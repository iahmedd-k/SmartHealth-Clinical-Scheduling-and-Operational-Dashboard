# Environment Variables

SmartHealth is configured entirely through environment variables. Settings are loaded using `pydantic-settings` from `.env` and optionally `.env.production` (production values override local defaults).

## Loading Order

The `Settings` class loads from:

1. `.env` (required baseline)
2. `.env.production` (optional production overrides)

This is defined in `backend/app/core/settings.py`:

```python
model_config = SettingsConfigDict(
    extra="ignore",
    validate_by_name=True,
    validate_by_alias=True,
    env_file=(".env", ".env.production"),
    env_file_encoding="utf-8",
)
```

## Example Files

| File | Purpose |
|------|---------|
| `backend/.env.example` | Local development defaults |
| `backend/.env.production.example` | Production template |

Copy `.env.example` to `.env` for local development. For production, copy `.env.production.example` to `.env.production` and update secrets.

## Required Environment Variables

### Application

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_ENV` | `local` | Environment name (`local`, `production`, `test`) |
| `BUILD_REVISION` | `development` | Build revision identifier |
| `LOG_LEVEL` | `INFO` | Logging level |
| `API_VERSION` | `1.0.0` | API version |

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | None (SQLite fallback in local) | SQLAlchemy connection string |
| `DB_POOL_SIZE` | 5 | Connection pool size |
| `DB_MAX_OVERFLOW` | 10 | Max overflow connections |
| `DB_POOL_TIMEOUT` | 30 | Pool checkout timeout (seconds) |
| `DB_POOL_RECYCLE` | 1800 | Connection recycle time (seconds) |

### Authentication (JWT)

| Variable | Default | Description |
|----------|---------|-------------|
| `JWT_SECRET` | `change-me-locally` | JWT signing key (32+ chars in production) |
| `JWT_ALGORITHM` | `HS256` | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | 60 | Access token expiry (minutes) |
| `REFRESH_TOKEN_EXPIRE_DAYS` | 7 | Refresh token expiry (days) |

### Temporal

| Variable | Default | Description |
|----------|---------|-------------|
| `TEMPORAL_HOST` | `localhost:7233` | Temporal frontend address |
| `TEMPORAL_NAMESPACE` | `default` | Temporal namespace |
| `TEMPORAL_TASK_QUEUE` | `app-workflow` | Workflow task queue |
| `TEMPORAL_WORKFLOW_TIMEOUT_MINUTES` | 30 | Workflow timeout (minutes) |

### Redis

| Variable | Default | Description |
|----------|---------|-------------|
| `REDIS_URL` | `redis://localhost:6379/0` | Idempotency store connection |

### Celery

| Variable | Default | Description |
|----------|---------|-------------|
| `CELERY_BROKER_URL` | `redis://localhost:6379/1` | Celery broker |
| `CELERY_RESULT_BACKEND` | `redis://localhost:6379/2` | Celery result backend |
| `CELERY_TASK_ALWAYS_EAGER` | False | Execute tasks eagerly (testing) |

### Kafka

| Variable | Default | Description |
|----------|---------|-------------|
| `KAFKA_ENABLED` | false | Enable Kafka event publishing |
| `KAFKA_BOOTSTRAP_SERVERS` | `localhost:9092` | Kafka broker address |
| `KAFKA_CONSUMER_GROUP` | `app-analytics` | Consumer group |
| `KAFKA_TOPIC_PREFIX` | `app` | Topic prefix |
| `KAFKA_DLQ_TOPIC_SUFFIX` | `dlq` | Dead letter queue suffix |
| `KAFKA_CONSUMER_MAX_RETRIES` | 3 | Max consumer retries |

### CORS

| Variable | Default | Description |
|----------|---------|-------------|
| `CORS_ALLOWED_ORIGINS` | [] | Comma-separated allowed origins |

### Rate Limiting

| Variable | Default | Description |
|----------|---------|-------------|
| `AUTH_LOGIN_RATE_LIMIT` | `5/minute` | Login rate limit |
| `AUTH_REGISTER_RATE_LIMIT` | `3/hour` | Register rate limit |
| `AUTH_PASSWORD_RESET_RATE_LIMIT` | `3/hour` | Password reset rate limit |

### Embeddings

| Variable | Default | Description |
|----------|---------|-------------|
| `EMBEDDING_PROVIDER` | `huggingface` | Embedding provider |
| `EMBEDDING_API_KEY` | "" | Embedding API key |
| `EMBEDDING_MODEL` | `microsoft/harrier-oss-v1-0.6b` | Embedding model |
| `EMBEDDING_DIMENSIONS` | 1024 | Embedding dimensions |
| `EMBEDDING_BATCH_SIZE` | 32 | Embedding batch size |

### Retrieval (Semantic Search)

| Variable | Default | Description |
|----------|---------|-------------|
| `RETRIEVAL_TOP_K` | 5 | Max semantic search results |
| `RETRIEVAL_MIN_SIMILARITY` | 0.60 | Minimum similarity threshold |

