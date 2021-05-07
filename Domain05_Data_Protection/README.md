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

- suporta chaves simétricas e assimétricas;
- não esquecer das roles para quem acesso o KMS;
- alias da chave é obrigatório;
- cmk (customer master key, chave gerada pelo cliente / não gerenciada pela AWS);
- key id: id da cmk;
- ciphertext: objeto/arquivo criptogradado;
- devido a limitação do tcp/ip, vc só pode criptogradar 4k com uma CMK (customer master key);
- devio a latências ou oscilações de redes, AWS recomenda  o uso de CMK + Data Key (envelope encryption)
- envelope encryption: o processo funciona da seguinte forma:
1. primeiramente é feito uma chamada para o KMS (generate data key), usando o seu key-id (cmk) para criar uma data key
2. a aws retorna a data key de 2 formas: plaintext e encrypted data key
3. localmente vc encrypta o arquivo com o plaintext, exclui o plaintext da memória e guarda somente a data key encryptado;
4. quando precisar descriptografar, vc chamada o kms (decrypt) para decriptografar a data key;
5. com o plaintext data key, vc descriptografa o arquivo, exclui o plaintext da memória.
- chave assimétrica: usado o método chave pública / privada. imagine um chaveiro/chave: o chaveiro é a chave pública, onde vc pode compartilhar e a chave para abrir, é a chave privada (vc deve guardar)
- exemplo de como criptogradar algo com uma chave pública assimétrica gerada pelo KMS:

```console
# openssl rsautl -encrypt -oaep -in file.txt -out encrypted.txt -pubin -inkey mykey_pub
# aws kms decrypt --key-id "925760b0-17aa-47be-bf6d-e8a550cd62cb" --encryption-algorithm RSAES_OAEP_SHA_1 --ciphertext-blob fileb://encrypted.txt --query Plaintext --output text | base64 -d
```

- Outra função da chave assimétrica no kms é para fazer assinatura digital, onde vc assina (sign) com a chave privada, envia o arquivo e a chave pública para o destino, e ele verifica (verify) o conteúdo se está íntegro com a chave pública.

```console
# aws kms sign --key-id "64bcd1b9-0b2f-4924-89f1-91dfb000c6c6" --message fileb://demo.txt --signing-algorithm RSASSA_PKCS1_V1_5_SHA_256 --query Signature --output text | base64 -d > sign.txt
# aws kms verify --key-id "64bcd1b9-0b2f-4924-89f1-91dfb000c6c6" --message fileb://demo.txt --signature fileb://sign.txt --signing-algorithm RSASSA_PKCS1_V1_5_SHA_256
```

- Data Key caching: feito para diminuir a quantidade de requisições por segundo feitas na api do KMS, cacheando a data key e o plaintext data key sempre que necessário, diminuindo a frequência da requisições, diminuindo assim latência por exemplo; se atentar ao tradeoff dessa operação, pois eleva um pouco o risco comparado a forma tradicional. AWS só recomenda essa função em casos muito específicos (milhares de chamadas);

- Deletar CMK:
1. remoção é irreversível;
2. uma vez deletada, não é mais possível decriptar algo com ess CMK;
3. aws enforce um período de espera (mínimo 7 dias, maximo 30, padrão é 30);
4. durante esse período, nenhuma operação é permitida;
5. para deletar, vai em key actions e em schedule key deletion;
6. key pode ser desabilitado ao invés de deletada caso necessário.
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