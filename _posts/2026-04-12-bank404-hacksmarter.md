---
layout: post
title: "bank404 - HackSmarter (WriteUp)"
date: 2026-04-12
categories: writeups
---

# Resumo:
O desafio começou com a enumeração inicial utilizando o RustScan, que revelou diversas portas abertas, indicando tratar-se de um Domain Controller. Durante a análise, uma aplicação web expunha nomes de funcionários, permitindo a criação de uma wordlist personalizada. Além disso, foi encontrado um executável contendo uma senha em Base64, que após decodificação foi utilizada em um password spraying. Sem sucesso imediato via Kerberos, a exploração evoluiu com o uso do BloodHound, identificando permissões abusáveis como GenericWrite e ForceChangePassword, possibilitando movimentações laterais até obter acesso remoto via RDP.

Já dentro da máquina, foi identificada uma porta interna que levou ao uso do Ligolo-ng para pivoting e acesso a redes internas. Nesse processo, foi possível recuperar um backup contendo credenciais privilegiadas, que permitiram avançar na exploração do ambiente. A enumeração do ADCS revelou uma vulnerabilidade do tipo ESC4, possibilitando o abuso de templates de certificados. Com isso, foi possível emitir um certificado em nome do administrador do domínio e autenticar-se como tal, resultando no comprometimento total do ambiente e obtenção de privilégios de Domain Admin.

# Escopo:
O 404 Bank, um elemento básico da comunidade financeira local, está a realizar a sua avaliação anual de segurança. Para defender seu lema de ser "Comprovado, local, forte", o banco encomendou o Equipe Vermelha para realizar um teste de penetração interna.

# Reconnaissance:
Como se tratava de uma operação black box. Só havia um ip do alvo, então useu o rustscan para fazer um escaner de portas, usando as flags -Pn(não completa o three way handshake) e o -sV (versão do serviço que esta rodando)

<p align="center">
<img width="697" height="436" alt="image" src="https://github.com/user-attachments/assets/445c5cb8-6137-436e-8932-3d881888673f" />
</p>

80/tcp open  http syn-ack

88/tcp open kerberos-sec syn-ack

DNS_Domain_Name: 404finance.local

DNS_Computer_Name: DC-404.404finance.local

Sendo assim, já salvo o dominio no meu arquivo hosts, para que o ip do servidor seja resolvido na minha máquina localmente, não precisando buscar por um servidor dns fora.

<p align="center">
<img width="544" height="282" alt="image" src="https://github.com/user-attachments/assets/b489f024-0771-4149-82e1-d6d1cc8e2033" />
</p>

Como durante o scan de portas, a porta 80 que roda um serviço http estava aberta. Fui direto ao navegador enumerar que tipo de serviço web rodava.

<p align="center">
<img width="701" height="508" alt="image" src="https://github.com/user-attachments/assets/d3e42b7c-d11c-4a30-9a11-650a370bc7d4" />
</p>

Durante a enumeração havia os algumas informações sobre alguns funcionários da empresa. Ótimo, poderia usar isso para criar uma wordlist com alguns formatos usados por empresas para usernames. Claro usei um IA para isso.

<p align="center">
<img width="696" height="537" alt="image" src="https://github.com/user-attachments/assets/397b3fe8-f7ff-4fa4-b5e8-64d2f712b133" />
</p>

Diante de uma lista de usuário, fiz uma enumeração de usuários via LDAP usando autenticação Kerberos sem senha. Isso me traria como respostas, usuários válidos e usuários não válidos na rede. Apenas observando a resposta do comando.

<p align="center">
<img width="699" height="348" alt="image" src="https://github.com/user-attachments/assets/98ed18b9-d4fa-4e6d-868d-9a6a9a272da2" />
</p>

A resposta do comando trouxe dois usuários, sendo eles:

404finance.local\robert.graef: KDC_ERR_PREAUTH_FAILED

404finance.local\karl.hackermann: KDC_ERR_PREAUTH_FAILED

Esse erro mostra que o usuário existe, porém ele não se autentica sem senha. Ele precisa de senha. Já os outros usuários não existem na rede.

Antes de força um brute force de senha, voltei para página web para continuar a enumeração. Então notei que havia um executável que podia baixar para minha máquina. Era como um software da empresa. Então baixei ele para minha máquina local.

<p align="center">
<img width="697" height="480" alt="image" src="https://github.com/user-attachments/assets/657759d3-9a32-492b-953f-4f378439edae" />
</p>

O comando string é uma ferramenta de análise estática básica para extrair informações ocultas em binários. E assim encontrei uma coisa bem interessante no executável.

<p align="center">
<img width="695" height="399" alt="image" src="https://github.com/user-attachments/assets/b18cc71a-8902-49c2-bc3d-41f0efadf388" />
</p>

Se tratava de um base64. Para decodar esse base64 usei o próprio base64 -d do linux para isso.

