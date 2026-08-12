---
name: docker
description: >
  Docker best practices. Use when writing or modifying Dockerfiles, docker-compose files,
  or any container configuration. Covers image hygiene, security, multi-stage builds,
  secrets management, and Compose conventions. Auto-activates on Dockerfile and .yml
  container config files.
allowed-tools: Read, Bash, Grep, Glob
---

# docker - Docker Best Practices

## Dockerfile - Image Hygiene

Always pin the base image to a specific version tag. Never use `latest`.

```dockerfile
# ❌ Never
FROM node:latest

# ✅ Always
FROM node:22-alpine
```

Prefer slim or alpine variants to minimize attack surface and image size.

## Dockerfile - Layer Optimisation

Order instructions from least to most frequently changed to maximise cache reuse.
Group related `RUN` commands into a single layer to reduce image size.

```dockerfile
# ❌ Creates 3 layers, busts cache on every change
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# ✅ Single layer, clean cache in the same step
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

Copy dependency manifests before source code so the package install layer is cached:

```dockerfile
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
```

## Dockerfile - Multi-Stage Builds

Always use multi-stage builds for production images. Builder stage installs tools and
compiles assets; final stage contains only the runtime artefacts.

```dockerfile
# ── Builder ──────────────────────────────────────────
FROM node:22-alpine AS builder

RUN corepack enable
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm run build           # Next.js: requires output "standalone" in next.config.ts

# ── Runtime ──────────────────────────────────────────
FROM node:22-alpine AS runtime

WORKDIR /app
ENV NODE_ENV=production

# Copy only the build output, not the build tooling
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

CMD ["node", "server.js"]
```

## Dockerfile - Security

**Never run as root.** Create a dedicated non-root user.

```dockerfile
RUN groupadd --gid 1001 appgroup \
    && useradd --uid 1001 --gid appgroup --shell /bin/sh --create-home appuser

USER appuser
```

**Never bake secrets into the image.**

```dockerfile
# ❌ Secret visible in every layer and in docker history
RUN echo "//registry.example.com/:_authToken=TOKEN" > .npmrc && pnpm install --frozen-lockfile

# ✅ Pass secrets at build time using BuildKit secret mounts
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc pnpm install --frozen-lockfile

# ✅ Or at runtime via environment variables
ENV DATABASE_URL=""   # placeholder - real value injected at runtime
```

**Always include a `.dockerignore`** to prevent leaking secrets and bloating the build context:

```
.git
.env
.env.*
*.env
node_modules
.next
dist
coverage
*.log
.DS_Store
```

## Dockerfile - Health Checks

Every long-running service must declare a health check:

```dockerfile
# HTTP service
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# TCP-only service
HEALTHCHECK --interval=30s --timeout=5s \
  CMD nc -z localhost 5432 || exit 1
```

## Dockerfile - Metadata

Label every image for traceability:

```dockerfile
LABEL maintainer="team@example.com"
LABEL org.opencontainers.image.title="my-service"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.source="https://github.com/org/repo"
```

## docker-compose - Structure

Use a `base` / `dev` / `prod` split, not a single file:

```
docker-compose.yml          # base: shared service definitions
docker-compose.dev.yml      # dev overrides: volumes, ports, hot-reload
docker-compose.prod.yml     # prod overrides: replicas, resource limits
```

```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## docker-compose - Services

```yaml
services:
  api:
    build:
      context: .
      target: runtime          # reference the multi-stage target explicitly
    image: my-api:${TAG:-local}
    restart: unless-stopped
    environment:
      DATABASE_URL: ${DATABASE_URL}   # always from .env - never hardcoded
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy   # wait for health check, not just start
    ports:
      - "127.0.0.1:3000:3000"        # bind to localhost only in dev
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

## docker-compose - Volumes & Networks

Name volumes and networks explicitly - never rely on auto-generated names:

```yaml
volumes:
  postgres_data:
    name: myproject_postgres_data

networks:
  backend:
    name: myproject_backend
    driver: bridge
```

Do not use `network_mode: host` in production.

## docker-compose - Secrets & Environment

Never commit `.env` files. Always provide `.env.example`:

```bash
# .env.example - commit this
DATABASE_URL=postgres://user:password@db:5432/mydb
SECRET_KEY=change-me
DEBUG=false
```

For production secrets, use Docker secrets or an external secrets manager:

```yaml
secrets:
  db_password:
    external: true   # managed outside Compose (e.g. Docker Swarm / Vault)

services:
  api:
    secrets:
      - db_password
```

## Tooling Workflow

```bash
# Build with BuildKit enabled (always)
DOCKER_BUILDKIT=1 docker build -t my-service:local .

# Check image size and layers
docker image inspect my-service:local
docker history my-service:local

# Scan for vulnerabilities
docker scout cves my-service:local
# or: trivy image my-service:local

# Lint the Dockerfile
hadolint Dockerfile

# Start dev stack
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build

# Clean up
docker compose down --volumes --remove-orphans
```

## Rules

- Pin every base image to a specific version - never `latest`
- Use multi-stage builds for all production images
- Never run containers as root
- Never hardcode secrets - use env vars or Docker secrets
- Always include `.dockerignore`
- Always define a `HEALTHCHECK` for long-running services
- Split Compose files into `base` / `dev` / `prod`
- Bind dev ports to `127.0.0.1` only
- Name all volumes and networks explicitly
- Run `hadolint` on every `Dockerfile` before committing
