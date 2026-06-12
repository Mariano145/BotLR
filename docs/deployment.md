# BotLR Deployment

## Prerequisites

- Docker 24.x+
- Docker Compose 2.20+
- Node.js 18 LTS (frontend builds)
- Python 3.11+ (backend runtime)
- Git 2.40+

---

## Local Development

```bash
# 1. Start services
up docker compose -f docker-compose.dev.yml up -d

# 2. Run migrations
alembic upgrade head

# 3. View logs
docker compose -f docker-compose.dev.yml logs -f backend

# 4. Stop
docker compose -f docker-compose.dev.yml down -v
```

**Env vars (`.env`):**

| Variable | Example | Purpose |
|----------|---------|---------|
| `DATABASE_URL` | `postgresql+asyncpg://botlr:pass@localhost:5432/botlr_dev` | Postgres connection |
| `REDIS_HOST` | `localhost` | Redis host |
| `SECRET_KEY` | `change-me-in-prod-32-chars` | JWT signing |
| `APP_PORT` | `8000` | FastAPI port |
| `NEXT_PUBLIC_API_URL` | `http://localhost:8000/v1` | Frontend API base |
| `NEXT_PUBLIC_SSE_URL` | `http://localhost:8000/v1/events/orders` | Frontend SSE endpoint |

| Service | URL |
|---------|-----|
| Dashboard | http://localhost:3000 |
| API | http://localhost:8000/v1 |
| API docs | http://localhost:8000/docs |
| Health | http://localhost:8000/v1/health |

---

## Production

- Use Docker secrets or cloud secret manager (AWS Secrets Manager, GCP Secret Manager)
- Run behind reverse proxy (Caddy or Nginx) with automatic HTTPS
- JWT secret must be 32+ random characters
- PostgreSQL credentials rotated every 90 days
- Enable webhook signature verification
- Container images scanned in CI (Trivy / Snyk)

---

## CI/CD

```
┌───────┐     ┌───────┐      ┌───────┐      ┌───────┐      ┌───────┐
│ Push  │ ──▶ │ Lint  │ ──▶ │ Test  │ ──▶ │ Build  │ ──▶ │ Deploy│
│(feat) │     │(ruff, │      │(pytest│      │(Docker│      │(stage/│
│       │     │eslint)│      │, jest)│      │images)│      │ prod) │
└───────┘     └───────┘      └───────┘      └───────┘      └───────┘
    │                                           │
    ▼                                          ▼
 PR to develop                            Merge to main
 (code review)                           (triggers prod)
```

- **Lint:** `ruff` (backend), `eslint` (frontend)
- **Test:** `pytest` with ≥ 80% coverage, `jest` with ≥ 70%
- **Build:** Docker images pushed to GHCR
- **Deploy:** Webhook or SSH to staging (`develop`) / production (`main`)

---

## Monitoring

- **Health:** `GET /health` (liveness) and `GET /health/ready` (readiness)
- **Logs:** Structured JSON logs; `LOG_LEVEL=INFO`, `LOG_FORMAT=json`
- **Metrics:** Sentry for error tracking; Grafana + Prometheus (or Datadog) for metrics
- **Alerts:**
  - Critical: > 5% 5xx in 5 min, DB connection failures
  - Warning: > 10% SSE drops, webhook p99 > 2s, disk > 80%

---

## Rollback

```bash
# App rollback: pin previous image in docker-compose.prod.yml, then:
docker compose -f docker-compose.prod.yml up -d

# DB rollback: snapshot first, then:
alembic downgrade -1

# Full rebuild: stop, restore snapshot, pin previous image, start:
docker compose -f docker-compose.prod.yml down
pg_restore -d botlr_prod backup.dump
# edit image tag
docker compose -f docker-compose.prod.yml up -d
```

---

*End of Deployment Guide*
