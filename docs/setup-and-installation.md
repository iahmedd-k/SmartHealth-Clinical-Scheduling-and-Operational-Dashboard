# Setup and Installation

This guide covers local environment setup and running the SmartHealth project.

## Prerequisites

- Python 3.11+
- Docker and Docker Compose (recommended for full stack)
- Node.js 18+ and npm (for frontend)
- Git

## Project Structure

```
SmartHealth/
  backend/          # FastAPI application
  frontend/         # React dashboard
  docs/             # Project documentation
```

## Option 1: Docker Compose (Recommended)

The full stack (PostgreSQL, Redis, Kafka, Zookeeper, Temporal, API, Celery, Prometheus) can be started with Docker Compose.

### 1. Clone the repository

```bash
git clone https://github.com/iahmedd-k/Clinical-Scheduling-and-Operational-Dashboard-FastApi-React.git
cd Clinical-Scheduling-and-Operational-Dashboard-FastApi-React
```

### 2. Configure environment

Copy the example environment file:

```bash
# Windows (PowerShell)
copy .env.example .env

# Linux/macOS
cp .env.example .env
```

Update values in `.env` as needed. The `.env.example` contains safe local defaults.

### 3. Start the stack

```bash
docker compose up --build
```

This starts:
- PostgreSQL (pgvector)
- Redis
- Kafka + Zookeeper
- Temporal + Temporal UI
- API service
- Celery worker + beat
- Temporal worker
- Analytics consumer
- Prometheus

### 4. Access the API

Open `http://localhost:8000/docs` for Swagger UI.

The API applies migrations automatically on startup via the `migrate` service.

### 5. Seed data (optional)

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

## Option 2: Manual Local Setup

### 1. Backend

#### Install dependencies

```bash
cd backend
pip install -r requirements.txt
```

#### Configure environment

```bash
copy .env.example .env    # Windows
cp .env.example .env      # Linux/macOS
```

#### Start supporting infrastructure

Use Docker Compose for infrastructure only:

```bash
docker compose up -d postgres redis kafka temporal
```

#### Apply migrations

```bash
alembic upgrade head
```

#### Start the API

```bash
uvicorn app.main:app --reload
```

The API runs at `http://localhost:8000`.

#### Start background workers

```bash
# Celery worker
celery -A app.workers.celery_app worker --loglevel=info

# Celery beat (for scheduled reminders)
celery -A app.workers.celery_app beat --loglevel=info

# Temporal worker
python -m app.workers.temporal.worker

# Kafka analytics consumer
python -m app.workers.kafka
```

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at `http://localhost:5173` by default (Vite).

## Verification

| Check | Command / URL |
|-------|---------------|
| API health | `http://localhost:8000/health` |
| Swagger docs | `http://localhost:8000/docs` |
| Prometheus metrics | `http://localhost:8000/metrics` |
| Temporal UI | `http://localhost:8080` |
| Prometheus targets | `http://localhost:9090/targets` |

## Stopping the Stack

```bash
docker compose down
```

To remove volumes (database data will be lost):

```bash
docker compose down -v