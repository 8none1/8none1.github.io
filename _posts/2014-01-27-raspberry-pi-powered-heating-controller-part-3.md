---
id: 561
title: 'Raspberry Pi powered heating controller (Part 3)'
date: '2014-01-27T14:38:03+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=561'
permalink: /2014/01/27/raspberry-pi-powered-heating-controller-part-3/
categories:
    - 'Making the world a better place'
    - RaspberryPi
---

I’ve had all the parts hooked up on breadboard for a few weeks now, and in theory everything works. I haven’t actually tested the prototype with the real heating system yet as it’s been cold and I don’t want to risk blowing anything up and having to deal with a cold house and an angry family. Sometime in the next month or so I will do that, but as far as I can tell it will Just Work (lolz).

![The final circuit design](/wp-content/uploads/2014/01/final_breadboard.jpg)

The final circuit design

In the meantime I’ve been thinking about the software to drive the thing.

My programming skills are pretty basic so I have been trying to keep everything as simple as possible. It became apparent that I was going to need some inter-process communication so that I could handle switching things on and off from outside the controller program (for web control etc). This presented me with a problem as all the basic reading from files/FIFOs were blocking – in that they would sit and wait for a “command” to be received before continuing – and that was too limiting for my needs. I was going to have to deal with some kind of threading. This troubled me deeply. The good people of Google+ suggested a few options and in the end I decided a RESTful interface was the way to go as it should be accessible from the widest range of other languages I might have to use, and especially easy from a web browser.

Originally I had imagined that I would have a single Python script to handle pretty much everything, including:

- Switching the relays on/off
- Responding to button presses
- Proving a scheduling engine for standard “on at this time, off at this time” behaviours
- Providing a way for other things to control the system
- Monitoring temperatures around the house
- Intelligent switching (e.g. it’s extra cold this morning, turn on early)

That way, I figured, I could just write one script and each function could interact with each other function very easily. But following a conversation with Mark S. a few years ago, and then more recently with Stuart L. I was starting to think that the monolithic architecture was not the way to go and instead I should push the intelligence out to the edges and the main loop should just deal with the basic on/off functionality. I was worried that if something went wrong with the electronics, for example the heating turned on but never off, then it would be difficult to spot from outside the main loop. But, given that the heating system has a lot of built-in safety features like room stats, boiler stats and hot water stats which are all wired in series with the mains that won’t be a problem, and besides the current controller doesn’t do anything more intelligent than ON or OFF.

To that end I decided that I’d go with this architecture:

![Heating System Architecture](/wp-content/uploads/2014/01/Heating-System-Architecture.jpg)

The Relay Controller would handle switching the power on and off in the right order to the right outputs, it would handle the switches for manual override, provide feedback via LEDs, and it would provide a REST interface for anything else to control the system or find out what the current state is. This way I can write a much more simple script to manage the scheduler for example. It can run independently and then simply poke the relay controller at the right time. I also decided that the relay controller would only have the ability to switch **off** after a certain period of time. The default will be 60 mins. This further simplifies things and puts just the right amount of intelligence in the core. The scheduler just says “switch on now, and off again in 90 minutes”. If the scheduler crashes the correct state is maintained by the controller and things just pick up from where they were when the scheduler starts again. If the controller crashes then the power is removed from the system and it “fails safe”.

The REST API is simple:

- /get/ch *or* hw/ – gets the current state of the system as a JSON object. e.g. “http://10.0.0.1/get/ch”
- /set/*ch* or *hw*/on *or* off – switch the system on or off. e.g. “http://10.0.0.1/set/ch/on” switches the central eating on for the default time (60 minutes)
- /set/ch *or* hw/on/n – switch the system on for n minutes. e.g. “http://10.0.0.1/set/hw/on/90” switches hot water on for 90 minutes

I used the Python SimpleHTTPServer, the ThreadedHTTPServer and threading.Thread.

I’m certain that it could be rewritten using proper OO methods and whatnot, and there is quite a lot of code which started out as an idea which then became redundant, but if you’re interested it’s available here: [piheat](/wp-content/uploads/2014/01/piheat.txt)

Next on the agenda is to draw a PCB and get it made (once I’ve tested the circuit properly) and work on the scheduling engine. At this rate I should be finished right around the time when we don’t need to use the heating any more.

## Further Updates

- [Part 1](https://www.whizzy.org/2014/01/04/raspberry-pi-powered-heating-controller-part-1/ "Raspberry Pi powered heating controller (Part 1)")
- [Part 2](https://www.whizzy.org/2014/01/11/raspberry-pi-powered-heating-controller-part-2/ "Raspberry Pi powered heating controller (Part 2)")
- [Part 3](https://www.whizzy.org/2014/01/27/raspberry-pi-powered-heating-controller-part-3/ "Raspberry Pi powered heating controller (Part 3)")
- [Part 4](https://www.whizzy.org/2014/02/04/raspberry-pi-powered-heating-controller-part-4/ "Raspberry Pi powered heating controller (Part 4)")
