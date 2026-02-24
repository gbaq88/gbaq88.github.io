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
         888                      .d8888b.  .d8888b.  
        888                     d88P  Y88bd88P  Y88b 
        888                     Y88b. d88PY88b. d88P 
 .d88b. 88888b.  8888b.  .d88888 "Y88888"  "Y88888"  
d88P"88b888 "88b    "88bd88" 888.d8P""Y8b..d8P""Y8b. 
888  888888  888.d888888888  888888    888888    888 
Y88b 888888 d88P888  888Y88b 888Y88b  d88PY88b  d88P 
 "Y8888888888P" "Y888888 "Y88888 "Y8888P"  "Y8888P"  
     888                     888                     
Y8b d88P                     888                     
 "Y88P"                      888                     


       welcome to the gbaq88 notes
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
