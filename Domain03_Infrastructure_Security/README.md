Bastion Host
-------------

- Nunca guardar as chaves privadas dentro do Bastion, ao invés disso, usar o SSH agent para fazer forward da chave.

VPN
-----

- VGW: virtual private gateway; possui nativamente alta disponibilidade (2 HA endpoints, uma em cada AZ); nesse caso, no customer site, vc cria 2 túneis, uma para cada eni do VGW;
- CGW: customer gateway, basicamente o endpoint da ponta de destino da VPN (por exemplo o firewall on premises)
- VPN S2S (IPsec): não esquecer de inserir rota para a rede de destino da VPN, para sair pelo VGW; não esquecer que um VGW só pode ser atachado em uma VPC.

VPC Peering
-------------

- não faz transito, ou seja, se lembrar que se houver um peering entre VPC_A <-> VPC_B e VPC_B <-> VPC_C, vc não consegue chegar via roteamento/peering entre VPC_A -> VPC_C.
- não esquecer que ela precisa de rota entre os CIDRs saindo pelo peering (em ambas as VPCs);
- não esquecer que ainda assim os Security Groups controlam os acessos;

VPC Endpoint
-------------

- Gateway Endpoint: nesse caso, o endpoint é criado fora da sua VPC e o tráfego basicamente é roteado até o gateway via Route Table; possível de controlar os acessos via IAM (Policy);
- como o gateway endpoint é criado fora da sua VPC, vc não consegue acessar os recursos dele via VPN ou Direct Connect por exemplo;
- Interface Endpoint: criado dentro da sua VPC, via ENI / private IP e controlado via security group;
- importante lembrar que o VPC Endpoint funciona para serviços na mesma região; um caso que pode confundir é quando listamos os buckets por exemplo e ele mostra todos na saída, porém isso é porque nesse momento ele está conectado na console do S3, porém quando tentar acessar algum conteúdo dentro de um bucket de outra região, isso irá falhar;
- gateway endpoint atualmente só existe para S3 e DynamoDB;

NACL
------

- stateless por padrão, ou seja, a regra de retorno precisa ser explicitamente controlada (ele não mantém o estado da conexão, como o statefaul);
- normalmente as portas 1-1023 são portas reservadas conhecidas, acima disso são ephemeral; cada S.O geralmente tem um padrão de portas altas que iniciam a conexão (client);
- opera na camada da subnet e não da instância (eni)
- por padrão, toda VPC é criada com uma NACL que libera todo tráfego de saída e entrada
- numa regra da NACL, quanto menor valor, maior prioridade (as regras são lidas como ACL, deu match, é respeitado); o "*" é a regra padrão;
- uma NACL pode ser associado a várias subnets;

EBS
-----

- para elimiar definitivamente os dados (wipe) de um volume EBS, tem 2 formas:
1. antes de terminar a instância, manualmente o cliente pode fazer um wipe por conta própria;
2. AWS automaticamente faz um Wipe dos dados imediatamente antes do EBS ser marcado para reuso;
- quando o storage/disco atingir o "end of use" eles são decomissionados seguindo alguns documentos de compliance de segurança;

CloudFront
-----------

- SNI
1. usado para  habilitar múltiplos certificados/domínios num mesmo servidor
2. navegadores mais modernos suportam SNI
3. antigamente, caso fosse necessários vários domínios/certificados no mesmo servidor, era necessário alocar IPs dedicados para cada serviço para funcionamento do SSL (dns->ip->ssl por domínio)
5. o cloudfront tem uma função bem cara ($600/mês) que habilita IPs dedicados em cada edge location para permitir funcionamento de navegadores legados (isso garante acesso total ao site)

- OAI: usado para permitir acesso na origem somente pelo CloudFront, por exemplo, evitando expor um bucket S3 diretamente pro mundo; necessário habilitar o OAI, incluir uma bucket policy, bloquear acesso público.

- Signed URLs or Signed cookie
1. possível implementar URLs assinadas diretamente a nível do Cloudfront permitindo acesso a algum objeto somente a pessoas autorizadas
2. possui um Trusted Signers, outras contas além de vc podem criar URLs assinadas para seus objetos
3. Os cloudfront key pairs precisam ser gerados via root no IAM (security credentials)
4. precisa ser habilitado na distribuição.
