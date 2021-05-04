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