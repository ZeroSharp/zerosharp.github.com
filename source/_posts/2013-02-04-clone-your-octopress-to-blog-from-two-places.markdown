---
layout: post
title: "Clone your Octopress to blog from two places"
date: 2013-02-01 11:59
comments: true
categories: [octopress]
description: How to recreate an accidentally deleted Octopress blog. How to blog from two different places.
---
{% img right /images/blog/clone-your-octopress-001.png %}

This post covers how recreate a local repository of your Octopress blog. Perhaps you've accidentally lost it, or perhaps you would like to be able to blog from two different places. Recently [I bought a new computer](/the-best-pc-laptop-is-a-mac/) and I wanted to be able to blog from both my desktop and my laptop.

### How Octopress works ###

Octopress repositories have two branches, `source` and `master`. The `source` branch contains the files that are used to generate the blog and the `master` contains the blog itself.

When the local folders are initially configured according to the [Octopress Setup Guide](http://octopress.org/docs/setup/), the `master` branch is stored in a subfolder named '_deploy'. Since the folder name begins with an underscore, it is ignored when you `git push origin source`. Instead, the `master` branch (which contains your blog posts) gets updated when you `rake deploy`.

## Recreating a local Octopress repository ##

To recreate the local directory structure of an existing Octopress blog, follow these instructions.

### Clone your blog to the new machine ###

First you need to clone the `source` branch to the local octopress folder.

    $ git clone -b source git@github.com:username/username.github.com.git octopress

Then clone the `master` branch to the _deploy subfolder.

    $ cd octopress
    $ git clone git@github.com:username/username.github.com.git _deploy 

Then run the rake installation to configure everything

    $ gem install bundler
    $ rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
    $ bundle install
    $ rake setup_github_pages

It will prompt you for your repository URL.

    Enter the read/write url for your repository
    (For example, 'git@github.com:your_username/your_username.github.com)

That's you setup with a new local copy of your Octopress blog.

### Pushing changes from two different machines ###

If you want to blog from more than one computer, you need to make sure that you push everything before switching computers. From the first machine do the following whenever you've made changes:

    $ rake generate
    $ git add .
    $ git commit -am "Some comment here." 
    $ git push origin source  # update the remote source branch 
    $ rake deploy             # update the remote master branch

Then on the other machine, you need to pull those changes.

    $ cd octopress
    $ git pull origin source  # update the local source branch
    $ cd ./_deploy
    $ git pull origin master  # update the local master branch

Of course, it might be easier to deploy everything from a thumb drive instead... 
