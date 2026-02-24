---
layout: default
title: gbaq88
description: "0x00 - breaking things to understand them"
---

<style>
body {
  display: flex;
  margin: 0;
  background-color: black;
  color: #00ff00;
  font-family: monospace;
  line-height: 1.4em;
}

nav {
  width: 220px;
  padding: 20px;
  border-right: 1px solid #00ff00;
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

main {
  flex: 1;
  padding: 20px;
  position: relative;
}

.cursor {
  display: inline-block;
  width: 10px;
  background-color: #00ff00;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
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

root@gbaq88:~# ls posts
{% for post in site.posts %}
- {{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})
{% endfor %}
</pre>

<span class="cursor">&nbsp;</span>

<p>
<a href="https://github.com/gbaq88/gbaq88.github.io" target="_blank">View on GitHub</a>
</p>
</main>
