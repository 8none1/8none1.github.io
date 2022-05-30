---
id: 452
title: 'Convincing MythTV to tune in to DVB-T2 MUXes'
date: '2013-11-25T19:09:38+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=452'
permalink: /2013/11/25/convincing-mythtv-to-tune-in-to-dvb-t2-muxes/
categories:
    - linux
    - tv
---

So that I remember for next time, and so I can start writing down some of the issues I’ve had to sort out since re-building my servers, here is what you have to do to tune to a DVB-T2 mux in MythTV (0.27 fixes).

This is the frequency for the PSB3 mux on Sandy Heath, I expect the encoding scheme will work pretty much where ever. At the very least it should get you started:

```
Frequency:  474200000
Bandwidth:  8 MHz
Inversion:  Auto
Constellation:  QAM 256
LP Coderate: 2/3
HO Coderate: Auto
Trans. Mode: 8K
Guard Interval: 1/32
Hierarchy:  None
```

If that doesn’t work, and you know it should (as in you are on the same transmitter) try setting the tuning timeouts higher. Also, just blindly re-trying because “WHY DOESN’T IT WORK” seemed to help.