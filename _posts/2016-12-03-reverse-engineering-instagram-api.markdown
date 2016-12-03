---
layout: post
title:  "Reverse Engineering the Android Instagram App's Private API"
date:   2016-12-02 23:22:15
comments: true
categories: reverseengineering
---

For fun, I wanted to write a simple bot to interact with the Instagram API. Instagram has a public API you can use, but it is a little restricted, and some API endpoints have to go through a review process before you can use them. Since I'm just doing to for fun and to learn a few tools, I figured I'd just grab the access token from the Instagram app on my device, and that's that.

Turns out that Instagram's private API is a little secure, and it won't be that simple.

For starters, the API uses SSL, which means that you can't just hook up a proxy and sniff the traffic. No problem, you can use a proxy like Charles, [install the Charles Root Certificate to your device][InstallCharlesUrl], and you're all set and ready to go! This is usually all I've needed to do.

I tried this with the Instagram API, and the server rejected the Charles root certificate! Damn! That means Instagram is using certificate pinning, where the server will only accept it's own certificates, and rejects the Charles root certificate that it doesn't recognize.

Well, I have a rooted Android phone with [XPosed][XposedUrl] installed, and there's a nifty module called [JustTrustMe][JustTrustMeUrl]. This appears to stub out the system's trust store, so it accepts _any_ certificate as valid. And it works! I can now see all the traffic my device sends and receives from the Instagram private API. (By the way - make sure to turn off JustTrustMe after you're done using it, or anyone will be able to MITM you. Not good.)

Ok, so now that I can pull my access token from one of the requests, I should be able to script that and issue my own requests to their API, right? Actually... wrong.

Instagram has another layer of security, where the requests are signed by a signature key on the client. So each request includes the json params, and a hmac hash of the json params. The server will then use the same signing key to hash the json params, and make sure the hashes match. If they don’t match (ie not signed with the correct key) the server will reject the request.

Ok.. so we need a way to get Instagram's signing key. How? The Instagram app on my device must have access to it, or else it wouldn't be able to interact with the server.

Let's break out the big guns then. [Frida][frida-url] is a tool that allows you to inject JS into the running process and sniff or modify memory, function arguments, and return values.


Here's the Frida script that I used, modified from the Frida docs and the folks [here][ctf-url] which did the hard dissasembly work to find the actual function we need to hook:


```python
import frida, sys

def on_message(message, data):
    print(message)

process = frida.get_usb_device().attach('com.instagram.android')

jscode = """
Interceptor.attach(Module.findExportByName("libscrambler.so", "_ZN9Scrambler9getStringESs"), {
    onLeave: function (retval) {
        console.log(Memory.readCString(retval));
    }
});
"""

script = process.create_script(jscode)
script.on('message', on_message)
print('[*] Running sniffer')
script.load()
sys.stdin.read()
```

This attaches an interceptor to the `_ZN9Scrambler9getStringESs` function in `libscrambler.so`, which is the function the Instagram app is using to retrieve the signature key, and logs the value to the command line. We've done it!

Now that we have the signature key, we can construct the signed body using the following:

`JSON={"_csrftoken":"xxx","user_id":"xxx","_uid":"xxx","_uuid":"xxx"}`

Put in whatever values are yours, from sniffing the API

I use the following alias to URL encode JSON:

```
alias urlencode='python -c "import sys, urllib as ul; \
    print ul.quote_plus(sys.argv[1])"'
```

And to construct the signed body:

`SIGNED_BODY=$(echo -n $JSON | openssl dgst -sha256 -hmac $KEY).$(urlencode $JSON)"&ig_sig_key_version=4"`

You can use this signed body and now script your own requests to Instagram's private API, as if you were issuing those requests directly from your Instagram app. Anything the app can do, your scripts can now do!

[frida-url]: http://www.frida.re
[JustTrustMeUrl]: https://github.com/Fuzion24/JustTrustMe
[XposedUrl]: http://repo.xposed.info/
[InstallCharlesurl]: http://eliasbagley.github.io/android/2016/10/26/charles-cert-without-lockscreen.html
[ctf-url]: https://github.com/ctfs/write-ups-2016/blob/39e9a0e2adca3a3d0d39a6ae24fa51196282aae4/cyber-security-challenge-belgium-2016-qualifiers/Mobile%20Security/Phishing-is-not-a-crime/README.md
