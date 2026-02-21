# [Atlas-Vital]-loadbalancer

## Descrição

Este projeto implementa um Load Balancer utilizando Nginx com 5 nós,
aplicando o algoritmo Round Robin para distribuir as requisições de uma aplicação front-end desenvolvida em ReactJs na disciplina de Programação Web 2.

O ambiente foi configurado utilizando comandos básicos do Docker.

---

## Estrutura do Projeto

O projeto contém:

- dist/ → aplicação React compilada
- nginx.conf → configuração principal do Nginx
- default.conf → configuração do Load Balancer

---

## Arquitetura

Cliente  
↓  
Nginx (Load Balancer)  
↓  
5 Nós (Nginx servindo aplicação React)

---

## Algoritmo Utilizado

O Nginx utiliza por padrão o algoritmo Round Robin quando múltiplos servidores são definidos em um bloco upstream.

Isso significa que as requisições são distribuídas sequencialmente entre:

- node1
- node2
- node3
- node4
- node5

---

## Configuração do Nginx

### nginx.conf

Arquivo responsável pelas configurações globais do serviço:

```nginx
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    include /etc/nginx/conf.d/*.conf;
}
default.conf

Arquivo responsável pela configuração do Load Balancer:

upstream frontend_nodes {
    server node1:80;
    server node2:80;
    server node3:80;
    server node4:80;
    server node5:80;
}

server {
    listen 80;

    location / {
        proxy_pass http://frontend_nodes;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

O envio do IP real da requisição é realizado através dos cabeçalhos:

X-Real-IP

X-Forwarded-For

 Configuração do Ambiente com Docker
 Criar a rede Docker
docker network create rede_lb
 Criar os 5 nós da aplicação React

Considerando que a pasta dist já contém a aplicação compilada:

docker run -d --name node1 --network rede_lb -v $(pwd)/dist:/usr/share/nginx/html nginx:alpine
docker run -d --name node2 --network rede_lb -v $(pwd)/dist:/usr/share/nginx/html nginx:alpine
docker run -d --name node3 --network rede_lb -v $(pwd)/dist:/usr/share/nginx/html nginx:alpine
docker run -d --name node4 --network rede_lb -v $(pwd)/dist:/usr/share/nginx/html nginx:alpine
docker run -d --name node5 --network rede_lb -v $(pwd)/dist:/usr/share/nginx/html nginx:alpine

Cada container serve a aplicação React através do Nginx.

 Criar o container do Load Balancer
docker run -d \
  --name loadbalancer \
  --network rede_lb \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf \
  -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf \
  nginx:alpine
 Testando o Load Balancer

Após a execução dos comandos, acesse no navegador:

http://localhost

O Nginx irá distribuir as requisições entre os 5 nós utilizando Round Robin.
