---
id: 311
title: 'Asterisk VoIP calls causing PPP to drop on ADSL modems'
date: '2010-10-18T16:02:28+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=311'
permalink: /2010/10/18/asterisk-voip-calls-causing-ppp-to-drop-on-adsl-modems/
categories:
    - asterisk
---

The internet FTW.

I’ve been having this odd problem since I got my Asterisk box back up and running in that whenever a call came in and hung up I’d lose internet connectivity for a few seconds.

I tracked it down to the router dropping the PPP connection, which initially made me think that the polarity reversal indicating the call had hung up was causing the modem to b0rk, perhaps due to the distance between the phone socket and the modem, or my dodgy cat5 cabling, or something.

Turns out it was none of the above. EDIT: Bit hasty there. Hasn’t fixed it al all.

This post:

http://forums.contribs.org/index.php/topic,43733.msg209081.html#msg209081

points to exactly my problem. I disabled the DDoS protection and no more dropped internet connections. I don’t worry about DDoS attacks too much, so for now I’m going to leave it off.

UPDATE: None of the above actually worked. In the end it turned out the modem had simply broken. The NV RAM wasn’t remembering the config after a proper reboot and it was generally broken. So I replaced the whole thing with a new DrayTek modem. The old one did pretty well, it worked perfectly for about 3 years non-stop.

Search hints:

DrayTek VoIP Asterisk dropping ADSL PPP drop DDoS SIP