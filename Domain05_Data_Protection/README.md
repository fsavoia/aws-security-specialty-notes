HSM
------

- hardware security models: são dispositivos físicos especiais para gerenciar armazenamento de chaves;
- AWS Cloud HSM é single tenanted (um dispositivo dedicado para vc);
- usado dentro da sua VPC;
- pode ser integrado com red shift ou rds oracle;
- suporte a alta disponibilidade com um cluster de HSM;
- AWS usa um appliance para o HSM chamado Safenet Luna SA HSM;

KMS
------

- KMS é um serviço regional, por padrão, nenhuma chamada a CMK de outra região pode ser feita;
- não esquecer das roles para quem consumir o serviço do KMS;
- usar alias para chave é obrigatório;
- cmk (customer master key, chave gerada pelo cliente / não gerenciada pela AWS);
- key id: id da cmk;
- ciphertext: objeto/arquivo criptografado;
- KMS suporta chaves simétricas e assimétricas;
- chave assimétrica: usado o método chave pública / privada. imagine um chaveiro/chave: o chaveiro é a chave pública, onde vc pode compartilhar com alguém; a chave privada vc usava para abrir e deve guardar secretamente;
- exemplo de como criptografar algo com uma chave pública assimétrica gerada pelo KMS:

```console
# openssl rsautl -encrypt -oaep -in file.txt -out encrypted.txt -pubin -inkey mykey_pub
# aws kms decrypt --key-id "925760b0-17aa-47be-bf6d-e8a550cd62cb" --encryption-algorithm RSAES_OAEP_SHA_1 --ciphertext-blob fileb://encrypted.txt --query Plaintext --output text | base64 -d
```

- Outra função da chave assimétrica no kms é para fazer assinatura digital, onde vc assina (sign) com a chave privada, envia o arquivo e a chave pública para o destino, e ele verifica (verify) o conteúdo se está íntegro com a chave pública.

```console
# aws kms sign --key-id "64bcd1b9-0b2f-4924-89f1-91dfb000c6c6" --message fileb://demo.txt --signing-algorithm RSASSA_PKCS1_V1_5_SHA_256 --query Signature --output text | base64 -d > sign.txt
# aws kms verify --key-id "64bcd1b9-0b2f-4924-89f1-91dfb000c6c6" --message fileb://demo.txt --signature fileb://sign.txt --signing-algorithm RSASSA_PKCS1_V1_5_SHA_256
```
- devido a limitação do tcp/ip, vc só pode criptogradar arquivos de até 4k com uma CMK;
- devio a latências ou oscilações de redes, AWS recomenda  o uso de CMK + Data Key (usando conceito de envelope encryption)

- **Envelope Encryption**: o processo funciona da seguinte forma:
1. primeiramente é feito uma chamada para o KMS (generate data key), usando o seu key-id (cmk) para criar uma data key
2. a aws retorna a data key de 2 formas: plaintext e encrypted data key
3. localmente vc encripta o arquivo com o plaintext, exclui o plaintext da memória e guarda somente a data key encriptado;
4. quando precisar descriptografar, vc chamada o kms (decrypt) para decriptografar a data key;
5. com o plaintext data key, vc descriptografa o arquivo, exclui o plaintext da memória.

- Data Key caching: feito para diminuir a quantidade de requisições por segundo feitas na api do KMS, cacheando a data key e o plaintext data key sempre que necessário, diminuindo a frequência da requisições, diminuindo assim latência por exemplo; se atentar ao tradeoff dessa operação, pois eleva um pouco o risco comparado a forma tradicional. AWS só recomenda essa função em casos muito específicos (milhares de chamadas);

- Deletar CMK:
1. remoção é irreversível;
2. uma vez deletada, não é mais possível decriptar algo com esse CMK;
3. aws força um período de espera (mínimo 7 dias, máximo 30, padrão é 30);
4. durante esse período, nenhuma operação é permitida;
5. para deletar, vai em key actions e em schedule key deletion;
6. key também pode ser desabilitada ao invés de deletada caso necessário.
7. caso seja necessário dar rollback durante o período de espera, vc pode chamar a api CancelKeyDeletion e ele irá colocar a CMK no status de disabled;

