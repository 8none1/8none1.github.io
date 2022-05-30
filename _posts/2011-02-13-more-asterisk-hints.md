---
id: 322
title: 'More Asterisk hints'
date: '2011-02-13T19:21:08+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=322'
permalink: /2011/02/13/more-asterisk-hints/
categories:
    - asterisk
---

Wow – a day of fixing loads of niggling little Asterisk problems!

- Max duration

My calling plan gives me unlimited free calls as long as those calls are under an hour in duration. Pretty standard BT stuff. If you do make a call over an hour you don’t just get charged for that bit of the call over the hour, oh no, you get charged for the whole call.  
Anyway – we have the technology to defeat them!

In FreePBX under General Settings change your Asterisk Outbound Dial command options to include:

```
L(3360000,3240000,10000)
```

which will drop the call after 3360000ms (56 minutes) and should alert you at 3240000ms.

- Courtesy Tone

The above works very well and drops the calls, but without a bit of extra magic you don’t get the warning in your ear – it just drops the call. To enable the warning tones etc edit this:

```
/etc/asterisk/features_general_custom.conf
```

and add

```
courtesytone=beep
```

substituting “beep” for what ever noise you want to hear.

- Call Recording

By also adding:

```
Ww
```

to the Dial command options in #1 you can press \*1 when you’re in a call to start recording. It plays the courtesy tone to both parties though.