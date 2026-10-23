# Deployment

SmartHealth is designed for containerized deployment using Docker Compose. The backend exposes a FastAPI API; the frontend is a static React build served via Vercel or any static host.

## Production Deployment (Docker Compose)

### 1. Prepare production environment

Create a production environment file from the example:

```bash
copy .env.production.example .env.production    # Windows
cp .env.production.example .env.production      # Linux/macOS
```

Update the following values in `.env.production`:

| Variable | Production Value |
|----------|-----------------|
| `APP_ENV` | `production` |
| `JWT_SECRET` | A unique 32+ character secret |
| `LLM_API_KEY` | Production LLM API key |
| `DATABASE_URL` | Production PostgreSQL connection |
| `CORS_ALLOWED_ORIGINS` | Comma-separated allowed origins |
| `KAFKA_BOOTSTRAP_SERVERS` | Production Kafka brokers |
| `TEMPORAL_HOST` | Production Temporal address |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` | Production DB credentials |
| `TEMPORAL_IMAGE_TAG` | Approved Temporal image version |
| `TEMPORAL_UI_IMAGE_TAG` | Approved Temporal UI image version |

The app validates that `JWT_SECRET` is at least 32 characters and not a known demo secret in production. `LLM_API_KEY` must be set.

### 2. Run production stack

Using Make targets:

```bash
make prod-env      # Reminds to create .env.production
make prod-up       # Start stack with .env.production
make prod-migrate  # Run migrations
make prod-down     # Stop stack
```

Or directly with Docker Compose:

```bash
docker compose --env-file .env.production up -d --build
docker compose --env-file .env.production run --rm migrate
```

### 3. Production services

The `docker-compose.yml` defines these services:

| Service | Image | Purpose |
|---------|-------|---------|
| `migrate` | Build from Dockerfile | Runs `alembic upgrade head` |
| `api` | Build from Dockerfile | FastAPI application (port 8000) |
| `celery-worker` | Build from Dockerfile | Background task worker |
| `celery-beat` | Build from Dockerfile | Scheduled task scheduler |
| `temporal-worker` | Build from Dockerfile | Temporal workflow worker |
| `analytics-consumer` | Build from Dockerfile | Kafka event consumer |
| `prometheus` | `prom/prometheus:v2.53.0` | Metrics scraping (port 9090) |
| `postgres` | `pgvector/pgvector:pg16` | Primary database |
| `temporal` | `temporalio/auto-setup` | Workflow engine |
| `temporal-ui` | `temporalio/ui` | Temporal UI (port 8080) |
| `redis` | `redis:7-alpine` | Cache and idempotency |
| `kafka` | `confluentinc/cp-kafka:7.5.0` | Event streaming |
| `zookeeper` | `confluentinc/cp-zookeeper:7.5.0` | Kafka coordination |

### 4. Health checks

| Service | Health Check |
|---------|-------------|
| API | `GET http://127.0.0.1:8000/health/ready` |
| Celery | `celery inspect ping` |
| PostgreSQL | `pg_isready` |
| Redis | `redis-cli ping` |
| Kafka | `kafka-topics --list` |

## CI/CD

The repository includes a GitHub Actions workflow at `backend/.github/workflows/ci.yml`.

### CI Pipeline

The workflow runs on push and pull requests to `main`:

1. Set up Python
2. Install dependencies from `requirements.txt`
3. Run linting with `ruff`
4. Run tests with `pytest`

### Recommended Production Pipeline

Add the following stages to your CI/CD:

1. **Lint**: `ruff check app tests scripts`
2. **Test**: `python -m pytest -q`
3. **Build**: `docker compose build`
4. **Migrate**: `docker compose --env-file .env.production run --rm migrate`
5. **Deploy**: `docker compose --env-file .env.production up -d`

## Security Notes for Production

- Never commit `.env.production` with real secrets to version control.
- Use a secrets manager (HashiCorp Vault, AWS Secrets Manager) instead of `.env` files.
- Enable TLS termination at the ingress/load balancer; the app uses `HTTPSRedirectMiddleware` in production.
- Kafka is deployed PLAINTEXT/no-auth by default; enable SASL/TLS for production.
- Rotate `JWT_SECRET` regularly.

## Frontend Deployment (Vercel)

The frontend is configured for Vercel via `vercel.json`.

```json
{
  "version": 2,
  "builds": [{ "src": "package.json", "use": "@vercel/static-build" }]
}
```

Deploy by connecting the repository to Vercel. Set the API base URL in `frontend/src/api.js` to your production backend URL.

## Resource Sizing

Default container limits in `docker-compose.yml`:

| Service | Memory | CPU |
|---------|--------|-----|
| API | 512m | 1.0 |
| Celery worker | 512m | 1.0 |
| Celery beat | 256m | 0.5 |
| Temporal worker | 512m | 1.0 |
| Analytics consumer | 512m | 1.0 |

Adjust based on production load.
