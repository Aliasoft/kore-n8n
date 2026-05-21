# kore-n8n

Self-hosted [n8n](https://n8n.io) instance running in Docker, exposed via a Traefik v2 reverse proxy with automatic Let's Encrypt TLS.

This repository is a clean, reusable foundation · bring your own Traefik setup and domain, configure `.env`, and you're live.

---

## Stack

| Component | Role |
|-----------|------|
| **n8n** | Workflow automation engine |
| **Docker + Compose** | Container runtime |
| **Traefik v2** | Reverse proxy, TLS termination (external) |
| **Let's Encrypt** | Automatic SSL certificates (via Traefik) |
| **SQLite** *(default)* | Embedded database · swap for PostgreSQL in production |

---

## Prerequisites

- Linux VPS with Docker ≥ 24 and Docker Compose v2
- Traefik v2 already running and attached to a Docker network named `traefik_public`
- A domain (or subdomain) with a DNS A record pointing to your VPS
- Traefik configured with a `letsencrypt` certificate resolver and a `websecure` entrypoint

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/<you>/kore-n8n.git
cd kore-n8n
```

### 2. Create your environment file

```bash
cp .env.example .env
```

Edit `.env` and set at minimum:

| Variable | Description |
|----------|-------------|
| `N8N_HOST` | Your subdomain, e.g. `n8n.yourdomain.com` |
| `WEBHOOK_URL` | Full URL: `https://n8n.yourdomain.com/` |
| `N8N_ENCRYPTION_KEY` | Random secret · generate with `openssl rand -hex 32` |
| `GENERIC_TIMEZONE` | IANA timezone, e.g. `Europe/Paris` |

> **Note:** `TRAEFIK_ENTRYPOINT` and `TRAEFIK_CERT_RESOLVER` must match the names defined in your Traefik static configuration.

### 3. Ensure the Traefik network exists

```bash
docker network create traefik_public 2>/dev/null || true
```

If your network has a different name, update `traefik_public` in `docker-compose.yml` and the `traefik.docker.network` label.

### 4. Start the stack

```bash
docker compose up -d
```

n8n is now accessible at `https://<N8N_HOST>`.

---

## Configuration

### Switching to PostgreSQL

Set `DB_TYPE=postgresdb` in `.env` and fill in all `DB_POSTGRESDB_*` variables. Add a `postgres` service to `docker-compose.yml` if you want it co-located, or point to an external database.

### Pinning the n8n version

Set `N8N_VERSION` to a specific tag (e.g. `1.90.0`) to avoid unexpected upgrades:

```env
N8N_VERSION=1.90.0
```

Available tags: https://hub.docker.com/r/n8nio/n8n/tags

### SMTP / email

Set `N8N_EMAIL_MODE=smtp` and configure the `N8N_SMTP_*` variables to enable password-reset emails and team invitations.

---

## Operations

### View logs

```bash
docker compose logs -f n8n
```

### Upgrade n8n

```bash
# Update the version pin in .env, then:
docker compose pull
docker compose up -d
```

### Backup

The `n8n_data` Docker volume holds all workflows, credentials, and the SQLite database.

```bash
# One-shot backup to a local tar archive
docker run --rm \
  -v kore-n8n_n8n_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/n8n_backup_$(date +%F).tar.gz -C /data .
```

### Stop / remove

```bash
docker compose down          # stop, keep volumes
docker compose down -v       # stop and delete volumes (destructive)
```

---

## Security checklist

- [ ] `N8N_ENCRYPTION_KEY` is set to a unique random value
- [ ] `.env` is never committed (covered by `.gitignore`)
- [ ] `N8N_BASIC_AUTH_ACTIVE=true` or user management is enabled before exposing publicly
- [ ] n8n version is pinned to a specific tag
- [ ] Regular backups of the `n8n_data` volume are in place
- [ ] Traefik access logs are enabled on your VPS

---

## License

MIT
