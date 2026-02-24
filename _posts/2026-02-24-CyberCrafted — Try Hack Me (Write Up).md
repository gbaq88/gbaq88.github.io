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

<img width="1020" height="437" alt="image" src="https://github.com/user-attachments/assets/26fc1394-4465-4d0b-a5cb-4be6b72b9aba" />

No servidor roda um site, sem links e bem estático.

Como não tem nada para enumerar, ou clicar. Vou ao código fonte do site.

<img width="1017" height="608" alt="image" src="https://github.com/user-attachments/assets/a4bf7b2f-0e66-40b0-8188-594c22ebcc0c" />

Há um comentário informando que há outros subdominios. Para fazer uma enumeração de subdominio e vhosts, vou ultilizar o FFuF.

<img width="1019" height="204" alt="image" src="https://github.com/user-attachments/assets/19e53977-1170-4b45-ba9a-370b441422a5" />

O FFuF trouxe dois subdominios. O admin e o store.

Para que eles fiquem acessiveis, adciono eles ao meu arquivo hosts. Lado a lado do que já estava lá.

<img width="1019" height="308" alt="image" src="https://github.com/user-attachments/assets/6968bba3-8568-448e-94aa-2c537df5aeb1" />

Indo ao admin.cybercrafted, encontro uma página de login.

<img width="1013" height="536" alt="image" src="https://github.com/user-attachments/assets/29d18501-b537-4d1a-bbbd-1aaff7a87670" />

Já no store.cybercrafted não consigo acesso.

Então resolvo fazer um brute force de diretório usando o feroxbuster. Esse é o link da ferramenta no github https://github.com/epi052/feroxbuster.

Nele usei as flags:

-u: informar a url
-w: informar a wordlist
Fiz um brute force em todos os dominios encontrados.

Primeiro no dominio cybercrafted.thm

<img width="1019" height="228" alt="image" src="https://github.com/user-attachments/assets/1c7e2f14-0b29-424e-b8d9-a3d1af7ebc18" />

Nele existia um diretório chamado secret, mas não fez muito sentido ele. Apesar de ter usado o wget e baixado todas as imagens e analisados elas.

<img width="1017" height="642" alt="image" src="https://github.com/user-attachments/assets/f26cd7e3-6fc5-4bb9-b689-b5fb5e1124dd" />

No subdominio admin, no brute force de diretório, nada foi encontrado também.

<img width="1020" height="237" alt="image" src="https://github.com/user-attachments/assets/e53f87f6-f821-48d2-9864-f85a9c32e20f" />

Porém no brute force do subdominio store, um diretório chamou atenção.

<img width="1019" height="176" alt="image" src="https://github.com/user-attachments/assets/6f027e8c-a686-4f4b-9a4e-e541efb80564" />

/search.php fazia algum tipo de busca no banco de dados.

<img width="1022" height="407" alt="image" src="https://github.com/user-attachments/assets/e4521415-95d9-4658-a190-331a0f675f4e" />

# Vulnerability Analysis

Então quando um aplicação faz uma busca em um banco de dados, o primeiro teste a se fazer é tentar fazer com que a Query seja quebrada.

Quando uma aplicação realiza consultas ao banco de dados utilizando dados fornecidos pelo usuário, o primeiro teste em uma análise de segurança é verificar se essa entrada está sendo tratada corretamente. Caso a aplicação construa a query SQL por meio de concatenação direta, é possível injetar caracteres especiais como aspas simples para alterar a estrutura da consulta. Essa técnica, conhecida como SQL Injection, permite manipular a lógica da query, podendo resultar em vazamento de dados ou autenticação indevida. O teste inicial geralmente envolve inserir uma aspa simples para observar se ocorre erro de sintaxe, indicando que a aplicação está vulnerável.

Para isso, no campo de busca coloquei uma aspas simples ('). Mas a aplicação não retornou nada. Não desistindo, enviei (' OR 1=1 --). E essa me trouxe algo muito bom.

<img width="1020" height="656" alt="image" src="https://github.com/user-attachments/assets/7854cd7c-9e4d-4ec0-be4e-b2b63b80e8bc" />

Retornou todo o banco de dados. Mostrando que o campo de busca é vulnerável a SQLi.

Nesse momento, abri o burp e interceptei esa requisição, para enfim explorar esse SQLi com o sqlmap.

# Exploration

O primeiro comando que ultilizei foi o (sqlmap -r req ). Sendo o req, um arquivo de texto contendo a requisição capturada pelo burp.

<img width="1016" height="383" alt="image" src="https://github.com/user-attachments/assets/807dfbdf-aa7f-40bf-aee0-55a59addabca" />

