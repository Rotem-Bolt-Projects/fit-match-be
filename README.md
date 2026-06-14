# FitMatch

A secure peer-to-peer second-hand clothing platform — step-by-step lab for the secure development course (Java/Spring Boot).

Labs 00–03 (setup, authentication, JWT, permissions) are implemented. Later labs add listings, SSRF, file upload, and more.

## Lab order (current scope)

| # | Topic | Walkthrough |
|---|-------|-------------|
| 00 | Setup from scratch | [walkthroughs/00-setup-from-scratch.md](walkthroughs/00-setup-from-scratch.md) |
| 01 | Authentication | [walkthroughs/01-authentication.md](walkthroughs/01-authentication.md) |
| 02 | JWT authentication | [walkthroughs/02-jwt.md](walkthroughs/02-jwt.md) |
| 03 | Permissions | [walkthroughs/03-permissions.md](walkthroughs/03-permissions.md) |

## Project structure

```
fit-match-be/
├── frontend/          # Plain HTML/CSS/JS — served by the BFF
├── api/               # Spring Boot REST API — port 8080, JWT Bearer
├── bff/               # Spring Boot BFF — port 8081, session cookies
├── localstack/init/   # DB + JWT secrets in LocalStack
├── docker-compose.yml
└── walkthroughs/
```

**Browser:** http://localhost:8081 (BFF — JWTs stay server-side).

**Direct API:** http://localhost:8080 with `Authorization: Bearer <token>`.

## Prerequisites

- Docker daemon running (e.g. Colima: `colima start`)
- **`docker-compose`** CLI (not `docker compose` — see [QUICKSTART.md](QUICKSTART.md))
- Java 17+ (Java 21 recommended)

## Quick setup

See [QUICKSTART.md](QUICKSTART.md). **Default:** reuse Postgres (5432) and LocalStack (4566) already used by other labs — run `scripts/seed-fitmatch-on-existing-stack.sh` only; no changes to other projects.

```bash
POSTGRES_CONTAINER=securetask_java-postgres-1 ./scripts/seed-fitmatch-on-existing-stack.sh
# Terminal 1 — API
AWS_REGION=us-east-1 AWS_ACCESS_KEY_ID=test AWS_SECRET_ACCESS_KEY=test \
SECRETS_MANAGER_ENDPOINT=http://localhost:4566 \
DB_SECRET_NAME=fitmatch/db JWT_SECRET_NAME=fitmatch/jwt \
BFF_SERVICE_TOKEN=fitmatch-bff-token \
./gradlew :api:bootRun
# Terminal 2 — BFF
./gradlew :bff:bootRun
```

Open http://localhost:8081 — register with full name, email, and password. First user is **ADMIN**; others are **ACTIVE_USER**.

**Permissions (Lab 03):** Dashboard links to **Wishlist** (own items only), **Style Tips** (read all; admin edits), and **Admin Panel** (admin only).

## Secrets (LocalStack)

| Secret | Purpose |
|--------|---------|
| `fitmatch/db` | PostgreSQL (host `localhost`) |
| `fitmatch/db-docker` | PostgreSQL (Docker network) |
| `fitmatch/jwt` | HS256 JWT signing key |

## Tests

```bash
./gradlew :api:test
```

Requires Docker (Testcontainers PostgreSQL).

## Frontend repo

Auth UI lives in `frontend/` here. [fit-match-fe](https://github.com/) will host React later — see that repo’s README.
