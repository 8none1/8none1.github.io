---
id: 359
title: 'NOOP scheduler &#8211; not worth it for SSD'
date: '2011-08-16T09:45:33+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=359'
permalink: /2011/08/16/noop-scheduler-not-worth-it-for-ssd/
categories:
    - linux
---

I’ve got a new Thinkpad X220i with a 128GB SSD.

Reading around the internet I found a lot of stuff about squeezing a bit more throughput from your drive.

I did a couple of benchmarks in Ubuntu 11.04:

- Adding noatime,discard to /etc/fstab

**Result:**  No change. Not better or worse, but in theory the “discard” will help in the long term

**Decision:** Switch this on.

- Setting the scheduler to noop in /etc/rc.local

**Result:** Average read rate increased by 2MB/s but Average access time went from 0.2ms to 0.3 ms. In itself that’s not a real problem, but the graphs show a different picture.

Without NOOP

Note the scale on the right hand side.

![](/wp-content/uploads/2011/08/without_noop.png)

With NOOP

![](/wp-content/uploads/2011/08/with_noop.png)

The access speed is all over the place.

**Decision:** Switch this off

I’d be interested in hearing if your experience differs, but it seems to me that “Doing nothing” is a valid choice. Ubuntu, out of the box, doesn’t really require any fettling in order to get the best from your SSD.