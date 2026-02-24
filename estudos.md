---
layout: default
title: Estudos
permalink: /estudos/
---

<style>
body {
  margin: 0;
  background-color: black;
  color: #00ff00;
  font-family: monospace;
  line-height: 1.4em;
}
main {
  padding: 20px;
  max-width: 900px;
  margin: auto;
}
a {
  color: #00ff00;
  text-decoration: none;
}
a:hover {
  text-decoration: underline;
}
</style>

<main>
<h2>Estudos</h2>
<ul>
{% assign estudos = site.posts | where:"categories","estudos" %}
{% for post in estudos %}
  <li><a href="{{ post.url }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})</a></li>
{% endfor %}
</ul>
</main>
