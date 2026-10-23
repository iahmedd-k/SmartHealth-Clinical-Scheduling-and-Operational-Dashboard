# SmartHealth — Clinical Scheduling and Operational Dashboard

SmartHealth is a FastAPI backend and React frontend for clinical scheduling and healthcare operations. The platform supports patient booking, provider and service management, appointment lifecycle tracking, slot publishing and reservation, billing pre-checks, notifications, analytics, and an AI-powered healthcare assistant with safety controls.

## Repository Structure

```
SmartHealth/
├── backend/                 # FastAPI application
│   ├── app/                 # Application source
│   │   ├── api/v1/endpoints/# HTTP route handlers
│   │   ├── core/            # Settings, middleware, security, logging, metrics
│   │   ├── db/              # Database session and engine
│   │   ├── models/          # SQLAlchemy ORM models
│   │   ├── repositories/    # Data access layer
│   │   ├── schemas/         # Pydantic validation schemas
│   │   ├── services/        # Business logic layer
│   │   └── workers/         # Celery, Kafka, Temporal workers
│   ├── alembic/versions/    # Database migrations
│   ├── tests/               # Unit and integration tests
│   ├── scripts/             # Seed and utility scripts
│   ├── docker-compose.yml   # Multi-service orchestration
│   ├── Dockerfile           # API container image
│   └── requirements.txt     # Python dependencies
├── frontend/                # React dashboard (Vite + Tailwind)
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   └── views/           # Role-based dashboard views
│   └── package.json
└── docs/                    # Project documentation
    ├── architecture.md          # System architecture and data flow
    ├── api-documentation.md     # Endpoints, examples, authentication
    ├── database.md              # ERD, tables, migrations
    ├── setup-and-installation.md# Local environment setup
    ├── deployment.md            # Production deployment, Docker, CI/CD
    ├── testing.md               # Unit, integration, E2E testing
    ├── environment-variables.md # Required env variables
    ├── requirements.md          # Functional and non-functional requirements
    ├── security.md              # Authentication, authorization, secrets
    ├── decisions/               # Architecture Decision Records
    └── diagrams/                # Architecture and ERD diagrams
```

## Technology Stack

| Layer | Technology |
|-------|-----------|
| API framework | FastAPI 0.115.6 |
| ASGI server | Uvicorn 0.32.1 |
| ORM | SQLAlchemy 2.0.36 |
| Database | PostgreSQL 16 + pgvector |
| Migrations | Alembic 1.14.0 |
| Cache / idempotency | Redis 7 |
| Background jobs | Celery 5.5.3 |
| Event streaming | Kafka 7.5.0 + Zookeeper |
| Workflow orchestration | Temporal 1.31.0 |
| LLM provider | Groq / OpenAI |
| Embeddings | HuggingFace (pgvector) |
| Monitoring | Prometheus |
| Frontend | React + Vite + Tailwind CSS |
| Containerization | Docker / Docker Compose |

## Core Capabilities

- Secure authentication and role-based authorization (patient, provider, front_desk, admin)
- Patient, provider, department, and service management
- Slot publishing and reservation workflows
- Appointment booking, rescheduling, cancellation, and visit tracking
- Billing pre-check support (mock implementation)
- Asynchronous workflow processing through Temporal and Celery
- Structured logging, correlation IDs, and Prometheus metrics
- Appointment reminder tracking (database-only; external delivery out of scope)
- AI healthcare assistant with LLM-powered responses and safety checks
- Audit trail and observability across all operations

## Quick Start

### Option 1: Docker Compose (Recommended)

```bash
# Clone the repository
git clone https://github.com/iahmedd-k/Clinical-Scheduling-and-Operational-Dashboard-FastApi-React.git
cd Clinical-Scheduling-and-Operational-Dashboard-FastApi-React

# Configure environment
copy .env.example .env       # Windows
cp .env.example .env         # Linux/macOS

# Start the full stack
docker compose up --build
```

The stack starts PostgreSQL, Redis, Kafka, Zookeeper, Temporal, the API, Celery, the Temporal worker, the analytics consumer, and Prometheus.

Open `http://localhost:8000/docs` to inspect the API via Swagger UI.

### Option 2: Manual Backend Setup

```bash
cd backend
pip install -r requirements.txt
copy .env.example .env       # Windows
cp .env.example .env         # Linux/macOS

# Start infrastructure (PostgreSQL, Redis, Kafka, Temporal)
docker compose up -d postgres redis kafka temporal

# Apply migrations
alembic upgrade head

# Start the API
uvicorn app.main:app --reload

# Start background workers
celery -A app.workers.celery_app worker --loglevel=info
celery -A app.workers.celery_app beat --loglevel=info
python -m app.workers.temporal.worker
```

### Option 3: Frontend

```bash
cd frontend
npm install
npm run dev
```

## Seed Data

Populate sample records for local development:

```bash
docker compose run --rm api python scripts/seed.py
```

Sample accounts:
- `admin@example.com` / `secret123`
- `provider@example.com` / `secret123`
- `patient@example.com` / `secret123`

To create only demo login users:

