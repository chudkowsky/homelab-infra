# Nginx

## Overview

Nginx runs on the VPS and acts as a reverse proxy. Each domain has its own server block config. Traffic is forwarded through the WireGuard tunnel to the service running on the local machine.

---

## Config Structure

| Property | Value |
|----------|-------|
| Config directory | `/etc/nginx/sites-available/` |
| Enabled directory | `/etc/nginx/sites-enabled/` |
| Reload command | `sudo systemctl reload nginx` |
| Naming convention | Short name without domain extension (e.g. `chudkowsky`, `quiz`, `book`) |

---

## Server Block Templates

### Default (HTTP only)

Always start with this. Sufficient for internal tools or when SSL is not needed.

```nginx
server {
    listen 80;
    server_name DOMAIN;

    location / {
        proxy_pass http://10.0.0.2:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### With SSL (optional)

Start with the HTTP-only config, then run certbot with the `--nginx` flag. Certbot will automatically modify the config to add SSL and HTTP→HTTPS redirect.

```bash
sudo certbot --nginx -d DOMAIN
```

Renewal is handled automatically via systemd timer:

```bash
sudo systemctl status certbot.timer
```

