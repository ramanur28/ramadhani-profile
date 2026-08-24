# Deployment & Operations Guide: Ramadhani Brand Platform

This guide outlines how to run the site locally in development mode, deploy the production stack with automated TLS and self-hosted analytics on a single VPS, and configure automatic rebuilds when content is published via Decap CMS.

---

## 1. Local Development (Docker & Native)

### Option A: Via Docker Compose (Recommended for clean isolation)
```bash
# Start Astro dev server and Decap CMS local proxy
docker compose -f docker-compose.dev.yml up
```
- **Astro Site**: [http://localhost:4321](http://localhost:4321)
- **CMS Admin**: [http://localhost:4321/admin](http://localhost:4321/admin)
- **Decap Local Backend Proxy**: [http://localhost:8081](http://localhost:8081)

### Option B: Native Node.js
```bash
npm install
npm run dev
```

---

## 2. Production VPS Deployment

The production architecture is fully containerized with **Caddy** (serving pre-compressed static assets, enforcing security headers, and automating Let's Encrypt / ZeroSSL TLS certificates) and **Umami Analytics** with its own PostgreSQL database.

### Step 1: Clone Repository on VPS
```bash
git clone https://github.com/your-username/ramadhani-personal-brand.git /opt/ramadhani
cd /opt/ramadhani
```

### Step 2: Configure Environment Variables
```bash
cp .env.example .env
nano .env
```
Update `SITE_DOMAIN`, `SITE_URL`, `UMAMI_DB_PASSWORD`, and `UMAMI_APP_SECRET`.

### Step 3: Launch Production Stack
```bash
docker compose -f docker-compose.prod.yml up -d --build
```

### Step 4: Verify Deployment & Healthchecks
```bash
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs web
```

---

## 3. Automated Rebuild on CMS Article Publication

When an editor publishes or updates an MDX article via Decap CMS at `/admin`, Decap commits the new MDX file to the `main` branch of your GitHub repository.

To trigger an automatic rebuild without needing a complex CI pipeline, set up a simple lightweight webhook handler on your VPS (e.g., using `adnanh/webhook` or a 15-line Node.js script):

### Webhook Deploy Script (`/opt/ramadhani/deploy.sh`)
```bash
#!/bin/bash
set -e
cd /opt/ramadhani
git pull origin main
docker compose -f docker-compose.prod.yml build web
docker compose -f docker-compose.prod.yml up -d --no-deps web
echo "Production build successfully refreshed at $(date)"
```
Make the script executable:
```bash
chmod +x /opt/ramadhani/deploy.sh
```

### GitHub Webhook Setup
1. In your GitHub repository, go to **Settings > Webhooks > Add webhook**.
2. **Payload URL**: `https://ramadhani.dev/api/webhook-deploy` (or your webhook server endpoint).
3. **Content type**: `application/json`.
4. **Secret**: Value of `REBUILD_WEBHOOK_SECRET` in your `.env`.
5. **Events**: "Just the push event" on `main`.

---

## 4. Performance & Core Web Vitals Monitoring

- Run local Lighthouse audits:
  ```bash
  npm run build
  npx lhci autorun
  ```
- Target Budgets:
  - **Performance**: ≥ 95
  - **SEO**: 100
  - **Accessibility**: ≥ 95
  - **Best Practices**: ≥ 95
  - **LCP**: < 2.0s
  - **CLS**: < 0.1