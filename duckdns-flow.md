# Adding a New Service with DuckDNS (No Custom Domain)

For cases where a custom domain is not available. DuckDNS provides free subdomains under `duckdns.org`.

---

## Prerequisites

- DuckDNS account at [duckdns.org](https://www.duckdns.org)
- A claimed subdomain e.g. `yourname.duckdns.org`

---

## Flow

### 1. Point DuckDNS subdomain to VPS IP

Log in to DuckDNS dashboard and set the subdomain's IP to the VPS public IP.

### 2. Add nginx server block on VPS

Use the same HTTP-only template as for custom domains. Name the config file with a short name, not the full domain.

```bash
sudo cp /etc/nginx/sites-available/chudkowsky /etc/nginx/sites-available/SHORTNAME
# edit the file, set server_name to YOURSUBDOMAIN.duckdns.org and proxy_pass port, then:
sudo ln -s /etc/nginx/sites-available/SHORTNAME /etc/nginx/sites-enabled/SHORTNAME
sudo systemctl reload nginx
```

### 3. Start the service on local machine

```bash
cd ~/dev/chudas  # or ~/dev/piot
git clone GIT_URL
cd REPO_NAME
docker compose up -d
```

### 4. _(Optional)_ Issue SSL certificate with Certbot

Same command as for custom domains:

```bash
sudo certbot --nginx -d YOURSUBDOMAIN.duckdns.org
```

---

## Notes

- DuckDNS subdomains are `*.duckdns.org` — no custom branding
- The DuckDNS IP needs to be updated if the VPS IP changes (not an issue with Hetzner static IPs)
