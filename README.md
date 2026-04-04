# PayMyBizz

**Self-hosted invoicing, time tracking, and expense management for freelancers and small agencies.**

Run it on your own server. Your data stays yours.

![Dashboard overview](https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docs/images/dashboard-overview.png)

---

## What you get

| | |
|---|---|
| 🧾 **Invoicing** | Create, send, and track invoices with automatic overdue reminders |
| ⏱ **Time tracking** | Built-in timer, manual entry, bulk actions, and one-click billing to invoices |
| 💸 **Expenses** | Track business expenses with receipt attachments and billable flagging |
| 👥 **Clients & projects** | Manage clients, projects, billing types (hourly, retainer, fixed), and budgets |
| 📊 **Reports** | Revenue by client, tax summaries, and CSV export |
| 🔁 **Recurring invoices** | Auto-generate invoices on weekly, monthly, or custom schedules |
| 📅 **Calendar sync** | Sync time entries with Google Calendar (optional) |
| 🌍 **8 languages** | English, Spanish, French, Portuguese, German, Japanese, Korean, Chinese |
| 👥 **Team support** | Invite team members with role-based access (Owner, Admin, Accountant, Viewer) |
| 📱 **Works offline** | Progressive Web App — install it, use it offline, syncs when back online |
| 🔒 **2FA** | TOTP two-factor authentication for your account |

---

## Screenshots

<table>
<tr>
<td><img src="https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docs/images/invoices-page.png" alt="Invoices"></td>
<td><img src="https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docs/images/time-tracking-page.png" alt="Time tracking"></td>
</tr>
<tr>
<td align="center"><b>Invoices</b></td>
<td align="center"><b>Time Tracking</b></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docs/images/clients-page.png" alt="Clients"></td>
<td><img src="https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docs/images/reports-page.png" alt="Reports"></td>
</tr>
<tr>
<td align="center"><b>Clients</b></td>
<td align="center"><b>Reports</b></td>
</tr>
</table>

---

## Requirements

> **You only need Docker.** Everything else is optional and can be configured later from the app's Settings page.

| | What | Notes |
|---|---|---|
| ✅ **Required** | Docker + Docker Compose v2+ | The only hard requirement |
| ✅ **Required** | 2 GB free disk space | For the database and Docker images |
| ⬜ Optional | Email provider (Gmail or Resend) | Needed only to send invoice emails to clients. The app works fully without it. |
| ⬜ Optional | Google OAuth credentials | Only needed if you want **Sign in with Google**. Email + password login works without it — do not add these unless you want Google login. |
| ⬜ Optional | Stripe account | Only needed for online card payments. Manual payment tracking (bank transfer, Zelle, etc.) works without it. |
| ⬜ Optional | Cloudflare account | Only needed to expose the app to the internet via a tunnel. Not needed for local use. |

---

## Quick start

```bash
# 1. Download the compose file
curl -O https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docker-compose.yml

# 2. Create your config (safe defaults are pre-filled — no changes needed to get started)
curl -O https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/.env.example
cp .env.example .env

# 3. Create data directories (your database and keys are stored here)
mkdir -p data/convex data/keys data/auth-keys

# 4. Start
docker compose up -d
```

Open **http://localhost:8177** — the setup wizard will guide you through the rest.

The first account you create automatically gets **Owner** (admin) access.

---

## Configuration

All settings in `.env` are optional except for the ones marked required above. The setup wizard inside the app will walk you through connecting email, Google OAuth, and Stripe if you want them.

For a full list of environment variables, see the [Configuration guide](../../wiki/Admin-Guide-Configuration).

---

## Data persistence

PayMyBizz stores all data in local directories on your filesystem:

| Directory | Contents |
|---|---|
| `./data/convex/` | Database (all your business data) |
| `./data/keys/` | Convex admin key |
| `./data/auth-keys/` | Auth signing keys |

These directories are bind-mounted directly into the containers, so your data survives container restarts, image updates, and `docker compose down`. They are plain directories on disk — back them up like any other folder.

> **Never run `docker compose down -v`** — the `-v` flag removes any remaining Docker volumes. Your `./data/` directories are safe, but it's a destructive command that's easy to mistype.

### Migrating from Docker volumes (existing installs)

If you set up PayMyBizz before this change and your data is in Docker named volumes, follow these steps to move it to the local filesystem:

```bash
# 1. Stop the containers — do NOT use -v
docker compose down

# 2. Create the host directories
mkdir -p data/convex data/keys data/auth-keys

# 3. Copy volume data to host directories
docker run --rm \
  -v paymybizz_convex_data:/source \
  -v "$(pwd)/data/convex:/dest" \
  alpine sh -c "cp -a /source/. /dest/"

docker run --rm \
  -v paymybizz_convex_keys:/source \
  -v "$(pwd)/data/keys:/dest" \
  alpine sh -c "cp -a /source/. /dest/"

docker run --rm \
  -v paymybizz_auth_keys:/source \
  -v "$(pwd)/data/auth-keys:/dest" \
  alpine sh -c "cp -a /source/. /dest/"

# 4. Pull the updated docker-compose.yml
curl -O https://raw.githubusercontent.com/benmarte/paymybizz-selfhosted/main/docker-compose.yml

# 5. Start again
docker compose up -d

# 6. Confirm everything works, then remove the old volumes
docker volume rm paymybizz_convex_data paymybizz_convex_keys paymybizz_auth_keys
```

---

## Remote access (optional)

To access your instance from outside your local network, add a Cloudflare tunnel token to `.env`:

```env
TUNNEL_TOKEN=your_cloudflare_tunnel_token
```

Then restart with the tunnel profile:

```bash
docker compose --profile tunnel up -d
```

See the [Installation guide](../../wiki/Admin-Guide-Installation) for Cloudflare setup steps.

---

## Updating

```bash
docker compose pull
docker compose up -d
```

> **Never run `docker compose down -v`** — the `-v` flag deletes your data volumes permanently. Plain `docker compose down` (no `-v`) is safe.

---

## Documentation

Full user and admin documentation is available in the [Wiki](../../wiki).

**User Guide**
- [Getting Started](../../wiki/Getting-Started)
- [Invoices](../../wiki/Invoices)
- [Time Tracking](../../wiki/Time-Tracking)
- [Clients](../../wiki/Clients)
- [Reports](../../wiki/Reports)
- [Settings](../../wiki/Settings)

**Admin Guide**
- [Installation](../../wiki/Admin-Guide-Installation)
- [Configuration](../../wiki/Admin-Guide-Configuration)
- [Backup & Restore](../../wiki/Admin-Guide-Backup-Restore)
- [Team Roles](../../wiki/Admin-Guide-Team-Roles)

---

## Support

- [Open an issue](https://github.com/benmarte/paymybizz-selfhosted/issues) for bugs or questions
- [Discussions](https://github.com/benmarte/paymybizz-selfhosted/discussions) for general help
