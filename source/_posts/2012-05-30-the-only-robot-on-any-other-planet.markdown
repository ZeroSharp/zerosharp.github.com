---
layout: post
title: "The only robot on any other planet"
date: 2012-05-30 08:44
comments: true
categories: [robots, space, C]
description: A mention of the hardware and software on board the Mars Opportunity Rover.
---

This post is part of an ongoing series about robots, past and present.  See [Musical Interlude](/musical-interlude/).

### The Mars _Opportunity_ Rover ###

_Opportunity_ landed on Mars in January 2004.  It is currently the only active robot on any planet other than Earth (until the scheduled arrival of [_Curiosity_](http://www.youtube.com/watch?v=P4boyXQuUIw) in August 2012).  The mission was originally supposed to last 90 days.

The rover contains pieces of the fallen World Trade Center which were turned into shields to protect cabling.

This photo of the Endeavour Crater on Mars was taken by the _Opportunity_ rover on March 9, 2012.

{% img /images/blog/the-only-other-robot-001.jpg %} 

#### Hardware - if it ain't broke, don't fix it ####

NASA prefers hardware that has been used successfully on previous missions, even if it means compromising on performance.

The computer on board is a [RAD6000](http://en.wikipedia.org/wiki/IBM_RAD6000). It is single board computer designed for the US Air Force by BAE Systems with a 32-bit processor based on the IBM RISC single chip CPU.  It has 128 MB of RAM.It is radiation hardened so that it can withstand the harsh environment of space. The RAD6000 is reported to cost between USD 200,000 and USD 300,000.

{% blockquote Peter Gluck, http://news.oreilly.com/2008/07/the-software-behind-the-mars-p.html The Software Behind the Mars Phoenix %}
It's a venerable board. We've used it ever since the Mars Pathfinder Mission what — about 12 years ago, 14 years ago?
{% endblockquote %}


#### Lifecycle ####

[{% img right /images/blog/the-only-other-robot-002.jpg Eagle Crater landing site %}](http://en.wikipedia.org/wiki/File:Opportunity_-_Cratera_Eagle.jpg)

The software on a Mars lander is expected take 3-4 years development.
It's designed according to a [Waterfall](http://en.wikipedia.org/wiki/Waterfall_model) model split roughly as follows: 

  - 9 months requirements definition
  - 12-18 months in design phase
  - 12-18 months test phase
  - Launch!


#### Software ####

{% blockquote left %}
One of the things about the software, one of the unique aspects about working software on a spacecraft is it's the only part of the system that you can change after launch.
{% endblockquote %}

The operating system is [VxWorks](http://en.wikipedia.org/wiki/VxWorks).  The software on board the spacecraft is entirely in *C*.  The software on the rover is about 650,000 lines of *C*.

    #include void main() { printf("Hello Mars\n"); }

No GitHub.  Most space technology software is considered arms technology, so it cannot be made open source.

#### References ####
- [Universe Today blog](http://www.universetoday.com/95358/opportunity-gets-a-view-from-the-edge/)
- [The Software Behind the Mars Phoenix](http://news.oreilly.com/2008/07/the-software-behind-the-mars-p.html) Audio interview with Peter Gluck
- Image credits NASA/JPL-Caltech