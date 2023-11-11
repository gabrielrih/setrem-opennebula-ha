# Geral

É necessário executar as configurações abaixo em todas as VMs:

## Configuração básica para conectividade e segurança
```yaml
services:
- frontend1 (192.168.20.21)
- frontend2 (192.168.20.211)
- frontend3 (192.168.20.212)
- cache (192.168.20.213)
- balancer (192.168.20.214)
- node (192.168.20.92)
```

- Atualização de pacotes
```sh
sudo su
apt update
apt upgrade -y
apt install net-tools
```

- Alterar o hostname de cada VM
```sh
vi /etc/hostname
```

- Adicionar no host o nome de cada VM (para facilitar a conexão)
```sh
vi /etc/hosts
```

No arquivo /etc/hosts
```conf
# OpenNebula
192.168.20.214  balancer
192.168.20.21   frontend1
192.168.20.211  frontend2
192.168.20.212  frontend3
192.168.20.213  cache
192.168.20.92   node
```

- Alterar a senha do usuário root
```sh
sudo passwd root
```
... e então informe a senha.


## Configuração do SSH

Editar as seguintes propriedades no arquivo: /etc/ssh/sshd_config

```conf
PermitRootLogin yes
PasswordAuthentication yes
```

Ao final reiniciar o serviço
```sh
sudo service sshd restart
```

