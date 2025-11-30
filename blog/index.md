---
layout: default
title: Blog
sidebar: blog
---

# Blog

Hier findest du alle aktuellen Artikel von Bozart Botanics.

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})  
  <small>{{ post.date | date: "%d.%m.%Y" }}</small>
{% endfor %}
