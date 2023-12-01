# Configurações que devem ser feitas em todas as VM de frontend

## Todos os frontends

Realize os passos abaixo em todos os FrontEnds:

Instalar o OpenNebula:
```sh
sudo su

apt-get update
apt-get -y install gnupg wget apt-transport-https

wget -q -O- https://downloads.opennebula.io/repo/repo2.key | apt-key add -
echo "deb https://downloads.opennebula.io/repo/6.6/Ubuntu/22.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list

apt update
apt-get -y install opennebula opennebula-sunstone opennebula-gate opennebula-flow opennebula-provision
```

Instalar o MySQL:
```sh
sudo su

apt install mysql-server
systemctl start mysql.service
systemctl status mysql
```

Configurando o MySQL:
```sh
sudo mysql_secure_installation
```

- Remove anonymous users? Yes
- Disallow root login remotely? No
- Remove test database and access to it? Yes
- Reload privilege tables now? Yes

## Somente no Frontend1 (Leader)

Conectar no MySQL e criar um usuário específico:
```sh
sudo su
mysql -u root
```

```sql
CREATE USER 'oneadmin' IDENTIFIED BY 'my-password-here';
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Configurar o MySQL no arquivo do OpenNebula:
```sh
vi /etc/one/oned.conf
```

```conf
DB = [ BACKEND = "mysql",
SERVER = "localhost",
PORT = 0,
USER = "oneadmin",
PASSWD = "my-password-here",
DB_NAME = "opennebula",
CONNECTIONS = 25,
COMPARE_BINARY = "no" ]
```

Configuração do OpenNebula:

Troque para o usuário oneadmin
```sh
sudo -u oneadmin /bin/bash
```

Crie o arquivo /var/lib/one/.one/one_auth com senha inicial no formato
```sh
echo 'oneadmin:my_password_here' > /var/lib/one/.one/one_auth
```

Inicie os serviços do opennebula:
```sh
systemctl start opennebula opennebula-sunstone opennebula-gate opennebula-flow
```

Habilite os serviços do opennebula:
```sh
sudo systemctl enable opennebula opennebula-sunstone opennebula-gate opennebula-flow
```

Configurar Alta Disponibilidade. Colocar o Frontend1 como o LEADER de um cluster de HA.

Primeiro encontrar a zona.
Aqui você pode ver que o servidor aparece como SOLO, ou seja, faz parte de um standalone e não de um cluster com mais nós.
```sh
onezone list
onezone show 0
```

Adicione o server na zona:
```sh
onezone server-add 0 --name frontend1 --rpc http://192.168.20.21:2633/RPC2
```

Onde, você deve colocar o --name como sendo o hostname da VM e o IP da VM.

Pare o serviço do opennebgula e altere o arquivo ```/etc/one/oned.conf``` adicionando o seguinte:
```sh
systemctl stop opennebula
vi /etc/one/oned.conf
```

```conf
FEDERATION = [
    MODE          = "STANDALONE",
    ZONE_ID       = 0,
    SERVER_ID     = 0, # changed from -1 to 0 (as 0 is the server id)
    MASTER_ONED   = ""
]
```
Note que o que mudou foi o SERVER_ID. Estava como -1 e agora mudamos para 0.

Agora você pode iniciar o serviço do openNebula novamente e verificar a zona:
```sh
systemctl start opennebula
onezone show 0
```

Note que agora o STATE do nó aparece como LEADER e não mais como SOLO.


Fazer o dump da database do OpenNebula no MySQL do Frontend1 e restaurar nos demais frontends (followers).

Backup:
```sh
onedb backup -u root -d opennebula
```

Enviar backup para cada um dos demais frontends (aqui utilizando o frontend2 como exemplo):
```sh
scp /var/lib/one/mysql_localhost_opennebula_2023-11-17_1:22:30.sql frontend2:/tmp
```

Copiar o conteúdo do .one directory (chaves que devem ser as mesmas entre todos os frontends)
Tome cuidado para preservar o owner: oneadmin
Caso o owner não seja preservado, mais abaixo temos um comando para mudar o owner destes arquivos em cada um dos demais frontends.
```sh
ssh frontend2 rm -rf /var/lib/one/.one
scp -r /var/lib/one/.one/ frontend2:/var/lib/one/
```

## Nos demais servidores de frontend (Followers)

Mudar o owner dos arquivos novos copiados (chaves que vieram do leader):
```sh
sudo chown -R oneadmin:oneadmin /var/lib/one/.one
```

Criar database opennebula no MySQL:
```sh
sudo su
mysql -u root
```

```sql
CREATE DATABASE opennebula;
```

Restaurar backup no novo frontend:
```sh
onedb restore -f -u root -d opennebula -f /tmp/mysql_backup.sql
```

Conectar no MySQL e criar um usuário específico para uso do OpenNebula:
```sh
sudo su
mysql -u root
```

```sql
CREATE USER 'oneadmin' IDENTIFIED BY 'my-password';
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Configurar o MySQL no arquivo do OpenNebula:
```sh
vi /etc/one/oned.conf
```

```conf
DB = [ BACKEND = "mysql",
SERVER = "localhost",
PORT = 0,
USER = "oneadmin",
PASSWD = "my-password-here",
DB_NAME = "opennebula",
CONNECTIONS = 25,
COMPARE_BINARY = "no" ]
```

- Voltando ao frontend1

Adicione o servidor novo na zona (mesma zona que o frontend1, leader, está inserido):
```sh
onezone server-add 0 --name frontend2 --rpc http://192.168.20.211:2633/RPC2
```

> Tome cuidado aqui para que o IP seja o IP do novo frontend. Além disso, o "server-1" deve ser uma string única. Use "server-0", "server-1", e assim por diante.

Após este comando você verá um erro no server caso execute: ```onezone show 0```
Para corrigir isso, você deve modificar o arquivo ```/etc/one/oned.conf``` no frontend novo e alterar o ```SERVER_ID``` de -1 para 1 (ou 2). Representando respectivamente o número daquele server.

> Lembrando que o leader é 0.

Lembre de reiniciar os serviços, ou caso ainda não tenha iniciado você tem que habilitar e iniciar os serviços:
```sh
sudo systemctl enable opennebula opennebula-sunstone opennebula-gate opennebula-flow
sudo systemctl start opennebula opennebula-sunstone opennebula-gate opennebula-flow
```


# Conectando no NODE via SSH

O OpenNebula tem uma particularidade que é conectar nos "nodes" através de SSH.
Para isso é necessário que todos os frontends consigam conectar nos "nodes" via SSH sem precisar informar usuário e senha automaticamente.

Para isso, o próprio OpenNebula já criou uma chave SSH. O que temos que fazer é configurar para que essa chave seja utilizada automaticamente toda vez que o OpenNebula tentar conectar no node:

> Lembrando que aqui estamos utilizando um node chamado "node2".

Em cada frontend executar:
```sh
su - oneadmin
ssh-keyscan node2 >> /var/lib/one/.ssh/known_hosts
ssh-copy-id -i /var/lib/one/.ssh/id_rsa.pub node2
```