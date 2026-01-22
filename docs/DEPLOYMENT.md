# Prazium Deployment Guide

## Overview

Prazium supports multiple deployment modes:
- **Self-hosted (Single Node)** - Docker Compose on a single server
- **Self-hosted (Kubernetes)** - Scalable K8s deployment
- **Hosted (prazium.com)** - Managed multi-tenant service

---

## Self-Hosted Deployment

### Prerequisites

- Docker Engine 24.0+
- Docker Compose v2.20+
- 4+ CPU cores
- 16GB+ RAM
- 100GB+ SSD storage
- Linux host (Ubuntu 22.04 recommended)

### Quick Start

```bash
# Clone repository
git clone https://github.com/prazium/prazium.git
cd prazium

# Copy environment template
cp .env.example .env

# Edit configuration
nano .env

# Start services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

---

## Docker Compose Configuration

### docker-compose.yml

```yaml
version: '3.8'

services:
  # ===================
  # Web UI
  # ===================
  ui:
    image: prazium/ui:latest
    build:
      context: ./packages/ui
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://api:3001
      - NEXT_PUBLIC_WS_URL=ws://api:3001
    depends_on:
      - api
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ===================
  # Orchestrator API
  # ===================
  api:
    image: prazium/api:latest
    build:
      context: ./packages/api
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://prazium:${DB_PASSWORD}@postgres:5432/prazium
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
      - LLM_API_KEY=${LLM_API_KEY}
      - LLM_PROVIDER=${LLM_PROVIDER:-openai}
      - SESSION_SECRET=${SESSION_SECRET}
      - DATA_DIR=/data
      - LOG_LEVEL=${LOG_LEVEL:-info}
    volumes:
      - prazium-data:/data
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ===================
  # Worker
  # ===================
  worker:
    image: prazium/worker:latest
    build:
      context: ./packages/worker
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://prazium:${DB_PASSWORD}@postgres:5432/prazium
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
      - LLM_API_KEY=${LLM_API_KEY}
      - LLM_PROVIDER=${LLM_PROVIDER:-openai}
      - DATA_DIR=/data
      - DOCKER_HOST=unix:///var/run/docker.sock
      - CONCURRENCY=${WORKER_CONCURRENCY:-4}
      - LOG_LEVEL=${LOG_LEVEL:-info}
    volumes:
      - prazium-data:/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      replicas: ${WORKER_REPLICAS:-1}

  # ===================
  # PostgreSQL
  # ===================
  postgres:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=prazium
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=prazium
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - "5432:5432"
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U prazium"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===================
  # Redis
  # ===================
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===================
  # Agent Image Builder (one-time)
  # ===================
  agent-builder:
    image: docker:24-dind
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./packages/agent:/workspace:ro
    command: >
      sh -c "docker build -t prazium/agent:latest /workspace"
    profiles:
      - setup

volumes:
  prazium-data:
    driver: local
  postgres-data:
    driver: local
  redis-data:
    driver: local

networks:
  default:
    name: prazium-network
```

### Environment Variables (.env)

```bash
# ===================
# Database
# ===================
DB_PASSWORD=your-secure-database-password

# ===================
# Redis
# ===================
REDIS_PASSWORD=your-secure-redis-password

# ===================
# LLM Configuration
# ===================
LLM_PROVIDER=openai
LLM_API_KEY=sk-your-openai-api-key

# Alternative: Anthropic
# LLM_PROVIDER=anthropic
# LLM_API_KEY=sk-ant-your-anthropic-key

# ===================
# Security
# ===================
SESSION_SECRET=generate-a-32-byte-random-string

# ===================
# Worker Configuration
# ===================
WORKER_CONCURRENCY=4
WORKER_REPLICAS=1

# ===================
# Logging
# ===================
LOG_LEVEL=info

# ===================
# Optional: S3 Storage
# ===================
# S3_ENDPOINT=https://s3.amazonaws.com
# S3_BUCKET=prazium-artifacts
# S3_ACCESS_KEY=your-access-key
# S3_SECRET_KEY=your-secret-key
```

---

## Production Deployment

### Reverse Proxy (nginx)

```nginx
# /etc/nginx/sites-available/prazium
upstream prazium_ui {
    server 127.0.0.1:3000;
}

