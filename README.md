# Keycloak Docker setup with NGINX Proxy, Letsencrypt, hosted on AWS EC2 with managed Postgres DB on AWS RDS, and config for federation to windows active directory

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


## Connect Keycloak instance to LDAP/Active Directory

- Followed this guide to set up the CA Root certificate on the AD server :
https://pdhewaju.com.np/2016/04/08/installation-and-configuration-of-active-directory-certificate-services/

- Add LDAPS service on AD server for port 636:
https://pdhewaju.com.np/2017/03/02/configuring-secure-ldap-connection-server-2016/#:~:text=Open%20your%20machine%2C%20go%20to,not%20have%20configured%20secure%20LDAP.

- AD integration guide for Keycloak:
https://dmc.datical.com/administer/configure-keycloak-ldap.htm#:~:text=Navigate%20to%20the%20Keycloak%20tab,the%20list%20of%20users%20imported

- Add CA Root certificates on Keycloak server:
https://dale-bingham-soteriasoftware.medium.com/linking-keycloak-to-microsoft-active-directory-and-its-nuances-c145857710ca

- Map a local folder to a volume in documer-compose

```
    volumes:
      - /home/ubuntu/keycloak-docker/disable-theme-cache.cli:/opt/jboss/startup-scripts/disable-theme-cache.cli
      - /home/ubuntu/keycloak-docker/certs:/opt/jboss/keycloak/certs  
      - /home/ubuntu/keycloak-docker/LDAP-certs-SPI.cli:/opt/jboss/startup-scripts/LDAP-certs-SPI.cli
```

The `LDAP-certs-SPI.cli` script tells keycloak to load the cert required for an AD connection

This script corresponds to the following in the `standalone-ha.xml` file

```
<spi name="truststore">
  <provider name="file" enabled="true">
    <properties>
      <property name="file" value="/opt/jboss/keycloak/certs/truststore.jks"/>
      <property name="password" value="changeit"/>
      <property name="hostname-verification-policy" value="ANY"/>
      <property name="enabled" value="true"/>
    </properties>
  </provider>
</spi>
```
