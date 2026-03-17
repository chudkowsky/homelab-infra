# Plans & Migrations

---

## Migrate nginx to local machine

**Current:** VPS nginx handles per-domain server blocks and SSL termination.

**Target:**
```
Internet → VPS (TCP stream proxy, ports 80/443) → WireGuard → Local nginx (per-domain) → Services
```

**Changes required:**
- Replace VPS nginx config with a static stream block (one-time, never changes again)
- Install and configure nginx on local machine
- Move all per-domain server blocks to local machine
- Move certbot to local machine

**Benefit:** VPS becomes fully static — no changes needed there when adding new services. All automation runs on local machine only.

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
