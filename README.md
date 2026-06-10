# service-bunker

Bunkerweb (security-focused web server) deployment via Podman Quadlet.

## Structure

```
service-bunker/
├── containers/
│   ├── bunker-nginx/
│   │   └── Containerfile
│   └── bunker-scheduler/
│       └── Containerfile
├── quadlets/
│   ├── bunker-nginx.container
│   ├── bunker-scheduler.container
│   ├── volumes/
│   │   ├── bw-data.volume
│   │   └── bw-nginx-data.volume
│   └── networks/
│       └── shared-network.network
└── ansible-role/
    └── bunker_service/
        └── tasks/
            └── main.yml
```

## Container Build

Build Bunkerweb container images:

```bash
cd containers/bunker-nginx
podman build -t ghcr.io/your-org/bunker-nginx:latest .
podman push ghcr.io/your-org/bunker-nginx:latest

cd ../bunker-scheduler
podman build -t ghcr.io/your-org/bunker-scheduler:latest .
podman push ghcr.io/your-org/bunker-scheduler:latest
```

## Quadlet Services

- `bunker-nginx.container` - Main Bunkerweb nginx instance
- `bunker-scheduler.container` - Let's Encrypt renewal scheduler

### Volume Definitions

- `bw-data.volume` - Bunkerweb configuration and data
- `bw-nginx-data.volume` - Nginx-specific data (certificates, etc.)

### Network

`shared-network.network` - Bridge network (10.89.0.0/24) shared with Nextcloud.

## Ansible Deployment

Use the included Ansible role to deploy Quadlet services:

```yaml
- hosts: all
  roles:
    - role: bunker_service
      bunker_service_user: proxy
      bunker_service_home: /var/services/proxy
```

See `ansible-role/README.md` for details.

## Configuration

Bunkerweb configuration is managed via environment variables in `.env` files:

```bash
cp configs/bunkerized_nginx.env.example configs/bunkerized_nginx.env
# Edit with your domain and security settings
```

Key settings:
- `SERVER_NAME` - Your domain
- `REVERSE_PROXY_HOST` - Backend service (nextcloud-web)
- `AUTO_LETS_ENCRYPT` - Automatic SSL certificates
- `USE_MODSECURITY` - WAF protection

## Network Architecture

```
+------------------+
|   Internet       |
+------------------+
        |
        v
+------------------+
|  Bunkerweb       |
|  (nginx:443)     |
+------------------+
        |
        v
+------------------+
|  nextcloud-web   |
|  (nginx:80)      |
+------------------+
```

Bunkerweb acts as reverse proxy and security layer for Nextcloud.

## Security Features

- WAF (ModSecurity)
- Rate limiting
- Bot protection
- Automatic SSL (Let's Encrypt)
- Geo-blocking
- Header security

## Requirements

- Podman 4.0+ (Quadlet support)
- systemd user instances
- Btrfs filesystem (recommended for snapshots)

## License

MIT - See LICENSE file
