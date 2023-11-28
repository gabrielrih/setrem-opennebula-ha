# Node

A instalação começa com uma atualização do sistema:

```sh
sudo su
apt-get update
apt-get -y install gnupg wget apt-transport-https
```

Adicione o repositório (com o usuário root):
```sh
sudo su
wget -q -O- https://downloads.opennebula.io/repo/repo2.key | apt-key add -
echo "deb https://downloads.opennebula.io/repo/6.6/Ubuntu/22.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list
```

```sh
sudo su
apt-get update
sudo apt-get install opennebula-node-kvm
```

Reinicie o Libvirt
```sh
sudo service libvirtd restart
```

É importante ainda definir uma senha para o usuário oneadmin, com o comando:
```sh
passwd oneadmin
```