---
layout: default
title: gbaq88
description: "0x00 - breaking things to understand them"
---

<style>
body {
  display: flex;
  flex-direction: row;
  margin: 0 auto;
  max-width: 1200px;
  background-color: black;
  color: #00ff00;
  font-family: monospace;
  line-height: 1.4em;
}

nav {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  overflow-y: auto;
  background-color: black;
  border-right: 1px solid #00ff00;
  width: 220px;
  padding: 20px;
  order: 0;
}

main {
  margin-left: 240px;
  padding: 20px;
  flex: 1;
  max-width: 900px;
  min-width: 400px;
  box-sizing: border-box;
  order: 1;
}

nav ul {
  list-style: none;
  padding: 0;
}

nav li {
  margin: 10px 0;
}

nav a {
  color: #00ff00;
  text-decoration: none;
}

nav a:hover {
  text-decoration: underline;
}

pre {
  margin: 0 0 20px 0;
}
</style>

<nav>
  <ul>
    <li><a href="/writeups/">Write-ups</a></li>
    <li><a href="/artigos/">Artigos</a></li>
    <li><a href="/estudos/">Estudos</a></li>
  </ul>
</nav>

<main>
<pre style="line-height:1.1em;">
   ____ ____    _    _     _ 
  / ___| __ )  / \  | |   | |
 | |  _|  _ \ / _ \ | |   | |
 | |_| | |_) / ___ \| |___| |___
  \____|____/_/   \_\_____|_____|  

       welcome to the gbaq88 lab
</pre>

<pre>
root@gbaq88:~# whoami
gbaq88

root@gbaq88:~# echo "hacker style blog online"
hacker style blog online
</pre>

<h3>Posts recentes:</h3>
<ul>
{% for post in site.posts %}
  <li><a href="{{ post.url }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})</a></li>
{% endfor %}
</ul>

<p>
<a href="https://github.com/gbaq88/gbaq88.github.io" target="_blank">View on GitHub</a>
</p>
</main>
