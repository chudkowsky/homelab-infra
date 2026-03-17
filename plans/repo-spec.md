# Repo Compatibility Spec

Defines the contract a repository must meet to be deployable by the PaaS automation.

---

## Requirements

### 1. `docker-compose.yml` at repo root

Required. The automation reads this file to understand how to build and run the service.

### 2. Single service (iteration 1)

Only one service in `docker-compose.yml` is supported for now. Multi-container setups (e.g. app + database) are not yet supported.

### 3. Build or image

The service must declare either:

- `build: .` — requires a `Dockerfile` at repo root
- `image: <name>` — must reference a trusted registry (Docker Hub official images or `ghcr.io/chudkowsky/*`)

### 4. Port label

The service must declare the internal port it listens on:

```yaml
services:
  app:
    build: .
    labels:
      homelab.port: "3000"
```

The automation reads this label to assign a host port and generate the nginx config.

### 5. Environment variables (optional)

If the service requires env vars:

- Ship a `.env.example` listing required variable names (no values)
- The user provides a `.env` file before deploying

**Security rules enforced by the automation:**
- Env var names must match `^[A-Z_][A-Z0-9_]*$` — file is rejected otherwise
- Env vars are passed to Docker Compose via `--env-file` only — never interpolated into shell commands

---

## Minimal valid example

```yaml
# docker-compose.yml
services:
  app:
    build: .
    labels:
      homelab.port: "3000"
```

```dockerfile
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm ci
CMD ["node", "server.js"]
```