```bash
docker compose run --rm api python -m scripts.seed_users
```

## API Overview

All authenticated routes are versioned under `/api/v1`. Public catalog endpoints require no authentication.

### Authentication
- `POST /auth/register` — create a user account
- `POST /auth/login` — validate credentials, return access token

### Health and Monitoring
- `GET /health` — liveness check
- `GET /health/ready` — readiness check
- `GET /metrics` — Prometheus-formatted metrics
- `GET /docs` — Swagger UI

### Core Domain APIs
- `POST/GET /api/v1/departments` — department catalog
- `POST/GET /api/v1/providers` — provider profiles
- `GET /api/v1/providers/{id}/slots` — provider slots
- `POST/GET /api/v1/services` — service offerings
- `POST /api/v1/services/{id}/publish` — publish service
- `POST/GET /api/v1/slots` — availability slots
- `POST /api/v1/appointments` — book an appointment
- `GET /api/v1/appointments/{id}/state` — appointment state
- `POST /api/v1/appointments/{id}/cancel` — cancel appointment
- `POST /api/v1/appointments/{id}/reschedule` — reschedule appointment
- `POST /api/v1/appointments/{id}/visit/check-in` — check in
- `POST /api/v1/appointments/{id}/visit/start` — start visit
- `POST /api/v1/appointments/{id}/visit/complete` — complete visit
- `GET /api/v1/public/services` — public service catalog

### AI Assistant
- `POST /assistant/ask` — safety-checked healthcare Q&A
- `POST /assistant/report` — utilisation reports

See [docs/api-documentation.md](docs/api-documentation.md) for full endpoint details.

## Environment Variables

Configuration is managed through environment variables loaded from `.env` and optionally `.env.production`.

Key variables:

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | SQLAlchemy database connection |
| `JWT_SECRET` | JWT signing key (32+ chars in production) |
| `TEMPORAL_HOST` | Temporal frontend address |
| `REDIS_URL` | Idempotency store connection |
| `CELERY_BROKER_URL` | Celery broker connection |
| `KAFKA_ENABLED` | Enable Kafka event publishing |
| `LLM_API_KEY` | LLM provider API key (required in production) |
| `USE_FAKE_LLM` | Use deterministic fake LLM for testing |

See [docs/environment-variables.md](docs/environment-variables.md) for the complete reference.

## Testing

Run the test suite:

```bash
cd backend
pytest -q
```

Test suites include unit tests (`tests/unit/`), integration tests (`tests/integration/`), and workflow/concurrency demo tasks (`tests/demo_tasks/`). A deterministic FakeLLM enables offline AI testing.

See [docs/testing.md](docs/testing.md) for details.

## Deployment

### Production (Docker Compose)

```bash
copy .env.production.example .env.production    # Windows
cp .env.production.example .env.production      # Linux/macOS

# Update secrets in .env.production, then:
docker compose --env-file .env.production up -d --build
docker compose --env-file .env.production run --rm migrate
```

Make targets: `make prod-env`, `make prod-up`, `make prod-migrate`, `make prod-down`.

See [docs/deployment.md](docs/deployment.md) for production deployment details.

### CI/CD

GitHub Actions workflow at `backend/.github/workflows/ci.yml` runs linting (`ruff`) and tests (`pytest`) on push and pull requests.

## Documentation

| Document | Description |
|----------|-------------|
| [docs/architecture.md](docs/architecture.md) | System architecture, components, data flow |
| [docs/api-documentation.md](docs/api-documentation.md) | Endpoints, request/response examples, authentication |
| [docs/database.md](docs/database.md) | ERD, tables, relationships, indexes, migrations |
| [docs/setup-and-installation.md](docs/setup-and-installation.md) | Local environment setup |
| [docs/deployment.md](docs/deployment.md) | Production deployment, Docker, CI/CD |
| [docs/testing.md](docs/testing.md) | Unit, integration, end-to-end testing |
| [docs/environment-variables.md](docs/environment-variables.md) | Required environment variables |
| [docs/requirements.md](docs/requirements.md) | Functional and non-functional requirements |
| [docs/security.md](docs/security.md) | Authentication, authorization, secrets, security controls |
| [docs/decisions/](docs/decisions/) | Architecture Decision Records (ADRs) |
| [docs/diagrams/](docs/diagrams/) | Architecture, ERD, sequence, and workflow diagrams |

## Scope and Limitations

### Included
- Complete healthcare scheduling backend with appointment workflows
- Multi-step orchestration via Temporal sagas with compensation
- Event streaming via Kafka with PHI sanitization
- AI assistant with medical advice refusal and PHI scoping
- Role-based and resource-ownership authorization
- Redis-backed idempotency for safe retries
- Structured logging, correlation tracing, and Prometheus metrics

### Out of Scope
- Real email/SMS delivery (reminders tracked in DB only)
- Real payment processing (billing pre-check is mock)
- Calendar sync (Google Calendar, Outlook, iCalendar)
- Document parsing / OCR
- Multi-clinic tenancy
- Clinical records (EHR)
- Provider time-off / vacation scheduling
- Production secrets management (use a vault in production)

## License

MIT License
