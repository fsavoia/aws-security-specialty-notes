IAM
-----

- resource based policy são policies que são atachadas nos recursos, exemplo: s3, kms, sqs, etc; se não houver nenhum explicity denied e houver um IAM policy + resource based policy, os acessos serão somados no final;
- statement element é obrigatório numa definição de policy: {}; se conter mais de um statement, ele fica entre [{},{}]
- effect: só possui 2 valores possíveis: Allow / Denied
- action: lista de ação que são permitidas ou negadas
- resource: define qual recurso o statement se refere (conjunto todo)
- External ID: é uma configuração que vc coloca na API do AssumeRole do STS, é usado como quase um duplo fator de segurança (quase como um MFA); mas só funciona quando faz um AssumeRole via CLI ou SDK; se tentar via console, dá acesso negado.
- external ID é uma best practice quando um terceiro precisa assumir uma role na sua conta  
- Role Architecture: Quando uma role é atachada num ec2, o sts gera a credencial e inputa dentro do metadata da máquina: 169.254.169.254/latest/meta-data/iam/security-credentials/first-iam-role. Para bloquear o acesso local (de um ou mais usuários) ao serviço de metadata para evitar brecha de segurança, pode ser usado um iptables por exemplo.
- version element: 2012-10-17: versão mais novas, com funções mais modernas, como por exemplo variaveis {aws.username}; 2008-10-17: versão antiga
- policy version é diferente do version element: policy version é apenas um versionamento de alterações das policies.
- NotPrincipal element: IMPORTANTE: não é porque existe um NotPrincipal dentro de um Deny que vc automaticamente tem um Allow para esse usuário. Isso só garante a vc o não bloqueio, mas a permissão precisa ser configurada na policy do seu usuário.
- IAM boundaries: quando um boundarie é configurado, ele não automaticamente lhe garante nenhuma permissão, ele é apenas uma régua limitando o máximo de permissão possível que ele pode ter, ou seja, vc ainda precisa estipular o identitie policy;
- IAM database authentication: dá pra se autenticar via IAM no MySQL e PostgreSQL; ele possui suporte a usar um token ao invés de uma senha; toda comunicação de autenticação é criptografada via SSL; permite gerenciar centralmente via IAM ao invés de conectar nos bancos individualmente; para aplicações rodando em EC2, dá para usar um profile credentials ao invés de usar senhas.


SSO
------

- possível de se autenticar via CLI: aws sso login --profile account (já pre configurado com link, conta, role, região)
- para habilitar SSO, o Organizations precisa ser habilitado;

S3
-----

- qundo vc habilita versionamento no bucket, não é mais possível remover o versionamento, somente pausar;
- cross region replication: ambos buckets precisam de versionamento habilitado;
- object lock: WORM (write once, read many), usado para bloquear escrita/modificação num objeto após criação; no modo governance mode, alguns usuários IAM podem remover o lock, já no caso do compliance mode, ninguém pode alterar o lock; versionamento é pré-req;
- quando a permissão é concedida no bucketname/, o acesso é a nível de bucket;
- quando a permissão é concedida no bucketname/*, o acesso é a nível de objeto;
- quando a permissão é concedidade no bucketname*, o acesso é total e em qualquer coisa que contenha bucketnameXXXXXX (cuidado com essa permissão, não recomendado);
- quando houver necessidade de criptografia em transito (SSL), pode ser usado uma bucket policy negando quando houver aws:SecureTransport:false.