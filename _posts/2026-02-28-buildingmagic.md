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
E são elas:
<p align="center">
<img width="564" height="182" alt="image" src="https://github.com/user-attachments/assets/639dbef7-3159-4307-bac7-3d006f1926e8" />
</p>
Indo ao navegador verificar o que roda no serviço http, só há um serviço padrão do windows.
<p align="center">
<img width="698" height="347" alt="image" src="https://github.com/user-attachments/assets/3fcb13a0-0308-409f-a374-ab1f98c1d2e3" />
</p>
# Delivery

Password spraying é uma técnica de ataque onde você testa uma mesma senha comum em vários usuários, em vez de tentar várias senhas em um único usuário. Então como há 2 senhas em texto plano, vou usar elas num passord spraying nos usuários que vazados no reconhecimento.
Para essa técnica, vou usar o netexec (https://github.com/Pennyw0rth/NetExec) com a flag continue-on-sucess, ou seja, continuar mesmo depois de encontrar um usuário válido.

<p align="center">
<img width="698" height="389" alt="image" src="https://github.com/user-attachments/assets/7442a960-685c-4bd6-8acb-dcd69832387a" />
</p>
Com a técnica de password spraying, encontro um usuário válido para uma das senhas encontradas. Agora, com o prórpio netexec válido a credencial na rede, usando o protocolo ldap.

<p align="center">
<img width="697" height="121" alt="image" src="https://github.com/user-attachments/assets/236f8a03-af84-4a3b-a0e6-cc64bff8d9ec" />
</p>
# Exploitation

Com um credencial válida, posso tentar algum tipo de ataque ao protocolo kerberos. UM deles é o kerberoasting.

"O kerberoasting é uma técnica onde você solicita tickets de serviço do Kerberos e tenta quebrar offline a senha de contas de serviço a partir desses tickets. Ela explora uma limitação do protocolo Kerberos, onde qualquer usuário autenticado pode solicitar tickets de serviço (TGS), que são criptografados usando a senha da conta de serviço associada. Esses tickets podem ser capturados e levados para ataque offline, permitindo que um atacante tente quebrar a senha sem gerar alertas no sistema. Assim, a técnica depende principalmente de senhas fracas ou mal configuradas em contas de serviço, tornando possível obter credenciais válidas e avançar no ambiente."

<p align="center">
<img width="698" height="246" alt="image" src="https://github.com/user-attachments/assets/70c74c18-d94e-4110-bde9-c117d295ca75" />
</p>

Como visto no print, há um usuário vulnerável ao kerberoasting. Com isso, a hash foi do usuário foi capturada, podendo assim, usar a ferramenta john the ripper (https://github.com/openwall/john) para quebrar off-line, mas claro se a senha for fraca e estiver na wordlist usada.

<p align="center">
<img width="697" height="193" alt="image" src="https://github.com/user-attachments/assets/68190a3c-c19b-4e37-86df-d8ee317096a9" />
</p>
Já que a consegui quebrar a hash off-line, vou a validar ela com o netexec.

# Lateral Moviment
<p align="center">
<img width="694" height="105" alt="image" src="https://github.com/user-attachments/assets/d21ac185-b411-4082-9799-624b698c84a9" />
</p>

Para continuar a enumeração, agora vou ultilizar o bloodhound. Para fazer essa coleta, continuo usando o netexec.

"O BloodHound é uma ferramenta que mapeia relações e permissões no Active Directory para identificar caminhos de escalonamento de privilégios. Em campanhas de red team, deve ser usado com cautela porque sua coleta de dados (LDAP, SMB, sessões, ACLs) gera muito tráfego e eventos, podendo ser facilmente detectado por sistemas de monitoramento e defesa."

<p align="center">
<img width="698" height="96" alt="image" src="https://github.com/user-attachments/assets/f0e4d26b-155f-4cd8-a6f2-4e6d9ae6eae3" />
</p>

Agora ja com a coletas de dados feita pelo netexec, inicio o bloodhound para uma enumeração melhor.

<p align="center">
<img width="699" height="286" alt="image" src="https://github.com/user-attachments/assets/dce05856-c3ab-4fde-b648-eb703032ac55" />
</p>

No bloodhound já encontro uma falha de permissão de ACL. ForceChangePassword é uma permissão no Active Directory que permite a um usuário resetar a senha de outro usuário sem saber a senha atual. Ou seja, o usuário que foi comprometido consegue resetar a senha de outro usuário, mesmo não sabendo a senha autal.

A ferramenta Impacket (https://github.com/fortra/impacket) existe um módulo para explorar essa permissão errada. 

<p align="center">
<img width="697" height="116" alt="image" src="https://github.com/user-attachments/assets/94f18882-c8f4-4e03-abde-b879b507d083" />
</p>

Na saída do comando, ele mostra que o reste de senha foi um sucesso, agora preciso apenas validar para confirmar mais um movimento lateral.

<p align="center">
<img width="698" height="235" alt="image" src="https://github.com/user-attachments/assets/e5a2e900-74bb-46ec-9480-ace9c6d51da8" />
</p>

Mas um movimento lateral feito, analisando o bloodhound, não encontro algo que podemos explorar, seja para um privilege escalation, seja para outro movimento lateral. Além disso, não tenho acesso remoto para máquina. Então partir para o compartilhamento de rede.

No compartilhamento de rede, tenho ler e escrever no diretório File-Share. Isso é bom pois posso usar o isso para capturar hash de usuário através de um ataque MITM. O ataque MITM com o Responder consiste em se posicionar na rede para responder a requisições de resolução de nomes (como LLMNR/NBT-NS) quando uma máquina não consegue encontrar um host. O atacante responde fingindo ser o destino solicitado e força a vítima a tentar autenticar nele via SMB/HTTP, capturando assim hashes NetNTLMv2. Esses hashes podem então ser quebrados offline ou usados em outros ataques, permitindo obter credenciais válidas sem precisar conhecer a senha diretamente.

Para ter a certeza que consigo subir um arquivo para esse compartilhamento, crio um arquivo de teste.

<p align="center">
<img width="619" height="171" alt="image" src="https://github.com/user-attachments/assets/4d084330-6708-48c5-8c47-0be6f2d3d723" />
</p>

Agora vou usando o smbclient para acessar o comprtilhamento.

<p align="center">
<img width="699" height="222" alt="image" src="https://github.com/user-attachments/assets/e80e2e10-8030-4d1b-9f30-07f5bea68894" />
</p>

Agora vou subir, usando o compando put, o arquivo para o compartilhamento.

<p align="center">
<img width="683" height="279" alt="image" src="https://github.com/user-attachments/assets/f1d676fc-4e6e-4b5f-9611-589dc85889ba" />
</p>

Como planejado. Consegui subir um arquivo para o compartilhamento.

Ntlm_left (https://github.com/Greenwolf/ntlm_theft) é um ferramenta usada para gerar arquivos maliciosos que forçam máquinas Windows a tentarem autenticar em um servidor controlado pelo atacante. No caso o servidor é minha máquina.

<p align="center">
<img width="697" height="334" alt="image" src="https://github.com/user-attachments/assets/dca883fd-97a7-45d7-8daf-165e626bd987" />
</p>

E ao tentar se autenticar no servidor, estarei usando o responder para capturar a hash, com um ataque MITM.

<p align="center">
<img width="640" height="317" alt="image" src="https://github.com/user-attachments/assets/f6e20889-f61f-4c3f-9d43-481175865097" />
</p>

Agora vou subir o arquivo malicioso criado com o ntlm_left e esperar um usuário desavisado abrir e se autenticar para abrir o arquivo.

<p align="center">
<img width="468" height="196" alt="image" src="https://github.com/user-attachments/assets/926460c5-3e30-43ae-a7b6-994a7870cb54" />
</p>

E um pouco tempo depois, conseguimos capturar uma hash.

<p align="center">
<img width="699" height="103" alt="image" src="https://github.com/user-attachments/assets/c83d93b3-9f56-449a-91d2-ed3dce4afb92" />
</p>

Agora, salvo ela em um arquivo na minha máquina e vou usar o john para quebrar ela off-line.

<p align="center">
<img width="698" height="183" alt="image" src="https://github.com/user-attachments/assets/5597835d-4e5a-4e4c-b917-05f487e05bff" />
</p>

Ao validar o movimento lateral na rede, confirmei que tenho acesso remoto à máquina. E usando o evil-winrm já faço esse acesso remoto e pego a flag de usuário.

<p align="center">
<img width="699" height="274" alt="image" src="https://github.com/user-attachments/assets/bf5e9285-17c9-4c5a-acbe-cfaa68ca9052" />
</p>

E ao enumerar os privilégios do usuário comprometido, encontro o SeBackpPrivilege.

"SeBackupPrivilege é um privilégio do Windows que permite a um usuário ler qualquer arquivo do sistema ignorando permissões de acesso, com o objetivo original de realizar backups."
<p align="center">
<img width="622" height="226" alt="image" src="https://github.com/user-attachments/assets/8ef1b74d-2a11-428a-b32c-ceae54ea05f4" />
</p>

Ou seja, com esse usuário consigo acessar arquivos protegidos (como SAM e NTDS) mesmo sem ser administrador, sendo frequentemente abusado para extrair credenciais e escalar privilégios.

# Privilege Escalation

Para explorar essa permissão, já crio um arquivo temporário. E baixo para ele os arquivos sam e system.

Os arquivos SAM e SYSTEM são componentes críticos do Windows usados para armazenamento e proteção de credenciais.

SAM (Security Account Manager): armazena os hashes de senha dos usuários locais da máquina.

SYSTEM: contém informações do sistema, incluindo a chave (bootkey) usada para descriptografar os dados protegidos do SAM.
<p align="center">
<img width="667" height="520" alt="image" src="https://github.com/user-attachments/assets/f5fb884b-937d-4df5-a38e-6b497fdd670c" />
</p>

E usando o próprio evil-winrm, faço o download para a minha máquina.

<p align="center>
<img width="470" height="216" alt="image" src="https://github.com/user-attachments/assets/2596d6e7-4b2f-42ac-ad4d-f6cce85b1f43" />
</p>

"O Impacket, por meio do secretsdump, utiliza os arquivos SAM e SYSTEM para extrair credenciais do sistema Windows: ele primeiro obtém a bootkey presente no SYSTEM e a usa para descriptografar os dados protegidos do SAM, permitindo assim recuperar os hashes de senha (NTLM) dos usuários locais, que podem ser usados posteriormente para cracking ou ataques como pass-the-hash."

<p align="center">
<img width="694" height="283" alt="image" src="https://github.com/user-attachments/assets/3de78935-21e0-440f-8f73-e1b89297d8a4" />
</p>

Agora, com a hash do Administrator, posso fazer um pass-the-hash usando o evil-winrm.

<p align="center">
<img width="699" height="167" alt="image" src="https://github.com/user-attachments/assets/78078822-dcc8-4a0e-83df-7668dc098e1d" />
</p>

Mesmo após obter a hash do Administrator, o acesso via pass-the-hash falhou devido a restrições do serviço WinRM, evidenciando que nem toda credencial privilegiada garante acesso direto por todos os vetores. 
<p align="center">
<img width="700" height="120" alt="image" src="https://github.com/user-attachments/assets/94f5e05b-b917-4b09-a5a6-9f0d62545d63" />
</p>
Como alternativa, foi realizado password spraying, resultando na descoberta de um usuário com permissões adequadas para login remoto, permitindo a continuidade da exploração.
<p align="center">
<img width="699" height="306" alt="image" src="https://github.com/user-attachments/assets/392ef7b3-4179-4978-8d34-268784862a35" />
</p>
Além de acesso ao remoto, esse novo usuário ainda faz parte do grupo de administradores do dominio.


