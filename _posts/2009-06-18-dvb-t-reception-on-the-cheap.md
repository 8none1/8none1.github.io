---
id: 178
title: 'DVB-T reception on the cheap'
date: '2009-06-18T17:07:04+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=178'
permalink: /2009/06/18/dvb-t-reception-on-the-cheap/
categories:
    - linux
    - tv
---

In an effort to reduce noise and electricity consumption in my living room I’m removing the current MythTV frontend and replacing it with an Aspire Revo.

The box under the telly at the moment is a dual-core Pentium 4 somethingorother running at about 2Gz in a nice looking “media” case. It’s great and everything, and it decodes HD perfectly well but it has fans on the processor, fans on the video card and fans in the PSU which all add up to a noisy box.

So it’s going to get re-purposed as a general computer and the Revo will go in it’s place. Actually it will get bolted on to the back of the telly to keep it out of the reach of children who like to press buttons. I’ll report back on the building of a frontend on a Revo once I wrestle the package from the hands of the worlds most apathetic delivery company “Home Delivery Network”. (I think they use the word “delivery” quite incorrectly)

Anyway, as part of this upgrade I need to add a DVB-T tuner to the Revo but with no PCI slots the only option is to use a USB tuner. My father-in-law put me on to Kenable who have a lot of bits and bobs at good prices, specifically the PEAK DVB-T USB tuner for 15quid:

USB Peak DIGITAL DVB-T Freeview TV Card XP MCE Vista Dongle
[ http://www.kenable.co.uk/product\_info.php/cPath/171/products\_id/1435](http://www.kenable.co.uk/product_info.php/cPath/171/products_id/1435)

I wasn’t able to find much about about it before I bought it, but I thought I’d take a punt and get one anyway and worry about getting it working under Linux later.

It arrived the other day and I popped the lid off to find an AF9015 demodulator and a QT1010 tuner inside. I checked on the Linux TV wiki and things looked pretty good on the whole. When I plugged it in to an Ubuntu 9.04 box nothing happened which was a shame but a bit more digging made me think that I should download the latest Linux TV drivers and have another go.

I followed the instructions from here:

[http://www.linuxtv.org/wiki/index.php/How\_to\_Obtain,\_Build\_and\_Install\_V4L-DVB\_Device\_Drivers](http://www.linuxtv.org/wiki/index.php/How_to_Obtain,_Build_and_Install_V4L-DVB_Device_Drivers)

and downloaded, compiled and installed the new drivers. You’ll also need some different firmware to that which is supplied with Ubuntu, download 4.95.0 from here:

[http://www.otit.fi/~crope/v4l-dvb/af9015/af9015\_firmware\_cutter/firmware\_files/4.95.0/](http://www.otit.fi/~crope/v4l-dvb/af9015/af9015_firmware_cutter/firmware_files/4.95.0/)

and place the .fw file in /lib/firmware (not as most people say /lib/firmware/&lt;kernel version&gt;). You’ll need to overwrite the other version, or rename it or whatever.

Reboot and you’re golden. The stick is detected and the firmware loads and I have been watching live TV on my Aspire One netbook courtesy of Mplayer.

You probably need to bear in mind that if you apply any future kernel/driver updates from Ubuntu then your drivers might get over written. There’s a change that the new version of the Ubuntu drivers will include the required AF9015 driver, but it might not. Also, you probably don’t need to compile all the drivers, just the ones for the AF9015 and QT1010 modules. I’ll look in to this, and if you promise to be good I’ll provide a nice little package with everything in.

UPDATE: So here we are two and a bit years later. I’ve just found this same tuner in the bottom of a box and plugged it in to my 11.04 Ubuntu machine. When I plugged it in Ubuntu automatically suggested I download the firmware. Awesome. It now “just works”. Well pleased.