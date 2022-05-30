---
id: 299
title: 'LogWatch SMART monitoring'
date: '2010-09-27T12:09:23+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=299'
permalink: /2010/09/27/logwatch-smart-monitoring/
categories:
    - linux
    - 'Making the world a better place'
---

Why is one of my boxes reporting SMART data in the LogWatch and the others not?

I’d installed smartmontools but it just doesn’t seem to be working.

Finally sussed it. Have a look at /etc/default/smartmontools and enable deamon