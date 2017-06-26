---
layout: main
---

<header class='post-title'>
  <span class='date'>
      {{ page.date | date:"%A %-d %B %Y" }}
  </span>
  <h1 class='title'>{{ page.title }}</h1>
  <p class='subtitle'>{{ page.description }}</p>
</header>

{{ content }}
