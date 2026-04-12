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

