---
id: 275
title: 'A more resilient DNS set up'
date: '2010-06-15T11:23:43+01:00'
author: will
excerpt: 'A replacement for ZoneEdit?'
layout: post
guid: 'http://www.whizzy.org/?p=275'
permalink: /2010/06/15/a-more-resilient-dns-set-up/
categories:
    - 'Making the world a better place'
---

ZonEdit are suffering from a DDOS attack or something and their default free servers have stopped responding. (See: <http://www.zoneedit.com/status.html?>)

This has caused me a few problems, the main one being that my MX records are held at ZoneEdit and so my email has effectively stopped working. Initially I was thinking of a quick workaround, but actually this is working rather nicely.

What I’ve done is:

- Created an account at <http://www.web-dns.co.uk> and added similar entries as I did with ZoneEdit. Namely, my MX records pointing back to Google, the A record for this site and a few others and some CNAME records for other Google services.
- Updated the nameservers for the whizzy.org domain at my registrar to have ZoneEdit’s two free DNS servers as 1 and 2, and then the first web-dns server as number 3.

That’s it. If ZoneEdit stop working, then once the old cached entries have expired new DNS requests start being serviced by web DNS while the ZoneEdit servers are down. I could in theory have three different DNS services listed with my registrar but ZoneEdit have been really good, so I’m happy to stick with them for now. If this sort of thing happens in the future then maybe I’ll host the DNS myself.

Web DNS gives you much rawer access to the zone file compared to ZoneEdit, but a bit of googling and you’ll be fine. For example, for an MX record:

1. Leave the first box (the name box) emtpy.
2. Leave the TTL as 3600
3. Change the IN to MX
4. Set the number (auxiliary information)to zero for the first mail server, and 10 for the second, 20 for the third and so on.
5. Set the data (the last box) to, in the case of Google Mail (e.g. Google Apps for Domains) to ASPMX.L.GOOGLE.COM. (note the trailing dot, very important)
6. Add the other mail servers in the same way

For a straight forward hostname resolution e.g. www.whizzy.org:

1. Set the first box to WWW
2. Leave the TTL as 3600
3. Change the IN to A
4. Set the number to zero
5. Set the data to your server IP address, e.g. 174.133.50.212