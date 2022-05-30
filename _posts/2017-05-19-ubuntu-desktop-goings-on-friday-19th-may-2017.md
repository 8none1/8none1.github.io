---
id: 943
title: 'Ubuntu Desktop goings on.  Friday 19th May 2017'
date: '2017-05-19T14:45:55+01:00'
author: will
layout: post
guid: 'https://www.whizzy.org/?p=943'
permalink: /2017/05/19/ubuntu-desktop-goings-on-friday-19th-may-2017/
categories:
    - Ubuntu
---

# Ubuntu Desktop Newsletter

I’m going to start a weekly newsletter style update to keep people abreast of what’s been going on with Ubuntu Desktop as we move to GNOME Shell and build the foundations for 18.04 LTS. Here’s the first instalment:

## Friday 19th May 2017

### GNOME

We’re on to the last few MIR (<https://wiki.ubuntu.com/MainInclusionProcess>) reviews for the packages needed to update the seeds in order to deliver the GNOME desktop by default.  
We still have some security questions to answer about how we deal with updates to mozjs/gjs in an LTS release (where mozjs has a support period of 12 months but we need to offer support for a full five years). This is being looked at now, but for 17.10 we are set.  
We are aiming to have the seeds updated next week, and this will be the first milestone on the road to a fantastic GNOME experience in 17.10 Artful.

We’ve been fixing bugs in the Ambiance &amp; Radiance themes to make them look crisp on GNOME Shell.  
<http://www.omgubuntu.co.uk/2017/05/install-improved-ambiance-gnome-theme>

We’ve also triaged over 400 GNOME Shell bugs in Launchpad to allow us to more easily focus on the important issues.

We have been working on removing Ubuntu’s custom “aptdaemon” plugin in GNOME Software in favour of the upstream solution which uses PackageKit. This allows us to share more code with other distributions.

### LivePatch

<https://www.ubuntu.com/server/livepatch>

LivePatch delivers essential kernel security updates to Ubuntu machines without having to reboot to apply them. As an Ubuntu user you can sign up for a free account.  
We’re working on integrating LivePatch in to the supported LTS desktops to provide a friendly way to setup and configure the service.  
This week we started to investigate the APIs provided by the LivePatch services so we can report LivePatch activity to the user, obtain an API key on behalf of the user &amp; set up the service. Work has also started on the software-properties-gtk dialogs (aka Software &amp; Updates in System Settings) to add the options required for LivePatch.

### QA

Added upgrade tests from Zesty to Artful for Ubuntu and flavours. Working on making all these tests pass now so that everyone will have a solid and reliable upgrade path.  
Work is being done on the installer tests. This will extend the current installer tests to check that not only has the install completed successfully but that all desktop environment is working as expected, this had previously been covered with manual tests.

### Package Updates

- GStreamer is now at 1.12 final in 17.10.
- Chromium: stable 58.0.3029.110, beta 59.0.3071.47, dev 60.0.3095.5
- LibreOffice 5.3.3 is being tested.
- CUPS-filters: 1.14.0
- Snapd-glib: 1.12

### Snaps

More GNOME applications are being packaged as Snaps. There is still some work to do to get them fully confined and fully integrated into the desktop. We’re working on adding Snap support to Gtk’s Portals to allow desktop Snaps to access resources outside their sandbox.  
We will start tracking the Snaps here:  
<https://wiki.ubuntu.com/DesktopTeam/GNOME/Snaps>

### In the news

Interview with Ken VanDine on the GNOME Desktop in Ubuntu: [http://www.omgubuntu.co.uk/2017/05/ubuntu-switch-to-gnome-questions-answered  ](http://www.omgubuntu.co.uk/2017/05/ubuntu-switch-to-gnome-questions-answered)

There’s also a survey running to get feedback on some extensions which could be shipped with Ubuntu Desktop: <http://www.omgubuntu.co.uk/2017/05/ubuntu-desktop-gnome-extensions-survey-1710>

This was picked up by the Linux Unplugged podcast as their headline story: <http://www.jupiterbroadcasting.com/114701/that-new-user-smell-lup-197/>