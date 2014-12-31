---
layout: post
title: Order Jekyll Posts on the Same Day
date: 2014-12-31 18:00:00 +08:00
description: ""
headline: ""
categories: jekyll
tags: 
  - Jekyll
  - Programming
comments: true
mathjax: null
featured: false
share: true
published: true
---

In Jekyll, you can simply name your posts yyyy-MM-dd-title.md to define date of posts and display posts in the order of the date. However, if you have several posts that are posted on the same day, they will be ordered randomly.

To solve this, just add <code>date: yyyy-MM-dd HH:mm:ss</code> in the YAML front matter of each post.

For example, if you have 2 posts, post_1.md(the older one) and post_2.md(the latest one) on 2014/12/31, add <code>date: 2014-12-31 00:30:00</code> to post_1.md, and <code>date: 2014-12-31 00:40:00</code> to post_2.md, then they will be ordered with these attributes.

To display the correct date, you may need to add the time zone like this: <code>date: 2014-12-31 00:30:00 +08:00</code>.

That's it! Enjoy blogging :)
