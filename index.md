---
layout: home
title: Poiz's Blog
tagline: Hello, world!
---
{% include JB/setup %}

Hi, this is Hansen.

## Post List

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


