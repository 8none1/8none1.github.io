---
id: 425
title: 'Device control over HDMI via CEC.  libcec FTW.'
date: '2012-11-26T15:28:18+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=425'
permalink: /2012/11/26/device-control-over-hdmi-via-cec-libcec-ftw/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - linux
    - 'Making the world a better place'
    - tv
---

Blimey, it’s been a while. I’ve been a bit busy, and let’s be honest; writing up blog posts always sounds like a good idea, but when you get in to it – it’s really hard work.

Anyway – I finally got round to buying a Pulse-Eight CEC to USB adapter: <http://www.pulse-eight.com/store/products/104-usb-hdmi-cec-adapter.aspx>

This awesome little box of tricks makes up for the lack of CEC control in the vast majority of HDMI-Out equipped graphics cards. It’s the final piece in the jigsaw of a Linux based home entertainment device. It allows you to talk to the other devices in your HDMI network; your surround sound amplifier and your big screen TV being the best examples (assuming they support CEC of course).

CEC has been around for a long time but for some reason it doesn’t seem to be widely used, or very well implemented, in most consumer electronics devices. It’s a published standard but with OEM manufacturers wanting to differentiate their products and introduce a bit of vendor lock-in, they all call it something different. The fact is that your Toshiba Regza Link TV will talk to your Sony Bravia Link Amp just fine, for the most part. There might be a couple of proprietary things which don’t work, but on, off, volume up, volume down etc will all just work.

Pulse Eight’s USB to CEC adapter lets your computer get in on the act too, and opens up a whole realm of automatic switching, which really cuts down on the number of remote controls you need and the number of buttons you have to remember the purpose of.

The good folk at Pulse Eight have also made libCEC, an open source library to allow pretty much any software to take advantage of the USB adapter. <http://libcec.pulse-eight.com/>

It comes with C++, C and .NET interfaces, and a CLI utility called cec-client. XBMC &amp; MythTV already support libCEC and have some neat features baked right in. It’s good, but it’s not exactly what I was looking for – I want a bit more control.

Now, I don’t know anything about C++ or C or .NET, so until someone writes some Python bindings, my ticket to this party lies solely with the CLI utility cec-client. It can do most things on the “transmit” side, so you can send commands to your other CEC devices fairly easily. Acting on a request, or listening to the CEC traffic is a bit more complex – but not beyond the realms of possibility.

This weekend I wrote (and rewrote and rewrote) a couple of Bash scripts to:

- Let me control the system volume (i.e. the real hardware, not the mixer on the computer) on the TV &amp; Amp from the PCs remote control
- Let me switch the TV &amp; Amp on and off
- Activate proper muting, again not the mixer on the computer – the hardware itself
- Switch the amp &amp; TV to my MythTV PC
- Power off all the hardware when the PC suspends, and then switching it all back on again
- Shut the whole lot down when the screensaver on the PC kicks in.

I make these scripts available for your amusement:

- [cecserver.sh](/wp-content/uploads/2012/11/cecserver.sh_.txt)
- [cecsimple.sh](/wp-content/uploads/2012/11/cecsimple.sh_.txt)

</div>cecsimple is a client of the “server” which itself is a client of cec-client. Fire up the server and then issue it commands down the FIFO either directly or via the abstraction layer which is cecsimple.sh.

I hooked up the volume control and amp power via “irexec” and lirc. I tell irexec to execute, for example, “cecsimple.sh volup” or “cecsimple.sh ampon”. If the server component is already running then these commands are sent very quickly and you don’t really notice the lag.

To switch the TV off when the PC goes in to suspend mode I added a script in /etc/pm/sleep.d which calls “cecsimple.sh tvoff” and then “cecsimple.sh tvon” when it resumes. In theory if the TV is using the Amp to output surround sound audio then the TV will tell the Amp to turn off, it it’s not – it wont.

To switch things off when the screensaver kicks in, I simply “sudo pm-suspend” from an “xscreensaver-command -watch” script.

The practical upshot is that I can now control 99% of my media centre from a single remote control. I’ve opted to use the remote connected to the PC as I found it to be the least laggy – using the TV remote to send up/down/left/right etc to the PC was sluggish.

I think it should be possible to parse the log output from cec-client and write a “listener” component too, but it’s probably a better idea to learn some rudimentary C and do it properly. Or some Python bindings. Oh yeah, and you know what would be really cool, a hook in to MythTV so that when I’m watching something in surround sound the amp turns on automatically. That would be cool.

UPDATE 3 Dec 2013: When someone leaves the amp’s HDMI switch set on the PS3 and you switch the MythTV box on from the remote, the amp doesn’t automatically switch to MythTV. This has been annoying me for a while now, so I fixed it.

In the scripts linked to above the “active source” command does this:

```
send_command "tx 45 82 11 00"
```

4 (the MythTV device) to 5 (the amp) – 82 (switch active source) to 1.1.0.0 – but my Sony amp just ignores this request.

I spent some time trying out a few alternatives with cec-client and I’ve found one which works, and it kinda makes sense why:

```
send_command "tx 45 70 11 00"
```

4 (MythTV) to 5 (Sony amp) 70 (System Audio Mode) 1.1.0.0 (the input where MythTV is connected)

My assumption is that amp only speaks “system audio” – what with it being an amp. I’ve changed the “activesrc” with the 45:70:11:00 code and now it works! (It also switches the amp on, whether I like it or not – so it’s not perfect).

Useful links:

- <http://www.cec-o-matic.com/>
- <http://www.jwz.org/xscreensaver/man3.html>
- <https://github.com/Pulse-Eight/libcec>