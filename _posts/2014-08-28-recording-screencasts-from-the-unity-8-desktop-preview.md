---
id: 590
title: 'Recording screencasts from the Unity 8 Desktop Preview'
date: '2014-08-28T12:33:10+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=590'
permalink: /2014/08/28/recording-screencasts-from-the-unity-8-desktop-preview/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - linux
    - 'Making the world a better place'
    - Ubuntu
---

# Obtaining and running Unity 8 Desktop Preview

If you like playing with new toys you might have already downloaded and tried the Unity 8 Desktop Preview (available here: <http://cdimage.ubuntu.com/ubuntu-desktop-next/daily-live/current/>)

If you haven’t, you should take it for a spin. If you have an Intel graphics everything should be fine and dandy, if not YMMV at the moment.

1. Download the ISO
2. Find a spare &gt;1GB USB thumb drive
3. Run “disks” via the dash
4. Highlight your USB drive and from the cog icon on the right choose “Restore Disk Image…”
5. Select your ISO and “Start Restoring” – this will of course erase everything else on your USB stick
6. Done

You can now boot from your USB stick and have a play with Unity 8. Right now you’ll be seeing the Phone view of Unity 8, but that will all be changing in time.

# Capturing a screencast

Once you’ve got everything up and running you might like to make a few screencasts, so how do you do it? Well, the Mir developers have provided us with the mirscreencast tool so let’s use that:

Switch to tty1 (ctrl+alt+f1) and log in and run:

```
mirscreencast --file <output file> -m /run/lightdm-mir-0
```

Then switch back to tty8 (ctrl-alt-f8) and use Unity.

Your file will now be filling up FAST. Mirscreencast will be trying to write every raw frame to that file, probably at a rate of 60 frames a second. To kill mirscreencast I first hit ctrl-z and then:

```
 pkill -9 mirscreencast
```

but there is probably a better way.

# Capturing a better screencast

There are a few command line options for mirscreencast which will can help us shrink the file size a bit:

```
mirscreencast --file <output file> -m /run/lightdm-mir-0 -s 683 384 -n 3600
```

The option “-s” will resize the captured frames. Note that 683 384 is exactly half my native resolution, so you will need to adjust this to your display.

The option “-n” will capture n frames and then stop. At 60 frames a second, 3600 frames is one minute. If you use -n then mirscreencast will exit gracefully at the end.

# Playing back your screencast

I am lucky enough to have a spare machine with a touch screen just for running Unity 8 on (<http://www.dell.com/uk/dfh/p/inspiron-11-3137/pd>) so I SCP the raw video file on to my main machine for playback and editing.

I use mplayer for most of my video playback and encoding needs and it will happily play the raw video file, but it needs a few pointers:

```
mplayer -demuxer rawvideo -rawvideo fps=60:w=683:h=384:format=bgra <filename>
```

or, to convert the raw file into something which you can edit in OpenShot try this:

```
mencoder -demuxer rawvideo -rawvideo fps=60:w=683:h=384:format=bgra -ovc x264 -o <output filename> <filename>
```

And then you can edit and upload the processed file. When I export from OpenShot I use the “Web” profile, then target “YouTube-HD”, “HD 720p 29.97 fps” “Med” – it’s a bit overly compressed, but it looks OK.