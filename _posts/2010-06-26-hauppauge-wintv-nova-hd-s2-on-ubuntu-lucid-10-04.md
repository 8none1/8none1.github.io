---
id: 282
title: 'Hauppauge WinTV-NOVA-HD-S2 on Ubuntu Lucid 10.04'
date: '2010-06-26T22:01:47+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=282'
permalink: /2010/06/26/hauppauge-wintv-nova-hd-s2-on-ubuntu-lucid-10-04/
custom_header_001:
    - '<meta name="google-site-verification" content="sbNvBcaQw5KYqVYa8_Dyg2I8soCQcVYtESsbd8JfOFg" />'
categories:
    - Ubuntu
---

Ubuntu 10.04 doesn’t include the required firmware to get this card to work. Probably due to some licensing issues.

Although the card gets detected and you get the frontend device and the demod device a scan will fail to detect any channels.

The reason is that the firmware isn’t there.

Solution, <span style="text-decoration: line-through;">copy this file to /lib/firmware/ and reboot</span>

<span style="text-decoration: line-through;">[dvb-fe-cx24116.fw](http://www.whizzy.org/wp-content/uploads/2010/06/dvb-fe-cx24116.fw)</span>

Tsch. What a bell end. Instead, install the package:

# **linux-firmware-nonfree**

They don’t go out of their way to draw much attention to it, like telling anybody or anything.