# WAHA (WhatsApp admin alerts)

WAHA runs WhatsApp Web in Docker and gives Laravel an HTTP API to send alerts.
It is used by the `gods` Laravel project, whose settings live in
Admin → My Apps → WhatsApp Notification.

`.env` (API key, dashboard password) and `sessions/` (the linked WhatsApp
login) are git-ignored. Never commit them.

## Local (Mac, Apple Silicon)

```bash
cd ~/code/waha
docker run --rm -v "$(pwd)":/app/env devlikeapro/waha:arm init-waha /app/env
echo 'WAHA_IMAGE=devlikeapro/waha:arm' >> .env
# in .env, change WAHA_BASE_URL to http://127.0.0.1:3010
docker compose -f compose.local.yml up -d
```

- Joins the `code_laravel` Docker network (override with `LARAVEL_NETWORK=...`).
- Admin settings: **WAHA Server URL** `http://waha:3000`, **API Key** = `WAHA_API_KEY` from `.env`.
- Dashboard (optional): http://127.0.0.1:3010/dashboard/ (use `127.0.0.1`, not
  `localhost`: the port is only bound on IPv4).

## Production (DigitalOcean droplet, x86)

1. Point a DNS A record (e.g. `waha.yourdomain.com`) at the droplet.
2. Copy this folder (without `.env` and `sessions/`) to `~/waha` on the droplet, then:

```bash
cd ~/waha
docker run --rm -v "$(pwd)":/app/env devlikeapro/waha init-waha /app/env
echo 'WAHA_DOMAIN=waha.yourdomain.com' >> .env
# in .env, change WAHA_BASE_URL to https://waha.yourdomain.com
docker compose -f compose.prod.yml up -d
ufw allow OpenSSH && ufw allow 80 && ufw allow 443 && ufw --force enable
```

- Caddy gets the HTTPS certificate automatically. WAHA itself is not exposed.
- Admin settings: **WAHA Server URL** `https://waha.yourdomain.com`, **API Key** = `WAHA_API_KEY` from the droplet's `.env`.

## Linking the sender number

In the admin page, under **Sender Number**, press **Connect** and scan the QR
code from a spare WhatsApp number (Settings → Linked devices). Do not use the
main business number: WAHA is unofficial and WhatsApp may ban it.
