---
id: 443
title: 'Asterisk UK Caller ID &#038; SMS redux'
date: '2013-02-12T12:41:57+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=443'
permalink: /2013/02/12/asterisk-uk-caller-id-sms-redux/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - asterisk
---

**Update 28 Dec 2013:** As far as caller ID goes, I might have been barking up completely the wrong tree here. Have a look here: [http://forums.digium.com/viewtopic.php?f=1&amp;t=85028](http://forums.digium.com/viewtopic.php?f=1&t=85028). I’ve re-implemented that patch for Dahdi and I’m testing it now. Get in touch if you want to test it as well. Once I’m happy it’s actually working, I’ll post it here.

**tl;dr: Scroll down to “How to fix Caller ID in the UK on Asterisk 11 and Dahdi 2.6.1″**

I’ve been having some problems with Asterisk 1.6 for a while now, in that when I’m dialled in to conferences via SIP, or when I call someone via the work Asterisk server from my Asterisk server at home I get cut off every 15 minutes. My colleagues have all kind of accepted this now, it was starting to get annoying. It was a known problem in Asterisk 1.6, and so I was going to have to upgrade.

The Ubuntu Asterisk packages are fairly up-to-date but FreePBX isn’t packaged, and it’s a pain in the backside to get set up with the correct permissions, especially if you install Asterisk from packages. I had a spare machine anyway so I decided that I’d give FreePBX Distro a go and once I’d burnt the ISO to a CD (ya rly) I found the set up process to be simple and quick.

FreePBX Distro is a stripped down CentOS (2.6.32 kernel) which boots quick, has minimal footprint and bundles in loads and loads of FreePBX “apps” like conferences, voicemail, blacklisting and makes setting up extensions a breeze. It also includes things like kernel source for so building extra modules from source is pretty easy. Which is a good job, because it was about to get messy.

If you’ve found this page via Google then this will probably sound pretty familiar:

- You got a TDM400p four port FXO/FXS interface card, or a clone from someone like OpenVox (which work perfectly by the way – as do those cheap daughter cards off eBay).
- You’ve upgraded to **Asterisk 11**, or **Asterisk 1.8** perhaps, or changed to the **Dahdi 2.6.1** drivers and all of a sudden your ”**UK Caller ID**” has “**stopped working**” or at the very least become “**intermittent**“.
- It was working fine before, so it’s **not a hardware issue** and your phone provided have **NOT suddenly stopped sending caller ID information**, regardless of what people of forums tell you. No, in fact this is clearly a bug which has crept in somewhere along the line.
- You’ve searched and searched and tried things like setting **rxgain=x.x** and **txgain=x.x** or whatever. (Side note: setting txgain and rxgain in chan\_dahdi\_channels\_custom.conf has no effect, the only place is seems to work is in **chan\_dahdi\_groups.conf**)
- You might have even stumbled across a post hinting that **cid\_rxgain=x.x** will solve your problems, only it didn’t

If that sounds like you, dear reader, then read on. I am your salvation.

1. This page: http://downloads.openvox.cn/pub/drivers/callerid\_patches/ (I can only attribute this discovery to divine intervention)
2. Find a new wctdm.c driver here: http://downloads.openvox.cn/pub/drivers/callerid\_patches/2.6.1-wctdm.c
3. Download the 2.6.1 Dahdi source from here: http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-2.6.1+2.6.1.tar.gz
4. Replace dahdi-linux-complete-2.6.1+2.6.1/linux/drivers/dahdi/wctdm.c with the version downloaded from openvox
5. Note it doesn’t compile, add the missing semicolon to line 333 (or thereabouts), successfully build and install the new driver
6. You might need to change /etc/modprobe.d/dahdi.conf and add:

```
options wctdm opermode=UK fwringdetect=1 battthresh=4
```

1. restart Dahdi and Asterisk and caller ID should now be fixed! Yay!

# How to fix Caller ID in the UK on Asterisk 11 and Dahdi 2.6.1

Here is a patch against wctdm.c from stock Dahdi 2.6.1 with the fixes in: [wctdm.c](/wp-content/uploads/2013/02/wctdm.c.patch).patch

**OR:** Replace dahdi-linux-complete-2.6.1+2.6.1/linux/drivers/dahdi/wctdm.c with this one: [wctdm.c](/wp-content/uploads/2013/02/wctdm.c)

**OR:** If you’re running FreePBX distro 64 bit “BETA-3.211.63-5 Release Date-01-24-13″ (or close) here is a binary driver: [wctdm](/wp-content/uploads/2013/02/wctdm.ko).ko Move it to /lib/modules/2.6.32-279.11.1.el6.x86\_64/dahdi/wctdm.ko

It works for me. I have not had to increase the rxgain, or txgain or anything else. It just works.

Once that’s working, we can move on to…

# How to send &amp; receive SMS in the UK on FreePBX Distro Beta 3.211

I’ve talked about receiving and sending SMSes before, but that was quite a long time ago, and things have moved on a bit since then. I’ve also learnt a bit more about the Asterisk dial plan, and have a slightly cleaner way of doing it now.

You need to have working caller ID. Hint: see above.  
You need to install smsq.  
smsq doesn’t come built in FreePBX Distro so you have to build it yourself if you want to send SMS. This is pretty easy:

1. Download the Asterisk 11.2.1 source from here: http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-11.2.1.tar.gz
2. Install popt: yum install popt-devel.i686 popt.i686 popt.x86\_64 popt-devel.x86\_64 (prerequisite for smsq)
3. In the Asterisk source directory do “./configure”
4. run “make menuselect”
5. Scroll down to “Utilities”
6. In the right hand pane, check “smsq”
7. Save &amp; Exit
8. run “make”
9. DO NOT run “make install” as this will trash your FreePBX config
10. Once that’s done you should find an executable smsq in the utils directory

Just to be sure, create /var/spool/asterisk/sms/motx and /var/spool/asterisk/sms/mtrx  
You can find more details on building Asterisk here: http://blogs.digium.com/2012/11/05/how-to-install-asterisk-11-on-centos-6/

You need to be able to send SMS in order to register with the BT SMS system, otherwise when you receive an SMS you will get a phone call from the BT robots who will read the text to you, badly.

To register with BT try this:

```
<path to smsq>/smsq –motx-channel=DAHDI/2/17094009 -d 00000 -m “test”
```

where DAHDI/2 is the channel number of your outgoing line.

If you look in the Asterisk logs you should see the message going out with some TX and RX hex dumps. If it works, a short while later you will receive a phone call from either the SMS system trying to talk modem at you, or the robot. Either way, this means your text went out. Great success!

Now we need to set up receive, which is pretty straight forward.

Edit /etc/asterisk/extensions\_custom.conf and add the below. Note – I’ve added a lot of comments to explain what’s going on, it might improve readability if you take them out.

```
[from-internal-additional-custom]
;; This makes sure our new 'app' gets included in the FreePBX dial plan
include => app-rx-sms
[app-rx-sms]
;; This is the app to receive an SMS which will be called when we match
;; the caller ID of an incoming call to the BT computer
exten => s,1,Answer()
;; This wait seems to help
exten => s,n,Wait(0.5)
;; We simply use the built in SMS function
exten => s,n,SMS(default,a)
;; Hangup with code 16, normal termination.
exten => s,n,Hangup(16)
;; Done
;; We also extend the from-pstn context with a branch to the SMS app
;; if the caller ID matches, hence why it's necessary to get caller
;; id working first
[from-pstn-custom]
;; we add ourselves in to the end of the current dialplan
;; priority 7 gets us there...
exten => s,7,GotoIf($["${CALLERID(num)}" = "08005875290"]?app-rx-sms,s,1)
;; pretty simple - if the caller ID matches, go to the sms app
exten => s,n(dest-ext),Goto(ext-group,1100,1)
;; Note: 1100 is my "ring all" group. You will need to change 1100
;; to what ever you use. You should see this at the end of the
;; from-pstn context in extensions_additional.conf
```

Save that, and in asterisk do a “dialplan reload”. From smsq send an SMS to 00000 saying “register”· If everything works you should get a text file in /var/spool/asterisk/sms/mtrx.