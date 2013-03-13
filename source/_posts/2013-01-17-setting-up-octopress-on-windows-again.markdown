---
layout: post
title: "Setting up Octopress on Windows Again"
date: 2013-01-17 11:20
comments: true
categories: [octopress, github]
description: How to set up Octopress on Windows and host your blog with GitHub.
---

My [very first blog post](/setting-up-octopress-on-windows/) was about setting up Octopress. The following is an updated version of those instructions for setting up Octopress with Windows, ruby 1.9.3, python 2.7.3.

This is quick guide to setting up [Octopress](http://octopress.org/) to publish to GitHub pages. I'm using Windows 8 64-bit, but the instructions should work with other versions of Windows.

### Get with GitHub ###

First, get an account on GitHub and follow the excellent instructions for Windows here.  [here](http://help.github.com/win-set-up-git/).

Once you are set up with GitHub, get [yari](https://github.com/scottmuc/yari) by opening a command prompt:

Then create a directory for all your github projects.  You could put this anywhere, e.g., `%USERPROFILE%\github`.  I chose to create a directory `C:\projects\github\`.

### Installing Octopress ###

{% img right /images/blog/octopress_logo.png %} 

#### Use yari instead of RVM/rbenv ####

Scott Muc has written [yari](https://github.com/scottmuc/yari) which lets you switch between Windows Ruby versions.  Get it with the following command.

    cd C:\projects\github\
    git clone git://github.com/scottmuc/yari.git yari

Once this has completed you can setup the required Ruby environment with the command:

    yari 1.9.3

Then follow the rest of the instructions from [the Octopress setup instructions](http://octopress.org/docs/setup/).

    git clone git://github.com/imathis/octopress.git octopress
    cd octopress
    ruby --version  # Should report Ruby 1.9.3 thanks to yari

Next install dependencies

    gem install bundler
    bundle install

Install the default Octopress theme.

    rake install

### Deploying to GitHub ###

The instructions are [here](http://octopress.org/docs/deploying/github/).  The only difficulty I had was working out which URL to use after:

    rake setup_github_pages

This will ask you for your Github Pages repository url, but it is not clear that this is the SSH one, e.g., `git@github.com:ZeroSharp/zerosharp.github.com.git`.  You can find it near the top of [the repository page](https://github.com/ZeroSharp/zerosharp.github.com).

### Configuring your blog ###

Follow the instructions [here](http://octopress.org/docs/configuring/) to configure your blog.

#### Markdown ####
It is quite difficult to find good help on the markdown syntax other than for the codeblocks section.  For instance, it is not clear how to generate a codeblock with no line numbers, like this one:

    How do I output a codeblock with no line numbers?

It turns out it either requires a `<pre>` tag instead of a `{ codeblock }` tag, or alternatively, start the line with four spaces.  The official syntax is [here](http://daringfireball.net/projects/markdown/syntax), but I also found [this cheat sheet](http://warpedvisions.org/projects/markdown-cheat-sheet/).

#### Fonts and Styles ####
See [Lee's Bigdinosaur blog](http://blog.bigdinosaur.org/changing-octopresss-header/).

#### Problem with CodeBlocks ####
I had a problem with codeblocks which seems to be specific to Windows.  It seems that whenever you include a `lang` parameter in a `codeblock`, you get:

    Liquid error: No such file or directory - python2.7 -c “import sys; print sys.executable”

There are two issues:

 * Syntax highlighting requires Python which is not automatically installed.
 * There is a problem with pythonexec.rb which does not seem to support Windows Python installations very well.

Luckily we can fix them both.  Install [Python 2.7.3](http://www.python.org/getit/releases/2.7.3/).  Actually I used [ninite.com](http://ninite.com/) to install it.

Then modify the pythonrubyexec.rb which is in the depths of the yari subdirectory.  Try here: `.\yari\ruby-1.9.3-p194-i386-mingw32\lib\ruby\gems\1.9.3\gems\rubypython-0.5.3\lib\rubypython\pythonexec.rb`

At the top of pythonexec.rb, modify the file as follows:

{% codeblock pythonexec.rb lang:diff %}
class RubyPython::PythonExec
  def initialize(python_executable)
    @python = python_executable || "python"

+    if (@python.include? 'python2.7')
+      @python = "python"
+    end

    @python = %x(#{@python} -c "import sys; print sys.executable").chomp

    @version = run_command "import sys; print '%d.%d' % sys.version_info[:2]"

-    @dirname = File.dirname(@python)
+    @dirname = "C:/Python27"

{% endcodeblock %}
