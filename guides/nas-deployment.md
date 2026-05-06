# NAS / VPS deployment guide

Deploys the full backend stack with HTTPS via Nginx Proxy Manager. The React
frontend is deployed separately to GitHub Pages.

> **VPS vs NAS difference:** A VPS (e.g. Akamai) has a static public IP — skip
> Steps 1 and 2, remove the `duckdns` service from docker-compose.yml, and set
> your DuckDNS subdomain to the VPS IP once at provisioning. Everything else is
> identical. Skip the router port forwarding step — cloud VMs are directly
> internet-accessible on all ports by default (lock down unused ports in the
> provider's firewall panel).

## Architecture

```
Internet → Router (80/443) → NAS
                               ├── Nginx Proxy Manager  (port 80, 443, 81)
                               │     └── yourdomain.duckdns.org → gateway:5000
                               └── Job Tracker stack
                                     ├── gateway          (port 5000)
                                     ├── job-service      (port 5001)
                                     ├── job-service-consumer (port 5008)
                                     ├── contact-service  (port 5002)
                                     ├── journal-service  (port 5003)
                                     ├── resume-service   (port 5004)
                                     ├── notification-service (port 5005)
                                     ├── postgres         (port 5432)
                                     ├── kafka            (port 9092)
                                     ├── minio            (port 9000/9001)
                                     └── duckdns updater

GitHub Pages → https://markriggs-dev.github.io
  calls → https://yourdomain.duckdns.org/api/...
```

## Prerequisites

- Docker and Docker Compose installed on the NAS
- SSH access to the NAS
- A GitHub account (for DuckDNS login)

---

## Step 1: Create a DuckDNS subdomain

1. Go to [duckdns.org](https://www.duckdns.org) and log in with GitHub
2. Create a subdomain — e.g. `job-tracker` → `job-tracker.duckdns.org`
3. Copy your **token** from the top of the page

---

## Step 2: Forward ports on your router

Log into your router's admin panel and add two port forwarding rules:

| External port | Internal IP      | Internal port |
|---------------|------------------|---------------|
| 80            | \<NAS local IP\> | 80            |
| 443           | \<NAS local IP\> | 443           |

Your NAS local IP is typically something like `192.168.1.x`. Check your router's
DHCP client list or the NAS network settings to find it.

---

## Step 3: Create the shared Docker network

Run this once on the NAS. Both the NPM stack and the job tracker stack attach
to this network so NPM can proxy to the gateway container.

```bash
docker network create proxy
```

---

## Step 4: Start Nginx Proxy Manager

```bash
cd ~/job-tracker-app-infrastructure
docker compose -f nginx-proxy-manager/docker-compose.yml up -d
```

Open the NPM admin UI at `http://<nas-ip>:81`.
Default credentials: `admin@example.com` / `changeme` — change these immediately.

---

## Step 5: Clone the repos on the NAS

SSH into the NAS and clone all service repos into the same parent folder:

```bash
mkdir ~/job-tracker && cd ~/job-tracker

git clone https://github.com/markriggs-dev/job-tracker-app-infrastructure
git clone https://github.com/markriggs-dev/job-tracker-app-gateway
git clone https://github.com/markriggs-dev/job-tracker-app-job-service
git clone https://github.com/markriggs-dev/job-tracker-app-job-service-consumer
git clone https://github.com/markriggs-dev/job-tracker-app-contact-service
git clone https://github.com/markriggs-dev/job-tracker-app-journal-service
git clone https://github.com/markriggs-dev/job-tracker-app-resume-service
git clone https://github.com/markriggs-dev/job-tracker-app-notification-service
```

---

## Step 6: Configure environment variables

```bash
cd ~/job-tracker/job-tracker-app-infrastructure
cp .env.example .env
nano .env
```

Fill in all values. Key production settings:

```env
CORS_ALLOWED_ORIGINS=https://markriggs-dev.github.io
DUCKDNS_SUBDOMAINS=job-tracker
DUCKDNS_TOKEN=<your-duckdns-token>
```

---

## Step 7: Start the job tracker stack

```bash
cd ~/job-tracker/job-tracker-app-infrastructure

# Build images and start all backend services (skip web-react — GitHub Pages handles the frontend)
docker compose up -d --scale web-react=0
```

Watch the logs to confirm all services start cleanly:

```bash
docker compose logs -f gateway job-service job-service-consumer
```

---

## Step 8: Configure the NPM proxy host

1. In the NPM admin UI, go to **Proxy Hosts → Add Proxy Host**
2. Fill in:
   - **Domain name:** `job-tracker.duckdns.org`
   - **Scheme:** `http`
   - **Forward hostname/IP:** `gateway`
   - **Forward port:** `8080`
   - Enable **Websockets Support** (required for SignalR)
3. On the **SSL** tab:
   - Select **Request a new SSL certificate**
   - Enable **Force SSL**
   - Enable **HTTP/2 Support**
   - Agree to Let's Encrypt ToS
4. Save — NPM will request the cert automatically

---

## Step 9: Update Auth0 settings

In the Auth0 dashboard, add the production domain to:

- **Allowed Callback URLs:** `https://markriggs-dev.github.io`
- **Allowed Logout URLs:** `https://markriggs-dev.github.io`
- **Allowed Web Origins:** `https://markriggs-dev.github.io`
- **Allowed Origins (CORS):** `https://markriggs-dev.github.io`

---

## Step 10: Deploy React to GitHub Pages

In the `job-tracker-app-web-react` repo, create a `.env.production` file:

```env
VITE_GATEWAY_URL=https://job-tracker.duckdns.org
VITE_AUTH0_DOMAIN=<your-auth0-domain>
VITE_AUTH0_CLIENT_ID=<your-auth0-client-id>
VITE_AUTH0_AUDIENCE=<your-auth0-audience>
```

Build and deploy:

```bash
npm run build
# then push dist/ to gh-pages branch (or use a GitHub Actions workflow)
```

---

## Updating the stack

To pull new images and restart:

```bash
cd ~/job-tracker/job-tracker-app-infrastructure
git pull
docker compose pull   # if using pre-built images from a registry
# or rebuild from source:
docker compose up -d --build --scale web-react=0
```

## Running EF Core migrations on the NAS

Migrations are owned by the Job Service repo. Run them from the NAS before
starting the stack for the first time, or after any schema-changing deploy:

```bash
cd ~/job-tracker/job-tracker-app-job-service
dotnet ef database update --project src/JobService.Infrastructure --startup-project src/JobService.Api
```

Alternatively, configure the Job Service to run migrations on startup by
calling `dbContext.Database.Migrate()` in `Program.cs`.
