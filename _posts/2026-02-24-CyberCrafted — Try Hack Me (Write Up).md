---
layout: post
title: "CyberCrafted — Try Hack Me (Write Up)"
date: 2026-02-24
categories: writeups
---

# Resumo:

A máquina apresentava inicialmente apenas duas portas abertas, 22 (SSH) e 80 (HTTP). A enumeração do serviço web revelou múltiplos subdomínios expostos — www, admin, store — ampliando significativamente a superfície de ataque. Durante a análise do subdomínio store, identifiquei um diretório de busca vulnerável a SQL Injection, que foi explorado utilizando o sqlmap, permitindo a extração de credenciais do banco de dados.

Com acesso inicial obtido, avancei na enumeração do sistema, identificando um servidor Minecraft rodando com plugins configurados de forma insegura. A análise do plugin de autenticação revelou hashes MD5 e, posteriormente, credenciais em texto claro armazenadas em logs — um erro crítico de segurança. Utilizando essas credenciais, consegui acesso via SSH como o usuário cybercrafted. A etapa final envolveu a enumeração de permissões sudo, onde foi identificado que o usuário podia executar screen -r como root. Ao anexar-se à sessão existente do screen (iniciada pelo root), foi possível obter shell privilegiada e concluir a escalada de privilégios.

A exploração destacou falhas clássicas: má configuração de subdomínios, vulnerabilidade de SQL Injection, armazenamento inseguro de credenciais e permissões sudo excessivas — um encadeamento realista de vetores comuns em ambientes mal configurados.

# Reconnaissance

Para iniciar a fase de reconhecimento ultilizei o rustscan com as flags:

-Pn : não realizar o three-way handshake.

-sC: Procurar por scripts padrão para o serviço que esta rodando

<img width="1048" height="306" alt="image" src="https://github.com/user-attachments/assets/c0626d53-4305-4110-8d7a-5bb48d95ed2e" />

Nesse scan, indentifiquei 3 portas abertas:

22/tcp open ssh syn-ack

80/tcp open http syn-ack

25565/tcp open minecraft syn-ack

Além de :

ttp-title: Did not follow redirect to http://cybercrafted.thm/

Para partir para enumeração dos serviços que rodam em cada porta, principalmente a http (80). Vou adcionar o dominio encontrado, no arquivo host da minha máquina. Para que o ip do alvo seja resolvido nesse diretório e não procurar por um servidor externo.

<img width="753" height="445" alt="image" src="https://github.com/user-attachments/assets/873b7f45-96f4-41b9-9b51-ffc5f114a9d9" />

# Enumeration

Para enumeração, vou iniciar indo ao navegador e verificar o que roda na porta 80 de serviço/aplicação.
