# PayMyBizz — Self-Hosted

<p align="center">
  <img src="paymybizz.svg#gh-light-mode-only" alt="PayMyBizz" width="200">
  <img src="paymybizz-white.svg#gh-dark-mode-only" alt="PayMyBizz" width="200">
</p>

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

## Convex Dashboard

The Convex Dashboard lets you browse your database, inspect function logs, run queries, and manage environment variables.

It's available at **http://localhost:6791** while the stack is running.

> **Do not expose port 6791 publicly.** It provides full read/write access to your database with no additional authentication.

### Logging in

The dashboard requires an admin key. Retrieve it after the first run:

```bash
docker run --rm -v paymybizz-selfhosted_convex_keys:/keys alpine cat /keys/admin-key
```

Open **http://localhost:6791**, enter `http://localhost:3210` as the deployment URL, and paste the admin key.

### Accessing the dashboard remotely (SSH tunnel)

If your server doesn't have a browser, forward the port over SSH:

```bash
ssh -L 6791:localhost:6791 user@your-server
```

Then open **http://localhost:6791** in your local browser.

### Accessing via Cloudflare Access (optional)

If you use a Cloudflare Tunnel and want dashboard access without SSH:

1. In Cloudflare Zero Trust → Access → Applications, add a new application
2. Point it at `http://localhost:6791` (or `dashboard:6791` internally)
3. Add an access policy requiring your email — this gates the dashboard behind Cloudflare login

---

## Persisting Data

A `docker-compose.override.yml` is included that mounts all data to `./data` on your local filesystem. Just create the directories before the first run:

```bash
mkdir -p data/convex data/keys data/auth-keys
docker compose up -d
```

Your data will now survive restarts and `docker compose down -v`.

---

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

All configuration is in `.env`. See `.env.example` for all available options.

---

## Optional Features

### Google Sign-In & Calendar Sync

Google OAuth enables two things: **sign in with Google** and **Google Calendar sync** for time entries.

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → Create a new project
2. Enable the **Google Calendar API** (APIs & Services → Library)
3. Go to APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID
4. Application type: **Web application**
5. Add these **Authorized redirect URIs** (adjust domains for your setup):

   ```
   # Auth login callback (Convex site proxy port)
   http://localhost:3211/api/auth/callback/google

   # Calendar sync callback (frontend port)
   http://localhost:8177/api/google-calendar/callback
   ```

   If using a Cloudflare Tunnel, also add:
   ```
   https://site.yourdomain.com/api/auth/callback/google
   https://app.yourdomain.com/api/google-calendar/callback
   ```

6. Copy the Client ID and Secret into `.env`:

   ```env
   GOOGLE_CLIENT_ID=your-client-id
   GOOGLE_CLIENT_SECRET=your-client-secret
   ```

7. Restart the stack: `docker compose up -d`

---

### Email

Email is used to send invoice PDFs and password reset links. Choose one provider:

**Option A — Gmail**

1. Enable 2-Step Verification on your Google account
2. Go to Google Account → Security → App Passwords → generate a password for "Mail"
3. Add to `.env`:

   ```env
   GMAIL_USER=you@gmail.com
   GMAIL_APP_PASSWORD=your-16-char-app-password
   ```

**Option B — Resend**

1. Sign up at [resend.com](https://resend.com) and create an API key
2. Verify your sending domain in the Resend dashboard
3. Add to `.env`:

   ```env
   RESEND_API_KEY=re_xxxxx
   RESEND_FROM=invoices@yourdomain.com
   ```

Restart after changes: `docker compose up -d`

---

### Stripe Invoice Payments

Enables a **Pay Now** button on invoices so clients can pay by card.

1. Create or log into your [Stripe Dashboard](https://dashboard.stripe.com/)
2. Copy your API keys (Developers → API keys) into `.env`:

   ```env
   STRIPE_SECRET_KEY=sk_live_xxxxx
   STRIPE_PUBLISHABLE_KEY=pk_live_xxxxx
   ```

3. Create a webhook (Developers → Webhooks → Add endpoint):
   - **Endpoint URL**: `https://site.yourdomain.com/api/stripe` (must be publicly accessible)
   - **Events to listen for**: `checkout.session.completed`

4. Copy the webhook signing secret into `.env`:

   ```env
   STRIPE_WEBHOOK_SECRET=whsec_xxxxx
   ```

5. Restart: `docker compose up -d`

> For local testing, use the [Stripe CLI](https://stripe.com/docs/stripe-cli) to forward webhook events: `stripe listen --forward-to localhost:3211/api/stripe`

## Source Code

The source code for PayMyBizz lives at [github.com/benmarte/paymybizz](https://github.com/benmarte/paymybizz) (private). Docker images are published automatically to `ghcr.io/benmarte/paymybizz` on every release.