### LLM (AI Assistant)

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_PROVIDER` | `openai` | LLM provider (`openai`, `groq`) |
| `LLM_API_KEY` | "" | LLM API key (required in production) |
| `LLM_BASE_URL` | `https://api.openai.com/v1` | LLM base URL |
| `LLM_MODEL` | `gpt-4o-mini` | LLM model |
| `LLM_TIMEOUT_SECONDS` | 60 | LLM request timeout (seconds) |
| `USE_FAKE_LLM` | False | Use deterministic fake LLM (testing) |
| `AI_RATE_LIMIT_PER_MINUTE` | 30 | AI requests per minute |
| `AI_CACHE_TTL_SECONDS` | 300 | AI cache TTL (seconds) |
| `AI_CONVERSATION_KEY` | "" | AI conversation key |

### Booking

| Variable | Default | Description |
|----------|---------|-------------|
| `BOOKING_DEMO_PAUSE_SECONDS` | 0 | Demo pause (seconds) |
| `ASYNC_BOOKING_ENABLED` | false | Enable async booking |
| `BOOKING_WORKFLOW_TIMEOUT_MINUTES` | 30 | Booking workflow timeout |
| `BOOKING_FORCE_FAILURE` | false | Force booking failure (testing) |
| `BILLING_FORCE_FAILURE` | false | Force billing failure (testing) |
| `SLOT_RESERVATION_TTL_MINUTES` | 15 | Slot reservation TTL |
| `SERVICE_PUBLISH_STALE_MINUTES` | 30 | Service publish stale threshold |

### Security

| Variable | Default | Description |
|----------|---------|-------------|
| `ALLOW_SELF_SERVICE_ADMIN_REGISTRATION` | false | Allow self-service admin registration |

## Production Example

```env
APP_ENV=production
BUILD_REVISION=prod-build
LOG_LEVEL=INFO
DATABASE_URL=postgresql+psycopg://appuser:apppassword@prod-db:5432/appdb
JWT_SECRET=replace-with-a-32-character-production-secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=7
TEMPORAL_HOST=temporal:7233
TEMPORAL_NAMESPACE=default
TEMPORAL_TASK_QUEUE=app-workflow
TEMPORAL_WORKFLOW_TIMEOUT_MINUTES=30
REDIS_URL=redis://redis:6379/0
CELERY_BROKER_URL=redis://redis:6379/1
CELERY_RESULT_BACKEND=redis://redis:6379/2
CORS_ALLOWED_ORIGINS=https://api.example.com,https://app.example.com
KAFKA_ENABLED=true
KAFKA_BOOTSTRAP_SERVERS=kafka:29092
KAFKA_CONSUMER_GROUP=app-analytics-prod
KAFKA_TOPIC_PREFIX=app
EMBEDDING_PROVIDER=huggingface
EMBEDDING_API_KEY=
EMBEDDING_MODEL=microsoft/harrier-oss-v1-0.6b
EMBEDDING_DIMENSIONS=1024
EMBEDDING_BATCH_SIZE=32
RETRIEVAL_TOP_K=5
RETRIEVAL_MIN_SIMILARITY=0.50
LLM_PROVIDER=groq
LLM_API_KEY=replace-with-production-llm-key
LLM_BASE_URL=https://api.groq.com/openai/v1
LLM_MODEL=llama-3.1-8b-instant
LLM_TIMEOUT_SECONDS=60
USE_FAKE_LLM=false
AI_RATE_LIMIT_PER_MINUTE=30
AI_CACHE_TTL_SECONDS=300
AI_CONVERSATION_KEY=
BOOKING_DEMO_PAUSE_SECONDS=0
ASYNC_BOOKING_ENABLED=false
BOOKING_WORKFLOW_TIMEOUT_MINUTES=30
BOOKING_FORCE_FAILURE=false
BILLING_FORCE_FAILURE=false
AUTH_LOGIN_RATE_LIMIT=5/minute
AUTH_REGISTER_RATE_LIMIT=3/hour
AUTH_PASSWORD_RESET_RATE_LIMIT=3/hour
DB_POOL_SIZE=10
DB_MAX_OVERFLOW=20
DB_POOL_TIMEOUT=30
DB_POOL_RECYCLE=1800
ALLOW_SELF_SERVICE_ADMIN_REGISTRATION=false
```

## Docker Compose Environment

Services load `.env.example` (required) and `.env` (optional). Database credentials use `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` from the shell environment.

Temporal image tags require `TEMPORAL_IMAGE_TAG` and `TEMPORAL_UI_IMAGE_TAG` to be set.

## Validation

The `Settings` class validates in production:
- `JWT_SECRET` must be at least 32 characters and not a known demo secret.
- `LLM_API_KEY` must be set.
- `LLM_PROVIDER` must be `openai` or `groq`.
- `DATABASE_URL` is required outside local/test/development environments.