upstream prazium_api {
    server 127.0.0.1:3001;
}

server {
    listen 80;
    server_name prazium.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name prazium.example.com;

    ssl_certificate /etc/letsencrypt/live/prazium.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/prazium.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;

    # UI
    location / {
        proxy_pass http://prazium_ui;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # API
    location /api/ {
        proxy_pass http://prazium_api/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket
    location /api/ws {
        proxy_pass http://prazium_api/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
    }

    # SSE
    location /api/v1/projects/ {
        proxy_pass http://prazium_api/v1/projects/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Connection "";
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 86400;
    }
}
```

### SSL Certificate

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d prazium.example.com

# Auto-renewal
sudo systemctl enable certbot.timer
```

---

## Railway Deployment

### railway.json

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "numReplicas": 1,
    "sleepApplication": false,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Dockerfile (Multi-stage)

```dockerfile
# ===================
# Build Stage
# ===================
FROM node:20-alpine AS builder

WORKDIR /app

# Install pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# Copy package files
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY packages/api/package.json ./packages/api/
COPY packages/ui/package.json ./packages/ui/
COPY packages/shared/package.json ./packages/shared/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source
COPY . .

# Build
RUN pnpm build

# ===================
# Production Stage - API
# ===================
FROM node:20-alpine AS api

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@latest --activate

COPY --from=builder /app/packages/api/dist ./dist
COPY --from=builder /app/packages/api/package.json ./
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3001

CMD ["node", "dist/main.js"]

# ===================
# Production Stage - UI
# ===================
FROM node:20-alpine AS ui

WORKDIR /app

COPY --from=builder /app/packages/ui/.next/standalone ./
COPY --from=builder /app/packages/ui/.next/static ./.next/static
COPY --from=builder /app/packages/ui/public ./public

EXPOSE 3000

CMD ["node", "server.js"]
```

### Railway Services

```yaml
# railway.toml
[services.api]
  dockerfile = "Dockerfile"
  target = "api"
  
[services.ui]
  dockerfile = "Dockerfile"
  target = "ui"

[services.worker]
  dockerfile = "packages/worker/Dockerfile"

[services.postgres]
  plugin = "postgresql"

[services.redis]
  plugin = "redis"
```

---

## Export Bundle Specification

### Bundle Contents

```
my-app-export/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── api/
│   ├── components/
│   ├── lib/
│   └── types/
├── supabase/
│   ├── migrations/
│   └── config.toml
├── public/
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── tailwind.config.js
├── next.config.js
├── README.md
└── README_DEPLOY.md
```

### .env.example

```bash
# ===================
# Supabase
# ===================
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# ===================
# Application
# ===================
NEXT_PUBLIC_APP_URL=http://localhost:3000

# ===================
# Optional
# ===================
# NEXT_PUBLIC_ANALYTICS_ID=
```

### Dockerfile (Generated App)

```dockerfile
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@latest --activate

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN pnpm build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### docker-compose.yml (Generated App)

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_SUPABASE_URL=${NEXT_PUBLIC_SUPABASE_URL}
      - NEXT_PUBLIC_SUPABASE_ANON_KEY=${NEXT_PUBLIC_SUPABASE_ANON_KEY}
      - SUPABASE_SERVICE_ROLE_KEY=${SUPABASE_SERVICE_ROLE_KEY}
    restart: unless-stopped
```

### README_DEPLOY.md (Generated)

```markdown
# Deployment Guide

## Prerequisites

- Node.js 20+
- pnpm 8+
- Supabase account
- Docker (optional)

## Local Development

\`\`\`bash
# Install dependencies
pnpm install

# Set up environment
cp .env.example .env.local
# Edit .env.local with your Supabase credentials

# Run development server
pnpm dev
\`\`\`

## Supabase Setup

1. Create a new Supabase project
2. Run migrations:
   \`\`\`bash
   npx supabase db push
   \`\`\`
3. Copy credentials to .env

## Deploy to Railway

\`\`\`bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Deploy
railway up
\`\`\`

## Deploy to Vercel

\`\`\`bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel
\`\`\`

## Deploy with Docker

\`\`\`bash
# Build image
docker build -t my-app .

# Run container
docker run -p 3000:3000 --env-file .env my-app
\`\`\`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| NEXT_PUBLIC_SUPABASE_URL | Yes | Supabase project URL |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | Yes | Supabase anonymous key |
| SUPABASE_SERVICE_ROLE_KEY | Yes | Supabase service role key |
```

---

## Backup & Recovery

### Database Backup

```bash
# Manual backup
docker compose exec postgres pg_dump -U prazium prazium > backup.sql

# Automated backup script
#!/bin/bash
BACKUP_DIR=/backups
DATE=$(date +%Y%m%d_%H%M%S)
docker compose exec -T postgres pg_dump -U prazium prazium | gzip > $BACKUP_DIR/prazium_$DATE.sql.gz

# Keep last 7 days
find $BACKUP_DIR -name "prazium_*.sql.gz" -mtime +7 -delete
```

### Git Repository Backup

```bash
# Backup all repositories
tar -czf repos_backup.tar.gz /data/repos/

# Restore
tar -xzf repos_backup.tar.gz -C /
```

### Full System Backup

```bash
#!/bin/bash
# Stop services
docker compose stop

# Backup volumes
docker run --rm \
  -v prazium-data:/data:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/prazium-data.tar.gz /data

docker run --rm \
  -v postgres-data:/data:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres-data.tar.gz /data

# Start services
docker compose start
```

---

## Monitoring

### Health Checks

```bash
# API health
curl http://localhost:3001/health

# UI health
curl http://localhost:3000/api/health

# Database
docker compose exec postgres pg_isready -U prazium

# Redis
docker compose exec redis redis-cli -a $REDIS_PASSWORD ping
```

### Prometheus Metrics

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'prazium-api'
    static_configs:
      - targets: ['api:3001']
    metrics_path: '/metrics'

  - job_name: 'prazium-worker'
    static_configs:
      - targets: ['worker:9090']
```

### Logging

```yaml
# docker-compose.override.yml
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"

  worker:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
```

---

## Scaling

### Horizontal Scaling

```yaml
# docker-compose.scale.yml
services:
  worker:
    deploy:
      replicas: 4
      resources:
        limits:
          cpus: '2'
          memory: 4G
```

```bash
# Scale workers
docker compose up -d --scale worker=4
```

### Resource Recommendations

| Component | CPU | Memory | Storage |
|-----------|-----|--------|---------|
| UI | 0.5 | 512MB | - |
| API | 1 | 1GB | - |
| Worker | 2 | 4GB | - |
| PostgreSQL | 1 | 2GB | 50GB |
| Redis | 0.5 | 1GB | 10GB |
| Data Volume | - | - | 100GB |

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Container won't start | Missing env vars | Check .env file |
| Database connection failed | Wrong credentials | Verify DB_PASSWORD |
| Worker can't spawn sandboxes | Docker socket permission | Add user to docker group |
| Out of disk space | Build artifacts | Clean old exports |
| Slow builds | Insufficient resources | Scale workers |

### Debug Commands

```bash
# View logs
docker compose logs -f api
docker compose logs -f worker

# Enter container
docker compose exec api sh
docker compose exec worker sh

# Check disk usage
docker system df
du -sh /data/*

# Clean up
docker system prune -a
```

---

## Upgrade Procedure

```bash
# Pull latest images
docker compose pull

# Backup database
docker compose exec postgres pg_dump -U prazium prazium > backup.sql

# Stop services
docker compose down

# Start with new images
docker compose up -d

# Run migrations
docker compose exec api pnpm db:migrate

# Verify
docker compose ps
curl http://localhost:3001/health
```

---

## Related Documentation

- [Architecture Overview](./ARCHITECTURE.md)
- [Security Model](./SECURITY.md)
- [API Specification](./API.md)
