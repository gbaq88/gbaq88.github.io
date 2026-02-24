---
layout: default
title: Artigos
permalink: /artigos/
---

<main>
<h2>Artigos</h2>
<ul>
{% assign artigos = site.posts | where:"categories","artigos" %}
{% for post in artigos %}
  <li><a href="{{ post.url }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})</a></li>
{% endfor %}
</ul>
</main>
