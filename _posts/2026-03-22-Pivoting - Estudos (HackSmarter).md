---
layout: post
title: "Pivoting - Estudos"
date: 2026-02-24
categories: estudos
---

# Resumo:
Pivoting é a técnica de utilizar um host previamente comprometido como ponto intermediário para alcançar, enumerar e explorar ativos adicionais em segmentos de rede que não são diretamente acessíveis a partir da máquina do atacante.

Do ponto de vista operacional, envolve o estabelecimento de mecanismos de encaminhamento de tráfego (traffic forwarding) — como port forwarding, SOCKS proxies ou túneis — que permitem redirecionar requisições através do host comprometido, efetivamente expandindo o alcance do atacante para redes internas, sub-redes isoladas ou zonas protegidas por controles de acesso (firewalls, ACLs, segmentação).

Essa técnica é fundamental em cenários de pós-exploração, pois possibilita:

Descoberta de novos hosts (network discovery) em redes internas
Enumeração de serviços não expostos externamente
Movimentação lateral (lateral movement) entre sistemas
Escalada de privilégios em ambientes distribuídos
Encadeamento de compromissos (attack chaining) até ativos críticos

Em ambientes corporativos, o pivoting geralmente explora a confiança implícita entre hosts internos e a falta de visibilidade ou inspeção de tráfego lateral, permitindo que o atacante opere “de dentro para fora” com maior furtividade e eficácia.
<img width="707" height="404" alt="image" src="https://github.com/user-attachments/assets/10ba660b-3661-4118-b905-ccaa0914b071" />

