---
layout: default
title: Write-ups
permalink: /writeups/
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
<h2>Write-ups</h2>
<ul>
{% assign writeups = site.posts | where:"categories","writeups" %}
{% for post in writeups %}
  <li><a href="{{ post.url }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})</a></li>
{% endfor %}
</ul>
</main>
