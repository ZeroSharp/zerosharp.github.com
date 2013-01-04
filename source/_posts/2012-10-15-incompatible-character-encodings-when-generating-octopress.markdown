---
layout: post
title: "Incompatible character encodings when generating Octopress"
date: 2012-10-15 16:26
comments: true
categories: [octopress]
description: A fix for 'Liquid error incompatible character encodings' when running 'rake generate'
---

I started getting an encoding error whenever I tried to generate my Octopress blog with `rake generate`. The error message was:

    Liquid error: incompatible character encodings: UTF-8 and IBM437

I tried to save all my recently modified files with UTF-8 encoding, but I couldn't get the error to go away until
I found a solution on [this blog in Chinese](http://chxt6896.github.com/blog/2012/02/13/blog-jekyll-native.html#comment-678936946). 

Set two environment variables (via Control Panel/System/Advanced System Settings/Environment Variables) as follows:

    LC_ALL = en_US.UTF-8
    LANG = en_US.UTF-8

Now my blog generates without error.