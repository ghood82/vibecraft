# Docker & Container Patterns

## Dockerfile for Node.js (Multi-Stage)

```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Security: don't run as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## Dockerfile for Next.js

```dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000
CMD ["node", "server.js"]
```

Requires in `next.config.js`:
```javascript
module.exports = { output: "standalone" };
```

## Dockerfile for Python (FastAPI)

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .

RUN useradd --create-home appuser
USER appuser

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## .dockerignore

```
node_modules
.next
.git
.env
.env.local
*.md
docker-compose*.yml
Dockerfile*
.dockerignore
.github
tests
coverage
.turbo
```

## Docker Best Practices

### Image Size Optimization
```dockerfile
# Use alpine base (5MB vs 900MB for full)
FROM node:20-alpine

# Use multi-stage builds (shown above)
# Only copy what's needed in the final stage

# Combine RUN commands to reduce layers
RUN apk add --no-cache curl && \
    npm ci --only=production && \
    npm cache clean --force

# Use .dockerignore to exclude unnecessary files
```

### Security
```dockerfile
# Never run as root
USER appuser

# Don't store secrets in the image
# Use build args for build-time only, env vars for runtime
ARG NPM_TOKEN
RUN echo "//registry.npmjs.org/:_authtoken=${NPM_TOKEN}" > .npmrc && \
    npm ci && \
    rm .npmrc  # Remove after install

# Scan for vulnerabilities
# docker scout cves myapp:latest

# Pin versions
FROM node:20.11.1-alpine  # Not just "node:20-alpine"
```

### Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1
```

## Common Commands

```bash
# Build
docker build -t myapp .
docker build -t myapp --target builder .  # Stop at a stage

# Run
docker run -p 3000:3000 --env-file .env myapp
docker run -d --name myapp -p 3000:3000 myapp  # Detached

# Compose
docker compose up -d          # Start all services
docker compose down            # Stop all
docker compose logs -f app     # Follow app logs
docker compose exec app sh     # Shell into running container
docker compose build --no-cache  # Rebuild from scratch

# Debug
docker logs myapp              # View logs
docker exec -it myapp sh       # Shell access
docker stats                   # Resource usage

# Cleanup
docker system prune -a         # Remove all unused images/containers
docker volume prune            # Remove unused volumes
```

## CI/CD with Docker

```yaml
# .github/workflows/docker.yml
name: Build & Push Docker
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## Fly.io Deployment (Alternative to Vercel for containers)

```toml
# fly.toml
app = "my-app"
primary_region = "iad"

[build]

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 1

[checks]
  [checks.alive]
    type = "http"
    port = 3000
    path = "/api/health"
    interval = "15s"
    timeout = "2s"
```

```bash
flyctl launch      # Initial setup
flyctl deploy      # Deploy
flyctl logs        # View logs
flyctl ssh console # SSH into machine
```
