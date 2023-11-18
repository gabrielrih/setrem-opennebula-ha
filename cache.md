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

# Frontends

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