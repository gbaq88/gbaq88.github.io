---
layout: post
title: "Multi-Hop Pivot"
date: 2026-08-15
categories: artigos
---

# Definição:
O multi-hop pivot é uma técnica crítica em Red Team porque supera limitações de rede, como firewalls e segmentação, que impedem o acesso direto a sistemas críticos. Ela explora a confiança entre redes, permitindo ao atacante "se mover lateralmente" através do ambiente, muitas vezes usando apenas ferramentas legítimas do sistema para camuflar suas atividades.

Imagine que o invasor (atacante) está em uma rede externa. Ele compromete o Host A (primeiro pivô), que tem acesso a uma rede interna. Dentro dessa rede, ele compromete o Host B (segundo pivô), que por sua vez tem acesso a uma rede ainda mais restrita, onde está o Host C (alvo final). O fluxo de ataque segue a cadeia: Atacante -> Host A -> Host B -> Host C.

<p align="center">
<img width="465" height="350" alt="image" src="https://github.com/user-attachments/assets/454f50ba-7b42-4e12-996c-e57f072a936f" />
</p>
