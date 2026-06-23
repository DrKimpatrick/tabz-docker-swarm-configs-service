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

## `ms-notification-tabz` (`.env-notification`)

The **`notification`** service in `docker-compose.staging.yml` loads **`.env-notification`**. Copy **`.env-notification.example`** for a blank template.

### Swarmbyte jackpot collections

| Variable | Purpose |
|----------|---------|
| `COLLECTION_PAYMENT_PROVIDER` | `SWARMBYTE` (default), `NEXUMPAY`, or `DMARK` |
| `SWARMBYTE_BASE_URL` | Swarmbyte API host (e.g. `https://stg-api.swarmbyte.com`) |
| `SWARMBYTE_CLIENT_ID` / `SECRET` / `WALLET_ADDRESS` | **Primary** merchant (jackpot pool) |
| `SWARMBYTE_BACKUP_CLIENT_ID` / `SECRET` / `WALLET_ADDRESS` | **Backup** merchant (separate account; all three required for routing) |
| `SWARMBYTE_WALLET_ROUTING_ENABLED` | `true` — per-USSD-code primary % caps; `false` — always primary |
| `API_BASE_URL_PRIMARY` / `SWARMBYTE_WEBHOOK_PUBLIC_BASE_URL` | Public HTTPS origin for `POST /v1/collections/swarmbyte/callback` |

Primary and backup each use their **own OAuth credentials**. Status polling uses the same merchant as the original collect.

**Webhooks:** set **`API_BASE_URL_PRIMARY=https://ussd.tabzmedia.org`** (and/or `SWARMBYTE_WEBHOOK_PUBLIC_BASE_URL`) so `POST /v1/collect` sends a `webhookUrl` that reaches this service.

**Nginx:** **`ussd.tabzmedia.org`** proxies `/` to the notification container; **`api.tabzmedia.org`** `/` goes to the main Tabz API — Swarmbyte must **not** use `api.tabzmedia.org` for `/v1/collections/swarmbyte/callback` (404).

**Enable backup routing:** fill all three `SWARMBYTE_BACKUP_*` vars, then set `SWARMBYTE_WALLET_ROUTING_ENABLED=true`, redeploy stack:

```bash
docker stack deploy --with-registry-auth -c docker-compose.staging.yml tabz-media-services
```

