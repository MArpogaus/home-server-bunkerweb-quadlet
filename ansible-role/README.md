# Bunkerweb Service Ansible Role

Deploy Bunkerweb Quadlet services via Ansible.

## Variables

| Variable | Default | Description |
|---|---|---|
| `bunker_service_user` | `proxy` | Service account username |
| `bunker_service_home` | `/var/services/proxy` | Home directory |

## Usage

```yaml
# In your playbook
- hosts: all
  roles:
    - role: bunker_service
      bunker_service_user: proxy
      bunker_service_home: /var/services/proxy
```

## Files Deployed

- Quadlet container definitions (`.container`)
- Volume definitions (`.volume`)
- Network definitions (`.network`)

## Requirements

- Ansible 2.21+
- Target system with systemd user instances
- Quadlet support (Podman 4.0+)
