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
<img width="1402" height="444" alt="image" src="https://github.com/user-attachments/assets/a9adee4e-fe91-41e3-bffe-463763e67f05" />
</p>
O diagrama de rede, basicamente assim. No print acima.

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

Como podemos ver, tenho acesso a maquina comprometida na rede 172.16.0.0/16. Na qual antes não tinhamos acesso. Certo, agora vou acessar via RDP essa máquina.

<p align="center">
<img width="1315" height="679" alt="image" src="https://github.com/user-attachments/assets/6083dd86-e480-4267-91bc-371f2561fe42" />
</p>

Acesso a maquina confirmado. E analisando as interfaces de rede, notamos que a maquina tem uma outra interface de rede, que provavelmente não temos acesso. Mas vou verificar e confirmar isso.

<p align="center">
<img width="847" height="234" alt="image" src="https://github.com/user-attachments/assets/bf0c0f81-9a0b-4788-83e8-113234855d73" />
</p>

Confirmado, não tenho acesso a outra rede da maquina. Então vou configurar o ligolo-mp para fazer um novo pivot.
<p align="center">
<img width="1360" height="645" alt="image" src="https://github.com/user-attachments/assets/82e0f52d-906e-494a-adc0-3df6f6f692ae" />
</p>

Agora vou adionar um redirector.
<p align="center">
<img width="440" height="327" alt="image" src="https://github.com/user-attachments/assets/df355e26-4914-459c-bb7b-5b779ed1e4be" />
</p>

Todo o tráfego recebido na porta 4444 será encaminhado para o IP e porta do servidor ligolo-mp (neste caso, para a máquina parrot) através do túnel TLS existente.
<p align="center">
<img width="1351" height="594" alt="image" src="https://github.com/user-attachments/assets/ef5494ef-16a3-4bb7-9274-8ef506235e10" />
</p>

Agora preciso criar um novo binario e transferir para essa maquina. O binário tem ser para a rede que a segunda maquina tem acesso a primeira
<p align="center">
<img width="837" height="562" alt="image" src="https://github.com/user-attachments/assets/7683b106-3b70-4a36-90ca-6ad8c3c50f78" />
</p>

E como no redirectory esta ouvindo na porta 4444 nosso binário tem que conectar nessa porta.
<p align="center">
<img width="806" height="397" alt="image" src="https://github.com/user-attachments/assets/71f0c0d5-e9d4-4a0e-9e11-fa29edad5982" />
</p>

Como podemos ver criei um novo binário e logo depois inicio ele para enviar para primeira maquina comprometida.
<p align="center">
<img width="906" height="295" alt="image" src="https://github.com/user-attachments/assets/b2c7df69-cf0a-4c9b-8836-fafc25029da2" />
</p>

E usando, outra vez, o invoke-webrequest baixo ele na maquina remota.
<p align="center">
<img width="843" height="241" alt="image" src="https://github.com/user-attachments/assets/853ab45e-73a1-4937-b807-4ed05a90ba39" />
</p>

Agora vou transferir para a segunda máquina.
<p align="center">
<img width="840" height="339" alt="image" src="https://github.com/user-attachments/assets/728f1845-d7f1-42ff-822a-57cca2ac9d10" />
</p>
Criando um compartilhamento para transferir o binário para segunda máquina
Agora apenas copio o binário para um diretório
<p align="center">
<img width="840" height="355" alt="image" src="https://github.com/user-attachments/assets/af907190-c984-4e5e-b67e-70241d79e6a7" />
</p>

Com o binário já na segunda máquina, ja posso executar ele e receber a conexão no ligolo.
<p align="center">
<img width="1349" height="648" alt="image" src="https://github.com/user-attachments/assets/d4901ea7-5d21-41ea-90b3-bf44f6ab7545" />
</p>
Como podemos notar no painel, ja tenho uma segunda conexão, agora preciso apenas configurr a rota.
<p align="center">
<img width="1347" height="619" alt="image" src="https://github.com/user-attachments/assets/57bb0072-1688-482e-934c-9c3371a3ec08" />
</p>
Logo depois de criar uma nova rota, inicio ela.
Agora volto a ultilizar o utilitário ping, para ver se consigo acessar maquina que foi comprometida nessa rede.
<p align="center">
<img width="894" height="231" alt="image" src="https://github.com/user-attachments/assets/995c32e9-5ec7-44ad-a5ac-61095bce107b" />
</p>
O teste mostra que tenho acesso a uma maquina comprometida em uma outra rede, agora vou acessar ela usando o rdp.
<p align="center">
<img width="1365" height="683" alt="image" src="https://github.com/user-attachments/assets/28e40bfe-96e7-46eb-b310-02d1ea485a4e" />
</p>

Podemos ver, inclusive no painel do remmina onde mostra a maquina que estamos acessando que são 3 ip's em redes diferentes.

ROMANOS 11:36


