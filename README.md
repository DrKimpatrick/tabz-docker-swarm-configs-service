# kps-docker-swarm-configs

Repository for docker swarm configs for the kapeesa backend micro-services

# to access private docker registry images here are the steps

- Login to the private registry and the credentials will be saved in the docker config file

e.g login to github docker registry

# To successfully deploy the stack

- use the following command

```
    docker stack deploy --with-registry-auth -c <compose_file> <stack_name>
    sudo docker stack deploy --with-registry-auth -c docker-compose.yml tabz-media-services
    docker stack deploy --with-registry-auth -c docker-compose.prod.yml tabz-media-services
    docker stack deploy --with-registry-auth -c docker-compose.staging.yml tabz-media-services
```

when using docker stack deploy in a swarm the `--with-registry-auth` option will forward the login information to other node

## SwarmByte jackpot webhooks (`ms-notification-tabz`)

- **`API_BASE_URL_PRIMARY=https://ussd.tabzmedia.org`** in `.env-notification` so `POST /v1/collect` sends a `webhookUrl` that reaches the notification service.
- Nginx: **`ussd.tabzmedia.org`** proxies `/` to the notification container; **`api.tabzmedia.org`** `/` goes to the main Tabz API — SwarmByte must **not** use `api.tabzmedia.org` for `/v1/collections/swarmbyte/callback` or you get **404**.
- Set **`SWARMBYTE_CALLBACK_SECRET`** (or Swarm secret) to match the merchant **`callbackSecret`** in SwarmByte Payments when verifying `X-Swarmbyte-Signature`.

