AWS Inspector
--------------

- CVE: dicionário público de vulnerabilidades conhecidas (mantida pelo NIST)
- CVSS: analisa o CVE e dá uma nota para classificar a importância da vulnerabilidade
- scan de vulnerabilidade em EC2 (necessita de agente)
- analisa com base em CVE, security (S.O) e network (portas abertas via igw / sem agente) e CIS benchmarks (best practices de segurança/hardening)
- opção de buscar ec2 via tags
- possível instalar o agente durante a configuração do assessment target (mas é necessário o SSM agent instalado, bem como a role com "Run command")
- assessment templates (onde vc configura a análise, bem o tempo, os rules packages (CVE, CIS , etc)), schedule
- assessment run: onde roda manualmente o template configurado e onde vc baixa o Report (com o relatório completo)
- findings, onde estão as análises via console

Security Hub
-------------- 

- Centraliza os alertas de segurança de vários serviços de segurança da AWS (guard duty / inspector / Macie )
- Também tem sua própria análise seguindo alguns padrões (CIS AWS Foundations / PCI DSS); necessário AWS Config
- Alertas são mostrados APÓS habilitado via Security HUB, os anteriores não são indexados;
- Funciona em apenas 1 região

WAF
----

- Web ACL é o conjunto geral dos rules, rules statements e associations;
- Rule Statements: onde vc define qual regra vc quer que seja analisado durante uma requisição (ex: bloqueio acessos vindos da ìndia ; bloqueai acesso a url /admin); admite regras customizadas, bem como padrões da AWS
- Rules (regras de ações baseadas nos statements), podem ser de 2 tipos: 
1. regular: pode combinar vários Rule Statements e criar regras do tipo AND, OR, NOT; ex: se o request tiver um código SQL e tiver IP de origem x.x.x.x;
2.  rate-based: regular rule + rate limiting, exemplo: se a requisição vier do IP 172.x.x.x e se a requisição exceder 1000 requests em 10 minutos;
- Association:
1. não possível de ser atachado diretamente em uma ec2;
2. support association: alb, cloudfront, api gateway;
3. 1500 WCU no total disponíveis;
4. Cada rule tem uma prioridade e se a requisição bater, nenhuma outra rule será inspecionada

SSM
----

- suporte ec2, on-premises e VMs
- necessário agente no EC2 (exceto Amazon Linux AMI) / role para o SSM / saída para os endpoints do SSM via 443
- Session Manager
1. Centraliza acesso dos ec2;
2. Audita os acessos (logs das sessões podem ser exportados para um Bucket/cloudwatch com todo histórico de comandos (inclusive os outputs), qual usuário iam, etc;
3. Sem necessidade de portas abertas nos SG.
- Run Command: permite rodar comandos remotos nos ec2, usando templates
- Patch Manager: permite gerenciar os patches dos ec2, usando baselines já prontos ou criados, onde vc define janela de manutenção, janela de reboot se necessário, export dos logs da aplicação, se vc quer fazer só um scan ou scan e aplicar, entre outras opções, como a possibilidade de usar hooks durante a aplicação do patch (antes/depois por exemplo), uso de grupos/tags, etc;
- Parameter Store: permite armazenar senhas e configurações (chave-valor), como database strings, senhas, etc; possível criptografar usando KMS. Ex via linha de comando: aws ssm get-parameter --name parameter, se for criptografado vc pode usar com a opção --with-decryption.

Athena
-------

- analisa log files do S3 usando SQL;
- exemplo de caso 1: análise de logs de auditoria no S3 que foram a partir de um cloudtrail
- exemplo de caso 2: um pico de tráfego na sua rede e vc quer analisar se o tráfego é válido ou um possível ataque; usa o Athen para analisar o VPC Flow Log enviado para o S3 para verificar o número e quais IPs foram rejeitados, qual ENI recebeu maior quantidade de tráfego, etc.

CloudTrail
------------

- monitora todas as chamadas de API feitas na conta, usado para auditoria (ex.: quem, quando, de onde foi deletado o bucket XXXX);
- possível criar trails (um é de graça), onde vc joga os logs para o S3, possível habilitar para todas as contas de um Organizations;
- possui integração com cloudwatch para monitorar os trails e notificar quando algo acontece;
- no Event history só é possível visualizar os últimos 90 dias de logs, mais do que isso é necessário criar um trail e jogar para o S3.
- Log file integrity: usado para garantir a integridade dos logs do trail, evitando assim que alguém manipule o arquivo de log. Ele faz isso usando hash e assinatura digital:

```console
$ aws cloudtrail describe-trails // pesquisar todas os trails
$ aws cloudtrail validate-logs --trail-arn [ARN-HERE] --start-time 20190101T19:00:00Z // validar integridade do trail
```

AWS Config
------------

- utilizado para monitorar alterações nas configurações dos recursos com o tempo. ex.: uma ec2 que rodava um site nos últimos 90 dias começou a dar vários problemas. O que foi que houve? o que foi alterado?
- usado também para audit e compliance, pois vc consegue criar as Rules que monitora os recursos e alerta os Findings que estão compliances/não compliances. Ex.: cria uma rule que diz que uma determinada ami está pré-aprovada para isso e ele monitora ec2 que foram lançadas sem essa AMI;
- rules podem ser gerenciadas pela AWS (aws managed ou criada customizada);
- conformance pack: basicamente uma coleção de AWS Config Rules e remediações que podem ser configuradas. ex: ao invés de selecionar uma rule específica sobre S3, eu posso escolher um pack com as best practices incluindo todas as rules do S3;
- por muitas vezes uma rule (aws managed) vem de um conformance pack.
- para habilitar que ele monitore todas as regiões, é necessário habilitar global resource check durante a configuração (The global resource types that AWS Config supports are IAM users, groups, roles, and customer managed policie)
- best practices: https://aws.amazon.com/blogs/mt/aws-config-best-practices/

Trusted Advisor
----------------

- best practices da AWS evolvendo os principais pilares: cost optimization, performance, security, fault tolerance e service limits;
- para liberar todas as features do trusted advisor, precisa ter um plano de suporte business ou enterprise, caso contrário fica limitado somente algumas features de segurança e service limits;
- exemplos de best practices: idle load balancers, EIP sem associação (cost optimization); segurança (access key sem rotação por no mínimo 90 dias e etc)

AWS Macie
----------

- Macie usa sistema de machine Learning para analisar dados dentro de bucket s3 a fim de proteger contra vazamento de dados, informações sensíveis e demais informações críticas de segurança. Ele se baseia em alguns padrões, como o PII para buscar por exemplo informações críticas bancárias/cartões. Acesso keys colocadas indevidamente, entre outros. Pode ser customizado se necessário e integrado com sistemas de eventos para remediar alguma coisa específica.

VPC Flow Logs
---------------

- Formato do log

```console
2 123456789010 eni-1235b8ca123456789 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK
```

1. version (flow logs version)
2. account-id
3. eni id
4. source
5. destiny
6. source port
7. dest port
8. protocol number (6 é tcp por exemplo)
9. pacotes transferidos
10. bytes transferidos
11. start time in unix seconds
12. end time in unix seconds
13. ação (accept / reject)
14. log status do flow log