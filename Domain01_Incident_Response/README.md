AWS Abuse Notice
-----------------

- Caso encontre algum tipo de abuso/comportamento malicioso vindo de algum recurso AWS, um ec2 por exemplo, é possível abrir um "Report Amazon Abuse" preenchendo os dados para investigação.

Guarduty
---------

- Monitora logs de serviço (monitoração inteligente) para buscar riscos de segurança (cloudtrail/vpc flow logs, dns logs / não monitora outros logs)
Exemplo: ec2 sendo usados para minerar bitcoins / ec2 sofrendo tentativas de bruteforce / portscan, etc
- Trusted IP List: possível customizar algum ip/rede para o guarduty não gerar os findings ; possível extrair localmente ou de uma lista vinda de um bucket. Suporta alguns formatos (plaintext / csv, por exemplo)
- Threat lists: oposto do Trusted IP List, IPs maliciosos conhecidos, são gerados findings para todas as requisições vinda deles.
- Os Findings types suportados estão na documentação da AWS para análise: https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html;
- Os findings podem ser arquivos ou filtrados (supress).
- Central Architecture: É possível enviar os Findings de todos os member account para uma master aws account.

Incident Response
------------------

- É uma forma estuturada para identificar e gerenciar as ações necessários pós-ocorrido de um incidente de segurança dentro da sua organização, limitando os danos causados, reduzindo tempo de recuperação e os custos causados pelo incidente;

- AWS Access Key e Secret Key expostas:
1. valida quais acessos a Key exposta possui;
2. invalida a credencial;
3. invalida qualquer acesso temporário (STS), incluindo uma policy Explicity Denied;
4. cria uma nova credencial e migra;
5. valida seu conta (cloudtrail, recursos, etc), para identificar se algo pode ter acontecido.

- EC2 comprometido:
1. isola a instância, só permite acesso se for para análise forense ou algo do tipo;
2. tira um snapshot do EBS;
3. tira um dump de memória para análise forese poterior (caso o ofensor esteja rodando em memória);
4. análise forense / causa raíz;
5. termina a instância.

Incident Response Plan
-----------------------

- Quando um incidente ocorre, é importante você já ter um plano preparado para seguir. Algumas etapas:
1. Preparação (logs habilitados, uso do Organizations para segmentação de contas, etc)
2. Detecção (monitora a infra para detectar os comportamento maliciosos, como tentativas de logins com falha em X tempo)
3. Conter (por exemplo, use uma automação que inclui um SG restritivo numa EC2)
4. Investiga (investiga causa raíz)
5. Recupera (restaura ambiente)
6. Lições aprendidas (o que fazer para evitar?!)

Pen Test in AWS
----------------

- Antigamente era necessário abrir um chamado na AWS para fazer PenTest, mas recentemente isso mudou e agora é possível fazer o teste diretamente contra sua infra, em até 8 serviços (pré-aprovados)
- ec2, rds, cloudfront, autora, apigw, lambda, lambda edge, beanstalk
- Alguns testes são proibidos: Dos, DDoS, Port Flooding, DNS zone walking, etc
- instancias pequenas não são suportadas;