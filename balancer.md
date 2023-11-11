# balancer

O que precisa ser executado na VM que será utilizada como Balanceador de Carga.

## Instalação do NGinx

Instalar NGINX Open Source:
```sh
sudo chmod 744 ./scripts/install_nginx.sh
./scripts/install_nginx.sh
```

Testar se o NGinx funcionou:
```sh
curl -I 127.0.0.1
```

[Referência](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)

## Habilitar SSL no NGinx

Criar um certificado autoassinado a ser utilizado pelo NGinx:
```sh
cd /etc/nginx
mkdir certificates
cd certificates

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt

chmod 644 *
```

[Referência](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04#adjust-the-nginx-configuration-to-use-ssl)

Configurar o NGInx para habilitar a porta 443 utilizando o certificado SSL criado.
Adicionar no arquivo _/etc/nging/nging.conf_ as seguintes linhas:

```conf
http {
    ...

    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              80;
        listen              443 ssl;
        server_name         balancer;

        keepalive_timeout   70;

        ssl_certificate     /etc/nginx/certificates/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/certificates/nginx-selfsigned.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        
        ...

    }
}
```

[Referência](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/)


## Direcionar requisições para o frontend

O que fazemos aqui é basicamente seguir a documentação deste link: https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/

No mesmo arquivo _/etc/nging/nging.conf_ temos que adicionar as seguintes linhas:
```sh
...

upstream backend {
    least_conn;
    server frontend1:9869;
    server frontend2:9869;
    server frontend3:9869;
}

server {
    ...
    location / {
        proxy_pass http://backend;
    }
}
```

> Obviamente, os servidores de frontend só devem ser adicinados aqui quando eles estiverem com o Sunstone corretamente configurado.

Quanto a linha _least_conn_, é onde é definido o algoritmo de balanceamento que será utilizado. Neste caso o "Least connections". Isso significa que uma requisição será redirecionada para o nó que está com menos conexões ativas.