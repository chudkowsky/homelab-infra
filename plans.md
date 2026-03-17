# Plans & Migrations

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