<p align="center">
<img width="575" height="123" alt="image" src="https://github.com/user-attachments/assets/52dbb9a7-d120-4887-936f-519a11d61985" />
</p>

A saída trouxe um md5. Para crackear o md5 usei o site crackstation(https://crackstation.net/).

<p align="center">
<img width="697" height="368" alt="image" src="https://github.com/user-attachments/assets/ba255a83-76c7-41d4-8d78-69d909033e43" />
</p>

# Delivery:
Password spraying é uma técnica de ataque onde você testa uma mesma senha comum em vários usuários, em vez de tentar várias senhas em um único usuário. Então como tenho uma senha em texto plano, vou usar elas num passord spraying nos usuários que encontrados no reconhecimento. Para essa técnica, vou usar o netexec (https://github.com/Pennyw0rth/NetExec).

<p align="center">
<img width="695" height="375" alt="image" src="https://github.com/user-attachments/assets/18270025-26e0-4ee6-a440-e2607c267fc2" />
</p>

# Exploration:
Já com uma credencial válida, já posso fazer uma enumeração melhor. Como usar o smb para listar os usuários da rede. Para isso usei o netexec e a flag smb.

<p align="center">
<img width="698" height="255" alt="image" src="https://github.com/user-attachments/assets/8298142d-3cef-466d-8a70-71dbc2451310" />
</p>

E logo depois de listar os usuários, copio para um arquivo na minha máquina.

<p align="center">
<img width="440" height="299" alt="image" src="https://github.com/user-attachments/assets/140ae738-72f7-42fe-a791-19c7f7b6083c" />
</p>

Ataques comuns como Kerberoasting e AS-REP Roasting não obtiveram sucesso nesse cenário, exigindo a adoção de técnicas mais avançadas de enumeração e exploração dentro do ambiente Active Directory.

<p align="center">
<img width="694" height="184" alt="image" src="https://github.com/user-attachments/assets/1b8aac72-0b58-4b4c-995f-50cf83b2e58d" />
</p>

Assim como listar os compartilhamentos do AD que não mostrou nenhum arquivo suspeito.

<p align="center">
<img width="698" height="298" alt="image" src="https://github.com/user-attachments/assets/61b17d35-b678-49f8-bd58-0f32511866b3" />
</p>

# lateral moviment:

Para coletar dados do AD e ultilizar o bloodhound, vou ultilizar o netexec.

<p align="center">
<img width="696" height="154" alt="image" src="https://github.com/user-attachments/assets/39df6466-9111-4526-8ef9-77865482ed1f" />
</p>

O BloodHound é uma ferramenta que mapeia relações e permissões no Active Directory para identificar caminhos de escalonamento de privilégios. Em campanhas de red team, deve ser usado com cautela porque sua coleta de dados (LDAP, SMB, sessões, ACLs) gera muito tráfego e eventos, podendo ser facilmente detectado por sistemas de monitoramento e defesa.

<p align="center">
<img width="698" height="299" alt="image" src="https://github.com/user-attachments/assets/cc187466-b9c0-4706-a5fa-84f979088cb6" />
</p>

O usuário karl.hackermann possui permissão GenericWrite sobre o objeto tom, o que significa que ele detém a capacidade de modificar atributos desse usuário no Active Directory. A permissão GenericWrite é um direito amplo de escrita que permite alterar diversos campos do objeto, como description, scriptPath, servicePrincipalName e, em alguns casos, até atributos relacionados à autenticação.

Para explorar essa permissão, foi utilizado o script targeted Kerberoasting (targetedKerberoast.py), que permite abusar diretamente do direito GenericWrite sobre o usuário alvo. A técnica consiste em modificar o atributo servicePrincipalName (SPN) do usuário tom, adicionando um SPN controlado pelo atacante, tornando a conta elegível para requisições de tickets de serviço (TGS).

Após a inclusão do SPN, o script solicita um TGS para o usuário modificado, permitindo a extração do ticket criptografado associado à conta. Esse ticket pode então ser submetido a técnicas de cracking offline para recuperação da senha em texto claro.

<p align="center">
<img width="701" height="257" alt="image" src="https://github.com/user-attachments/assets/c49e09e9-c42a-4da0-8367-3398c8ec9a96" />
</p>

Agora posso ultilizar o john para crackear offline.

"O John the Ripper é uma ferramenta de quebra de senhas (password cracking) amplamente utilizada em segurança ofensiva. Ele funciona realizando ataques offline contra hashes, tentando descobrir a senha original através de técnicas como wordlists, regras de mutação e brute force."

<p align="center">
<img width="696" height="217" alt="image" src="https://github.com/user-attachments/assets/f3b59bc1-c051-42bb-8b96-d93aea20de1c" />
</p>

E assim consigo a senha do usuário Tom, porém ainda tenho privilégios limitados. Então resolvo voltar para o bloodhound e enumerar o atributos do usuários.

<p align="center">
<img width="696" height="314" alt="image" src="https://github.com/user-attachments/assets/569eca70-9860-43c8-b92b-a7d3f9a15b78" />
</p>

ForceChangePassword é uma permissão no Active Directory que permite a um usuário resetar a senha de outro usuário sem saber a senha atual.

A ferramenta Impacket (https://github.com/fortra/impacket) existe um módulo para explorar essa permissão errada.

<p align="center">
<img width="700" height="114" alt="image" src="https://github.com/user-attachments/assets/a74d0778-2826-4155-adc9-5716cdb82f0d" />
</p>

Confirmo que o comando deu certo, porém o usuário ainda tem poucos privilégios. E ainda sem acesso remoto.

<p align="center">
<img width="700" height="171" alt="image" src="https://github.com/user-attachments/assets/838bf1c5-ffe9-46c6-954a-fffaee8838c8" />
</p>
Não desistindo, fui ao bloodhound enumerar o usuário Robert.

<p align="center">
<img width="699" height="321" alt="image" src="https://github.com/user-attachments/assets/95bf0832-718a-48e0-9d94-cadc3eea4b79" />
</p>

Posso analisar que foi uma descoberta boa. Apesar de não esta no grupo do usuários remotos, posso adcionar qualquer um. Para verificar se é isso mesmo vou ultilizar o ultiliário net.

"O utilitário net no Linux faz parte do pacote Samba e serve para interagir com serviços de rede Windows/SMB, especialmente em ambientes de Active Directory. Com ele, você consegue consultar usuários e grupos (net rpc group members, net rpc user), alterar senhas (net rpc password), enumerar recursos compartilhados (net rpc share), e até executar ações administrativas remotas dependendo das permissões. Na prática, ele funciona como uma “ponte” entre Linux e redes Windows, permitindo administrar contas e recursos SMB diretamente pelo terminal, sendo muito usado em auditorias, administração de domínios e cenários de pós-exploração."

<p align="center">
<img width="697" height="163" alt="image" src="https://github.com/user-attachments/assets/4b1400a6-d94b-41f8-a68d-704c12c98d3b" />
</p>

No print acima, primeiro verifiquei quais usuários estava no grupo dos usuários remotos. A saída do comando veio vazia, significando que não havia usuário com acesso RDP. No comando seguinte adcionei o próprio usuário robert ao grupo. Depois verifiquei novamente os usuários do grupo e lá estava o usuário Robert.

<p align="center">
<img width="699" height="420" alt="image" src="https://github.com/user-attachments/assets/9b223d58-aa90-4264-a35c-e92232c69ce5" />
</p>

Acessando remotamente o alvo, não foi encontrado nada de interessante com o usuário, mesmo enumerando os privilégios.

Como tenho como mudar a senha de alguns usuários vou comecar a enumeração, primeiro jan.tresor

<p align="center">
<img width="698" height="127" alt="image" src="https://github.com/user-attachments/assets/ca993e12-b515-4d35-ae3e-2234b3ce80ab" />
</p>

Uso o ultilitário net para alterar a senha do usuário e logo após usando o netexec verifico se o comando foi concretizado.

<p align="center">
<img width="698" height="135" alt="image" src="https://github.com/user-attachments/assets/b990a092-f0b8-4c90-907d-9dfd0b437e82" />
</p>

Logo após alterar a senha, adciono o usuário ao grupo Remote Desktop Users. Como mostra na saída, tenho dois usuários nesse grupo. Então hora de usar o rdp para acesso remoto.

<p align="center">
<img width="694" height="366" alt="image" src="https://github.com/user-attachments/assets/4894228e-86a6-48cc-83fc-e5b1173b8a4f" />
</p>

Ja no acesso remoto do usuário, já encontro que a lixeira esta cheia. E ao analisar a lixeira, encontro um e-mail contendo novas credenciais para um acesso.

<p align="center">
<img width="645" height="438" alt="image" src="https://github.com/user-attachments/assets/40da92d3-763d-4182-9806-2ba3791035ab" />
</p>

Então válido as credenciais na rede e logo depois usando o evil-winrm faço acesso remoto a máquina. E já concluo a primeira etapa pegando a flag de usuário.

<p align="center">
<img width="693" height="287" alt="image" src="https://github.com/user-attachments/assets/a9f25a5d-5034-41c0-9e73-8295a7b67916" />
</p>

Enumerando o usuário não consegui nada interessante. Porém no bloodhound já encontro uma falha de permissão de ACL. ForceChangePassword é uma permissão no Active Directory que permite a um usuário resetar a senha de outro usuário sem saber a senha atual. Ou seja o usuário Daniel pode mudar a senha do usuário Webadmin sem precisar saber a senha dele.




