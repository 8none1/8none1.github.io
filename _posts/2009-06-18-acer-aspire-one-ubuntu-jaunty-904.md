---
id: 174
title: 'Acer Aspire One &#038; Ubuntu Jaunty 9.04'
date: '2009-06-18T13:30:12+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=174'
permalink: /2009/06/18/acer-aspire-one-ubuntu-jaunty-904/
categories:
    - linux
tags:
    - linux
    - ubuntu
---

I’m really happy with my new Aspire One. I was a bit late coming to the whole netbook party. In the last few months I’ve been looking at the various options and the Aspire One was still the best deal in my opinion.

I’ve upgraded the RAM by adding another 1GB (you can’t add any more than this, it doesn’t pass POST if you do) and given it a custom paint job (pictures coming soon).

While I was playing around with various OS options I was caught by a bug which corrupts the SD card (left hand side) if you’ve opted to put /home on there and suspended. See this page for more info:

[https://bugs.launchpad.net/ubuntu/+source/pm-utils/+bug/342096](http://blog3.robertalks.com/index.php/2009/06/17/revision-3-for-kernel-2630-final-released/)

The fix is to recompile the kernel and change a few options from the Ubuntu defaults. While I was waiting for the source to download I had a quick poke around on Google and found that Robert had already built a custom kernel for the Aspire complete with all the Aspire One specific changes needed to make the WiFi LEDs work, and fan control and what have you. I posted a comment on Robert’s site asking if he had also included the fix from the Ubuntu bug as detailed above, he hadn’t but about 30 mins later a new release was ready to roll. It fixes the corrupt SD problem and saved me a lot of menuconfig work. Thanks Robert!

You can get more details from here:

<http://blog3.robertalks.com/index.php/2009/06/17/revision-3-for-kernel-2630-final-released/>