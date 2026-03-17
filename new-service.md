# Adding a New Service

## With Cloudflare domain (chudkowsky.com)

1. Add a DNS **A record** on Cloudflare pointing the new subdomain to `178.104.50.250`, proxy status **grey cloud (DNS only)**
2. Copy the HTTP-only nginx template on the VPS, update `server_name` and port:
```bash
sudo cp /etc/nginx/sites-available/chudkowsky /etc/nginx/sites-available/SHORTNAME
# edit the file, then:
sudo ln -s /etc/nginx/sites-available/SHORTNAME /etc/nginx/sites-enabled/SHORTNAME
sudo systemctl reload nginx
```
3. Clone the repo and start the service on the homeserver:
```bash
cd ~/dev/chudas  # or ~/dev/piot
git clone GIT_URL
cd REPO_NAME
docker compose up -d
```
4. _(Optional)_ Run certbot: `sudo certbot --nginx -d SUBDOMAIN.chudkowsky.com`

See [nginx.md](./nginx.md) for config templates.

---

## Without a custom domain (DuckDNS)

See [duckdns-flow.md](./duckdns-flow.md).
