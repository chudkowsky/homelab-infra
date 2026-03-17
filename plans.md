# Plans & Migrations

---

## Migrate nginx to local machine

**Current:** VPS nginx handles per-domain server blocks and SSL termination.

**Target:**
```
Internet → VPS (TCP stream proxy, ports 80/443) → WireGuard → Local nginx (per-domain) → Services
```

**Benefit:** VPS becomes fully static — no changes needed there when adding new services. All automation runs on local machine only.

**Status:** ✅ Completed 2026-03-17.

---

### Step-by-step

#### 1. Local machine — recreate all server blocks

Copy/rewrite each existing VPS server block under `/etc/nginx/sites-available/` on the local machine.
Adjust `proxy_pass` to use `localhost` (or `127.0.0.1`) instead of `10.0.0.2`.

Current domains to migrate:

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

Run certbot for each domain (local machine must be reachable on port 80 — it will be after step 3, so use DNS challenge or do this after the VPS switch):

```bash
sudo certbot --nginx -d chudkowsky.com
sudo certbot --nginx -d quiz.chudkowsky.com
sudo certbot --nginx -d book.chudkowsky.com
```

> Alternatively: use `--nginx` after the VPS stream proxy is live, since port 80 will then reach local nginx.

#### 3. VPS — replace all site configs with a single stream proxy

Remove all per-domain configs:
```bash
sudo rm /etc/nginx/sites-enabled/*
sudo rm /etc/nginx/sites-available/*   # or archive them
```

Add stream block. Create `/etc/nginx/stream.d/proxy.conf`:
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

Enable the stream module in `/etc/nginx/nginx.conf` (add at top level, outside `http` block):
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

- Remove certbot and its timers from the VPS (optional but clean).
- Update `nginx.md` to reflect the new architecture.

---

## PaaS automation

Automate the manual flow for adding a new service:

1. User provides a git repo URL
2. Script clones repo to local machine
3. Cloudflare API creates DNS A record for new subdomain
4. Nginx server block generated from template on local machine
5. Certbot issues certificate
6. Docker Compose starts the service

**Constraint:** Only Docker Compose repos supported.

**Depends on:** nginx migration above being completed first.

**TODO:** Define Docker Compose structure convention (port exposure, required fields) before building automation. This will become the standard for all repos under `~/dev/piot/`.
