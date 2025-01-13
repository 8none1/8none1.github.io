---
id: 291
title: 'Hauppauge WinTV Nova-S Plus on Linux'
date: '2010-08-04T18:15:09+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=291'
permalink: /2010/08/04/hauppauge-wintv-nova-s-plus-on-linux/
categories:
    - linux
    - 'Making the world a better place'
    - tv
---

The Nova-S Plus is a good card. [http://www.hauppauge.co.uk/site/products/data\_novasplus.html](http://www.hauppauge.co.uk/site/products/data_novasplus.html)

But, it would appear there is a defect in these boards, or at least a strange design, which means that they won’t lock on to some frequencies which require the 22kHz tone sending to the LNB with new drivers because there’s no link between the flange and dolphin-points. There’s plenty to read about here:

[https://bugzilla.kernel.org/show\_bug.cgi?id=9476](https://bugzilla.kernel.org/show_bug.cgi?id=9476)

And there’s a patch which fixes the problem by controlling the tone generator directly but it’ll never get in to the main kernel. For your convenience here is a link to a binary driver built for Ubuntu Lucid kernel version 2.6.32-23-generic:

</wp-content/uploads/2010/08/isl6421.ko>

Replace the current isl6421.ko from /lib/modules/2.6.32-23-generic/kernel/drivers/media/dvb/frontends/isl6421.ko with this one. It might also work for newer kernel versions, or not. Who knows? Not me.

I’ve also got a Hauppauge S2 HD and this patched driver doesn’t seem to effect it.

Search hints:

Hauppauge Nova S plus linux won’t lock horizontal 22khz tone can’t pick up some channels.
