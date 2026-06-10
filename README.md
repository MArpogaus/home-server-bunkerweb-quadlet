# service-bunker

Bunkerweb reverse proxy v1.6.11 via Podman Quadlet.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Bunkerweb Service (Quadlet)                        │
│                                                     │
│  bunker-nginx     ─ Bunkerweb nginx (v1.6.11)       │
│  bunker-scheduler ─ Bunkerweb scheduler             │
│                                                     │
│  Network: shared-network (bridge 10.89.0.0/24)      │
│  Ports: 80:8080/tcp, 443:8443/tcp                   │
│  Proxy: cloud.arpogaus.de → http://nextcloud-web:80  │
└─────────────────────────────────────────────────────┘
```

## Quick Start

1. Copy `.env.example` to `.env` and fill in settings:
   ```bash
   cp .env.example .env
   ```

2. Deploy Quadlet files to systemd user directory:
   ```bash
   cp containers/* ~/.config/systemd/user/
   cp volumes/* ~/.config/systemd/user/
   cp networks/*.network ~/.config/systemd/user/
   ```

3. Reload and enable:
   ```bash
   systemctl --user daemon-reload
   systemctl --user enable --now bunker-nginx bunker-scheduler
   ```

4. Verify:
   ```bash
   podman ps
   systemctl --user list-units --type=service | grep bunker
   ```

## Reverse Proxy Targets

| Host | Upstream | Rate Limits |
|---|---|---|
| `cloud.arpogaus.de` | `http://nextcloud-web:80` (Nextcloud) | apps: 5r/s, preview: 5r/s, push: 8r/s, memories/api: 8r/s |

## Environment Variables

| Variable | Description | Example |
|---|---|---|
| `SERVER_NAME` | Domain name | `cloud.arpogaus.de` |
| `AUTO_LETS_ENCRYPT` | Enable Let's Encrypt | `yes` |
| `WHITELIST_COUNTRY` | Allowed countries | `DE CH AT` |
| `LIMIT_REQ_RATE` | Default rate limit | `3r/s` |
| `API_WHITELIST_IP` | Trusted IPs | `127.0.0.1 10.0.0.0/8` |

## Volumes

| Volume | Purpose | SELinux |
|---|---|---|
| `bw-nginx-data` | Bunkerweb nginx config | `:Z` |
| `bw-data` | Bunkerweb data | `:Z` |

## Notes

- Uses official Bunkerweb images (no custom build needed)
- Rate limiting configured per URL path in `bunkerized_nginx.env`
- JSON analytics logging via mounted `json_analytics.conf`
