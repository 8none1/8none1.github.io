---
id: 184
title: 'Aspire Revo as a MythTV frontend'
date: '2009-06-22T11:34:33+01:00'
author: will
excerpt: 'Yes - you should buy that Acer Revo.'
layout: post
guid: 'http://www.whizzy.org/?p=184'
permalink: /2009/06/22/aspire-revo-as-a-mythtv-frontend/
categories:
    - linux
    - tv
tags:
    - linux
    - mythtv
    - nettop
    - revo
    - ubuntu
---

**It’s brilliant!**

If you’re reading this then I will assume you already know about MythTV and you’ve searched Google to find out if the Aspire Revo box will make a decent MythTV frontend. In short, yes – it works fantastically well.

I bought the 8GB SSD Linux version from Play for about 150 quid (get a Play credit card and you can knock off about another 15 quid in vouchers and get 9 months interest free credit, for what it’s worth), and I also bought 2 x 2GB SODIMM.

The first thing I did was take the lid off to have a look inside. This wasn’t as easy as I thought. There’s one little screw under a sticker that says something about a warranty and then you just have to prise the lid off. It’s pretty stiff, to the extent that I was convinced there was another screw somewhere, but it comes off in the end. I removed the WiFi card since I won’t use it and it might reduce the heat/power. The RAM swap presented no surprises, but the appearance of a 160GB HDD did.

I had sort of decided that the SSD was the better option for me for two reasons; less heat and less noise. But, seeing as I’ve been gifted 160GB of disk space and under use the HDD makes no noise, I’m very happy!

I decided on XUbuntu over the normal version partly because of the reduced overheads and software bloat, I really don’t need Open Office and The Gimp installed on this box and I can’t be bothered with manually selecting packages at install, so I downloaded the XUbuntu ISO and stuck it on a USB pen drive with [Unetbootin](http://unetbootin.sourceforge.net/). I had a few problems booting from the pen drive, it kept complaining that the initrd was corrupt, so in the end I had to use the alternative version and run through the install on the command line. I blew away all the partitions on the disk, I won’t need any of the Acer software – there’s nothing much of use on there anyway.

Once XUbuntu was installed I downloaded and compiled the latest NVIDIA drivers:

[http://www.nvidia.com/object/linux\_display\_ia32\_185.18.14.html](http://www.nvidia.com/object/linux_display_ia32_185.18.14.html)

(this is the latest version at I write this, check if there is a newer one)

and then I downloaded the SVN version of Myth and enabled VDPAU. If you’re looking for help setting up Myth and VDPAU check the MythTV wiki, there’s more information there than I can recreate here. Read the [“Installing SVN on Breezy”](http://www.mythtv.org/wiki/Installing_MythTV_SVN_on_Ubuntu_Breezy) document to give you a hand getting all the bits and bobs installed to allow you to compile Myth. Note: some libraries have changed name, e.g. liblame is now libmp3lame.

Then it’s a case of enabling VDPAU at compile time using the configure script and then creating a playback profile to use VDPAU from within the frontend.

My old frontend had a dual core 2GHz processor in and would sit at about 80 to 90 percent usage on both cores while watching 1080p video and sucked somewhere in the region of 80 to 100 watts. The Revo’s Atom processor sits at about 10% usage (obviously the graphics card is doing the work) and sucks less than 20watts, while also being nearly silent.

Sound was a bit of a faff – the Revo has an HDMI connector with the audio path built in. To hear anything I needed to enable and unmute the IEC958 (spdif) channel and then tell Myth to use ALSA:hdmi for audio. Ubuntu detected the sound card perfectly well, so trust me when I say you do not need to download a later version of ALSA, you just need to get the settings right. You might also need to tell your telly to pick up digital audio, not analogue. I haven’t got system sounds going down the HDMI just yet , but I don’t think this will be a problem.

Summary then; very very good choice for a small, quiet and cheap frontend that can also double as an Internet browser.

I think I might buy another one.