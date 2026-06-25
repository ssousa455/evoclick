# EvoClick — Self-Hosted Click Tracker for Affiliates

> **Credits:** This project is based on [EvoClick](https://github.com/EricFrancis12/evoclick) by [EricFrancis12](https://github.com/EricFrancis12). Massive thanks for the amazing work!

EvoClick is an open-source, self-hosted click tracking platform for media buyers and affiliate marketers. It's the free alternative to paid SaaS trackers like Voluum, RedTrack, and Binom.

---

## ✨ Features

- **Campaign Management** — Create and manage tracking campaigns with configurable flows
- **Click Tracking** — Track every visitor view, click, and conversion in real time
- **Real-time Reporting** — Dashboard with charts (revenue, visits, costs, ROI)
- **Smart Routing** — 14+ targeting rules (country, device, OS, browser, ISP, language, IP)
- **Multi-Entity Management** — Affiliate Networks, Offers, Landing Pages, Traffic Sources, Saved Flows
- **Conversion Tracking** — Postback URL integration with any affiliate network
- **Token Replacement** — Dynamic URL tokens (`{externalID}`, `{cost}`, `{country}`, `{device}`, etc.)
- **Tab-Based Navigation** — Open multiple report tabs for drill-down analysis
- **JWT Authentication** — Secure login with configurable credentials
- **Redis Caching** — High-performance caching layer (optional)
- **Go + Next.js** — High-performance Go API for click tracking, Next.js frontend for the admin panel

---

## 🧰 Prerequisites

- **VPS** with Ubuntu 20.04/22.04 (or similar) — at least 2 vCPUs and 4 GB RAM recommended
- **Docker** and **Docker Compose** installed
- A **domain** pointing to your VPS IP
- A **GitHub** account (for repository hosting and Dokploy integration)

### One-Click Deploy with Dokploy (Recommended)

This repository is designed to work seamlessly with [Dokploy](https://dokploy.com) — an open-source deployment platform.

---

## 📦 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/ssousa455/evoclick.git
cd evoclick
```

### 2. Configure environment variables

Create a `.env` file:

```env
APP_DOMAIN=evoclick.yourdomain.com
API_DOMAIN=evoclick-api.yourdomain.com
ROOT_USERNAME=admin
ROOT_PASSWORD=YourStrongPassword
POSTGRES_PASSWORD=YourDbPassword
JWT_SECRET=your-random-64-char-secret
PORT=3000
API_PORT=3001
```

### 3. Deploy with Dokploy

1. Create a new **Project** in Dokploy
2. Connect your GitHub repository
3. Set the environment variables (same as `.env`)
4. Click **Deploy**

### 4. Configure domains in Dokploy

In the Dokploy UI, go to the service → **Domains** tab:

- **next-app** → `evoclick.yourdomain.com` (port 3000)
- **api** → `evoclick-api.yourdomain.com` (port 3001)

### 5. Access the panel

Open `https://evoclick.yourdomain.com` and log in with your credentials.

---

## 🐳 Docker Compose (Local / Manual Deploy)

```bash
docker compose up -d
```

The app will be available at `http://localhost:3000`.

---

## ⚠️ Known Issues & Fixes

| Issue | Fix |
|---|---|
| **Prisma OpenSSL error** | Use `node:18-bullseye` instead of `node:18-alpine` in Dockerfiles |
| **Empty database tables** | `command: sh -c "npx prisma db push && npm start"` on `next-app` service |
| **Cloudflare proxy interference** | Keep subdomains in **DNS only** (gray cloud) during setup |
| **Duplicate domain config** | Use either Dokploy UI **or** `docker-compose.yml` labels — not both |

---

## 🏗️ Architecture

```
                     Internet
                        |
                  [Traefik Proxy]
                   /            \
         (Port 3000)          (Port 3001)
        +-----------+       +------------+
        | Next.js   |       | Go API     |
        | (Frontend |       | (Click     |
        |  + Admin  |       |  Tracking  |
        |  UI)      |       |  Engine)   |
        +-----------+       +------------+
              |                    |
        +-----------+       +------------+
        | PostgreSQL|       |   Redis    |
        | (Primary  |       |   (Cache)  |
        |  Store)   |       |            |
        +-----------+       +------------+
```

### Services

| Service | Tech | Port | Purpose |
|---|---|---|---|
| `next-app` | Next.js 14 (Node.js) | 3000 | Frontend + Admin API |
| `api` | Go 1.22 | 3001 | Click tracking engine |
| `postgres` | PostgreSQL 16 | 5432 | Database |
| `redis` | Redis 7 | 6379 | Cache (optional) |

### API Endpoints (Go)

| Endpoint | Method | Description |
|---|---|---|
| `/t?g={campaignId}` | GET | Track visit → redirect based on rules |
| `/click` | GET | Register click → redirect to offer |
| `/postback?pid={clickId}&payout={amount}` | GET | Receive conversion from affiliate network |

---

## 📹 How It Works

```
1. Visitor clicks your ad/tracking URL
   → https://evoclick-api.yourdomain.com/t?g=CAMPAIGN_ID&external_id=XXX&cost=0.50

2. EvoClick records the visit, evaluates routing rules (country, device, OS, etc.)
   → Redirects to landing page or directly to offer

3. If landing page → visitor clicks → /click endpoint records click
   → Redirects to offer URL with tokens replaced

4. Affiliate network sends postback on conversion
   → /postback?pid=CLICK_ID&payout=5.00
   → EvoClick records revenue and conversion time
```

---

## 🧪 Demo Mode

Enable demo mode by setting `DEMO_MODE=1` in your environment variables. This pre-populates the dashboard with sample data for testing.

---

## 🤝 Credits

- [EricFrancis12](https://github.com/EricFrancis12) — Original creator of EvoClick
- Open-source community — Docker, Prisma, Traefik, and all the amazing tools that make this possible

---

## 📄 License

MIT License — see the [original repository](https://github.com/EricFrancis12/evoclick) for details.
