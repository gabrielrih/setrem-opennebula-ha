# Instalação e configuração (Atividade prática)

__NGinx__
- Instalação no Ubuntu: https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/

- Personalizar configurações:    
    https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/
    - Worker processes: https://nginx.org/en/docs/ngx_core_module.html#worker_processes
    - Content caching: https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/

- Load balancing:
    - Upstream: https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/
    - Usar método least_conn: https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn.
    -  Passive Health Check: https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#passive-health-checks
        Me parece que eu terei timeout na conexão com a versão OpenSource do NGinx. Isso porque o sistema passivo marca o servidor como DOWN após esperar uma resposta de uma requisição de não receber. 
    - Sharing data with multiple worker processes: https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sharing-data-with-multiple-worker-processes
        Criar uma zona no upstream. É para o uso correto do algoritmo de balanceamento (Least Connections). Usar 32k é mais do que suficiente.


Para tarefas programadas no Frontend, posso adicionar a opção _down_ no server (configuração de upstream do NGinx): https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/.



__MemCached__

A fazer.


__Frontends__

- Floating IP should be used for monitoring daemon parameter MONITOR_ADDRESS in /etc/one/monitord.conf
Fiz esta alteração apontando para um IP fixo (o frontend2). Porém não tenho certeza se isso resolveu.
https://docs.opennebula.io/6.6/installation_and_configuration/ha/frontend_ha.html#sunstone-and-fireedge
