---
layout: archivepost
title: "Adventures in SSL land"
date: 2012-08-13 10:42
author: Jeremy Gray
comments: true
categories: [software]
tags: [.NET Bug, fiddler2, HTTP, jeremy, software, SSL, TLS]
---
I've been working on a problem for a lot longer than I care to admit.  I've been trying to connect a .NET client to a webservice.  Pretty much the easiest task you could ever imagine.  It's day 1 stuff, really, and I'm around day 3285.

[Here is the project that wasn't working](https://github.com/TheJeremyGray/whyunowork).

Pretty standard stuff, I was originally using standard web references.  Then I switched it over to WCF because I thought I might get a little more information.  But five minutes into this project I realized there was something very wrong.  I could hit the secure server through a browser and authenticate just fine.  But the SSL handshake would never complete and eventually their server would forcibly close the connection.  I spent some time on the phone with the vendor, mostly I wanted their server log to tell me why the connection was closed.  Unfortunately I couldn't get the correct person on the phone to give me that information or make it known that the process was failing before the application logs would pick up the error.

I pulled out every trick in my book, ignoring invalid certificates, pre-authenticating, sharing connection pools, trying to find interop issues between JAVA AXIS servers and .NET clients.  Finally, I recognized that I was going at the problem the wrong way and started talking to people.  The error I was getting all along was a forcibly closed connection error.

*System.Net.WebException was unhandled*
*The operation has timed out*

All along I was only getting one request/response from the server ([status code 200](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes), which is why I was pretty sure the issue was on my end.)

<a href="{{ site.baseurl }}/images/untitled1.png">![Fiddler Requst]({{ site.baseurl }}/images/untitled1.png)</a>

So I called my friend [Arthur](http://devarthur.blogspot.com) and asked him to take a look.  He updated the timeout to 5 minutes and took the maxReceivedMessageSize up to int.MaxValue.  He sent me a different error, below, that I was able to reproduce (HTTP stayed the same).

*System.ServiceModel.CommunicationException was unhandled*
*An error occurred while making the HTTP request to https://service. This could be due to the fact that the server certificate is not configured properly with HTTP.SYS in the HTTPS case. This could also be caused by a mismatch of the security binding between the client and the server.*

This turned out to be exactly what I needed.  I was pretty sure the server certification was configured properly because I was able to authenticate via the browser, so I focused on the mismatch.

Here are the ciphers in the SSL handshake, if you are interested in learning more about how an SSL session is created...try this [link](http://lmgtfy.com/?q=SSL+Handshake)

**Me**

~~~~~~~~
[002F] TLS_RSA_AES_128_SHA
[0035] TLS_RSA_AES_256_SHA
[0005] SSL_RSA_WITH_RC4_128_SHA
[000A] SSL_RSA_WITH_3DES_EDE_SHA
[C013] TLS1_CK_ECDHE_RSA_WITH_AES_128_CBC_SHA
[C014] TLS1_CK_ECDHE_RSA_WITH_AES_256_CBC_SHA
[C009] TLS1_CK_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
[C00A] TLS1_CK_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
[0032] TLS_DHE_DSS_WITH_AES_128_SHA
[0038] TLS_DHE_DSS_WITH_AES_256_SHA
[0013] SSL_DHE_DSS_WITH_3DES_EDE_SHA
[0004] SSL_RSA_WITH_RC4_128_MD5
~~~~~~~~

**Them**

~~~~~~~~
[0005] SSL_RSA_WITH_RC4_128_SHA
[000A] SSL_RSA_WITH_3DES_EDE_SHA
[0013] SSL_DHE_DSS_WITH_3DES_EDE_SHA
[0004] SSL_RSA_WITH_RC4_128_MD5
[00FF] TLS_EMPTY_RENEGOTIATION_INFO_SCSV
~~~~~~~~

My problem has always been that I got the initial 200 response from the server and then nothing else ever happened.  This finally started to get more clear.  I was asking for *[002F] TLS_RSA_AES_128_SHA* and they were sending back *[00FF] TLS_EMPTY_RENEGOTIATION_INFO_SCSV.* I've never dealt with this renegotiate flag before and had to look up the specifics in [RFC 5746 Section 3.3](http://tools.ietf.org/html/rfc5746).  In plain English this means, I want TLS, but they only support SSLv3.  But somehow we aren't communicating the renegotiation.

The fix, completed in a single line of code, forced my end to use SSLv3

[ServicePointManager.SecurityProtocol](http://msdn.microsoft.com/en-us/library/system.net.servicepointmanager.securityprotocol.aspx) = SecurityProtocolType.Ssl3;

This fixes the problem but I still didn't understand why fiddler2 gave me an unexpected EOF or 0 byte error and why WCF just stopped.  I did more searching for unexpected EOF's during SSL connections and found that apparently .NET "[chokes](http://social.msdn.microsoft.com/Forums/nl/ncl/thread/9e48f5d5-9c99-4d83-a5fa-7ba21dc5f934)" on this flag.  Not sure how reliable this is but it seems to make sense.  I'll call it case closed for now.
