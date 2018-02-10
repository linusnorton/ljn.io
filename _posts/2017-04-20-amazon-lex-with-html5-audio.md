---
title: Amazon Lex with HTML5 audio
date: 2017-04-20 08:00
lastmod: 2017-04-20 08:00
description: Using Amazon Lex and the HTML5/Web Audio API to create a virtual assistant inside the browser
layout: post
---

Amazon Lex is a relatively new service from Amazon that can be used to create interactive bots. It's currently only available in the Virginia region and you need to request access before you can use it. The service essentially mixes voice recognition with programmable scripts that return voice responses using the Polly (text-to-speech) service. This allows you to create the same two way conversation you might have with Amazon's Alexa.

I was recently taking part in the [Assertis Hackathon](https://medium.com/@assertis/hackathon-marathon-for-hackers-7fac00b38889) and thought it would be the perfect service to take for a spin. Our team's idea was to turn it into an interactive voice assistant for our website using the [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API). 

There are [many](https://www.youtube.com/watch?v=7uG9cuxNo5k) [tutorials](https://www.youtube.com/watch?v=tAKbXEsZ4Iw) on creating bots but all the integration examples tend to be with Facebook Chat or a native application rather than a website so I thought I'd jot down a few notes here.

# Web Audio API / HTML 5 Audio

First off it's worth saying that I'm totally green when it comes to audio processing, so my immediate thought when I encountered HTML5 Web Audio API is that it rather complicated. 

It turns out that there are two main browser based audio APIs. HTML5 audio revolves around the `<audio>` tag and plays basic media. There is also the JavaScript based Web Audio API that can be used for more complicated audio processing. 

Looking through the [Web Audio API documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) was a intimidating, the class hierarchy alone was difficult to grasp but to be fair the documentation does have links to various pages explaining all the concepts. In the midsts of a hackathon I had to set aside my desire to get to grips with some of the finer details and focus on getting results. With that in mind I did what any bad developer would do and started hacking together code from different tutorials.

The gist of it was to create an audio context, then use the `navigator.getUserMedia` method to get a stream, using that stream ask the audio context to create a media stream source, connect the source stream to a recorder, then take the buffer from the recorder and send it to Amazon using their SDK. Simple, right?

# Amazon API SDK

The good news in the midsts of this was that the Amazon SDK was actually really simple to use. In fact the Amazon Lex API is really simple to use as we were only using a single API endpoint to take the audio buffer input, and return the audio buffer response.

# Sending Audio to AWS

Being inexperienced with audio processing it's not surprising we hit a few issues along the way. The first thing to note is the format of the API's - the Web Audio API returns a ArrayBuffer (or AudioSomethingSomething) of Float32 but the Amazon SDK expects an array of Uint8 - not being well versed in these things I can't tell you why, the Float32 makes more sense to me but I assume it's to lower the size of the stream from 32-bit integers to 8 bit integers.

Luckily some curisory Googling turned up this snippet from Stack Overflow:

```javascript
function convertFloat32ToInt16(buffer) {
    var l = buffer.length;
    var buf = new Int16Array(l);
    while (l--) {
        buf[l] = Math.min(1, buffer[l]) * 0x7FFF;
    }
    return buf.buffer;
}
```

After we'd done that conversion Lex did start to return responses but they were always incorrect and only a single word. After a bit of head scratching I remembered that the audio stream we are receiving from the Web Audio API is 44.1khz and the Lex API only allows up to 16khz. 

After another copy paste from Stackoverflow:

```javascript
function downsampleBuffer(buffer, rate) {
    // 44100 is what we get from input (at least on Google Chrome) :)
    sampleRate = 44100;

    if (rate == sampleRate) {
        return buffer;
    }
    if (rate > sampleRate) {
        throw "downsampling rate show be smaller than original sample rate";
    }
    var sampleRateRatio = sampleRate / rate;
    var newLength = Math.round(buffer.length / sampleRateRatio);
    var result = new Float32Array(newLength);
    var offsetResult = 0;
    var offsetBuffer = 0;
    while (offsetResult < result.length) {
        var nextOffsetBuffer = Math.round((offsetResult + 1) * sampleRateRatio);
        var accum = 0, count = 0;
        for (var i = offsetBuffer; i < nextOffsetBuffer && i < buffer.length; i++) {
            accum += buffer[i];
            count++;
        }
        result[offsetResult] = accum / count;
        offsetResult++;
        offsetBuffer = nextOffsetBuffer;
    }
    return result;
}
```

We downsampled the stream and sent it on - finally some sensible results from Lex.

# Processing the audio response

Unfortunately there seems to be a bug in the Amazon API now where opting for the binary audio response in the API caused a permissions issue with Polly. In the stress of a hackathon we didn't have too much time to dig into that so we took the text response and sent it on to Polly ourselves. 

There was no need for the audio trickery when processing the response as the browser was quite happy to play the mp3 stream using the HTML5 Audio API, or at least the `<audio>` element. The audio buffer response was captured into an inline url for the audio tag to play. 

```javascript
polly.synthesizeSpeech(params, function (err, data) {
    if (err) console.log(err, err.stack); // an error occurred
    else {
        var uInt8Array = new Uint8Array(data.AudioStream);
        var arrayBuffer = uInt8Array.buffer;
        var blob = new Blob([arrayBuffer]);

        var url = URL.createObjectURL(blob);
        audioElement.src = url;
        audioElement.play();                 
    }
});
```

I was considering turning all this into a library but I am certain that some of the processing we're doing is suboptimal and I wasn't sure my use of the Web Audio API is correct.

Better leave this stuff to the professionals. 

# Did it work?

Yes, all in all the project was a success. After we'd navigated the minefields of Web Audio and HTML5 audio the Lex API was quite simple. You could press a button on our website, then say "plan me a journey from Norwich to Coventry on the 21th April" and it would do it. There were some issues on the bot side of things - we used the cities dictionary for the station names and that did not work in many cases (i.e. Canterbury East) but it's possible to upload your own dictionary.

There are many potential uses for this and as people become more accustomed to voice interfaces via their mobile devices or home assistants I can see there being a place for voice interfaces on the web too. Modern screen readers are excellent but I'd like to think that a voice assistant would be a nice accessibility choice to have. 

