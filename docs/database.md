# Database Documentation

SmartHealth uses PostgreSQL 16 with the pgvector extension for vector embeddings. Database versioning is managed with Alembic. The ORM is SQLAlchemy 2.0.

## Connection

The database connection is configured via `DATABASE_URL`:

```
postgresql+psycopg://appuser:apppassword@postgres:5432/appdb
```

In local/test/development environments without a `DATABASE_URL`, the app falls back to SQLite (`sqlite+pysqlite:///./app.db`).

## Entity-Relationship Diagram

```mermaid
erDiagram
    User ||--o{ Patient : "has"
    User ||--o{ Provider : "has"
    User ||--o{ Appointment : "books"
    Department ||--o{ Provider : "contains"
    Department ||--o{ Service : "offers"
    Service ||--o{ Slot : "has"
    Service ||--o{ ContentChunk : "indexed"
    Slot ||--o{ Appointment : "booked into"
    Slot ||--o{ SlotReservation : "reserved"
    Provider ||--o{ Slot : "provides"
    Appointment ||--o{ Visit : "has"
    Appointment ||--o{ Billing : "has"
    Appointment ||--o{ Notification : "triggers"
    Appointment ||--o{ WaitlistEntry : "waitlisted"
    Appointment ||--o{ AuditLog : "audited"
    Appointment ||--o{ AIInteraction : "generates"
    Appointment ||--o{ GeneratedContent : "produces"
```

## Tables

### Core Domain

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `users` | User accounts with roles | id, email, password_hash, role, created_at |
| `patients` | Patient demographics | id, user_id, date_of_birth, phone, address |
| `providers` | Provider profiles | id, user_id, department_id, specialization, license_number |
| `departments` | Department catalog | id, name, description, location |
| `services` | Service offerings | id, department_id, name, description, price, duration_minutes, status |
| `slots` | Availability slots | id, service_id, provider_id, start_time, end_time, status |
| `slot_reservations` | Temporary slot reservations | id, slot_id, patient_id, expires_at, status |
| `appointments` | Booked appointments | id, slot_id, patient_id, provider_id, service_id, status, created_at |
| `appointment_status_history` | Status transition audit | id, appointment_id, from_status, to_status, changed_at |
| `visits` | Visit tracking | id, appointment_id, check_in_at, start_at, complete_at, status |
| `billings` | Billing records | id, appointment_id, amount, status, checked_at |
| `waitlist_entries` | Waitlist for unavailable slots | id, slot_id, patient_id, status, joined_at |
| `notifications` | Appointment reminders | id, appointment_id, type, status, scheduled_at, sent_at |

### Observability & Events

| Table | Description |
|-------|-------------|
| `audit_logs` | Audit trail across operations |
| `outbox_events` | Outbox pattern events for reliable messaging |
| `processed_events` | Deduplicated processed Kafka events |
| `failed_jobs` | Failed background job records |
| `idempotency_keys` | Idempotency keys for safe retries |

### AI & Search

| Table | Description |
|-------|-------------|
| `content_chunks` | Vector-indexed content chunks for semantic search |
| `ai_interactions` | AI assistant conversation logs |
| `generated_content` | LLM-generated appointment summaries and drafts |

### Analytics

| Table | Description |
|-------|-------------|
| `analytics_daily` | Daily operational counters |
| `analytics_appointment_daily` | Per-day appointment aggregates |
| `analytics_service_daily` | Per-day service aggregates |
| `analytics_processed_event` | Processed event analytics |

## Relationships

- **User → Patient/Provider**: One user can have one patient profile and/or one provider profile.
- **Department → Provider/Service**: A department contains multiple providers and services.
- **Service → Slot**: A service has many availability slots.
- **Slot → Appointment**: A slot can be booked into one appointment.
- **Appointment → Visit/Billing/Notification**: One appointment generates visit, billing, and notification records.
- **Appointment → AIInteraction/GeneratedContent**: AI interactions and generated content are linked to appointments.

## Indexes

Indexes are defined in migration `018_add_query_indexes.py` and subsequent migrations. Key indexed columns include:

- `appointments(status, patient_id, provider_id, created_at)`
- `slots(status, start_time, service_id, provider_id)`
- `services(department_id, status)`
- `notifications(status, scheduled_at)`
- `audit_logs(appointment_id, created_at)`
- `outbox_events(processed, created_at)`
- `content_chunks` (pgvector embedding index)

## Migrations

Alembic migrations are stored in `backend/alembic/versions/`. There are 29 migrations:

| Migration | Description |
|-----------|-------------|
| `001_initial_schema.py` | Core users, departments, providers, services, slots |
| `002_add_appointments.py` | Appointments and status history |
| `003_add_billings.py` | Billing records |
| `004_add_services_status.py` | Service status tracking |
| `005_add_pgvector_embeddings.py` | pgvector extension and embeddings |
| `006_add_service_chunk_metadata.py` | Content chunk metadata |
| `007_add_waitlist.py` | Waitlist entries |
| `008_complete_demo_schema.py` | Demo schema completion |
| `009_add_audit_logs.py` | Audit log table |
| `010_add_booking_statuses.py` | Additional booking statuses |
| `011_add_outbox_events.py` | Outbox event table |
| `012_allow_rebooking_cancelled_slots.py` | Rebooking support |
| `013_add_vector_metadata.py` | Vector metadata |
| `014_add_chunk_content_hash.py` | Content chunk hashing |
| `015_add_service_price.py` | Service pricing |
| `016_add_booking_key.py` | Idempotency booking keys |
| `017_required_domain_models.py` | Required domain models |
| `018_add_query_indexes.py` | Query performance indexes |
| `019_analytics_daily_counters.py` | Daily analytics counters |
| `020_notification_lifecycle.py` | Notification lifecycle fields |
| `021_visit_history_fields.py` | Visit history fields |
| `022_reconcile_model_columns.py` | Column reconciliation |
| `023_update_embedding_dimensions.py` | Embedding dimension updates |
| `024_add_analytics_and_failed_jobs.py` | Analytics and failed jobs |
| `025_add_ai_interaction_question.py` | AI interaction questions |
| `026_add_ai_interaction_telemetry.py` | AI telemetry |
| `027_add_ai_question_text.py` | AI question text |
| `028_add_generated_content_audit.py` | Generated content audit |
| `029_reconcile_ai_constraints.py` | AI constraint reconciliation |

## Running Migrations

```bash
# Apply all migrations
alembic upgrade head

# Create a new migration
alembic revision --autogenerate -m "description"

# Check current revision
alembic current

# Downgrade one revision
alembic downgrade -1
```

## Connection Pool Settings

Configured via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_POOL_SIZE` | 5 | Connection pool size |
| `DB_MAX_OVERFLOW` | 10 | Max overflow connections |
| `DB_POOL_TIMEOUT` | 30 | Pool checkout timeout (seconds) |
| `DB_POOL_RECYCLE` | 1800 | Connection recycle time (seconds) |

## Docker Database

PostgreSQL runs as a service in `docker-compose.yml` using the `pgvector/pgvector:pg16` image. Environment variables `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` must be set.

Data is persisted in the `pgdata` Docker volume.
