Resumo:

A máquina apresentava inicialmente apenas duas portas abertas, 22 (SSH) e 80 (HTTP). A enumeração do serviço web revelou múltiplos subdomínios expostos — www, admin, store — ampliando significativamente a superfície de ataque. Durante a análise do subdomínio store, identifiquei um diretório de busca vulnerável a SQL Injection, que foi explorado utilizando o sqlmap, permitindo a extração de credenciais do banco de dados.

Com acesso inicial obtido, avancei na enumeração do sistema, identificando um servidor Minecraft rodando com plugins configurados de forma insegura. A análise do plugin de autenticação revelou hashes MD5 e, posteriormente, credenciais em texto claro armazenadas em logs — um erro crítico de segurança. Utilizando essas credenciais, consegui acesso via SSH como o usuário cybercrafted. A etapa final envolveu a enumeração de permissões sudo, onde foi identificado que o usuário podia executar screen -r como root. Ao anexar-se à sessão existente do screen (iniciada pelo root), foi possível obter shell privilegiada e concluir a escalada de privilégios.

A exploração destacou falhas clássicas: má configuração de subdomínios, vulnerabilidade de SQL Injection, armazenamento inseguro de credenciais e permissões sudo excessivas — um encadeamento realista de vetores comuns em ambientes mal configurados.
