# Keycloak Docker setup with NGINX Proxy and Letsencrypt hosted on AWS EC2 with managed Postgres DB on AWS RDS

A simple Keycloak setup using NGINX Reverse Proxy and Letsencrypt with a database hosted on AWS RDS

originally cloned from here: https://github.com/selloween/docker-keycloak-letsencrypt


### Reference
* NGINX Proxy Docker image: https://github.com/jwilder/nginx-proxy
* Letsencrypt NGINX Docker Companion: https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion

## Guide

First set up your AWS RDS postgres database, including a security group to allow ingres from the internal IP of your EC2 instance.

Copy sample.env to .env and adjust the environment variables.
Then ```docker-compose up -d```

Note: ssl option in JDBC_PARAMS is set to false, as the nginx proxy will handle SSL.
```
DB_VENDOR=postgres
DB_ADDR=endpoint.rds.amazonaws.com:5432
DB_DATABASE=postgres
DB_USER=keycloak_db_user
DB_PASSWORD=Passw0rd!
KEYCLOAK_HOSTNAME=example.com
KEYCLOAK_HTTP_PORT=8080
KEYCLOAK_USER=admin
KEYCLOAK_PASSWORD=Passw0rd!
JDBC_PARAMS="ssl=false" 
VIRTUAL_HOST=example.com
VIRTUAL_PORT=8080
PROXY_ADDRESS_FORWARDING=true
LETSENCRYPT_HOST=example.com
LETSENCRYPT_EMAIL=user@example.com
```
