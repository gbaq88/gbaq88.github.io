---
layout: post
title: "BuildingMagic - HackSmarter (WriteUp)"
date: 2026-02-24
categories: writeups
---

# Resumo:
Neste CTF, a exploração começou com a quebra de hashes MD5 fornecidas, permitindo acesso inicial via password spraying. 
Em seguida, foi realizado Kerberoasting para obter novas credenciais, e com auxílio do BloodHound foi identificado um caminho de escalonamento explorando a permissão ForceChangePassword. 
Após assumir outro usuário, foi conduzido um ataque MITM com Responder para capturar hashes NetNTLMv2, que também foram quebradas. 
Com acesso mais privilegiado, utilizou-se o privilégio SeBackupPrivilege para extrair os arquivos SAM e SYSTEM, e através do Impacket foi possível obter hashes válidas e realizar Pass-the-Hash, culminando no comprometimento administrativo da máquina alvo.

