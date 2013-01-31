---
layout: post
title: "A C# Developer's Adventures in iOS - Getting Started"
date: 2013-01-30 11:18
comments: true
categories: [ios, monotouch, xcode, iphone, mobile]
description: How to get started with iPhone development from a C# developer's perspective.
---
## First steps in iPhone development ##

This is the first in a series of posts about developing an iOS application from the perspective of a Windows C# developer.
### Startup Weekend ###

On a recent month-long trip to South America, I took part in the [Buenos Aires Startup Weekend](http://buenosaires.startupweekend.org/) hackathon and my team came second!](/how-i-went-from-c-number-developer-to-iphone-developer-in-a-weekend). The application we built is a treasure hunt game for tourists which helps them discover Buenos Aires. At the beginning of the weekend I knew nothing about mobile development. It was the first time I developed software using a Mac.

First, let's look at the basic requirements for the environment.

{% img right /images/blog/adventures-in-ios-getting-started-001.png 450 %}

### Xamarin MonoTouch ###

[MonoTouch](http://xamarin.com/monotouch) is the easiest approach for C# developers. It allowed me to write a basic native iPhone app in a weekend with no prior mobile development experience. I'd never used MonoDevelop or XCode either.

MonoTouch is a software development kit for MacOS that cleverly enables you to use C# and the common .NET libraries (via Mono) along with bindings for the [Apple Cocoa Touch](https://developer.apple.com/technologies/ios/cocoa-touch.html) framework to create native applications for iPhone, iPad and iPod. A lot of the tools are the same as for Objective-C developers: you use the iPhone Simulator and the XCode Interface Builder.

If you are a C# developer still wondering whether to learn Objective-C, there is [a magnificent StackOverflow answer](http://stackoverflow.com/a/1634592/1077279) comparing MonoTouch and Objective-C.

### Costs ###

There are some costs associated with becoming a MonoTouch iOS developer.

You need a Mac. I'm using a 15" MacBook Pro. I have it configured for dual boot, but I'm using the MacOS side for this project.

You will need to sign up as an iOS developer with Apple. This costs **$99** per year. More information about [Apple developer programs on their site](https://developer.apple.com/programs/).

You will need Xamarin MonoTouch which currently costs **$399** for a one year single seat license for a single developer. More information on pricing at the [Xamarin store](https://store.xamarin.com/).

### Setting up the environment ###

Xamarin have an [excellent installation guide](http://docs.xamarin.com/ios/Guides/Getting_Started/Installation) which guides you through installing both XCode and MonoTouch.

### Hello World ###

The next step is to create your first iOS application running in the iPhone simulator. Go through the samples in the [MonoTouch getting started guide](http://docs.xamarin.com/ios/guides/getting_started).

### Next up ###

Over the next few weeks I'm aiming to finish the app I built for Startup Weekend and I'll be sharing what I learn along the way.

