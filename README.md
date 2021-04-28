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

- É uma forma estuturada para identificar e gerenciar as ações necessários pós-ocorrido de um incidente de segurança dentro da sua organização, limitando os dadnos causados, reduzindo tempo de recuperação e os custos causados pelo incidente;
