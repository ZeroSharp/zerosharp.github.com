---
layout: post
title: "The best PC laptop is a Mac"
date: 2012-10-09 17:09
comments: true
categories: [macbook, hardware, c#]
description: My first impressions of using a Macbook Pro with retina screen for Windows development.
---
### A month ago ###

I've never liked laptops. The screen seems so small. A powerful laptop is too heavy; a light one doesn't cut the mustard. 
I can't even find the on/off switch.

Then there are those fancy new **Macbook Pro**s with retina displays. They seem light and powerful and the screen looks great. I don't know much about Macs, but I'd like to learn.

{% blockquote %}
But I need Microsoft Windows and Visual Studio. I know theoretically I can run a virtual machine or setup a dual boot, but I'll bet it's all a bit of a hassle and rather clunky...
{% endblockquote %}

### Three hours after my Macbook Pro arrived ###

Windows 8 is installed and looks absolutely beautiful! The font is small, but the screen is so sharp that it is quite readable. Visual Studio looks great with the font size in the editor Window set to 14 or 15.

The installation was much simpler than most other Windows installations I've done. I followed [this tutorial on iClarified](http://www.iclarified.com/entry/index.php?enid=20961) to set up a dual boot. You use a tool called the Bootcamp Advisor which is already installed on your Mac to create an bootable USB for installing Windows. Since all Macs are the same, the Bootcamp Advisor sets up all the drivers correctly for you.

### Three weeks later ###

{% blockquote %}
This is by far the most pleasant laptop I've ever used for development.
{% endblockquote %}

The screen is wonderful - the machine seems very fast and it is no weight to carry around.

### A tiny little gripe ###

There is a little annoyance with the Bootcamp Control Panel which won't start properly, but Thomas Jespersen found a [workaround](http://apple.stackexchange.com/a/59132/30710), [which I've put in a batch file](https://gist.github.com/3859956). You need this to make the F1-12 keys work as they would in Windows.

### Some help with the Mac keyboard and trackpad ###

I'm very picky about keyboards and I need to be able to find all my Visual Studio shortcuts in the same places. All in all I find the keyboard fine to work with.

The `delete` key works like a PC `backspace` so you have to do `fn+delete` to delete forwards. Here's a list of other shortcuts.

PC (running Windows) | &nbsp;Macbook (running Windows) 
--- | --- 
_Windows_ key             | &nbsp;_command_ 
_Del_                     | &nbsp;_fn+delete_  
_Ins_                     | &nbsp;_fn+return_ 
_Home_                    | &nbsp;_fn+left_ 
_End_                     | &nbsp;_fn+right_ 
_PgUp_                    | &nbsp;_fn+up_ 
_PgDn_                    | &nbsp;_fn+down_ 
_Ctrl+Home_               | &nbsp;_fn+control+left_ 
_Ctrl+End_                | &nbsp;_fn+control+right_ 

&nbsp;
There are some keys which you will still not find, for instance, `NumLock`, `PrtScn`, `Break`.

If I need these, I use the On-Screen keyboard (type 'osk' into the search bar). If you cannot see the `NumLock`, the options allow you to show the numeric keypad. Alternatively, you could always plug in a USB keyboard.

The trackpad works great including the two-finger click for right-click and two-finger swipe for scroll.