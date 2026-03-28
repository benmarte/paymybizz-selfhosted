# PayMyBizz — Self-Hosted

[![License: PolyForm Noncommercial](https://img.shields.io/badge/license-PolyForm%20Noncommercial-blue)](LICENSE)

Self-hosted freelance invoicing, time tracking, and business management.

> No source code required — runs entirely from pre-built Docker images.
>
> **Personal use only.** Commercial use requires a license — contact info@paymybizz.com.

## Requirements

- [Docker](https://docs.docker.com/get-docker/) & Docker Compose

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/benmarte/paymybizz-selfhosted.git
cd paymybizz-selfhosted

# 2. Configure your environment
cp .env.example .env
# Edit .env — defaults work out of the box for local access

# 3. Start everything
docker compose up -d

# 4. Visit the app
open http://localhost:8177
```

The first run takes a minute — the init container pushes Convex functions and sets up authentication automatically.

## URLs

| Service | URL |
|---------|-----|
| App | http://localhost:8177 |
| Convex Backend | http://localhost:3210 |
| Convex Dashboard | http://localhost:6791 |

## Remote Access (Cloudflare Tunnel)

To access from your phone or outside your local network:

1. Create a free tunnel at [Cloudflare Zero Trust](https://one.dash.cloudflare.com/) → Networks → Tunnels
2. Add three public hostnames pointing to `frontend:3000`, `backend:3210`, and `backend:3211`
3. Add these to your `.env`:

```env
TUNNEL_TOKEN=your-token-here
CONVEX_CLOUD_ORIGIN=https://api.yourdomain.com
CONVEX_SITE_ORIGIN=https://site.yourdomain.com
NEXT_PUBLIC_DEPLOYMENT_URL=https://api.yourdomain.com
AUTH_SITE_URL=https://app.yourdomain.com
```

4. Start with the tunnel profile:

```bash
docker compose --profile tunnel up -d
```

## Updating

```bash
docker compose pull
docker compose up -d
```

## Stopping

```bash
# Stop containers (keeps data)
docker compose down

# Stop and wipe all data
docker compose down -v
```

## Configuration

All configuration is in `.env`. See `.env.example` for all available options including:

- Google OAuth (optional)
- Email via Gmail or Resend (optional)
- Stripe invoice payments (optional)
- Stripe SaaS billing (optional, for multi-tenant deployments)

## Source Code

The source code for PayMyBizz lives at [github.com/benmarte/paymybizz](https://github.com/benmarte/paymybizz) (private). Docker images are published automatically to `ghcr.io/benmarte/paymybizz` on every release.
