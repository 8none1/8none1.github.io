---
id: 1020
title: 'Double helping of Pi Hole'
date: '2021-01-02T17:45:02+00:00'
author: will
excerpt: 'Improve the performance of Pi Hole by running it on a more powerful computer. Durr.'
layout: post
guid: 'https://www.whizzy.org/?p=1020'
permalink: /2021/01/02/double-helping-of-pi-hole/
categories:
    - linux
    - RaspberryPi
    - Ubuntu
---

In [episode 100 of Late Night Linux](https://latenightlinux.com/late-night-linux-episode-100/) I talked a little bit about trying out [Pi Hole](https://pi-hole.net/) and [AdGuard](https://adguard.com/en/welcome.html) to replace my home grown ad blocker based on [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) and a massive hosts file.

I came down in favour of Pi Hole for a couple of reasons but the deciding factor was that Pi Hole felt a bit more open and that it was built on top of `dnsmasq` which allowed me to reuse config for TFTP which netboots some devices which needed it.

Now that I’ve been using Pi Hole for a few months I have a much better understanding of its limitations and the big one for me is performance. Not the performance when servicing DNS requests but performance when querying the stats data, when reloading block lists and when enabling and disabling certain lists. I suspect a lot of the problems I was having is down to flaky SD cards.

I fully expect that for most people this will never be a problem, but for me it was an itch I wanted to scratch, so here’s what I did:

![](/wp-content/uploads/2021/01/Double-Pi-Hole.png)

Through the actually quite generous [Amazon Alexa AWS Credits promotion](https://developer.amazon.com/en-US/alexa/alexa-skills-kit/new/aws-promotional-credits) I have free money to spend on AWS services, so I spun up a `t2.micro` EC2 instance (1 vCPU, 1GB RAM – approx £10 a month) running Ubuntu.

I installed Pi Hole on that instance along with Wireguard which connects it back to my local network at home. I used [this guide from Linode](https://www.linode.com/docs/guides/set-up-wireguard-vpn-on-ubuntu/) to get Wireguard set up.

The Pi Hole running in AWS hosts the large block files and is configured with a normal upstream DNS server as its upstream (I’m using Cloudflare).

![](/wp-content/uploads/2021/01/image-2.png)
Pi Hole running in AWS configured with Cloudflare as its upstream DNS

I use three Ad block lists:

- `OISD:` <https://dbl.oisd.nl/>
- `Wally3k:` <https://v.firebog.net/hosts/static/w3kbl.txt>
- `Polish Filters Team: `[https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts\_without\_controversies.txt](https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts_without_controversies.txt)

![](/wp-content/uploads/2021/01/image-1.png)

Pi Hole running on a `t2.micro` instance is really speedy. I can reload the block list in a matter of seconds (versus minutes on the Pi) and querying the stats database no longer locks up and crashes Pi Hole’s management engine FTL.

The Pi Hole running on my LAN is configured to use the above AWS based Pi Hole as its upstream DNS server and also has a couple of additional block lists for [YouTube](https://raw.githubusercontent.com/8none1/pihole-blocklists/main/youtube/hosts) and [TikTok](https://raw.githubusercontent.com/llacb47/mischosts/master/social/tiktok-block).

![](/wp-content/uploads/2021/01/image.png)

This allows me use Pi Hole on a Pi as the DHCP server on my LAN and benefit from the GUI to configure things. I can quickly and easily block YouTube when the kids have done enough and won’t listen to reason and the heavy lifting of bulk ad blocking is done on an AWS EC2 instance. The Pi on the LAN will cache a good amount of DNS and so everything whizzes along quickly.

Pi Hole on the LAN has a block list of about 3600 hosts, whereas the version running in AWS has over 1.5 million.

All things considered I’m really happy with Pi Hole and the split-load set up I have now makes it even easier to live with. I would like to see an improved Pi Hole API for enabling and disabling specific Ad lists so that I can make it easier to automate (e.g. unblock YouTube for two hours on a Saturday morning). I think that will come in time. The split-load set up also allows for easy fallback should the AWS machine need maintenance – it would be nice to have a “DNS server of last resort” in Pi Hole to make that automatic. Perhaps it already does, I should investigate.

Why not just run Pi Hole on a more powerful computer in the first place? That would be too easy.

If you fancy trying out Pi Hole in the cloud or just playing with Wireguard you can get $100 free credit with Linode with the Late Night Linux referral code: <https://linode.com/latenightlinux>