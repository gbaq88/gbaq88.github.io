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

Para esse estudo, vou ultilizar a room da plataforma Hacksmarter chamada PivotSmarter.Você pode encontrar esse lab no link https://www.hacksmarter.org/catalog.

No cenário proposto pelo laboratório, assume-se que um Windows Server já foi previamente comprometido, sendo fornecidas credenciais válidas e acesso remoto ao sistema. A partir desse ponto inicial de acesso, o objetivo é conduzir atividades de pós-exploração com foco em movimentação lateral dentro da rede interna.

Especificamente, o desafio consiste em comprometer um segundo host localizado em um segmento de rede não diretamente acessível a partir da máquina do atacante. Para isso, o host comprometido deve ser utilizado como um ponto de pivot, atuando como intermediário para o encaminhamento de tráfego e viabilizando a comunicação com ativos internos.

Dessa forma, técnicas de pivoting são empregadas para transformar a máquina comprometida em uma ponte de acesso (proxy/túnel), permitindo a enumeração, identificação de serviços e eventual exploração do alvo interno. Esse processo simula um cenário realista de rede segmentada, onde o acesso direto é restrito e depende da exploração de relações de confiança e conectividade entre hosts internos.

Como possuo credencial válida, já ultilizo o evil-winrm para esse acesso remoto.

<img width="696" height="148" alt="image" src="https://github.com/user-attachments/assets/bc439d09-bd00-43bb-9480-d832961e0d5c" />

O Evil-WinRM é uma ferramenta que permite obter acesso remoto interativo a máquinas Windows via WinRM (Windows Remote Management), usando credenciais válidas, funcionando como um shell remoto para execução de comandos e pós-exploração.

Como ja visto no print acima, o acesso foi confirmado. Então, para esse caso de estudo, vou ultilizar, para pos exploração, o Sliver C2. Ele pode ser encontrado nesse link https://github.com/BishopFox/sliver.

<img width="699" height="355" alt="image" src="https://github.com/user-attachments/assets/1b5bf08c-e01a-493e-940c-ebbd1befcead" />

Já com o sliver-server iniciado, uso o comando generate para gerar um implant. Um implant é o binário/script que é entregue ao alvo, é executado na máquina comprometida que estabelece comunicação com o C2 e fornece acesso remoto (beacon ou session).

<img width="672" height="205" alt="image" src="https://github.com/user-attachments/assets/6c2a83b3-b462-431f-900e-5127f93e3969" />

Para enviar o implant ao alvo comprometido, inicio um servidor local com o python. E ainda no Sliver inicio um ouvinte usando o comando mtls.

<img width="432" height="252" alt="image" src="https://github.com/user-attachments/assets/ba18caaa-2cd5-472d-869f-a25ea1299d68" />

Com o acesso remoto, o comando Invoke-WebRequest foi ultilizado para baixar o implant direto do servidor python iniciado na máquina local. E logo após já executo ele usando o Start-Process.

<img width="697" height="64" alt="image" src="https://github.com/user-attachments/assets/ee077216-ec06-4f46-babb-7c83f9ab588c" />

Logo após executar o binário, recebo a conexão no Sliver C2. 

<img width="697" height="258" alt="image" src="https://github.com/user-attachments/assets/912fc6e9-6932-4e0d-bf6f-da3ec50599e1" />

Para interagir com essa conxão é ultilizado o comando "use id". 

<img width="660" height="121" alt="image" src="https://github.com/user-attachments/assets/41dcfd84-22e2-4bbd-aa2d-f794ce2a1475" />

Para a realização de pivoting, é necessário dispor de uma sessão interativa, uma vez que o modo beacon opera de forma assíncrona e não suporta diretamente funcionalidades como tunelamento dinâmico (ex.: SOCKS proxy).

Interactive - Esse comando instrui o implant a estabelecer uma conexão ativa e contínua com o servidor C2, permitindo execução de comandos em tempo real e habilitando recursos avançados de pós-exploração, como pivoting, port forwarding e criação de proxies.

<img width="697" height="147" alt="image" src="https://github.com/user-attachments/assets/70a3900e-555f-4f36-b5ee-8daa1fd2264d" />

Agora posso fazer o pivoting, para isso vou ultilizar o portfoward.

<img width="698" height="103" alt="image" src="https://github.com/user-attachments/assets/894e2d6a-5709-4755-8691-4b48bc04baf2" />

O comando portfwd add -b 127.0.0.1:8080 -r 10.1.91.55:80 no Sliver cria um encaminhamento de porta local (local port forwarding). Nesse caso, a porta 8080 da máquina atacante (localhost) é vinculada à porta 80 do host interno 10.1.91.55, utilizando o sistema comprometido como intermediário. Na prática, qualquer conexão feita para 127.0.0.1:8080 será automaticamente redirecionada através do túnel estabelecido até o servidor interno na porta HTTP.

Dessa forma, ao acessar http://127.0.0.1:8080 no navegador, estarei indiretamente acessando o serviço web hospedado em 10.1.91.55:80, mesmo sem conectividade direta com essa rede. Esse mecanismo permite interagir com serviços internos de forma transparente, como se estivessem expostos localmente, sendo amplamente utilizado em cenários de pivoting para exploração de aplicações web, painéis administrativos e outros serviços restritos à rede interna.

<img width="699" height="265" alt="image" src="https://github.com/user-attachments/assets/4535306c-dab9-4a7f-8021-7eda51f9453f" />

O objetivo foi conseguido. Usando o Windows Server, consigo acesso a rede interna e consequentemente acesso ao Web Server. Agora basta ultilizar o gobuster e fazer um bruteforce de diretório, com o objetivo de encontrar o diretório de login, para acessar o dashboard do usuário.

<img width="698" height="236" alt="image" src="https://github.com/user-attachments/assets/1decb3a9-7bb9-4af1-8e26-6f4a147db5e9" />

Não demora muito e encontro o diretório login.html. Onde o titulo do diretório ja informar para que serve.

<img width="696" height="301" alt="image" src="https://github.com/user-attachments/assets/6d6ce6cd-0db3-4557-b680-1ec42c7fc1e2" />

Agora basta usar as credenciais fornecidas para acessar o dashboard do usuário. E finalizar pegando a flag do laboratório.

<img width="697" height="273" alt="image" src="https://github.com/user-attachments/assets/15500e9f-4f73-446e-91b4-a76d0a22eb57" />

Conclusão:

O pivoting utilizando o Sliver demonstra, na prática, como o comprometimento inicial de um único host pode ser expandido para alcançar outros ativos em redes internas segmentadas. Por meio da criação de túneis, proxies e redirecionamento de portas, é possível contornar restrições de acesso e interagir com serviços que, originalmente, não estariam expostos ao atacante. Esse processo evidencia a importância da movimentação lateral como etapa crítica em cenários de pós-exploração.

Além disso, a utilização de sessões interativas em conjunto com técnicas de pivoting reforça a capacidade de exploração profunda do ambiente, permitindo não apenas a enumeração de novos alvos, mas também sua exploração efetiva. Em um contexto real, isso destaca a necessidade de controles de segurança mais robustos, como segmentação adequada de rede, monitoramento de tráfego lateral e restrições de acesso entre hosts, a fim de mitigar esse tipo de ameaça e reduzir a superfície de ataque interna.



















