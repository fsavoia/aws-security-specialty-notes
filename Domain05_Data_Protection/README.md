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