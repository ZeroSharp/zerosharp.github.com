---
layout: post
title: "Fix Visual Studio update links when running as Administrator"
date: 2013-09-02 09:58
comments: true
categories: [visualstudio]
description: A fix for an annoying Visual Studio issue related to Administrator mode.
---
This post is about fixing an annoyance whereby Visual Studio refuses to update extensions when running as an administrator.

I always had a problem when an update to an extension or tool tried to open the browser. For instance, I would see this notification in the system tray.

{% img /images/blog/visual-studio-updates-001.png %}

And then when I went to _Tools/Extensions and Updates..._ in Visual Studio, I'd get to something like this:

{% img /images/blog/visual-studio-updates-002.png %}

But the _Update_ button was not responding to any clicks. The only workaround I found was to restart Visual Studio under my normal user account and then the button would work.

## The fix at last ##

It turns out the problem rather specific. It only occurs when you run VS2012 as an administrator and you have Google Chrome set as your default browser. The problem is that older versions of Google Chrome failed to register themselves as the default browser for administrator users. You can confirm this by doing the following from an admin command prompt:

    start http://blog.zerosharp.com

You'll get a message saying _Class not registered_.

Unfortunately, updating to newer installations of Chrome does not fix the problem. You need to uninstall and reinstall. (This sounds like a lot of hassle, but if you are using Chrome sync it is very easy and only takes about a minute).

* Uninstall Chrome by going to _Control Panel/Uninstall a program_. * Open IE and download Chrome. 
* During the installation process it will ask you to enable sync. 
* A few seconds later everything should be back as at it was.

But better, because the _Update_ button in Visual Studio now works.



