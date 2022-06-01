---
layout: post
title: Using Jekyll for static hosting
date: '2022-06-01T08:24:00+01:00'
author: will
categories:
    - 'Making the world a better place'
---

You live and learn.

I'd previously stated that I had no interest in learning a new static blog hosting package.

I said that if I wanted to create a new blog post I'd simply spin up the old Wordpress VM, write the blog post in the WYSIWYG editor, export it to an HTML file, upload it to GitHub and be done.  Welp, I changed my mind.

As soon as it came to actually write a post, I found myself eager to avoid interacting with a heavy editor and decided that I'd be much happier just bashing out some markdown in a text editor instead.  So here we are.

## How I migrated from Wordpress to Jekyll

I did indeed fire up the Wordpress VM, that went exactly as planned. But I installed the (https://en-gb.wordpress.org/plugins/jekyll-exporter/)[Jekyll Exporter] plug-in.  This was very easy.
I installed it, clicked the `export` button and was given a zip file with all the text converted to markdown and images in a separate directory linked to correctly.
In short, it just worked.

There were a few strange formatting errors, but I think this comes from Wordpress (or me) embedding HTML directly in the post instead of using styles correctly.

## Hosting on GitHub

GitHub will build and host your Jekyll site direct from a repo.  This is well documented.  I opted to use the (https://beautifuljekyll.com/)[Beautiful Jekyll] theme which has the option to clone the repo in GitHub and have everything else pre-configured.  I had some problems with this where GitHub wouldn't build the project because it couldn't find the theme.  This was very strange.  In the end I think I had broken the `_config` file.  Replacing it with the stock Beautiful Jekyll file and then updating it to match what I wanted worked.

## Changes I made

All my posts in Wordpress had `categories` and a few also had `tags`.  The categories were transferred in the post frontmatter by Jekyll Exporter, but by default Beautiful Jekyll didn't display them.

I reused the `tags` logic to also display `categories` like this:

1. edit `_layouts/home.html` to include this.  It's a copy and paste of the tags handler updated to refer to categories instead.

```html
    {% if site.feed_show_tags != false and post.categories.size > 0 %}
    <div class="blog-tags">
      <span>Categories:</span>
      {% for tag in post.categories %}
      <a href="{{ '/categories' | absolute_url }}#{{- tag -}}">{{- tag -}}</a>
      {% endfor %}
    </div>
    {% endif %}
```

2. Create a new `categories.html` page.  This allows you to list all the categories in use:

```html
---
layout: page
title: 'Category Index'
---

{% assign date_format = site.date_format | default: "%B %-d, %Y" %}

{%- capture site_categories -%}
    {%- for tag in site.categories -%}
        {{- tag | first -}}{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
{%- endcapture -%}
{%- assign tags_list = site_categories | split:',' | sort -%}

{%- for tag in tags_list -%}
    <a href="#{{- tag -}}" class="btn btn-primary tag-btn"><i class="fas fa-tag" aria-hidden="true"></i>&nbsp;{{- tag -}}&nbsp;({{site.categories[tag].size}})</a>
{%- endfor -%}

<div id="full-tags-list">
{%- for tag in tags_list -%}
    <h2 id="{{- tag -}}" class="linked-section">
        <i class="fas fa-tag" aria-hidden="true"></i>
        &nbsp;{{- tag -}}&nbsp;({{site.categories[tag].size}})
    </h2>
    <div class="post-list">
        {%- for post in site.categories[tag] -%}
            <div class="tag-entry">
                <a href="{{ post.url | relative_url }}">{{- post.title | strip_html -}}</a>
                <div class="entry-date">
                    <time datetime="{{- post.date | date_to_xmlschema -}}">{{- post.date | date: date_format -}}</time>
                </div>
            </div>
        {%- endfor -%}
    </div>
{%- endfor -%}
</div>
```

## First impressions

Jekyll is written in Ruby which means interacting with it's Gems package handler.  This is quite opaque to me.  I should probably have used something in Python if I wanted to understand what was going on.  However, GitHub recommend Jekyll so that's what I used.  Once I learned to stop caring *how* it worked and to just use it, everything went smoothly enough.  Battling to get the site built in GitHub using a combination of the docs provided by GitHub, the docs provided by Jekyll and the docs provided by Beautiful Jekyll was a confusing mess.  My advice is to either clone Beautiful Jekyll and then hack it in to what you want once it's running - or - follow the GitHub instructions.  Not both.

I quite liked how easy it was to include categories in the posts by hacking the existing tags handler. 

The Jekyll eco-system is advanced and can do way more than I need it to.  I can't really be bothered to learn all the intricacies but it's nice to know that they are there should I change my mind.

In the end it took me about a day to get all my old posts ported and fixed up and to get the site building in GitHub and serving pages.  From this point forwards writing a post should be a lot easier and more portable.  The best thing about Jekyll is that once your posts are in markdown you're not locked in to using it forever more.

I went through all this because I actually have something to write about.  I'll get on with that this weekend.
