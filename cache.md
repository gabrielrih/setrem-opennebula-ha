# Memcached

Instalação: https://github.com/memcached/memcached/wiki/Install

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

# Frontend1

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

<!--
Dependências
```sh
apt-get install libevent-dev
apt-get install build-essential
```

Memcached
```sh
wget http://memcached.org/latest
mv latest memcached.tar.gz
tar -zxf memcached.tar.gz
cd memcached-1.x.x
./configure --prefix=/usr/local/memcached
make && make test && sudo make install
```
-->