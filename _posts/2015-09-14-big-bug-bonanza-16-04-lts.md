---
id: 679
title: 'Big Bug Bonanza Ubuntu 16.04 LTS'
date: '2015-09-14T15:30:12+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=679'
permalink: /2015/09/14/big-bug-bonanza-16-04-lts/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - Ubuntu
---

The vast majority of Ubuntu desktop users prefer to stick with a long term support release (<https://wiki.ubuntu.com/LTS>) rather than the regular 6 monthly releases, so 16.04 LTS represents the next big upgrade for most Ubuntu users. 16.04 LTS will be running Unity 7 by default as it has done for the last six years and our focus for the Unity 7 stack is fixing bugs which adversely affect the user experience of the desktop.

Over the years the bug lists for Unity 7 and Compiz have grown to become unmanageable. To make sure we are focusing on the most important issues we have to do some serious tidying up of the bug lists and we need some help.

At the time of writing there are 2680 open bugs for Unity 7 (<https://launchpad.net/ubuntu/+source/unity/+bugs>) and 1455 for Compiz (<https://launchpad.net/ubuntu/+source/compiz/+bugs>) and 322 for nux, our graphical toolkit (<https://launchpad.net/ubuntu/+source/nux/+bugs>).

We’re proposing to cut this down to size with the following plan:

1. **Close all bugs which relate to an unsupported release of Ubuntu.** We will do a manual review of the high heat bugs affecting unsupported releases first, but low heat bugs will most likely be closed by a robot. The rationale is that the majority of these older bugs will have been fixed and that the original reporter is probably no longer affected by the bug and has forgotten to close it. Plus manually screening each of these bugs cannot be done at this scale in a reasonable timeframe. There will be some collateral damage which is an unfortunate but unavoidable side-effect. Sorry if this affects you, but please do re-open the bug against a supported release.
2. **Close all private apport bugs and review public ones with a view to closing them as well.** Apport is the automated error reporting tool which runs when it detects a crash. It can open a private bug in Launchpad, private because stack traces might contain sensitive information which shouldn’t be public. We have [errors.ubuntu.com](http://errors.ubuntu.com) which can monitor crashes and provide a *much* clearer picture of which crashers are affecting numerous people and which are one-offs. We will use errors.ubuntu.com instead of trying to triage the 250 or so bugs which fall in to this category.
3. **Manually try to reproduce bugs and flag those which are still a problem.** This is where we need the most help. We will create a list of bugs which need to be checked and then ask people to spend a few minutes trying to reproduce a chosen bug on 15.10. If it’s still a problem then the tester would mark the bug as triaged or add a specific tag, or if it cannot be reproduced then they would mark the bug as Invalid. This will give us a curated list of real bugs which we can then triage further to assess the impact and priority. We will work through the triaged list in an agile manner and have regular meetings to review what has been fixed and decide on which bugs to focus on next. By distributing this problem across many people we can get the job done in a reasonable time scale.

## How you can help

First of all we need help in triaging the bug list. You don’t need to be a superstar software developer to do this, everyone can help and contribute to Ubuntu. You will need a [Launchpad](https://launchpad.net/) account though. We will publish a link to a list of bugs in Launchpad for Unity 7 (and in time Compiz &amp; Nux) which we think need manual checking. The links are available at this wiki page: <https://wiki.ubuntu.com/BigDesktopBugScrub>

Please choose a bug from this list and try to recreate it in 15.10. If your main machine isn’t running 15.10 you could set up a virtual machine using VirtualBox.

1. **Choose a bug from the list.**  The heat metric is a good indication of which bugs are more important to a lot of people. The list is sorted by heat so selecting one from somewhere near the top is a good starting point. It’s possible that someone else will be working on the same bug as you so check the comments to see if anyone has added anything recently.
2. **Can you recreate the bug?** There are a number of possible outcomes when you attempt to recreate the bug. Listed below are the most common ones. If you can’t match one of these categories directly, or don’t know what to do just leave the bug where it is and try a different one. 
    1. **No – I can’t understand from the report what the problem is**: 
        1. Add a comment along the lines of: “Thank you for taking the time to report this bug. Unfortunately we can’t work out how to recreate this bug from your description. Please describe the process you go through to trigger this bug and then change the bug status to NEW. See this page for more information. https://wiki.ubuntu.com/BigDesktopBugScrub”
        2. Set the bug status to **Incomplete**
    2. **No – I’ve tried to but it doesn’t seem to be a problem any more**: 
        1. Add a comment along the lines of: “Thank you for taking the time to report this bug. We have tried to recreate this on the latest release of Ubuntu and cannot reproduce it. This bug is being marked as Invalid. If you believe the problem to still exist in the latest version of Ubuntu please comment on why that is the case and change the bug status to NEW.”
        2. Set the bug status to **Invalid**
    3. **Yes – it’s still a problem in 15.10**: 
        1. Add a comment along the lines of: “As part of the big bug review for 16.04 LTS I have tested this on 15.10 and the bug is still there.”
        2. Mark the bug as **Triaged** or, if you don’t have permission to do that, add the tag “desktop-bugscrub-triaged”
    4. **Yes – but I don’t think it’s really a bug (perhaps a feature request)**: 
        1. Add a comment along the lines of: “As part of the big bug review for 16.04 LTS I have tested this on 15.10 and the bug is still there. I think this is a feature request rather than a bug.”
        2. Mark the bug as “**Opinion**”, or if you don’t have permission to do that, add the tag “desktop-bugscrub-opinion”
3. **Thank you!** We’re one bug closer to perfection!
4. Lather, Rinse, Repeat

## What happens next

Once we have a list of high quality, reproducible bug reports which are affecting many people we can start to chip away at them in a logical manner. We will be using an Agile-like workflow:

1. Meet at the start of a “[sprint](http://scrummethodology.com/scrum-sprint/)” to discuss which of the most important bugs (importance will be decided on a mixture of bug heat and expert knowledge) will be working on during the sprint duration. We will decide how many of the bugs we think are fixable in that sprint and take them into our backlog. The backlog will be managed using Trello (<https://trello.com/b/9YvUSYqq/unity-7>).
2. The sprint will start and developers will take bugs (Cards) from the backlog to work on.
3. The card will move to the In Progress colum
4. If there is a problem the bug will move to the blocked column and these cards will be discussed at regular intervals during the sprint.
5. Once a bug is fixed it will move to the Review column. A code review will be done and if everything is OK then the fix will be merged and automatically tested. If there are problems it will move back to the In Progress column.
6. At the end of the sprint the fixes will be demonstrated and everyone will have a chance to spot any problems with the fix. If there is a problem the card will go back into the Backlog for more work next sprint. If everything is OK then the card is moved to Done and that bug is now fixed.
7. The next sprint will start and we will go back to step 1.

We will endeavour to do our reviews in a Hangout On Air so that everyone can join to see what progress is being made. We will also use our IRC channel on Freenode [\#ubuntu-desktop](http://is.gd/ubuntu_desktop_irc).

## Software developers who want to help

If you are a developer who wants to help fix the code as well as triage bugs please join us on IRC ([\#ubuntu-desktop on Freenode](http://is.gd/ubuntu_desktop_irc)) and introduce yourself. We can get you write access to the Trello board and invite you along to the Sprint planning and review meetings. We’d love you to get involved.

## Bug Squash Hours

In order to kick start the process we will be setting aside a few hours a week where a core Unity 7 developer will be available on IRC to help answer questions about bugs and we’ll be working through the list as well. Feel free to ask for help or come and join us while we work through the bug list. Exact schedule will be announced as soon as we know what it is.

## Ubuntu Online Summit

We will have a session at UOS to review how the bug triage is going, discuss our tooling and policy on which bugs to auto-close etc.