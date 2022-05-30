---
id: 468
title: 'Combining MythTV and Asterisk'
date: '2013-12-15T10:56:04+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=468'
permalink: /2013/12/15/combining-mythtv-and-asterisk/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - asterisk
    - linux
    - tv
---

I’ve had this idea for a while and with the discovery of the Google Text-to-speech and Voice Recognition AGI scripts from Zaf (<http://zaf.github.io/asterisk-googletts/> &amp; <http://zaf.github.io/asterisk-speech-recog/>) I’ve implemented a quick proof-of-concept.

You can see the results in this YouTube video:

<iframe allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" frameborder="0" height="433" loading="lazy" src="https://www.youtube.com/embed/7DMeRVU9hik?feature=oembed" title="MythTV, Asterisk and Speech Recognition" width="770"></iframe>

Over the next few days I’ll tidy up the code and write up a blog post about how to do it. It’s pretty straight forward though, using APIs provided by Google, MythTV and Asterisk and then just glueing them together.