O campo search foi devidamente explorado pelo sqlmap, e na saída da ferramenta mostra o tipo de sqli foi explorado.

O próximo comando ultilizado foi o ($sqlmap -r req --dbs). Esse comando vai retorna todos os banco de dados no servidor.

<img width="1014" height="474" alt="image" src="https://github.com/user-attachments/assets/f1f288f4-7529-4a99-97ed-78e119a018e8" />

Um deles chamou muito atenção. Então para listar as tabelas do webapp ultilizei o comando ($sqlmap -r req -D webapp --tables).

<img width="1019" height="504" alt="image" src="https://github.com/user-attachments/assets/7a28c89f-0b78-45b7-94aa-ef80b15262ca" />

Agora para ter uma visão melhor das colunas da tabela admin, vou usar o comando (sqlmap -r req -D webapp -T admin --columns).

<img width="1017" height="613" alt="image" src="https://github.com/user-attachments/assets/74d90b3d-5fc1-463c-b0c5-aea8fa18bdcd" />

Como podemos ver a tabela admin tem 3 colunas, agora posso fazer apenas um dump dessa tabela admin. Para isso vou usar o comando (sqlmap -r req -D webapp -T admin --dump).

<img width="1006" height="379" alt="image" src="https://github.com/user-attachments/assets/bb55483c-e6dd-4e90-9eb1-d3127fd8cb3e" />

Ótimo, foi feito um dump da hash do usuário e uma flag da sala. Como a hash tem 40 caractere, posso imaginar que seja um SHA1.

Então salvo ela num arquivo na minha máquina local e uso o hashcat para fazer a quebra dessa hash off-line.

<img width="1019" height="269" alt="image" src="https://github.com/user-attachments/assets/41bb123d-fcff-4453-9fe6-1c0fe6544975" />

Para esse ataque usei a wordlist rockyou. E depois de um certo tempo, consegui quebrar a hash e descobrir a senha.

<img width="1020" height="562" alt="image" src="https://github.com/user-attachments/assets/5bd613e3-5058-4f54-8ded-0c649bf4d9b0" />

# Initial Access

Voltando, agora com as credenciais, ao subdominio admin.cybercrafted.thm, faço o login e consigo acesso ao dashboard dele.

<img width="1031" height="506" alt="image" src="https://github.com/user-attachments/assets/6f255211-4ccb-4de7-ab7e-527db33aea63" />

O dash do usuário, caimos em um campo, informando para rodar comandos.

Diante disso, iniciei a enumeração usando os comando no diretório home. Logo depois listei o diretório do usuário que estava logado.

<img width="1003" height="741" alt="image" src="https://github.com/user-attachments/assets/c19dddce-66fb-49d2-8a1a-f48b2b7bdf0f" />

O diretório .ssh é uma pasta oculta presente no diretório pessoal de um usuário em sistemas Linux/Unix que armazena arquivos relacionados à autenticação via SSH (Secure Shell). Nele ficam guardadas chaves privadas (id_rsa, id_ed25519), chaves públicas (id_rsa.pub), o arquivo authorized_keys (que define quais chaves podem acessar a conta), além de configurações como config e o histórico de hosts conectados em known_hosts. Esse diretório é essencial para autenticação segura baseada em chave pública, permitindo login remoto sem senha quando configurado corretamente, e por isso deve possuir permissões restritas (geralmente 700 para a pasta e 600 para as chaves privadas), já que o vazamento da chave privada pode comprometer completamente o acesso ao servidor.

Acessando o diretório ssh, posso ver que temos acesso a chave privada do usuário. E se usar o comando cat para ler ela, posso copia-la.

<img width="896" height="740" alt="image" src="https://github.com/user-attachments/assets/0a3de011-4da2-466f-a94a-da69a960f842" />

Se olharmos bem acima da chave temos:

Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC

Significa que a chave esta protegida por senha. Mas para contorna esse problema o john tem um ultilitário chamado ssh2john que transforma ela em um formato que o próprio john possa quebrar ela off-line.

Mas primeiro dei as devidas permissões para o arquivo, ja na minha máquina local e usei o ultilitário e logo depois usei o john para quebrar ela off-line.

<img width="1017" height="367" alt="image" src="https://github.com/user-attachments/assets/0d6bb90c-05fe-4cf6-be45-25e46a424a87" />

E assim consigo a senha, agora basta acessar remotamente o servidor usando o ssh. Mas dessa vez ultilizando a flag -i e informar a chave id_rsa. Logo depois ele vai pedir a senha da chave privada, a qual ja tenho.

