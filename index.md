---
layout: default
title: gbaq88
description: "Hacking, experiments, and lab notes"
---

<style>
body {
  margin: 0;
  background-color: black;
  color: #00ff00;
  font-family: monospace;
  line-height: 1.4em;
}

nav {
  background-color: black;
  border-bottom: 1px solid #00ff00;
  padding: 10px 20px;
  display: flex;
  gap: 20px;
}

nav a {
  color: #00ff00;
  text-decoration: none;
  font-weight: bold;
}

nav a:hover {
  text-decoration: underline;
}

main {
  padding: 20px;
  max-width: 900px;
  margin: auto;
}

pre {
  margin: 0 0 20px 0;
  line-height: 1.1em;
  background-color: #111;
  padding: 15px;
  border-radius: 5px;
}

h3 {
  margin-top: 40px;
  color: #00cc00;
}
</style>

<nav>
  <a href="/writeups/">Write-ups</a>
  <a href="/artigos/">Artigos</a>
  <a href="/estudos/">Estudos</a>
</nav>

<main>
<pre>
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

root@gbaq88:~# echo "w3lc0m3 t0 my n0t3s"
w3lc0m3 t0 my n0t3s
</pre>

<h3>whoami...</h3>
<ul>
{% for post in site.posts %}
  {% if post.title == "Whoami" %}
    <li><a href="{{ post.url }}">whoami</a></li>
  {% endif %}
{% endfor %}
</ul>

<p>
<a href="https://github.com/gbaq88/gbaq88.github.io" target="_blank">View on GitHub</a>
</p>
</main>
