---
id: 753
title: 'Unity 7 Low Graphics Mode'
date: '2016-09-01T10:43:23+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=753'
permalink: /2016/09/01/unity-7-low-graphics-mode/
categories:
    - Ubuntu
---

Unity 7 has had a low graphics mode for a long time but recently we’ve been making it better.

[Eleni](https://wiki.ubuntu.com/hikiko) has been making improvements to reduce the amount of visual effects that are seen while running in low graphics mode. At a high level this includes things like:

- Reducing the amount of animation in elements such as the window switcher, launcher and menus (in some cases down to zero)
- Removing blur and fade in/out
- Reducing shadows

The result of these changes will be beneficial to people running Ubuntu in a virtual machine (where hardware 3D acceleration is not available) and for remote-control of desktops with VNC, RDP etc.

Low graphics mode should enable itself when it detects certain GL features are not available (e.g. in a virtualised environment) but there are times when you might want to force it on. Here’s how you can force low graphics mode on 16.04 LTS (Xenial) :

1. nano ~/.config/upstart/lowgfx.conf
2. Paste this into it:

```
start on starting unity7
pre-start script
    initctl set-env -g UNITY_LOW_GFX_MODE=1
end script
```

3. Log out and back in

If you want to stop using low graphics comment out the initctl line by placing a ‘#’ at the start of the line.

This hack won’t work in 16.10 Yakkety because we’re moving to systemd for the user session. I’ll write up some instructions for 16.10 once it’s available.

Here’s a quick video of some of the effects in low graphics mode:

<iframe allowfullscreen="allowfullscreen" frameborder="0" height="315" loading="lazy" src="//www.youtube.com/embed/gWUyP-oTRVg" width="560"></iframe>