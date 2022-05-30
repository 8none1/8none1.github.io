---
id: 878
title: 'Whizzy Labs Wireless Sensor'
date: '2019-06-11T15:36:26+01:00'
author: will
layout: post
guid: 'https://www.whizzy.org/?p=878'
permalink: /2019/06/11/whizzy-labs-wireless-sensor/
categories:
    - Arduino
    - IoT
---

Sheesh! Making a wireless sensor has proven to be a lot harder than I had expected.

Like a lot of weekend hardware hackers I thought it would be fun to build a wireless temperature sensor. I could use it as a feedback mechanism for my [Raspberry Pi heating controller](https://www.whizzy.org/2014/01/raspberry-pi-powered-heating-controller-part-1/), I could create graphs of temperatures in different rooms (everyone loves a graph right?), and with a little bit of forward planning I could make a fairly useful Arduino breakout board which could be used for lots of other fun wireless projects.

I’ve been through four iterations of the design before I finally found something that worked. I could have avoided some of this if I’d have spent more time testing on breadboard, but then I wouldn’t get to order colourful PCBs!

![](/wp-content/uploads/2017/03/photo_2017-06-19_16-39-12.jpg)

## The Brief

### Cheap

In common with a lot of my other weekend projects, this is very likely to be a white elephant, so it needs to be cheap to start with. There are lots of excellent wireless sensors already available, the [Moteino](https://lowpowerlab.com/shop/) is well regarded. Using that as a baseline, could I build something similar for less than £5 per unit? The answer is “nearly”, see the Bill Of Materials below.

### Use a ready made microcontroller board

I didn’t want to have to design a board which I would then have to solder an ATmega328/P to myself. Laying out that board would be hard, plus I would have to spec and solder all the supporting components. I could have done that with through-hole components I suppose, but then the board would have been massive, and I very much doubt it would have actually worked out any cheaper. Instead, I decided that a better idea was to solder on a ready-made Arduino clone, specifically the [Pro Mini](https://www.arduino.cc/en/Main/ArduinoBoardProMini). I buy them from [this supplier on eBay](http://rover.ebay.com/rover/1/710-53481-19255-0/1?icep_ff3=2&pub=5575128401&toolid=10001&campid=5337704861&customid=&icep_item=321413432145&ipn=psmain&icep_vectorid=229508&kwid=902099&mtid=824&kw=lg) and I’ve never had any problems. They are 3v / 5v switchable, very reliable and turn up quickly. Power consumption was going to be an important metric and running them off of a 3V supply would be essential.

### Use a ready made radio module

As with the MCU, I don’t want to be laying out a radio board or soldering surface mount components. I needed a ready-made radio module which was cheap, readily available and had low power requirements. I started off with the NRF24L01. It had a small footprint, cost around 99p a unit, had a built in antenna and was supported by the [RadioHead library](http://www.airspayce.com/mikem/arduino/RadioHead/). Initial testing on breadboard showed that the range of these devices would be sufficient for my house. I designed the first two revisions of the sensor board around this radio (and a CR2032, see the battery section below). Initial current draw was a bit too high for the CR2032, but I could have worked around that by using AAA batteries instead – but more annoying was that the range of the radios was not at all reliable. The 2.4GHz spectrum is very noisy and the addition of two baby monitors since the original breadboard test hasn’t helped, but in testing on the PCB I found these radios to be pretty much useless. I also tried using the version with the power amp as the central transceiver, this helped a little bit but they were still failing to get at least 50% of their transmissions to the other end, even over a couple of meters. Other people have reported really good range with these boards, but crucially those range tests were done outside. It’s my considered opinion that these radios and Wifi in the house do not co-exist.

So I gave up with those, and tried the **XL4432-SMT**. This is based on the Si443x chipset from [Silicon Labs](http://www.silabs.com/products/wireless/proprietary/ezradiopro-ism-band-transmitters-recievers-transceivers). They’re [readily available on eBay](http://rover.ebay.com/rover/1/710-53481-19255-0/1?icep_ff3=2&pub=5575128401&toolid=10001&campid=5337704861&customid=&icep_item=142189527394&ipn=psmain&icep_vectorid=229508&kwid=902099&mtid=824&kw=lg) for around £.170 each, so nearly twice the price of the NRF24L01. They’re well supported by the [RadioHead](http://www.airspayce.com/mikem/arduino/RadioHead/) library, can run down to 1.8V, have low current draw when on (virtually nothing when in stand-by), support a wide frequency range around the 433MHz ISM band and in range testing they out-performed the NRF24L01 by a long way. The downside is that these pre-made boards use 1.27mm pitch / castellated connections. I had to design an Eagle part to interface with them, but that wasn’t really too hard. See below for links to the parts I made. Another drawback was the antenna; being a much lower frequency means they need a much longer antenna so I would need to find a project box which could hold them.  
The Si443x also has a temperature sensor and a wake up timer on board. However, reading the errata from Silicon Labs it seems that the WUT is actually broken and the temperature sensor was returning very strange results. The datasheet says that you need to calibrate the temperature sensor so I tried doing this but go nowhere fast, and so I opted to use a 1wire sensor instead.

The other option for a radio would have been an ESP8266. You can get ready made boards cheap on eBay, and I could have done away with a separate MCU altogether but the power consumption of these devices is just too great for a project which needs to run of a couple of batteries for a year.

### Run for a long time on batteries

What’s the point in a wireless sensor if you have to plug it in to the mains. This project must run from batteries, and those batteries need to last a long time – having to change the batteries every few months would quickly get boring and the sensors would be left doing nothing. Obviously having the MCU go in to a sleep state between runs is going to be necessary, plus a radio which has modest power requirements when running. We can further reduce the power needs by making sure that the “work” that the sensor has to do is done as quickly as possible. The default 1wire temperature sensor code you find will typically takes around 2700msec to read. That’s a very long time. I changed the code a bit by hard-coding the hardware address of the sensor on the board and by only reading two bytes of temperature data (good for 0.5 degree accuracy). More information can be found below.

From a size perspective a CR2032 battery looks very appealing. Some early testing made me think that they would work fine but in real life I had a lot of problems. In hindsight I think I can put most of the problems down the Brown Out Detector on the Pro Mini being set to 2.8V, more on that in a moment.

UPDATE: Whoops. This post has been sat in drafts for nearly 2 years. Guess it’s not getting finished then. I’m posting this in case the above is interesting. Topics that I wanted to cover but haven’t are:

- Fit neatly in a box
- Read the temperature
- Be extensible
- Reading 1wire temperature sensors quickly
- Lower the BOD to 1.8V
- Bill Of Materials