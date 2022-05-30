---
id: 1110
title: 'Finally moving to a static site'
date: '2021-07-14T14:46:20+01:00'
author: will
layout: post
guid: 'https://www.whizzy.org/?p=1110'
permalink: /2021/07/14/finally-moving-to-a-static-site/
categories:
    - 'Making the world a better place'
---

Amazon have made some changes to the (admittedly generous) Alexa AWS credits programme which made me think about the services that I run there and how to reduce my bill a bit. This WP blog was hosted on a t2.micro instance running Apache, MySQL, PHP, and WordPress. You need to add a bit of swap in order to keep MySQL happy but generally speaking it will tick along at very low load. So low in fact that I thought about merging the blog hosting instance with another instance which runs various home automation services, but decided that was too risky.

Rather than a security bug in WordPress exposing my internal network to the world I’ve decided to move everything to static pages. I don’t really post here any more, so it’s not like I need everything that WordPress offers and there are lots of places which will host static pages for either free or very cheap, so why not try it out? Here’s what I decided on and how I did it.

## Hosting

I considered the following:

1. Creating a virtualhost in an already existing Apache set up
2. Uploading a static copy of these pages to S3
3. Uploading a static copy of these pages to Github Pages
4. Moving to a cheaper cloud provider

I ruled out 1, I don’t want any public facing services on my otherwise private servers. I know enough to know that I don’t know anything about securing web servers.

I think #2 and #3 are largely the same in the end. As you have probably worked out by now, I decided that Github was the one I would try. This was decided entirely on cost. S3 would have cost me a couple of pennies a month, Github was free.

I also considered spinning up a new instance in [Digital Ocean](http://do.co/lnl) or [Linode](https://linode.com/latenightlinux) (referral link, get $100 credit) which would have been about 20% cheaper for a t2.micro equivalent. Github pages are free though, and get a decent CDN behind them. WordPress on a small server is slow without doing work.

## Getting the data out of WordPress

I used the [Simply Static](https://patrickposner.dev/plugins/simply-static/) plugin for WordPress to export the pages and images to HTML. It worked pretty well, and I would recommend it. Simply Static produced a zip file with everything in. I unzipped it and copied everything to the root of a new Github repo, committed it, pushed it, and it worked.

Some tips for preparing your blog for export:

- Disable comments on all your posts. This is a faff as you have to do it on each existing post. In the end I followed this guide: <https://themeisle.com/blog/disable-comments-in-wordpress/>  
    See section 3 on how to remove the ability to comment on existing posts.
- Remove all the meta links etc from the footer and sidebar. I did this through the WordPress “Appearance” -&gt; “Customise” menu to remove widgets and links as required.

## Adding new content in the future

I have zero interest in learning some new static blogging platform or writing posts in HTML or Markdown or using some automated jiggery pokery. I really like WordPress for it’s WYSIWYG writing interface. I write about one post every two years. I intend to switch off the instance hosting WordPress until such time as I need to write a new post, then I’ll switch it on, write the post and export it using Simply Static. This will cost me a few cents for the disk storage, and zero effort.