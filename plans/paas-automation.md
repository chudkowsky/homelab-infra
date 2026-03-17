# PaaS Automation

Automate the manual flow for adding a new service:

1. User provides a git repo URL
2. Script clones repo to homeserver
3. Cloudflare API creates DNS A record for new subdomain
4. Nginx server block generated from template on homeserver
5. Certbot issues TLS certificate
6. Docker Compose starts the service

**Constraint:** Only repos conforming to [repo-spec.md](./repo-spec.md) are supported.

**TODO:** Define Docker Compose structure convention (port exposure, required fields) before building automation. This will become the standard for all repos under `~/dev/piot/`.
