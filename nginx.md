# Nginx

## Overview

Two nginx instances — one on VPS, one on homeserver.

- **VPS nginx**: static TCP stream proxy. Forwards ports 80 and 443 to homeserver over WireGuard. Config never changes when adding new services.
- **Local nginx**: handles all per-domain server blocks and TLS termination via certbot.

```
Internet → VPS nginx (stream proxy) → 10.0.0.2 → Local nginx → localhost:PORT
```

---

## VPS — Stream Proxy

**Config file:** `/etc/nginx/stream.d/proxy.conf`

```nginx
server {
    listen 80;
    proxy_pass 10.0.0.2:80;
}

server {
    listen 443;
    proxy_pass 10.0.0.2:443;
}
```

The `stream` module is loaded via `/etc/nginx/modules-enabled/` (package: `libnginx-mod-stream`).

`/etc/nginx/nginx.conf` must include at top level (outside `http` block):
```nginx
stream {
    include /etc/nginx/stream.d/*.conf;
}
```

**Reload:** `sudo systemctl reload nginx`

---

## Homeserver — Per-domain Config

| Property | Value |
|----------|-------|
| Config directory | `/etc/nginx/sites-available/` |
| Enabled directory | `/etc/nginx/sites-enabled/` |
| Reload command | `sudo systemctl reload nginx` |
| Naming convention | Short name without domain extension (e.g. `chudkowsky`, `quiz`, `book`) |

### Server Block Template (HTTP only)

```nginx
server {
    listen 80;
    server_name DOMAIN;

    location / {
        proxy_pass http://localhost:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Adding TLS

Start with the HTTP-only config, then run certbot:

```bash
sudo certbot --nginx -d DOMAIN
```

Certbot modifies the config to add TLS and HTTP→HTTPS redirect. Renewal is handled automatically via systemd timer:

```bash
sudo systemctl status certbot.timer
```
