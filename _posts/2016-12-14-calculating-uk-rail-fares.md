---
title: Calculating UK Rail Fares
date: 2016-12-14 08:00
lastmod: 2016-12-14 08:15
description: Calculating rail fares in the UK is a complicated process with no formal documentation, this guide covers the basics of  working out how much a train ticket should cost.
layout: post
---

**Please note this post is now out of date, please check [the wiki](https://github.com/open-track/fares-service/wiki/Fare-Lookup) for the latest version.**

Although the UK rail fare data is often described as "open" I would actually describe it as available rather than open. Rail Delivery Group (formerly ATOC) are the holders of the data and before handing it over you need to agree to a fairly restrictive license and sign up for an account that can be disabled at their discretion. On top of that the data is only updated three times a year whereas the [commercial data](dtdportal.atocrsp.org) is updated daily. 

What's worse is that the data is almost entirely meaningless to anyone not working in the industry. Using the data to calculate rail fares is a complicated process and there is no formal documentation public or private. This guide covers the basics of looking up a fare and working out how much a train ticket should cost. The information presented in this blog post is also available in a [wiki](https://github.com/open-track/fares-service/wiki/Calculating-UK-Rail-Fares).

It should also be noted that Advance tickets are quota controlled by a centralised yield management system called New Reservation System or commonly National Reservation System. Access to this requires a leased line and several thousand pounds so unless you are quite serious about fares I'm going to assume Advance tickets are off the table. It's a shame as they are the cheapest fares. The NRS system is currently being re-written but the new version will not be publically available either.

This post only covers fares in the [ATOC fares feed](http://data.atoc.org/). Much of the information in this guide is derived from the [RSP specification of the feed](http://www.atoc.org/download/clientfiles/files/RSPDocuments/RSPS5045%2001-00%20Fares%20and%20Associated%20Data%20Feed%20Interface%20Specification.pdf). Unfortunately there are always additional rules that are no in the data, or written down anywhere. I will update the wiki as I'm made aware of them.

## Query Structure

The basic structure of a query is:

 - origin ([CRS](http://nrodwiki.rockshore.net/index.php/CRS) or [NLC](http://nrodwiki.rockshore.net/index.php/NLC))
 - destination ([CRS](http://nrodwiki.rockshore.net/index.php/CRS) or [NLC](http://nrodwiki.rockshore.net/index.php/NLC))
 - date 
 - passengers
   - passenger type (Adult or Child)
 - railcards (optional List of [RailcardCode](https://github.com/open-track/fares-service/wiki/Railcard-Code))

These are the bare minimum fields required to perform a query rather than a comprehensive list.

## Query Locations

There are roughly 12500 locations in the feed and in order to minimize the amount of data in some of the files stations and [group stations](Group-Stations) are divided into station clusters.

When looking up a fare the first step is to create a list of origins and destinations that might be relevant to the query. The relevant stations are a [set](https://en.wikipedia.org/wiki/Set) of [NLCs](http://nrodwiki.rockshore.net/index.php/NLC) comprised of:

- input NLC
- any clusters that input NLC belongs to
- any group stations the input NLC belongs to
- any cluster stations the group stations belong to 

### London Terminals

Unfortunately London Terminals is a bit of a special case in that it is a group station, but the members of the group depend on the other station involved in the query. To correctly query fares for London Terminals you will need to take the London Terminals Mapping file (available via data.atoc.org) and calculate the group members when a query is received.

## Fare Querying

Fares are split into two primary groups; [flow fares](https://github.com/open-track/fares-service/wiki/Flow-Fares) and [non-derivable fares (https://github.com/open-track/fares-service/wiki/NDFs)](Non-Derivable-Fares). Flow fares are considered the norm even though there are more NDFs. It should be noted that NDFs override flow fares (as we will see later).

When looking up either type of date you should respect the start and end date ranges and the quote date as defined in the flow and NDF files. The start and end date ranges are for the query target time and the quote date is for the date of the query itself (the current date).

### Flow fares

To look up a flow fare, return any fare records that have a flow between any of the origin and destination NLCs, or the NLCs of group stations and station clusters the origin and destination belong to. Some flows have the reversible flag set, in which case the fare is the same in the other direction and the origin and destination can be swapped. If you are using a flow that is reversible you should switch the origin and destination of the flow so that it is processed correctly later on. 

Once you have the fares you can check which railcards may be applied and apply them. There are three things that should validated before a railcard can be applied:

#### 1) Does the railcard apply to the passenger set

Railcards have a number of min and max fields (quantity of adults, children or total passengers). The passengers in the passenger set must be within the limits of the railcard. The data in the feed is largely incorrect but can be found online.

#### 2) Railcard restrictions

The railcard restrictions may restrict the use of the railcard on a particular ticket codes, at a specific origin but it may also change the restriction code of a fare. 2TR railcard for example has a record that applies to any ticket type and at any location saying the fare must have a restriction code of R9. 

It appears multiple railcard restrictions can apply to a fare. One may change the restriction code and one may ban it's use.

#### 3) Non-derivable fares and non-derivable fare overrides

The non-derivable fare overrides data may contain suppression records for the entire flow (which we check later) or preventing the use of a railcard. If a suppression record exists that matches the origin, destination, route, ticket code and railcard code of the fare then the railcard cannot be applied. The railcard code is empty then no railcard can be applied to that fare.

The non-derivable fare data may contain an entry with an empty railcard code which means that use of all railcards is not allowed on that flow with that ticket type.

#### Applying the railcard

Once that railcard has been validated the ticket types discount category can be matched up status discount record for the status code(s) of the railcard. Each status discount has a discount indicator, which when combined with the status record controls the logic used to discount the fare. This is described in [RSPS5045](http://www.atoc.org/download/clientfiles/files/RSPDocuments/RSPS5045%2001-00%20Fares%20and%20Associated%20Data%20Feed%20Interface%20Specification.pdf). 

Some discount statuses have a discount indicator of X or N which means they cannot be applied.

Most railcards have a minimum fare that varies by ticket type. After applying the railcard the price should not less than the minimum fare. If it is, set it to the minimum fare.

Once you have applied the discount the correct rounding rules should be applied. The official rounding rules are not public so most people just round down to the nearest 5p.

The public railcard covers normal adult and child fares (status 000 and 001) and can be applied like any other fare.

If the flow has a non standard discount indicator then you cannot apply railcard discounts using the method above. Either the flow will be marked as not discountable or there will be an origin and destination that should be used to return another flow discount to be used as an "add on" to the original fare. Non standard discounts do not apply to status "000" (standard adult fares).

### Non derivable fares

To look up an NDF, match the origin and destination and any railcards that are relevant to your query in the non-derivable fare feed. Then check if that NDF is suppressed in the non-derivable fare override feed. Lastly combine this with any non suppression non derivable fares in the non derivable fare override feed.

NDF and NDF override records with a composite record of N can be ignored as the documentation suggests they are also in the flow file, rendering them useless.

It's possible railcard restrictions also need to be applied to NDFs but I can't see why they would need to be.

## Combining the results

As noted earlier non-derivable fares take precedence over flow fares so when creating the combined set of fares any flow fare with an origin, destination, route, ticket type and railcard that matches an NDF should be removed. 

## Season tickets

There are only entries in the fares feed for 7 day seasons. The rest are calculated by multiplying the 7 day fare:

http://www.railforums.co.uk/showpost.php?p=1132953&postcount=6

and

https://groups.google.com/forum/#!topic/uk.railway/UkrCEW5OTYY

## Journey Validity

To determine whether a journey is valid using a fare you need to process the fare restrictions, ticket type validity, route locations and national routeing guide. I will write another post on this in the new year.
