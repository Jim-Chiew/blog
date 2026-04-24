---
title: Karakeep - install on Portainer
description:
draft: false
tags:
  - docker
created: 23/44/2025 19:44
updated: 23/09/2026 21:09
---
This installation took longer then expected because of the fact that I refuse to use docker compose as I have set up Portainer and wanted to manage all my docker containers from there. 

# Compose File: The smallest change makes a big difference
In the portainers web UI. go to `Stacks` -> `Add stack`.


To run the recommended compose file on Portainers, just change `.env` from any `env_file` sections to `stack.env`.

NOTE: running as a specific user `user: "1000:1000"`
NOTE: Bind mounted `/home/user/karakeep:/data`.
```YAML
services:
  web:
    image: ghcr.io/karakeep-app/karakeep:latest
    restart: unless-stopped
    user: "1000:1000"
    volumes:
      # By default, the data is stored in a docker volume called "data".
      # If you want to mount a custom directory, change the volume mapping to:
      # - /path/to/your/directory:/data
      - /home/user/karakeep:/data
    ports:
      - 3000:3000
    env_file:
      - stack.env
    environment:
      MEILI_ADDR: http://meilisearch:7700
      BROWSER_WEB_URL: http://chrome:9222
      # OPENAI_API_KEY: ...

      # You almost never want to change the value of the DATA_DIR variable.
      # If you want to mount a custom directory, change the volume mapping above instead.
      DATA_DIR: /data # DON'T CHANGE THIS
  chrome:
    image: gcr.io/zenika-hub/alpine-chrome:123
    restart: unless-stopped
    command:
      - --no-sandbox
      - --disable-gpu
      - --disable-dev-shm-usage
      - --remote-debugging-address=0.0.0.0
      - --remote-debugging-port=9222
      - --hide-scrollbars
  meilisearch:
    image: getmeili/meilisearch:v1.13.3
    restart: unless-stopped
    env_file:
      - stack.env
    environment:
      MEILI_NO_ANALYTICS: "true"
    volumes:
      - meilisearch:/meili_data

volumes:
  meilisearch:
```
# Adding the environments: 
Either manually enter the require field or upload the .env file in Portainers ENV variable: 
![[karakeep env.png]]

```.env
DATA_DIR=/data
MEILI_ADDR=http://127.0.0.1:7700
MEILI_MASTER_KEY=SUPER_SUPER_SECRET_STRING
[generate with <openssl rand -base64 36 | tr -dc 'A-Za-z0-9'>]
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=SUPER_SECRET_STRING
[generate with <openssl rand -base64 36>]
```

For `MEILI_MASTER_KEY` use the following command in a separate terminal:
```bash
openssl rand -base64 36 | tr -dc 'A-Za-z0-9'
```

for `NEXTAUTH_SECRET`, use:
```bash
openssl rand -base64 36
```

# Deploy:
Finally hit deploy and you should have karakeep. running. 

Don't forget to give your stack a name.


# Resource:
https://github.com/karakeep-app/karakeep/blob/main/docker/docker-compose.yml
https://github.com/karakeep-app/karakeep/blob/main/docker/.env.sample

https://docs.karakeep.app/installation/docker/
https://docs.karakeep.app/configuration/

https://docs.portainer.io/user/docker/stacks/add#environment-variables
https://docs.portainer.io/faqs/troubleshooting/stacks-deployments-and-updates/environment-variable-management-in-docker-.env-vs.-stack.env
