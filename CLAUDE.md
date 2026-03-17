# CLAUDE.md — Homelab Context

This repo documents a homelab built by Mateusz Chudkowski. Read this before making any suggestions about infrastructure, deployment, or networking.

---

## Architecture

```
Internet → VPS (nginx TCP stream proxy, ports 80/443) → WireGuard → Local nginx (per-domain, TLS) → Services
```

- **VPS** (Hetzner, Ubuntu 24.04): public entry point, runs nginx as a static TCP stream proxy — config never needs to change when adding services
- **Homeserver** (Ubuntu 24.04, i3-8100): runs nginx with per-domain server blocks, handles TLS via certbot, runs all services, sits behind CGNAT (no public IP)
- **WireGuard**: bridges VPS and homeserver; homeserver also peers with Piotr's machine

---

## Network

| Machine | WireGuard IP |
|---------|-------------|
| VPS | `10.0.0.1` |
| Homeserver | `10.0.0.2` |
| Piotr's machine | `10.0.0.3` |

- WireGuard interface: `wg0`, port `51820`
- VPS public IP: `178.104.50.250`
- Domain: `chudkowsky.com` — registered and DNS-managed on Cloudflare

---

## Services & Ports

| Domain | Port | Repo |
|--------|------|------|
| `chudkowsky.com` | 3000 | [chudkowsky/personal-page](https://github.com/chudkowsky/personal-page) |
| `quiz.chudkowsky.com` | 8001 | [chudkowsky/interview](https://github.com/chudkowsky/interview) |
| `book.chudkowsky.com` | 8002 | [chudkowsky/howcryptoworksbook](https://github.com/chudkowsky/howcryptoworksbook) |
| Minecraft | 25565 | — running directly on host in tmux, whitelist-guarded |

Next available port: `8003`

---

## Key Conventions

- Nginx configs on **homeserver**: `/etc/nginx/sites-available/SHORTNAME` (no domain extension), proxy to `localhost:PORT`
- VPS nginx: static TCP stream proxy only — `/etc/nginx/stream.d/proxy.conf`, forwards ports 80/443 to `10.0.0.2`
- New services start with HTTP-only nginx config on homeserver; certbot (`sudo certbot --nginx -d DOMAIN`) is optional upgrade
- No custom domain: use DuckDNS, same certbot command applies
- All web services run via Docker Compose
- Repos are cloned under `~/dev/chudas/` (Mateusz's projects) or `~/dev/piot/` (Piotr's projects — currently empty)
- Docker Compose structure convention is TBD, will be defined during PaaS automation build

---

## Docs

- [nginx.md](./nginx.md) — config templates, certbot, VPS stream proxy setup
- [wireguard.md](./wireguard.md) — topology, peer configs, key rotation
- [new-service.md](./new-service.md) — manual flow for adding a service
- [duckdns-flow.md](./duckdns-flow.md) — flow for services without a custom domain
- [plans/](./plans/) — future migrations and automation plans
- [completed/](./completed/) — completed migrations with full step-by-step records
