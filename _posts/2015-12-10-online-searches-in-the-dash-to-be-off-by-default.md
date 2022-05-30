---
id: 721
title: 'Online searches in the dash to be off by default.'
date: '2015-12-10T10:53:10+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=721'
permalink: /2015/12/10/online-searches-in-the-dash-to-be-off-by-default/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - Ubuntu
---

Scopes are a leading feature of the Ubuntu Phone and of Unity 8 in general. That concept, the story of scopes, started out in Unity 7 and in 12.10 when we added results from online searches to the dash home screen. 

Well, we’re making some changes to the Unity 7 Dash searches in 16.04 LTS. On Unity 8 the Scopes concept has evolved into something which gives the user finer control over what is searched and provides more targeted results. This functionality cannot be added into Unity 7 and so we’ve taken the decision to gracefully retire some aspects of the Unity 7 online search features.

### What is changing?

First of all online search will be off by default. This means that out-of-the-box none of your search terms will leave your computer. You can toggle this back on through the Security &amp; Privacy option in System Settings. Additionally, if you do toggle this back on then results from Amazon &amp; Skimlinks will remain off by default. You can toggle them back on if you wish. Further, the following scopes will be retired from the default install and moved to the Universe repository for 16.04 LTS onwards:

1. Audacious
2. Clementine
3. gmusicbrowser
4. Gourmet
5. Guayadeque
6. Musique


The Music Store will be removed completely for 16.04 LTS onwards.

### Why now?

By making these changes now we can better manage our development priorities, servers, network bandwidth etc throughout the LTS period. We allow ourselves more freedom to make changes without further affecting the LTS release (e.g SRUs), specifically we can better manage the eventual transition to Unity 8 and not have to maintain two sets of scope infrastructure for the duration of the LTS support period of five years.

### What about previous supported releases?

Search results being off by default will not affect previous releases or upgrades, only new installs (i.e. we will not touch your existing settings). Changes to search results from Amazon &amp; Skimlinks will also only affect 16.04 and beyond. The removal of the Music Store will be SRU’d back to older supported releases and the option will be removed from the Dash.

### When will this happen?

We’re preparing the make the changes in the archive, to Unity 7 and to the Online Search servers right now. This will take a little while to test and roll out. We’ll let you know once all the changes are in Xenial.