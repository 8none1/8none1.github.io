---
id: 353
title: 'TCP sessions causing lag'
date: '2011-07-04T12:06:33+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=353'
permalink: /2011/07/04/tcp-sessions-causing-lag/
categories:
    - 'Making the world a better place'
---

I like to play Call Of Duty online. I find it’s a really good way to get angry and frustrated at inanimate objects. I also run a Bit Torrent client on my network.

I don’t suck at MW2 and I’m not too bad at Black Ops either, but on the PS3 Black Ops has been very unstable to the point of hanging the console.

Anyway, what really gets me cross is Lag. You can tell a match is going to be laggy the second the game opens. It feels as if you’re running through treacle. “GET A MOVE ON” I scream. And then it starts; you get shot while behind brick walls, your shots fail to connect even though “IT WAS CLEARLY A HEADSHOT WHY WONT YOU JUST DIE?”. And so on. You know what I’m talking about.

Originally I had a written a script to ping the PS3 and when it came online and tell Transmission to pause all the Torrents. When the PS3 went offline again it would start them. I thought this would be enough, but it isn’t. Even with Transmission paused and the bandwidth limited to zero kBps it still maintains a connection to other peers.

While this isn’t a significant amount of traffic over the WAN – it really does cause lag. Presumably having something like 1000 NAT sessions in the memory of my router is a bit of a stretch, so now I simply kill Transmission when the PS3 comes online. It takes a few minutes for the sessions to get pruned but it’s made a noticeable difference to the quality of my matches.

Of course, you’re still at the mercy of the internet connection of whoever hosts the match – but that’s beyond your control.