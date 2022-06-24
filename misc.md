---
layout: page
title: Miscellaneous
subtitle: "Posts about what I read and experience in day to day life."
---

<div id="misc">
  <ul class="posts">
    {% for post in site.posts %}
      {% if post.category == "misc" %}
      <li><a href="{{ post.url }}">{{ post.title }}</a><span> &raquo; {{ post.date | date: "%B %d, %Y" }}</span></li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
