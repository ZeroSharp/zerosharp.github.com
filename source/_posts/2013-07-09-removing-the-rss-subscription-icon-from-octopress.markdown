---
layout: post
title: "Removing the RSS subscription icon from Octopress"
date: 2013-07-09 09:13
comments: true
categories: [octopress]
---
A [fellow Octopress blogger](https://twitter.com/tlaynes) recently asked how I removed the RSS subscription icon from the Octopress navigation bar.

First, create a new site variable `show_feeds` by adding a line to the __config.yml_ file which is in the root folder of the Octopress source.

{% codeblock _config.yml lang:diff %}
  # RSS / Email (optional) subscription links (change if using something like Feedburner)
+ show_feeds: false
  subscribe_rss: http://feeds.feedburner.com/zerosharp
  subscribe_email:
  # RSS feeds can list your email address if you like
  email:
{% endcodeblock %}

Then modify the *source/_includes/navigation.html* as follows

{% codeblock navigation.html lang:diff %}
+ {% raw %}{% if site.show_feeds %}{% endraw %}
  <ul class="subscription" data-subscription="rss{% if site.subscribe_email %} email{% endif %}">
    <li><a href="{{ site.subscribe_rss }}" rel="subscribe-rss" title="subscribe via RSS">RSS</a></li>
    {% raw %}{% if site.subscribe_email %}{% endraw %} 
      <li><a href="{{ site.subscribe_email }}" rel="subscribe-email" title="subscribe via email">Email</a></li>
    {% raw %}{% endif %}{% endraw %} 
  </ul>
+ {% raw %}{% endif %}{% endraw %}
  {% raw %}{% if site.simple_search %}{% endraw %} 
  <form action="{{ site.simple_search }}" method="get">
    <fieldset role="search">
      <input type="hidden" name="q" value="site:{{ site.url | shorthand_url }}" />
      <input class="search" type="text" name="q" results="0" placeholder="Search"/>
    </fieldset>
  </form>
  {% raw %}{% endif %}{% endraw %} 
  {% raw %}{% include custom/navigation.html %}{% endraw %}
{% endcodeblock %}

Then you can toggle the visibility of the feed icon by changing the `show_feeds` setting in the `_config.yml` file.