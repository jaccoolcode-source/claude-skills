---
description: Analyze a project and generate Dockerfile, docker-compose.yml (with Traefik labels), and .env.example. Usage: /dockerize or /dockerize "app-name"
argument-hint: "[optional app-name override — defaults to directory name]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Dockerize

Analyze the current project and generate production-ready Docker configuration with Traefik reverse proxy integration.

User arguments (optional app name override): **$ARGUMENTS**

---

## Step 1 — Detect Project Type

Read the project root to identify the stack. Check for these files in order:

**Node.js / Bun**
- `package.json` → read it: check `scripts.build`, `scripts.start`, `main`, `type`
- `bun.lockb` → Bun runtime
- Look for framework signals in `dependencies`: `express`, `fastify`, `next`, `nuxt`, `vite`, `@nestjs/core`
- Check for separate frontend: `vite.config.*`, `next.config.*`, `src/` + `server/` coexistence

**Python**
- `requirements.txt` / `pyproject.toml` / `poetry.lock` / `Pipfile`
- Framework signals: `fastapi`, `flask`, `django`, `uvicorn`, `gunicorn`

**Go**
- `go.mod` → read module name and Go version

**Java / Kotlin**
- `pom.xml` → Maven, read `<artifactId>` and Java version
- `build.gradle` / `build.gradle.kts` → Gradle

**Static / Frontend-only**
- Only `index.html` + build tool, no server → nginx-served static

Also detect:
- **Exposed port**: check `EXPOSE` hints in existing Dockerfile, `PORT` env usage in code, framework defaults (Express 3000, FastAPI 8000, Django 8000, Spring 8080, Next.js 3000)
- **Databases**: presence of `pg`, `postgres`, `mysql`, `mongodb`, `redis` in dependencies or env vars → plan companion services
- **Build output directory**: `dist/`, `build/`, `.next/`, `out/`, `dist-server/`

Use Glob to scan: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Dockerfile`, `.env*`

---

## Step 2 — Determine App Name

1. If `$ARGUMENTS` is non-empty, use it as the app name (slugified: lowercase, hyphens)
2. Otherwise, derive from the current working directory name (basename, slugified)

The app name is used as:
- Docker service name
- Container name
- Traefik router/service name
- Subdomain: `<app-name>.localhost`

---

## Step 3 — Generate Dockerfile

Write a production-optimized, multi-stage `Dockerfile` appropriate for the detected stack.

### Node.js (Express/Fastify/custom server)

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# ── Runtime ───────────────────────────────────────────────────────────────────
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist
# Add other built dirs as needed (dist-server, public, etc.)

EXPOSE <PORT>

CMD ["node", "dist/index.js"]
```

Adjust `COPY --from=builder` lines based on actual build output dirs found in step 1.
If there's a separate server build (e.g. `dist-server/`), copy both.

### Node.js (Next.js)

Use standalone output mode pattern:
```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
EXPOSE 3000
CMD ["node", "server.js"]
```

### Python (FastAPI / uvicorn)

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Python (Django / gunicorn)

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "2"]
```

### Go

```dockerfile
FROM golang:1.23-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server ./cmd/server

FROM alpine:3.20

WORKDIR /app
COPY --from=builder /app/server .

EXPOSE 8080

CMD ["./server"]
```

### Static (Vite / React build / pure HTML)

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

---

## Step 4 — Generate docker-compose.yml

Always include:
- `restart: unless-stopped`
- Traefik labels on the main service
- `traefik` external network + `internal` bridge network
- Companion services for detected databases

**Traefik label pattern** (substitute APP_NAME and PORT):
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.<APP_NAME>.rule=Host(`<APP_NAME>.localhost`)"
  - "traefik.http.routers.<APP_NAME>.entrypoints=web"
  - "traefik.http.services.<APP_NAME>.loadbalancer.server.port=<PORT>"
```

**Networks block** (always at bottom of compose file):
```yaml
networks:
  traefik:
    external: true
  internal:
    driver: bridge
```

**PostgreSQL companion** (if detected):
```yaml
  db:
    image: postgres:16-alpine
    container_name: <APP_NAME>-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - ./data/db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - internal
```

**Redis companion** (if detected):
```yaml
  redis:
    image: redis:7-alpine
    container_name: <APP_NAME>-redis
    restart: unless-stopped
    volumes:
      - ./data/redis:/data
    networks:
      - internal
```

The main service should:
- Connect to `traefik` network (for reverse proxy) AND `internal` network (for database access)
- Use `depends_on` with health check condition for database services
- Pass `DATABASE_URL` or equivalent env var pointing to the companion service

---

## Step 5 — Generate .env.example

List all environment variables referenced in `docker-compose.yml` with sensible placeholder values. Example:

```env
# Database
POSTGRES_DB=myapp
POSTGRES_USER=myapp
POSTGRES_PASSWORD=changeme

# App
NODE_ENV=production
```

Do NOT generate `.env` — only `.env.example`.

---

## Step 6 — Check for Existing Files

Before writing any file, check if it already exists with Glob. If it does:
- Show the existing content
- Ask the user whether to overwrite, merge, or skip
- Never silently overwrite existing `Dockerfile` or `docker-compose.yml`

---

## Step 7 — Output Summary

After writing all files, print:

```
Files generated:
  Dockerfile
  docker-compose.yml
  .env.example  (copy to .env and fill in secrets)

App URL: http://<APP_NAME>.localhost
Deploy:  docker compose up --build -d

Traefik must be running with an external network named 'traefik'.
```

If any companion services were added, note their connection strings and that they are only reachable inside the `internal` network.
