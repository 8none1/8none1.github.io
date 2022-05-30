---
id: 415
title: 'Fixing &#8220;warning: Please check that your locale settings&#8221;'
date: '2012-08-16T17:01:58+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=415'
permalink: /2012/08/16/fixing-warning-please-check-that-your-locale-settings/
categories:
    - Ubuntu
---

I took an Amazon AWS t1.micro instance for a spin the other day. A free server is not to be sniffed at. Of course I installed Ubuntu 12.04 on it.

I was getting a lot of locale errors, things like this:

```
perl: warning: Setting locale failed.
 perl: warning: Please check that your locale settings:
 LANGUAGE = (unset),
 LC_ALL = (unset),
 LC_MESSAGES = "en_GB.UTF-8",
 LC_COLLATE = "en_GB.UTF-8",
 LC_CTYPE = "en_GB.UTF-8",
 LANG = "en_US.UTF-8"
 are supported and installed on your system.
```

I thought this would just go away by itself, but it didn’t – so I had to fix it. Note: I’m in the UK, so I’m using en\_GB as my locale, change yours to en\_US or whatever.

Type:

```
export LANGUAGE=en_GB.UTF-8
sudo locale-gen en_GB.UTF-8
sudo dpkg-reconfigure locales
```

And you should be all set.