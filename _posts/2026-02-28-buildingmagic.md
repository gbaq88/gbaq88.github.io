---
layout: post
title: "BuildingMagic - HackSmarter (WriteUp)"
date: 2026-02-28
categories: writeups
---

# Resumo:
Neste CTF, a exploração começou com a quebra de hashes MD5 fornecidas, permitindo acesso inicial via password spraying. 
Em seguida, foi realizado Kerberoasting para obter novas credenciais, e com auxílio do BloodHound foi identificado um caminho de escalonamento explorando a permissão ForceChangePassword. 
Após assumir outro usuário, foi conduzido um ataque MITM com Responder para capturar hashes NetNTLMv2, que também foram quebradas. 
Com acesso mais privilegiado, utilizou-se o privilégio SeBackupPrivilege para extrair os arquivos SAM e SYSTEM, e através do Impacket foi possível obter hashes válidas e realizar Pass-the-Hash, culminando no comprometimento administrativo da máquina alvo.

# Escopo:
Objetivo: Como testador de penetração no Hack Smarter Red Team, seu objetivo é alcançar um comprometimento total do ambiente do Active Directory.

Acesso inicial: Uma fase de enumeração anterior produziu um banco de dados vazado contendo credenciais de usuário (nomes de usuário e senhas com hash). Essas informações servirão como ponto de partida para obter acesso inicial à rede.

Execução: Sua tarefa é aproveitar as credenciais comprometidas para aumentar privilégios, navegar lateralmente pelo Active Directory e, finalmente, obter um comprometimento completo do domínio.

# Reconnaissance:
Como havia dito logo acima, durante o reconhecimento do alvo, a equipe conseguiu hash md5 vazadas. A partir dessa hash
iriamos iniciar nosso pentest na rede coorporativa do cliente.

Diante disso, partir para um reconhecimento do alvo, verificando as portas abertas usando o rustcan (https://github.com/bee-san/RustScan/). Uso o rustscan, pois ele é um pouco mais rápido que o nmap.

<p align="center">
<img width="569" height="630" alt="image" src="https://github.com/user-attachments/assets/1be98995-e268-4663-92e0-3b156c906602" />
</p>
Muitas portas abertas, sinal que trata-se de um DC. Como no print acima e também no recebimento do escopo, tenho o dominio e preciso colocar ele no arquivo hosts da minha máquina local. Para que esse ip seja resolvido na minha máquina, não precisando de um servidor de DNS para isso.

<p align="center">
<img width="536" height="116" alt="image" src="https://github.com/user-attachments/assets/96537042-720e-4420-ad9d-8663456670a7" />
</p>




# Weaponization:
Diante das credenciais vazadas, vou agora criar um arquivo com as hash md5 e outro arquivo com os usuários. Para que fique melhor para eu trabalhar com eles.
<p align="center">
<img width="701" height="457" alt="image" src="https://github.com/user-attachments/assets/c1bdbde7-32d3-46f0-8d6f-844dc56d6685" />
</p>

Usando o crackstation (https://crackstation.net/) para crackear a hash md5. Isso foi feito uma a uma.

<p align="center">
<img width="698" height="271" alt="image" src="https://github.com/user-attachments/assets/eaa467c1-e6b1-4d62-b9d2-fd99ba85aa4f" />
</p>
Conseguindo com todas aquelas hash, apenas 2 conseguimos crackear.

<p align="center">
<img width="697" height="224" alt="image" src="https://github.com/user-attachments/assets/7e7da0a4-9505-45b4-8db9-36ee1d8c7281" />
</p>



