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