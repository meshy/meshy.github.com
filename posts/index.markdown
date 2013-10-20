---
layout: main
bodyid: posts
title: Blog posts
---

{% for post in site.posts %}
  <div><a href='{{ post.url }}'>{{ post.excerpt }}</a></div>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
