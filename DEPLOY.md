# Deployment & Operations Guide: Ramadhani Brand Platform

This guide outlines how to build the production Docker image locally on your computer and deploy it to your remote production server (VPS at `/var/www/ramadhani`).

---

## 1. Local Development (Docker & Native)

### Option A: Via Docker Compose
```bash
docker compose -f docker-compose.dev.yml up
```
- **Astro Site**: [http://localhost:4321](http://localhost:4321)
- **CMS Admin**: [http://localhost:4321/admin](http://localhost:4321/admin)

### Option B: Native Node.js
```bash
npm install
npm run dev
```

---

## 2. Build Locally & Deploy to Production VPS

Building the Docker image locally saves CPU and RAM on your VPS and ensures zero build overhead in production.

### Method A: Direct Transfer (No Registry Required - Recommended)

#### Step 1: Build & Compress Image on Local Machine
Run this in WSL or your local terminal:
```bash
# 1. Build image for Linux VPS architecture (AMD64)
docker build --platform linux/amd64 -t ramadhani-web:latest .

# 2. Export and compress image to tar.gz
docker save ramadhani-web:latest | gzip > ramadhani-web.tar.gz
```

#### Step 2: Transfer Image & Configs to VPS
```bash
# Transfer image archive to VPS
scp ramadhani-web.tar.gz user@your-vps-ip:/var/www/ramadhani/

# Transfer docker-compose.prod.yml & .env if updated
scp docker-compose.prod.yml .env Caddyfile user@your-vps-ip:/var/www/ramadhani/
```

#### Step 3: Load & Run on Production VPS
SSH into your VPS:
```bash
ssh user@your-vps-ip

# Navigate to app directory
cd /var/www/ramadhani

# Load pre-built image into Docker
docker load < ramadhani-web.tar.gz

# Start or restart the stack (zero rebuild needed)
docker compose -f docker-compose.prod.yml up -d

# (Optional) Clean up the transferred archive
rm ramadhani-web.tar.gz
```

---

### Method B: Via Docker Hub or GitHub Container Registry (GHCR)

#### Step 1: Build & Push on Local Machine
```bash
# Login to Docker Hub or GHCR
docker login

# Build for Linux VPS architecture and push
docker build --platform linux/amd64 -t yourdockerhub/ramadhani-web:latest .
docker push yourdockerhub/ramadhani-web:latest
```

#### Step 2: Pull & Run on VPS
On your VPS in `/var/www/ramadhani`:
1. In `.env`, set `WEB_IMAGE=yourdockerhub/ramadhani-web:latest`
2. Run:
```bash
docker compose -f docker-compose.prod.yml pull web
docker compose -f docker-compose.prod.yml up -d
```

---

## 3. Server Firewall & DNS Checklist

### DNS Records:
- `A` Record: `@` &rarr; `<YOUR_VPS_PUBLIC_IP>` (`ramadhani.cloud`)
- `CNAME` / `A` Record: `www` &rarr; `ramadhani.cloud`
- `CNAME` / `A` Record: `analytics` &rarr; `ramadhani.cloud`

### VPS Firewall Ports (UFW):
```bash
sudo ufw allow 80/tcp comment "HTTP (Caddy)"
sudo ufw allow 443/tcp comment "HTTPS (Caddy)"
sudo ufw allow 443/udp comment "HTTP/3 QUIC (Caddy)"
sudo ufw allow 22/tcp comment "SSH"
sudo ufw reload
```

---

## 4. Verification & Health Monitoring

Check running containers:
```bash
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs web --tail 50
```

- **Website**: `https://ramadhani.cloud`
- **Analytics**: `https://analytics.ramadhani.cloud` (or port `3000`)
