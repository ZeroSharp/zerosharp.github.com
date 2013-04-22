---
layout: post
title: "Image captions for Octopress"
date: 2013-01-23 11:56
comments: true
categories: [octopress]
description: How to add pretty captions to images in Octopress.
---

I wanted to add captions to some of my images and came across [this forum thread](https://github.com/imathis/octopress/issues/124), from which I was able to piece together [Ted Kulp](http://tedkulp.com/)'s slick solution.

The following markdown 

    {% raw %} {% imgcap http://some.url.com/pic.jpg Leonhard Euler %} {% endraw %}

will render like this.

{% imgcap http://upload.wikimedia.org/wikipedia/commons/d/d7/Leonhard_Euler.jpg Leonhard Euler %}

It works with `left` and `right` too.

### The changes ###

First, create the image_caption_tag plugin and put it in the plugins subfolder.

{% codeblock plugins/image_caption_tag.rb lang:ruby %}

# Title: Image tag with caption for Jekyll
# Description: Easily output images with captions

module Jekyll

  class CaptionImageTag < Liquid::Tag
    @img = nil
    @title = nil
    @class = ''
    @width = ''
    @height = ''

    def initialize(tag_name, markup, tokens)
      if markup =~ /(\S.*\s+)?(https?:\/\/|\/)(\S+)(\s+\d+\s+\d+)?(\s+.+)?/i
        @class = $1 || ''
        @img = $2 + $3
        if $5
          @title = $5.strip
        end
        if $4 =~ /\s*(\d+)\s+(\d+)/
          @width = $1
          @height = $2
        end
      end
      super
    end

    def render(context)
      output = super
      if @img
        "<span class='#{('caption-wrapper ' + @class).rstrip}'>" +
          "<img class='caption' src='#{@img}' width='#{@width}' height='#{@height}' alt='#{@title}' title='#{@title}'>" +
          "<span class='caption-text'>#{@title}</span>" +
        "</span>"
      else
        {% raw %}"Error processing input, expected syntax: {% img [class name(s)] /url/to/image [width height] [title text] %}"{% endraw %}
      end
    end
  end
end

Liquid::Template.register_tag('imgcap', Jekyll::CaptionImageTag)

{% endcodeblock %}

Next, modify the _utilities.scss file as follows.

{% codeblock sass/base/_utilities.scss lang:diff %}
   border: $border;
 }
 
+@mixin reset-shadow-box() {
+  @include shadow-box(0px, 0px, 0px);
+}
+
 @mixin selection($bg, $color: inherit, $text-shadow: none){
   * {
     &::-moz-selection { background: $bg; color: $color; text-shadow: $text-shadow; }

{% endcodeblock %}

Finally, apply the following changes to the _blog.scss file.

{% codeblock sass/partials/_blog.scss lang:diff %}

   article {
     font-size: 2.0em; font-style: italic;
     line-height: 1.3em;
   }
-  img, video, .flash-video {
+  img, video, .flash-video, .caption-wrapper {
     @extend .flex-content;
     @extend .basic-alignment;
     @include shadow-box;
+    &.caption {
+      @include reset-shadow-box;
+    }
+  }
+  .caption-wrapper {
+    display: inline-block;
+    margin-bottom: 10px;
+    .caption-text {
+      background: #fff;
+      text-align: center;
+      font-size: .8em;
+      color: #666;
+      display: block;
+    }
   }
   video, .flash-video { margin: 0 auto 1.5em; }
   video { display: block; width: 100%; }
   
{% endcodeblock %}

Regenerate your Octopress site and you can start using `imgcap` instead of `img`.