<img width="1018" height="255" alt="image" src="https://github.com/user-attachments/assets/c091ee5e-ca0f-452d-819a-a09f407e6a3f" />

# Pos-exploration

Usando o comando id, me mostra que o usuário que estou logado pertence a um grupo interessante.

<img width="1015" height="69" alt="image" src="https://github.com/user-attachments/assets/1e6322ba-469b-4bc5-a670-40ae9e3b70ed" />

E depois de muito enumerar encontro o diretório válido para explorar.

<img width="1016" height="310" alt="image" src="https://github.com/user-attachments/assets/dfea5958-a5df-4c2b-abc2-ad80127353a8" />

O diretório cybercraft, apesar de não ser dono, mas os pertencentes ao grupo minecraft pode acessar.

<img width="1017" height="402" alt="image" src="https://github.com/user-attachments/assets/ca137285-efc0-426c-b8f9-9bbecf0c95c6" />

Mas antes de acessar o diretório, uso o cat para ler o conteúdo dos dois arquivos que estão nesse diretório. Note.txt é uma nota sobre um plugin no servidor. E no outro arquivo encontro uma nova flag.

Então, hora de ir ao diretório cybercrafted.

<img width="1020" height="516" alt="image" src="https://github.com/user-attachments/assets/9ed21841-2543-47fc-9ee1-0057cb327314" />

Como no arquivo note.txt falava sobre plugin, e nesse diretório tem um diretório chamado plugin, nada mais justo que acessar ele.

<img width="1021" height="172" alt="image" src="https://github.com/user-attachments/assets/4ee2de2d-65ef-4919-88ce-77852e527ba9" />

Como o arquivo .jar não é interessante nesse momento, acesso o diretório LoginSystem, para fazer uma enumeração melhor.

<img width="1017" height="254" alt="image" src="https://github.com/user-attachments/assets/0a4c341d-fd6b-4a7c-bcde-73f8fd75c0a3" />

Já dentro do diretório, encontro um arquivo chamado passwords.yml. Esse sim é um arquivo interessante. E ao ler esse arquivo, encontro dois usuários e as hash md5 deles.

<img width="1013" height="688" alt="image" src="https://github.com/user-attachments/assets/013b36d3-70e4-47a5-8af0-5d73626bb235" />

Salvo elas separadamente e uso o john para quebrar elas off-line. Apenas uma delas o john conseguiu quebrar. Porém a hash que quebrou é de um usuário que não existe, e ao tentar usar ela com o usuário cybercrafted ela não funcionou. Então voltei para a enumeração.

# Lateral Moviment

No diretório atual existe um arquivo chamdo log.txt. Nele contém os logs de todos os acessos ao servidor.

<img width="1018" height="392" alt="image" src="https://github.com/user-attachments/assets/4cbec087-e14e-4dfa-a850-ad29bca0f6d8" />

E para minha sorte, os logs de usuário e senha estão lá, inclusive o de senha esta em texto claro. Mostrando assim a senha do usuário que precisava.

Abrir uma nova janela no terminal da minha máquina e acessei o servidor com o ssh, logando com o usuário cybercrafted.

<img width="1016" height="515" alt="image" src="https://github.com/user-attachments/assets/8500bddb-712c-44a6-a687-b78f11ed7d8e" />

# Privilege Escalation

Usando o comando sudo -l, para verificar se o usuário há alguma vulnerabilidade, ou configuração incorreta ou algum privilégio especial.

<img width="1020" height="167" alt="image" src="https://github.com/user-attachments/assets/126a370b-f445-4e6a-84f9-481e746c11fe" />

A saída do comando mostra que o usuário root iniciou o screen e deixou a sessão anexável. Além de deixar que outro usuário( no caso o cybercrafted) anexar como root. E conseguindo acesso administrador a sessão ja existente. Para isso basta executar o screen.

Esse artigo mostra como essa vulnerabilidade é fácil de explorar.

https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/sudo-screen-privilege-escalation

Usando o screen, caio em uma sessão no servidor.

<img width="1022" height="473" alt="image" src="https://github.com/user-attachments/assets/c82ad8f5-e241-40ee-b82f-f47c3b13cf66" />

Agora basta executar Ctrl+a+c.

<img width="983" height="216" alt="image" src="https://github.com/user-attachments/assets/e248b4a2-1576-4b05-94c9-93745acfc0ab" />

Esse laboratório mostra que certas permissões, grupos e acessos, tem que ser bem monitorados, além de senhas fortes precisam ser adotadas pelas organizações e senhas fracas são facilmente quebradas off-line.

# ROMANOS 11:36


