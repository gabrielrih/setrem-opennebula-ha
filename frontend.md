# Frontends

Realize os passos abaixo em todos os FrontEnds:

Instalar o OpenNebula:
```sh
sudo apt-get update
sudo apt-get -y install gnupg wget apt-transport-https

sudo su
wget -q -O- https://downloads.opennebula.io/repo/repo2.key | apt-key add -

echo "deb https://downloads.opennebula.io/repo/6.6/Ubuntu/22.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list

sudo apt update

sudo apt-get -y install opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow opennebula-provision
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

Conectar no MySQL e criar um usuário específico:
```sh
sudo su
mysql -u root
```

```sql
CREATE USER 'oneadmin' IDENTIFIED BY 'Poscloud2023!';
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
PASSWD = "Poscloud2023!",
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
echo 'oneadmin:cloud2023' > /var/lib/one/.one/one_auth
```

## Somente no Frontend1

Inicie os serviços do opennebula:
```sh
systemctl start opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow
```

Habilite os serviços do opennebula:
```sh
sudo systemctl enable opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow
```

# FIX IT
# Colocar aqui como faz para colocar esse server como LEADER
#

## No Frontend2 e no Frontend3

Fazer o dump da database do OpenNebula no MySQL do Frontend1 e restaurar no frontend follower que estamos configurando agora.

- Executar no Frontend1

Backup:
```sh
onedb backup -u root -d opennebula
```

OBS: Senha em branco

Enviar backup para o novo frontend:
```sh
scp /var/lib/one/mysql_localhost_opennebula_2023-11-17_1:22:30.sql frontend2:/tmp
```

Copiar o conteúdo do .one directory
Tome cuidado para preservar o owner: oneadmin
```sh
ssh frontend2 rm -rf /var/lib/one/.one
scp -r /var/lib/one/.one/ frontend2:/var/lib/one/
```

- Executar no frontend novo

Mudar o owner dos arquivos novos copiados:
```sh
chown -R oneadmin:oneadmin /var/lib/one/.one
```

Restaurar backup no novo frontend:
```sh
onedb restore -f -u root -d opennebula -f /tmp/mysql_backup.sql
```

- Voltando ao frontend1

Adicione o servidor novo na zona:
```sh
onezone server-add 0 --name server-1 --rpc http://192.168.20.211:2633/RPC2
```

Onde 192.168.150.2 representa o IP do novo frontend.

Após este comando você verá um erro no server caso execute: ```onezone show 0```
Para corrigir isso, você deve modificar o arquivo ```/etc/one/oned.conf``` no frontend novo e alterar o ```SERVER_ID``` de -1 para 1 (ou 2). Representando respectivamente o número daquele server.

> Lembrando que o leader é 0.

Lembre de iniciar ou reiniciar os serviços:
```sh
systemctl start opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow
```


## Executar em todos

Desabilitar o fireedge
```sh
systemctl stop opennebula-fireedge
systemctl disable opennebula-fireedge
```