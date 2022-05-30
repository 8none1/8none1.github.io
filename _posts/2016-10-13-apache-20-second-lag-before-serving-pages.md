---
id: 780
title: 'Apache &#8211; 20 second lag before serving pages'
date: '2016-10-13T10:15:09+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=780'
permalink: /2016/10/13/apache-20-second-lag-before-serving-pages/
categories:
    - linux
    - 'Making the world a better place'
---

**TL;DR**: There is no such thing as a “none” directive in Apache 2. If you’ve got “deny from none” or “allow from none” then you’re doing DNS lookups on each host that connects regardless of whether you want to or not.

I was experiencing a very annoying problem trying to serve static HTML pages and CGI scripts from Apache 2 recently. The problem manifested itself like this:

- Running the scripts on the server hosting Apache shows they ran in well under a second
- Connecting to the Apache server from the LAN, everything was fine and ran in under a second
- Connecting to the Apache server from the Internet, but from a machine known to my network, ran fine
- Connecting from an AWS Lambda script, suddenly there is a 20 second or more delay before getting data back
- Connecting from Digital Ocean, there is a 20 second delay
- Connecting from another computer on the internet, there is a 20 second delay

What the heck is going on here?

I spent time trying to debug my CGI scripts and adding lots more logging and finally convinced myself that it was a problem with the Apache config and not something like MTUs or routing problems.

But what was causing it? It started to feel like like a DNS related issue since the machines where it ran fine where all known to me, and so had corresponding entries in my local DNS server. But but but… I clearly had “HostnameLookups Off” in my apache2.conf file. When I looked at the logs again, I noticed that indeed hostnames were being looked up, even though I told it not to.

![966381](/wp-content/uploads/2016/10/966381.jpg)]

Why? Because I don’t know how to configure Apache servers properly. At some point in time I thought this was a good idea:

```
<pre style="padding-left: 90px;">Order deny, allow
Deny from none
Allow from all
```

But, there is no such thing as a “none” directive>. Apache interprets “none” as a host name and so has to look it up to see if it’s supposed to be blocking it or not, which causes a DNS lookup delays and hostnames to appear in your Apache logs.

Enlightenment came from here:[ http://kb.simplywebhosting.com/idx/6/213/article/](http://kb.simplywebhosting.com/idx/6/213/article/)

There is also a suggestion that inline comments can do the same thing here: <https://www.drovemebatty.com/wp/entries/11>