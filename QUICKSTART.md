# Quick Start — FitMatch

## Prerequisites

- **Java 17+** (21 recommended)
- **Docker** daemon running (e.g. Colima: `colima start`)
- **`docker-compose`** (standalone CLI), not `docker compose`

Colima: Gradle tests set `DOCKER_HOST` automatically. For manual `docker-compose`:

```bash
export DOCKER_HOST=unix://$HOME/.colima/default/docker.sock
```

---

## Recommended: reuse your existing Postgres + LocalStack

If another project (e.g. SecureTask) already runs **Postgres on 5432** and **LocalStack on 4566**, you do **not** need a second stack or different ports. FitMatch only adds its own database and secrets — it does not change other projects.

### 1 — Seed FitMatch on the running stack

With SecureTask (or similar) already up:

```bash
chmod +x scripts/seed-fitmatch-on-existing-stack.sh

# LocalStack needs dummy AWS credentials for the CLI (same as bootRun)
AWS_ACCESS_KEY_ID=test AWS_SECRET_ACCESS_KEY=test \
./scripts/seed-fitmatch-on-existing-stack.sh
```

If your Postgres runs in Docker (typical for SecureTask):

```bash
POSTGRES_CONTAINER=securetask_java-postgres-1 \
PG_SUPERUSER=securetask_user \
PG_SUPERPASS=securetask_password \
./scripts/seed-fitmatch-on-existing-stack.sh
```

Adjust `POSTGRES_CONTAINER` to your container name (`docker ps`).

### 2 — Terminal 1: API

```bash
AWS_REGION=us-east-1 \
AWS_ACCESS_KEY_ID=test \
AWS_SECRET_ACCESS_KEY=test \
SECRETS_MANAGER_ENDPOINT=http://localhost:4566 \
DB_SECRET_NAME=fitmatch/db \
JWT_SECRET_NAME=fitmatch/jwt \
BFF_SERVICE_TOKEN=fitmatch-bff-token \
./gradlew :api:bootRun
```

### 3 — Terminal 2: BFF

If **Burp Suite** (or similar) listens on `127.0.0.1:8080`, use IPv6 for the API URL (default in `application.properties`):

```bash
BFF_SERVICE_TOKEN=fitmatch-bff-token ./gradlew :bff:bootRun
```

If login still fails, set explicitly: `API_BASE_URL=http://[::1]:8080 ./gradlew :bff:bootRun`

### 4 — Browser

**http://localhost:8081**

---

## Alternative: FitMatch-only infrastructure

Use this only when **nothing else** is bound to 5432/4566:

```bash
docker-compose --profile infra up -d postgres localstack
```

Then run the API/BFF with the same env vars as above (`SECRETS_MANAGER_ENDPOINT=http://localhost:4566`).

To run API+BFF in Docker as well:

```bash
docker-compose --profile infra --profile app up
```

---

## Tests

Uses Testcontainers (its own temporary Postgres — does not touch your dev databases):

```bash
./gradlew :api:test
```

---

## Ports (local dev)

| Service | Port |
|---------|------|
| BFF | 8081 |
| API | 8080 |
| Postgres (shared) | 5432 |
| LocalStack (shared) | 4566 |

Stop only FitMatch apps with Ctrl+C. Do not `docker-compose down` on another project's stack.
