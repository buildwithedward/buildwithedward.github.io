---
layout: archive
title: "Deep Learning - 120 Days"
permalink: /categories/deeplearning/
author_profile: true
---

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'deeplearning'" %}

{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
