---
layout: page
title: Hello
tagline: Supporting tagline
---
{% include JB/setup %}

This is Stoney's tec-blog, built with [Jekyll Bootstrap 3](http://github.com/dbtek/jekyll-bootstrap-3) and [Bootswatch](http://bootswatch.com). I use this blog as a notes collection of what I learnt in my work.

## Recent Posts

Here are recent "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
