---
id: 303
title: 'SMSq No call scheduled as already sending'
date: '2010-10-15T11:06:13+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=303'
permalink: /2010/10/15/smsq-no-call-scheduled-as-already-sending/
categories:
    - asterisk
---

In order to remind myself:

I kept getting this error when trying to send an SMS;

No call scheduled as already sending

It seems SMSq or Asterisk is quite picky about who sends the SMS. If I tired to send an SMS as myself it would fail, and then clog up /var/spool/asterisk/sms/motx with undead SMSes.

If I run smsq as the same user as Asterisk is running as it works (once you’ve deleted all the crap out of motx directory.

Just need to work out why now.