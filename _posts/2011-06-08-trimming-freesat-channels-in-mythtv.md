---
id: 342
title: 'Trimming Freesat Channels In MythTV'
date: '2011-06-08T12:42:58+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=342'
permalink: /2011/06/08/trimming-freesat-channels-in-mythtv/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - linux
    - 'Making the world a better place'
    - tv
---

There are loads and loads of free-to-air channels available on the Astra 28 constellation, the vast majority of which I do not watch.

So to make things a bit easier for me after a full re-scan, I’ve put together a list of the channels I don’t watch and with a tiny bit of SQL I can trim them from my channel list.

To make things a bit easier for *you* here is a SQL dump of my “unwatched channels” list:

[unwatched\_channels.sql](/wp-content/uploads/2012/08/unwatched_channels.sql_.txt)

And here is the SQL to trim these from your channel list:

`update channel set visible=0,useonairguide=0 where name in (select name from unwatched_channels)`

You’ll probably want to edit that list yourself to remove and add the channels as you prefer. Generally speaking, my list trims:

- Regional variations
- Specialist interest
- Shopping
- Games and other text based services

I’ll update this list occasionally, this page will always have my most up to date information.

- UPDATE: 6 Sept 11. Refreshed channel list
- UPDATE: 8 Oct 11. Refreshed channel list
- UPDATE: 14 Dec 2011. Refreshed channel list
- UPDATE: 4 Aug 2012. Refreshed channel list
- UPDATE: 29 Dec 2013. New list of channel IDs available here: [unwatched\_by\_chanid](/wp-content/uploads/2013/12/unwatched_by_chanid.csv). Add a new table and import that CSV file. Then do a “update channel set visible=0 where chanid in (select chanid from &lt;your new table&gt;”