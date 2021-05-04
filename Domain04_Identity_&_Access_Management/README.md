IAM
-----

- resource based policy são policies que são atachadas nos recursos, exemplo: s3, kms, sqs, etc; se não houver nenhum explicity denied e houver um IAM policy + resource based policy, os acessos serão somados no final;
- statement element é obrigatório numa definição de policy: {}; se conter mais de um statement, ele fica entre [{},{}]
- effect: só possui 2 valores possíveis: Allow / Denied
- action: lista de ação que são permitidas ou negadas
- resource: define qual recurso o statement se refere (conjunto todo)
- External ID: é uma configuração que vc coloca na API do AssumeRole do STS, é usado como quase um duplo fator de segurança (quase como um MFA); mas só funciona quando faz um AssumeRole via CLI ou SDK; se tentar via console, dá acesso negado.
- external ID é uma best practice quando um terceiro precisa assumir uma role na sua conta  