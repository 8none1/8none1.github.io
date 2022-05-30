---
id: 730
title: 'DHCP clients not registering hostnames in DNS automatically'
date: '2016-02-17T20:43:59+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=730'
permalink: /2016/02/17/dhcp-clients-not-registering-hostnames-in-dns-automatically/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - IoT
    - linux
    - RaspberryPi
---

To remind myself as much as anything:

I run a dnsmasq server on my router (which is a [Raspberry Pi 2](https://www.whizzy.org/2015/05/multipathrouting-rasppi2/)) to handle local DNS, DNS proxying and DHCP. For some reason one of the hosts stopped registering its hostname with the DHCP server, and so I couldn’t resolve its name to an IP address from other clients on my network.

I’m pretty sure it used to work, and I’m also pretty sure I didn’t change anything – so why did it suddenly stop? My theory is that the disk on the client became corrupt and a fsck fix removed some files.

Anyway, the cause is that the DHCP client didn’t know to send it’s hostname along with the DHCP request.

This is fixed by creating (or editing) `/etc/dhcp/dhclient.conf` and adding this line:

`send host-name = gethostname();`