- CMK deletion e EBS: quando usamos criptografia KMS com EBS, o KMS cria uma data key, encripta com sua CMK e envia a data key encriptada para o EBS; Em seguida, quando vc atacha um EBS numa instância, o EC2 chama o KMS (api Decrypt) para decriptar a data key e o KMS envia o plaintext data key para o EC2 também; portanto, quando vc agenda a remoção do CMK, não há efeito imediato no EC2, pq o EC2 está usando o plaintext data key (na memória) e não o CMK para encriptar o volume. Mesmo removendo o CMK, o efeito é o mesmo, só irá mudar em caso de detach/attach do EBC em outra EC2.
- Unmanageable CMK: boa prática, nunca remover o usuário IAM root do KMS policy, pois ele nunca pode ser removido, pois em caso de existe uma policy para um usuário específico, caso ele seja deletado por exemplo, o CMK fica ingerenciável;  
- Access policy: Para controlar as permissões da CMK via IAM policy, precisa haver uma perissão full dentro da Key policy, caso contrário a permissão é somada da IAM policy + key policy. Importante: se houver uma Key policy default com acesso full do usuário root, então todas as contas poderão controlar a CMK via IAM policy.
- KMS Grants: usado quando vc precisa ceder o uso (operação) a um usuário que não tem acesso a CMK, gerando assim um Token a partir de um usuário que tem acesso a CMK. Esse token pode ser revogado a qualquer momento. Exemplo de uso (gerando o token, usando e removendo):

    ```console
    aws kms create-grant /
    --key-id [KEY-ID] /
    --grantee-principal [GRANTE-PRINCIPLE-ARN] /
    --operations "Encrypt"

    aws kms encrypt --plaintext "hello world" 
    --key-id [KEY-ID] /
    --grant-tokens [GRANT TOKEN RECEIVED]

    aws kms revoke-grant --key-id [KEY-ID] --grant-id [GRANT-ID-HERE]
    ```
- kms:ViaService: limita/habilita o uso da CMK para requisições vindas de serviços específicos;
- migrando serviços usando KMS: em outra região, crie um snapshot, copia de região, e seleciona uma nova CMK da região de destino para proteger os dados (default encryption key não pode ser usado); na mesma região, vc pode usar a mesma CMK original ou especificar uma nova caso deseje;
- **muito importante:** caso esteja usando envelope encryption, usando data-keys, é necessário primeiro decriptar todo dado antes de migrar de região.

- KMS encryption context: In addition to limiting permission to the AWS KMS APIs, AWS KMS also gives you the ability to add an additional layer of authentication for your KMS API calls utilizing encryption context. The encryption context is a key-value pair of additional data that you want associated with AWS KMS-protected information. This is then incorporated into the additional authenticated data (AAD) of the authenticated encryption in AWS KMS-encrypted ciphertexts. If you submit the encryption context value in the encryption operation, you are required to pass it in the corresponding decryption operation. You can use the encryption context inside your policies to enforce tighter controls for your encrypted resources. Because the encryption context is logged in CloudTrail, you can get more insight into the usage of your keys from an audit perspective. Be aware that the encryption context is not encrypted and will be visible within CloudTrail logs. The encryption context should not be considered sensitive information and should not require secrecy.


Elastic Load Balancers
-----------------------

- Classic Load Balancers:
1. não suporta http2 nativamente
2. ip address não é suportado como um target
3. path based routing não suportado (camada 4)

- HTTP vs TCP listeners: quando o listener usa protocolo TCP, a conexão se inicia no client e é encaminhada diretamente para o target; quando usa HTTP, a conexão é feita até o balance, o cabeçalho é modificado e é iniciado uma nova conexão até o target.

DynamoDB Encryption
----------------------

- Dynamodb encryption client: se os dados salvos no Dynamodb forem sensíveis, o ideal é criptografar o dado o mais próximo possível da origem, nesse caso é possível usar o dynamodb encryption client para criptografar os dados da tabela ainda antes deles serem enviados para o Dynamodb (suporte ao KMS, HSM ou próprio sistema de criptografia).
- Dynamodb encryption at rest: suporta SSE usando KMS (AES256)

AWS Secrets Manager
--------------------

- suporte a gestão de senhas, com suporte nativo para RDS, DocumentDB, RedShift, outros bancos (ele conecta no banco e faz a gestão do acesso com todas as features automáticas); ele usa uma função lambda por trás para fazer isso (e ela precisa estar dentro da sua VPC);
- é usar versionamento, portanto as aplicações não quebram quando as senhas são roteacionadas;
- possível acessar as senhas via SDK;
- as senhas podem ser automaticamente rotacionadas (default 30 dias, mas pode variar entre 1 e 365 dias);
- suporte a IAM e resource based policies;
- suporte replicar a secret para outra região