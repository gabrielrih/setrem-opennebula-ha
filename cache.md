# Configurar cache para sessão de usuário

## Cache server

Instalação do Memcached: https://github.com/memcached/memcached/wiki/Install

```sh
apt install memcached
```

Porta padrão: 11211

Ajustar arquivo de configuração:
```sh
vi /etc/memcached.conf
```

No arquivo, alterar a linha ```-l 127.0.0.1``` para ```-l 0.0.0.0```.

Reiniciar o serviço:
```sh
service memcached restart
```

## Frontends

Após instalação e configuração do Memcached, temos que ajustar a configuração do Sunstone em todos os frontends para utilizar o memcached.

https://docs.opennebula.io/6.6/installation_and_configuration/large-scale_deployment/sunstone_for_large_deployments.html

```sh
vi /etc/one/sunstone-server.conf
```

Alterar os seguintes parâmetros:
```conf
:sessions: memcache

:memcache_host: cache
:memcache_port: 11211
:memcache_namespace: opennebula.sunstone
```

Reiniciar o serviço do sunstone:
```sh
sudo systemctl restart opennebula-sunstone
```

Checar se está utilizando o cache:
```sh
sudo systemctl status opennebula-sunstone
```

> Deve mostrar o host:port do servidor de cache.


# Criar discos NFS

Referência: https://computingforgeeks.com/configure-nfs-filesystem-as-opennebula-datastores/

## Cache server

Instalação do NFS server:
```sh
sudo su
apt update
apt -y install nfs-kernel-server
```

Criar diretórios:
```sh
sudo mkdir -p /mnt/opennebula/images
sudo mkdir -p /mnt/opennebula/system
sudo mkdir -p /mnt/opennebula/files
```

Configurar os exports do NFS.
Editar o arquivo ```/etc/exports``` e colocar o seguinte conteúdo nele:
```sh
/mnt/opennebula/images 192.168.20.0/24(rw,no_root_squash,no_subtree_check)
/mnt/opennebula/system 192.168.20.0/24(rw,no_root_squash,no_subtree_check)
/mnt/opennebula/files  192.168.20.0/24(rw,no_root_squash,no_subtree_check)
```

Reiniciar e habilitar o serviço:
```sh
sudo systemctl restart nfs-server
sudo systemctl enable nfs-server
```

Validar os exports:
```sh
sudo exportfs -rvv
```

## Frontends

Instalar o client do NFS:
```sh
sudo apt -y install nfs-common
```

Verifique se os diretórios de datastore existem no frontend, se não existir crie:
```sh
mkdir -p /var/lib/one/datastores/0
mkdir -p /var/lib/one/datastores/1
mkdir -p /var/lib/one/datastores/2
```

Montar os diretórios compartilhados via NFS em todos os frontends:

Edite o arquivo ```/etc/fstab``` e adicione o seguinte conteúdo:
```sh
# OpenNebula
192.168.20.213:/mnt/opennebula/system /var/lib/one/datastores/0 nfs defaults,soft,intr,rsize=32768,wsize=32768 0 0
192.168.20.213:/mnt/opennebula/images /var/lib/one/datastores/1 nfs defaults,soft,intr,rsize=32768,wsize=32768 0 0
192.168.20.213:/mnt/opennebula/files /var/lib/one/datastores/2 nfs defaults,soft,intr,rsize=32768,wsize=32768 0 0
```

> Note que o IP é do NFS server, ou seja, o servidor de cache. Além disso, os diretórios /var/lib/one/datastores/1 e /var/lib/one/datastores/2 representa o ID do disco já existente no OpenNebula. Cheque este valor.


Montar os diretórios NFS:
```sh
mount -av
```

Confirme o mount com o comando:
```sh
df -hT
```

Mudar permissões dos diretórios:
```sh
chown -R oneadmin:oneadmin /var/lib/one/datastores/0
chown -R oneadmin:oneadmin /var/lib/one/datastores/1
chown -R oneadmin:oneadmin /var/lib/one/datastores/2
```

<!--
## Nodes

Em todos as VM de nós execute os mesmos comandos que você executou no frontend, com a diferença que o diretório system também deve ser montado.
-->