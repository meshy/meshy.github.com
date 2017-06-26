---
layout: main
bodyid: posts
title: Blog posts
---

<div>{% for post in site.posts %}
  <div><a href='{{ post.url }}'>
    <header class='post-title'>
      <span class='date'>
          {{ post.date | date:"%A %-d %B %Y" }}
      </span>
      <h1 class='title'>{{ post.title }}</h1>
      <p class='subtitle'>{{ post.description }}</p>
    </header>
  </a></div>

  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}</div>
