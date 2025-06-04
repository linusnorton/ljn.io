---
title: "When size matters: the day a rogue getter swallowed our case data"
date: 2025-06-04 02:00:00
lastmod: 2025-06-04 02:00:00
description: >-
  
layout: post
---

# When size matters: the day a rogue getter swallowed our case data

## Where are my case notes?

HMCTS, the courts and tribunals service within the Ministry of Justice, handles millions of Civil, Family and Tribunal cases each year. All of those cases flow through a case management system called **CCD**. CCD has its quirks - it's configured via Excel spreadsheet and it implements business logic through an intricate system of callbacks into each team's APIs - but most of the time it just works.  Every now and then, though, case data goes missing, which leads to... distress.

I was working on one of the smaller services at HMCTS where an issue was reported on a case. After triggering an event, most of the case data simply vanished. Not *all* of it (that would have been too easy); specifically the caseworker's notes on the case had gone missing.

The audit trail showed nothing sinister: CCD's callback came in, our Spring Boot service answered `200 OK`, and everybody went back to work… until someone opened the case and found most of the data was missing.

## The only clue: a lonely warning

Buried in the logs there was a single line:

```
Response already committed. Ignoring: org.springframework.http.converter.HttpMessageNotWritableException
```

That warning was our only breadcrumb. No stack-trace, no 500, no "request failed" banner on CCD - just a shrug from Spring saying *too late, the headers are out the door*.

## A getter with a secret

With a debugger hooked into the callback we discovered the culprit: during the Jackson serialization a **getter** sometimes threw a `NullPointerException`. You would assume that throwing an exception during serialization would raise a `500`, but clearly this wasn't happening. We took a closer look at how Spring handles exceptions to find our log message:

```java
if (request instanceof ServletWebRequest servletWebRequest) {
    HttpServletResponse response = servletWebRequest.getResponse();
    if (response != null && response.isCommitted()) {
        logger.warn("Response already committed. Ignoring: " + ex);
        return null;
    }
}
return createResponseEntity(body, headers, statusCode, request);
```

For context, HTTP responses happen in two parts: first the status and headers are sent, then the body starts streaming. Tomcat keeps those headers in an internal buffer until it fills up or you explicitly flush. When that happens the response is marked **committed** (`HttpServletResponse#isCommitted()` returns `true`). Before that moment Spring can still change its mind and turn the call into a 500 error; after it, the status code is already on the wire and cannot be rewritten.

So in our world:

- If Jackson explodes **before** the headers are committed, Spring's error handler tears down the partial response and returns a clean `500 Internal Server Error`.
- If the exception happens **after** the commit, Spring can no longer touch the status code. All it can do is log *“response already committed”* and keep streaming - which leaves the client holding a valid but half‑filled JSON document.

## Re-creating the issue

At first we couldn’t reproduce the bug locally; every attempt gave the expected 500. The breakthrough came when we realised **two things mattered**:

1. **Size** - the JSON had to be big enough to flush the buffer early.
2. **Order** - the "big" field had to appear *before* the naughty getter.

So we inflated an innocent field (in our case the `applicantName`) until the payload passed the buffer threshold. Now the exception triggered *after* the response was committed, and boom - we hit the same "data-eating" path Spring had taken in prod.

CCD received a perfectly valid but **truncated** document. Fields up to, but not beyond, the failing getter survived; everything else evaporated.

## Why the JSON stayed valid

Jackson streams objects field-by-field. When the getter exploded, the serializer simply **abandoned** the remaining properties but still wrote the closing braces it had already buffered. The result was syntactically correct JSON - just missing half its content. Because the transport never broke, CCD had no reason to reject it.

## Takeaways

1. **Never put real logic in a getter.** Jackson will call it during serialisation; if it can throw, it *will* someday.
2. **Buffer thresholds matter.** A 5 kB object and a 50 kB object can follow completely different exception paths.
3. **Monitor warnings, not just errors.** When Spring says "response already committed" it's really saying *"I gave up"*.

We’ve patched the offending getter, cleaned up our DTOs, and set up an App Insights alert for that warning so the next silent 200 won’t slip through the cracks.