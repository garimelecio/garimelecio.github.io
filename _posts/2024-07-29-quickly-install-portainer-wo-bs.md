---
title: Quickly install Portainer without the BS
date: 2024-07-29 20:10:00 +0800
categories: [howtos]
tags: [portainer,docker,linux]
---

## Create the docker volume

```bash
docker volume create portainer_data
```

## Install Portainer

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

## Finishing

Access your new Portainer at `https://yourdockerhostip:8080` and create your creds.

**Done**