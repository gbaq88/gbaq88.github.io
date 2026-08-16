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

Há muitas formas, ferramentas e técnicas para executar essa "manobra", porém nesse artigo vou esta usando o ligolo-mp (https://github.com/ttpreport/ligolo-mp).

<p align="center">
<img width="723" height="534" alt="image" src="https://github.com/user-attachments/assets/c988b167-b300-468b-a481-f4dee713f087" />
</p>

Nesse artigo, vamos assumir que já conseguimos o acesso inicial a rede comprometendo uma máquina, com acesso administrador a ela. E na minha máquina parrot, verifico a instalação do ligolo-mp.

<p align="center">
<img width="680" height="401" alt="image" src="https://github.com/user-attachments/assets/04fd284a-ac2f-4d48-a69d-f6b210d02605" />
</p>

Como a instalação foi correta, agora apenas inicio ele, que vai abrir um painel.

<p align="center">
<img width="1353" height="615" alt="image" src="https://github.com/user-attachments/assets/ee41bf57-3a7f-497e-99bc-f347e663124a" />
</p>

Podemos ver no painel que tem um usuário administrador. E precionando o enter no teclado, ja caimos direto no painel de administrador.

<p align="center">
<img width="1348" height="621" alt="image" src="https://github.com/user-attachments/assets/d8e141ec-a5b4-4db7-ab70-6a61d54cec0b" />
</p>
No painel de administrador temos as seguintes funções:

Ctrl-A para funções de administrador

Ctrl-N para gerar um binário de agente

Ctrl-T para Traceroute

Tab para alternar o foco do painel

Ctrl-Q para sair

Como primeiro passo vou digitar ctrl+N para gerar o agente.

<p align="center">
<img width="1343" height="606" alt="image" src="https://github.com/user-attachments/assets/623c2369-92e9-4f4a-82a4-c7637bfec8f4" />
</p>

Save to: Especifica o caminho local onde o binário do agente gerado será salvo.

Servers: define o endereço de retorno de chamada e a porta à qual o agente se conectará novamente. (Digite o endereço do servidor onde o ligolo-mp está sendo executado. Nesse caso, inserimos o IP do Kali, já que o Ligolo está sendo executado no Kali.)

Proxy: permite rotear o tráfego do agente através de um servidor proxy especificado (se necessário).

Ignorar proxy env: ignora quaisquer configurações de proxy configuradas pelo sistema para conexão direta.

SO: Seleciona o sistema operacional de destino para o qual o binário do agente foi criado.

Arch: Determina a arquitetura da CPU (por exemplo, amd64) do agente gerado.

Ofuscar: Opção de aplicar ofuscação básica ao agente para evasão.

Depois de preencher, podemos clicar em enter. Ele vai gerar o binário e salvar no caminho especificado.

<p align="center">
<img width="908" height="288" alt="image" src="https://github.com/user-attachments/assets/f042e151-544a-4b22-bf29-735aa04db1b3" />
</p>

Agora inicio um servidor python para enviar o arquivo para maquina que temos acesso.
<p align="center">
<img width="912" height="318" alt="image" src="https://github.com/user-attachments/assets/c8cc26cc-eb30-4b81-ae48-3c25066c4ecb" />
</p>
Como ja tenho acesso a primeira maquina dentro da rede, uso o Invoke-WebRequest para baixar o binário para ela.

<p align="center">
<img width="896" height="245" alt="image" src="https://github.com/user-attachments/assets/9c83b7a8-fb0b-4ca7-aa70-1af37aed8a80" />
</p>

Agora com o binário na maquina comprometida, posso apenas executar ele e receber a conexão no painel do ligolo.

<p align="center">
<img width="1352" height="623" alt="image" src="https://github.com/user-attachments/assets/323a1eca-de98-412d-b932-338cb5a49557" />
</p>

Como podemos ver no painel do ligolo, recebo a conexão. E no painel chamado de interfaces,  a maquina tem duas interfaces de rede. Uma delas não tenho acesso. Agora vou configurar o pivot pra ela.

Primeiro vou criar uma rota para a rede que não tenho acesso.
<p align="center">
<img width="1350" height="590" alt="image" src="https://github.com/user-attachments/assets/85b90842-dbfc-4794-b2f6-79c7d7cf4ed0" />
</p>

E a rede que não tenho acesso é 172.16.0.0/16

<p align="center">
<img width="440" height="330" alt="image" src="https://github.com/user-attachments/assets/9cc3d91d-4589-4823-8aa0-5df13bf3eafa" />
</p>

Agora preciso iniciar ela, vou dar um enter e iniciar o relay.

<p align="center">
<img width="1347" height="583" alt="image" src="https://github.com/user-attachments/assets/b7069be2-abde-434c-bba2-87bbc962c651" />
</p>

Agora preciso apenas confirmar que tenho acesso a rede e a próxima máquina. Para isso vou ultizaar o utilitário ping.

<p align="center">
<img width="911" height="240" alt="image" src="https://github.com/user-attachments/assets/0faf4ce5-c1c1-45ca-8797-2df0a25e1d35" />
</p>



