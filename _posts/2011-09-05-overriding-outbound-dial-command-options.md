---
id: 365
title: 'Overriding Outbound Dial Command Options'
date: '2011-09-05T19:03:24+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=365'
permalink: /2011/09/05/overriding-outbound-dial-command-options/
categories:
    - asterisk
---

I’ve previously noted that you can apply time limits to calls going out of your Asterisk box:

<http://www.whizzy.org/2011/02/more-asterisk-hints/>

using the L(nnn,mmm,yyy) options for DIAL\_TRUNK\_OPTIONS. But, what if you don’t want to limit the length of calls for a specific trunk?

Well, FreePBX has a context called \[macro-dialout-trunk-predial-hook\] which lets you jump in at the very last moment and override any settings you like, which is perfect for this sort of thing.

I got the idea from here:

<http://www.freepbx.org/book/export/html/5893>

I’ve added this to extensions\_custom.conf:

```
[macro-dialout-trunk-predial-hook]
 exten => s,1,NoOp(Trunk ${OUT_${DIAL_TRUNK}} selected)
 exten => s,n,Gotoif($["${OUT_${DIAL_TRUNK}}" != "SIP/TRUNK_NAME"]?skip)
 exten => s,n,NoOp(Setting DIAL_TRUNK_OPTIONS to Ww)
 exten => s,n,Set(DIAL_TRUNK_OPTIONS="Ww")
 exten => s,n(skip),MacroExit()
```

Which tests the name of the trunk which is to be used and explicitly sets DIAL\_TRUNK\_OPTIONS if it matches, otherwise nothing happens. This is a very powerful feature and my extremely crude hack doesn’t do it justice. I wanted to make a note of this before I forgot.

A better way might be to look at the dial prefix? I’ll investigate that option, but for now that will have to do.