Bastion Host
-------------

- Nunca guardar as chaves privadas dentro do Bastion, ao invés disso, usar o SSH agent para fazer forward da chave.

VPN
-----

- VGW: virtual private gateway; possui nativamente alta disponibilidade (2 HA endpoints, uma em cada AZ); nesse caso, no customer site, vc cria 2 túneis, uma para cada eni do VGW;
- CGW: customer gateway, basicamente o endpoint da ponta de destino da VPC (por exemplo o firewall on premises)