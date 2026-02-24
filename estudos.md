---
layout: default
title: Estudos
permalink: /estudos/
---

<main>
<h2>Estudos</h2>
<ul>
{% assign estudos = site.posts | where:"categories","estudos" %}
{% for post in estudos %}
  <li><a href="{{ post.url }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})</a></li>
{% endfor %}
</ul>
</main>
