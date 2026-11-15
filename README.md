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
| `REDIS_URL` | `redis://redis:6379` — durable BullMQ queues for USSD collect jobs during traffic spikes |
| `MONGO_MAX_POOL_SIZE` | `100` recommended in production (was 30) |
| `MONGO_MIN_POOL_SIZE` | `10` recommended in production |

Primary and backup each use their **own OAuth credentials**. Status polling uses the same merchant as the original collect.

**Webhooks:** set **`API_BASE_URL_PRIMARY=https://ussd.tabzmedia.org`** (and/or `SWARMBYTE_WEBHOOK_PUBLIC_BASE_URL`) so `POST /v1/collect` sends a `webhookUrl` that reaches this service.

**Nginx:** **`ussd.tabzmedia.org`** proxies `/` to the notification container; **`api.tabzmedia.org`** `/` goes to the main Tabz API — Swarmbyte must **not** use `api.tabzmedia.org` for `/v1/collections/swarmbyte/callback` (404).

**Enable backup routing:** fill all three `SWARMBYTE_BACKUP_*` vars, then set `SWARMBYTE_WALLET_ROUTING_ENABLED=true`, redeploy stack:

```bash
docker stack deploy --with-registry-auth -c docker-compose.staging.yml tabz-media-services
```

### Swarmbyte payouts (jackpot winners and awards)

Money goes out from the **primary** jackpot merchant (`SWARMBYTE_CLIENT_ID` / `SECRET` / `WALLET_ADDRESS`) through `POST /v1/disburse`.

| Variable | Purpose |
|----------|---------|
| `SWARMBYTE_DISBURSEMENT_PIN` | Disbursement PIN set in the Swarmbyte merchant dashboard. **Empty = every payout is refused.** |
| `PAYOUT_TRACKING_START` | Winners from before this time (default `2026-11-15T00:00:00+03:00`) are not shown as pending and cannot be paid from Bingo; admins can mark them as paid elsewhere. |
| `PAYOUT_ALERT_NUMBERS` | Comma-separated numbers that get an SMS for every payout result |
| `PAYOUT_AWARD_MAX_PER_PAYOUT` / `PAYOUT_AWARD_MAX_PER_DAY` | Limits for ad-hoc awards (default 500,000 / 3,000,000 UGX) |
| `MONGO_AUTO_INDEX` | Must be exactly `true` (no comment on the same line) so the "one payout per winner" unique index exists |

**Swarmbyte dashboard, primary merchant:** enable *API disbursements*, set the disbursement PIN, and add the server's public IP to the IP allowlist (an empty allowlist returns 403).

**Webhook:** payouts report back to `https://ussd.tabzmedia.org/v1/disbursements/swarmbyte/callback` (same `ussd.tabzmedia.org` host as collections; no nginx change needed). Each callback is re-checked with Swarmbyte's status API, and a 10-second status poll covers missed callbacks.

**Safety:** a payout is sent once and never retried. A failed payout stays failed; an unclear result is marked `UNKNOWN` and must be checked in the Swarmbyte dashboard.
