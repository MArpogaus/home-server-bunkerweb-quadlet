# service-bunker

Bunkerweb — a security-focused web server and reverse proxy — deployed via Podman Quadlet.
Runs rootless under the `proxy` user.

## Structure

```
service-bunker/
├── quadlets/
│   ├── bunker-nginx.container       # Main Bunkerweb nginx instance
│   ├── bunker-scheduler.container   # Let's Encrypt renewal scheduler
│   ├── bw-data.volume               # Bunkerweb data volume
│   ├── bw-nginx-data.volume         # Nginx data volume (certs, etc.)
│   ├── shared-network.network       # Bridge network (10.89.0.0/24)
│   ├── configs/                     # Environment file examples
│   │   ├── bunkerized_nginx.env.example
│   │   ├── json_analytics.env.example
│   │   └── promtail-proxy.yaml
├── ansible-role/bunker_service/
│   ├── defaults/main.yml
│   ├── tasks/main.yml
│   └── templates/
│       ├── bunkerized_nginx.env.j2
│       └── json_analytics.env.j2
├── .env.example                     # Root env reference
└── LICENSE
```

## Quadlet Services

Both containers use `shared-network` (Podman bridge, `10.89.0.0/24`) to communicate with
other services (e.g. Nextcloud) on the same network.

| Service | Image | Purpose |
|---|---|---|
| `bunker-nginx` | `bunkerity/bunkerweb:1.6.11` | Reverse proxy, WAF, rate limiting, TLS termination |
| `bunker-scheduler` | `bunkerity/bunkerweb-scheduler:1.6.11` | Let's Encrypt renewal, config generation |

### Volumes

- `bw-data` — persistent Bunkerweb data (configs, cache)
- `bw-nginx-data` — nginx certificates and runtime data

## Configuration

Environment is supplied via `EnvironmentFile=` in the Quadlet unit files.
Copy the examples and edit:

```bash
cp quadlets/configs/bunkerized_nginx.env.example quadlets/configs/bunkerized_nginx.env
cp quadlets/configs/json_analytics.env.example quadlets/configs/json_analytics.env
```

Key variables in `bunkerized_nginx.env`:

| Variable | Description |
|---|---|
| `SERVER_NAME` | Your domain |
| `AUTO_LETS_ENCRYPT` | Automatic Let's Encrypt TLS |
| `LIMIT_REQ_RATE` | Rate-limit requests |
| `USE_MODSECURITY` | Enable ModSecurity WAF |
| `USE_REVERSE_PROXY` | Reverse proxy mode |
| `<domain>_REVERSE_PROXY_HOST` | Upstream backend |

## Ansible Deployment

The included Ansible role deploys the Quadlet files, templates environment configs, and starts
the services via `systemctl --user`.

Example play:

```yaml
- hosts: all
  roles:
    - role: bunker_service
      bunker_service_user: proxy
      bunker_service_home: /var/services/proxy
```

The role expects these variables (typically from `group_vars` or `secrets/vars.yml`):

| Variable | Default |
|---|---|
| `bunker_server_name` | (required) |
| `bunker_auto_lets_encrypt` | `yes` |
| `bunker_limit_req_rate` | `3r/s` |
| `bunker_whitelist_country` | `DE CH AT` |
| `bunker_api_whitelist_ip` | `127.0.0.1 10.0.0.0/8` |
| `bunker_use_modsecurity` | `no` |
| `bunker_max_client_size` | `10G` |

## Architecture

```
  Internet
     |
     v
  Bunkerweb (443)    ← TLS termination, WAF, rate limiting
     |
     v
  nextcloud-web (80) ← backend upstream
```

## Requirements

- Podman 4.0+ (Quadlet support)
- systemd user instances
- `shared-network` already created (`podman network create shared-network`)

## License

MIT — see [LICENSE](LICENSE).
