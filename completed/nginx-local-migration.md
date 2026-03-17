# Migrate nginx to local machine

**Completed:** 2026-03-17

**Before:** VPS nginx handled per-domain server blocks and SSL termination.

**After:**
```
Internet → VPS (TCP stream proxy, ports 80/443) → WireGuard → Local nginx (per-domain) → Services
```

**Benefit:** VPS is now fully static — config never changes when adding new services. All automation runs on local machine only.

---

### Step-by-step

#### 1. Local machine — recreate all server blocks

Copy/rewrite each existing VPS server block under `/etc/nginx/sites-available/` on the local machine.
Adjust `proxy_pass` to use `localhost` (or `127.0.0.1`) instead of `10.0.0.2`.

Domains migrated:

| Short name | Domain | Upstream port |
|------------|--------|---------------|
| `chudkowsky` | `chudkowsky.com` | 3000 |
| `quiz` | `quiz.chudkowsky.com` | 8001 |
| `book` | `book.chudkowsky.com` | 8002 |

Enable each:
```bash
sudo ln -s /etc/nginx/sites-available/NAME /etc/nginx/sites-enabled/NAME
```

Test config before touching the VPS:
```bash
sudo nginx -t
```

#### 2. Local machine — obtain SSL certificates

```bash
sudo certbot --nginx -d chudkowsky.com
sudo certbot --nginx -d quiz.chudkowsky.com
sudo certbot --nginx -d book.chudkowsky.com
```

#### 3. VPS — replace all site configs with a single stream proxy

Remove all per-domain configs:
```bash
sudo rm /etc/nginx/sites-enabled/*
sudo rm /etc/nginx/sites-available/*
```

Create `/etc/nginx/stream.d/proxy.conf`:
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

Enable the stream module in `/etc/nginx/nginx.conf` (at top level, outside `http` block):
```nginx
stream {
    include /etc/nginx/stream.d/*.conf;
}
```

Reload:
```bash
sudo nginx -t && sudo systemctl reload nginx
```

#### 4. Verify

- Hit each domain over HTTPS — cert should now be issued by local certbot.
- Check that `X-Real-IP` is preserved (nginx on local machine gets the real client IP via the stream proxy).

#### 5. Cleanup

Remove certbot and its timers from the VPS:
```bash
sudo systemctl stop certbot.timer certbot.service
sudo systemctl disable certbot.timer certbot.service
sudo apt purge certbot python3-certbot-nginx -y
sudo apt autoremove -y
sudo rm -rf /etc/letsencrypt
```
