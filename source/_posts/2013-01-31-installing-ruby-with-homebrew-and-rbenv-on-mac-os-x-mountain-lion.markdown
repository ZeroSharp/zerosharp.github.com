---
layout: post
title: "Installing Ruby with Homebrew and rbenv on Mac OS X Mountain Lion"
date: 2013-01-31 17:38
comments: true
categories: [ruby, mac, rbenv, octopress, homebrew]
---
## Install Ruby with rbenv ##

I decided to setup Octopress on my Mac so that I can publish blog posts from either Windows or MacOS. I'm on OS X 10.8.2.

I tried to follow the [Octopress instructions](http://octopress.org/docs/setup/rbenv/) for installing Ruby but ran into a few problems.

### Install Homebrew ###

[Homebrew](http://mxcl.github.com/homebrew/) is a package manager for OS X. Open Terminal and install Homebrew with:

    $ ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"

Once the installation is successful, you can run the following command to check your environment.

    $ brew doctor

Apparently, you should see:

    Your system is raring to brew

But instead I had 3 warnings.

<pre>
<span STYLE="color:red">Warning:</span> /usr/bin occurs before /usr/local/bin
This means that system-provided programs will be used instead of those
provided by Homebrew. The following tools exist at both paths:

    gcov-4.2

Consider amending your PATH so that /usr/local/bin
occurs before /usr/bin in your PATH.
</pre>

This one can be fixed by modifying your `.profile` file. Create it if it doesn't already exist. Use `nano ~/.profile` if you don't have a preferred editor. 

{% codeblock lang:bash .profile %}
# Fix $PATH for homebrew
homebrew=/usr/local/bin:/usr/local/sbin
export PATH=$homebrew:$PATH
{% endcodeblock %}

Google tells me the other two warnings are related to Mono being installed and can be ignored.

<pre>
<span STYLE="color:red">Warning:</span> /Library/Frameworks/Mono.framework detected
This can be picked up by CMake's build system and likely cause the build to
fail. You may need to move this file out of the way to compile CMake.

<span STYLE="color:red">Warning:</span> You have a non-Homebrew 'pkg-config' in your PATH:
  /usr/bin/pkg-config => /Library/Frameworks/Mono.framework/Versions/2.10.9/bin/pkg-config

This was most likely created by the Mono installer. `./configure` may
have problems finding brew-installed packages using this other pkg-config.
</pre>

Ploughing on...

## Install rbenv ##

Rbenv handles the installation of multiple Ruby environments. 

    $ brew update
    $ brew install rbenv
    $ brew install ruby-build

## Install gcc ##

If I try to install Ruby immediately, I get 

<pre>
<span STYLE="color:red">ERROR:</span> This package must be compiled with GCC, but ruby-build
couldn't find a suitable `gcc` executable on your system.
Please install GCC and try again.
...
</pre>

XCode used to ship with a compatible gcc, but no longer does. We can install it with Homebrew.

    $ brew update
    $ brew tap homebrew/dupes
    $ brew install autoconf automake apple-gcc42

The new gcc will coexist happily alongside the default one.

## Install Ruby 1.9.3 ##

Now we can install Ruby.

    $ rbenv install 1.9.3-p194
    $ rbenv rehash

Next run 

    $ ruby --version

Hmm... I get 

    ruby 1.8.7 (2012-02-08 patchlevel 358) [universal-darwin12.0]

Shouldn't it say ruby 1.9.3? It turns out you need to add the following to the end of your `.profile`.

{% codeblock lang:bash .profile %}
# Initialize rbenv
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi{% endcodeblock %}

Now quit and restart Terminal.

    $ rbenv global 1.9.3-p194
    $ ruby --version
    ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-darwin12.2.1]

Ruby 1.9.3 is installed correctly. If I quit and restart Terminal, `ruby --version` is still 1.9.3.





