---
layout: post
title: "Octopress on Windows and GitHub"
date: 2012-04-06 17:11
comments: true
categories: [octopress, github]
---

This is quick guide to setting up [Octopress](http://octopress.org/) on a Windows 7 machine to publish to GitHub pages.

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

    yari 1.9.2

Then follow the rest of the instructions from [the Octopress setup instructions](http://octopress.org/docs/setup/).

    git clone git://github.com/imathis/octopress.git octopress
    cd octopress
    ruby --version  # Should report Ruby 1.9.2 thanks to yari

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

Luckily we can fix them both.  Install [Python 2.7.2](http://www.python.org/getit/releases/2.7.2/).  Actually I used [ninite.com](http://ninite.com/) to install it.

Then modify the pythonrubyexec.rb which is in the depths of the yari subdirectory.  Try here: `.\yari\ruby-1.9.2-p290-i386-mingw32\lib\ruby\gems\1.9.1\gems\rubypython-0.5.1\lib\rubypython\pythonexec.rb`

At the top of pythonexec.rb, modify the file as follows:

{% codeblock pythonexec.rb %}
class RubyPython::PythonExec
  # Based on the name of or path to the \Python executable provided, will
  # determine:
  #
  # * The full path to the \Python executable.
  # * The version of \Python being run.
  # * The system prefix.
  # * The main loadable \Python library for this version.
  def initialize(python_executable)
    @python = python_executable || "python"

+    # <zerosharp+> Start of modification
+    if (@python.include? 'python2.7')
+      @python = "python"
+    end
+    # <zerosharp+> End of modification

    @python = %x(#{@python} -c "import sys; print sys.executable").chomp
{% endcodeblock %}

#### References ####

* [Lee's Bigdinosaur blog](http://blog.bigdinosaur.org/changing-octopresss-header/) helped with fonts and colours.
* [This blog post](http://translate.googleusercontent.com/translate_c?hl=en&ie=UTF8&prev=_t&rurl=translate.google.com&sl=zh-CN&tl=en&twu=1&u=http://blog.yesmryang.net/windows-octopress-python/&usg=ALkJrhhFZMKpT5j6_L8NXOL1S9H9CGjHNw) (translated from the Chinese via Google Translate) helped me solve the codeblock problem.
* [This issue](https://github.com/imathis/octopress/issues/262) and [this issue](https://github.com/github/gollum/issues/225) also helped.
* Markdown [syntax](http://daringfireball.net/projects/markdown/syntax) and [cheat sheet](http://warpedvisions.org/projects/markdown-cheat-sheet/)
