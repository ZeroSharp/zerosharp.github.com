---
layout: post
title: "A basic Chrome extension - analyze your chess.com games on lichess.org"
date: 2016-03-04 17:41
comments: true
categories: [chrome, extension, javascript, chess]
description: How I wrote my first Chrome extension to analyze finished chess.com games on lichess.org.
---
I've been trying to get better at chess by playing on [chess.com](http://www.chess.com). I often analyse my games with the help of a computer to see where I made mistakes. My favourite way of doing this is to make use of [Lichess.org](http://lichess.org)'s import functionality. The latest version of chess.com has greatly improved its own post game analysis, but I still much prefer Lichess's. So I would often download the finished game in pgn and upload it to Lichess.

Somewhere along the way I came across a Chrome extension written by Allan Rempel which would do this in one step. Unfortunately it stopped working when chess.com upgraded its site and Allan does not seem to be maintaining it any longer. I decided to jump in and try to fix it and maybe add a couple of features myself.

## The extension ##

When a chess.com game is finished, you can click on a little knight icon and jump straight to the analysis of the game in Lichess.

{% img /images/blog/chrome-extension-chessCom.png %}

It takes a few seconds to process but then you can see all your mistakes and blunders.

{% img /images/blog/chrome-extension-lichessOrg.png %}

You can play through the game and experiment with different variations. Chess.com's new v3 has greatly improved its own post-game analysis but it still lacks many of the features of the Lichess one.

Try it out! The extension is available [here](https://chrome.google.com/webstore/detail/chesscom-v3-analysis/bhjlkimpkkgkmfjlcfngmakenalgleap).

## How to modify an existing extension ##

First thing was to find the code of the existing broken extension. A bit of googling lead me to:

    $ cd ~/Library/Application\ Support/Google/Chrome/Default/Extensions/
    $ ls
    
    total 0
    drwx------+ 30 ra  staff  1020 29 Feb 20:39 .
    drwx------+ 66 ra  staff  2244 29 Feb 20:56 ..
    drwx------+  2 ra  staff    68 29 Feb 20:39 Temp
    drwx------+  3 ra  staff   102 14 Sep  2012 allibancfngcpjmjnnboebggipbaalhn
    drwx------+  3 ra  staff   102 28 Mar  2013 anfemphbgknehemmbbajjcmakfijiohp
    drwx------+  3 ra  staff   102 25 Oct 08:15 apdfllckaahabafndbhieahigkjlhalf
    drwx------+  3 ra  staff   102  5 Mar  2015 bepbmhgboaologfdajaanbcjmnhjmhfn
    drwx------+  3 ra  staff   102  5 Jun  2014 bfbameneiokkgbdmiekhjnmfkcnldhhm
    drwx------+  3 ra  staff   102 22 Feb 20:09 bhjlkimpkkgkmfjlcfngmakenalgleap
    drwx------+  3 ra  staff   102 24 Sep 20:33 blpcfgokakmgnkcojhhkbfbldkacnbeo
    drwx------+  3 ra  staff   102 29 Jul  2013 bmihblnpomgpjkfddepdpdafhhepdbek
    drwx------+  3 ra  staff   102 13 Sep  2012 boeajhmfdjldchidhphikilcgdacljfm
    ... etc.
    
Each extension is in a not very helpfully named folder. If you open `chrome://extensions/`, you can find the id of any installed extension and work out which directory corresponds to which extension.
 
So then I dug around in the folder and found `ext/background.js` where the bulk of the code lies. There is also a `manifest.json` file which is specifies various parameters such as what permissions are required and which websites will be affected by the extension. I copied all the important files into a development folder.

## Loading the extension from the development folder. ##

* Navigate to chrome://extensions
* Expand the developer dropdown menu and click "Load Unpacked Extension"
* Navigate to development folder

Assuming there are no errors, the extension should load into your browser. Don't forget to disable any conflicting versions you've installed from the Chrome Web Store.

{% img /images/blog/chrome-extension-load.png %}

I used the Chrome's built-in Developer Tools and peppered the code with `console.log()` to explore how it worked. There are plenty of good [resources](https://developer.chrome.com/extensions/tut_debugging) on the Chrome developer site. I managed to fix the broken parts and added support for analysis from chess.com's game archive. Then I posted the link on [reddit/r/chess](https://redd.it/47cdew).

The source code for the extension is on [GitHub](https://github.com/ZeroSharp/Chess.com_Analysis_Chrome_extension).
