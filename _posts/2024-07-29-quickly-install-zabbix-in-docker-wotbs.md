---
title: Quickly install Zabbix (via Portainer)
date: 2024-07-29 20:33:00 +0800
categories: [howtos]
tags: [zabbix,portainer,docker,nms,wotbs]
---

## Create 2 files

### zabbix.env

> **Caution!** Create a strong password and change `POSTGRES_PASSWORD` value below.

```bash
cat <<EOF > zabbix.env
POSTGRES_USER=zabbix
POSTGRES_PASSWORD=CHANGEME
POSTGRES_DB=zabbix
PHP_TZ=Europe/London
ZABBIX_FRONTEND_PORT=8080
ZBX_STARTVMWARECOLLECTORS=1
EOF
```

Here's a [list of other variables](https://hub.docker.com/r/zabbix/zabbix-server-pgsql/) that you can enable if you need to.

### docker-compose.yml

```bash
cat <<EOF > docker-compose.yml
version: '3.7'

services:
  postgresql-server:
    image: postgres:latest
    container_name: postgresql-server
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgresql-data:/var/lib/postgresql/data

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:latest
    container_name: zabbix-server
    restart: unless-stopped
    depends_on:
      - postgresql-server
    environment:
      DB_SERVER_HOST: postgresql-server
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      ZBX_STARTVMWARECOLLECTORS: ${ZBX_STARTVMWARECOLLECTORS}
    ports:
      - "10051:10051"
    volumes:
      - zabbix-server-data:/var/lib/zabbix
      - zabbix-snmptraps-data:/var/lib/zabbix/snmptraps
      - zabbix-export-data:/var/lib/zabbix/export

  zabbix-web-nginx-pgsql:
    image: zabbix/zabbix-web-nginx-pgsql:latest
    container_name: zabbix-web
    restart: unless-stopped
    depends_on:
      - postgresql-server
      - zabbix-server
    environment:
      DB_SERVER_HOST: postgresql-server
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: ${PHP_TZ}
    ports:
      - "${ZABBIX_FRONTEND_PORT}:8080"
    volumes:
      - zabbix-web-data:/usr/share/zabbix

  zabbix-agent:
    image: zabbix/zabbix-agent:latest
    container_name: zabbix-agent
    restart: unless-stopped
    depends_on:
      - zabbix-server
    environment:
      ZBX_HOSTNAME: "zabbix-server"
      ZBX_SERVER_HOST: zabbix-server
      ZBX_SERVER_PORT: '10051'
      ZBX_SERVER_ACTIVE: zabbix-server

volumes:
  postgresql-data:
  zabbix-server-data:
  zabbix-snmptraps-data:
  zabbix-export-data:
  zabbix-web-data:
EOF
```

## Create the stack in Portainer

- Click `Stacks` > `Add stack`
- Supply a `Name` to your stack (e.g. zabbix)
- Click `Upload` and upload your `docker-compose.yml` and `zabbix.env` files that was created above
- Click `Deploy` the stack
- Wait a bit for it to finish
- Browse to `https://yourdockerhostip:8080`. Default creds: `Admin` / `zabbix`

**Done**