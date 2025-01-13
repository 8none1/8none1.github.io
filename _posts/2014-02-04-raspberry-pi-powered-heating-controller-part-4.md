---
id: 571
title: 'Raspberry Pi powered heating controller (Part 4)'
date: '2014-02-04T19:25:00+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=571'
permalink: /2014/02/04/raspberry-pi-powered-heating-controller-part-4/
categories:
    - 'Making the world a better place'
    - RaspberryPi
---

![Heating PCB](/wp-content/uploads/2014/02/heating5_pcb.png)

It’s really happening! The breadboard prototype has been running the heating and hot water for a week or so now, and so far nothing has caught on fire. The relays are happy, they’re not getting warm or anything, the Pi is turning things on and off when it’s supposed to, the REST API is working, and I’ve knocked up a quick Web interface to switch things remotely. Last night for the first time I switched the heating on from the sofa, because I was a little bit cold.

## Great success!

![Borat](/wp-content/uploads/2014/02/borat.jpg)

I’ve turned the breadboard layout in to a PCB design and today at 09:11 UTC it was sent off for manufacture, I should have it back in a week. All the other components are on their way and so next weekend I should be able to put everything together and solder it to the board. With a bit of luck it will work first time.

I’m quite please with the board layout, it’s a fairly useful breakout board for the Raspberry Pi. In the future I would like to re-work it with smaller traces so that it can fit directly on top of the Pi.

The software to drive everything is rather basic, but I will make it available via Github anyway, it might come in handy. I will be working on this over the coming months, so it should improve soon. You’ll need a MySQL server set up. The schema for the database is included in the repo. The whole thing has grown organically and so the naming and structure is poor, but it works.

Code: <https://github.com/8none1/heating>

I’ve starting making some 1wire temperature sensors which I will place in various rooms and hook up back to the Pi via cat5.

![1wire_heat](/wp-content/uploads/2014/02/1wire_heat.jpg)

I won’t be able to control individual radiators at the moment, but I can set a target temperature for a given room and rely on the TRVs to control the temperature in other rooms.

I’ll also be adding outside temperature sensors and looking to replace the main room stat with another Pi in the future.

In summary then, I should have the final thing built in the next couple of weeks. More to follow when that happens. I’m considering looking at moving the whole thing to a micro controller rather than a Pi, that would reduce the BOM quite considerably, and might even warrant a commercial product in the long run – a hackable heating controller. Is this something people might be interested in?

## Further Updates

- [Part 1](https://www.whizzy.org/2014/01/04/raspberry-pi-powered-heating-controller-part-1/ "Raspberry Pi powered heating controller (Part 1)")
- [Part 2](https://www.whizzy.org/2014/01/11/raspberry-pi-powered-heating-controller-part-2/ "Raspberry Pi powered heating controller (Part 2)")
- [Part 3](https://www.whizzy.org/2014/01/27/raspberry-pi-powered-heating-controller-part-3/ "Raspberry Pi powered heating controller (Part 3)")
- [Part 4](https://www.whizzy.org/2014/02/04/raspberry-pi-powered-heating-controller-part-4/ "Raspberry Pi powered heating controller (Part 4)")
