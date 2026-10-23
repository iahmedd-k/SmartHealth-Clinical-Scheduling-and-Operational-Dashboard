# Requirements

## Functional Requirements

### Authentication & Authorization
- **FR-01**: Users can register with roles: patient, provider, front_desk, admin.
- **FR-02**: Users can log in and receive a JWT access token.
- **FR-03**: Role-based access control restricts endpoints by user role.
- **FR-04**: Resource ownership guards prevent unauthorized access to user-specific data.

### Patient Management
- **FR-05**: Patients can create and manage their profiles.
- **FR-06**: Patients can view available services and providers.

### Provider Management
- **FR-07**: Providers can manage their profiles and schedules.
- **FR-08**: Providers can view their appointments.

### Department Management
- **FR-09**: Administrators can create and list departments.
- **FR-10**: Departments group providers and services.

### Service Catalog
- **FR-11**: Services can be created with name, description, price, and duration.
- **FR-12**: Services can be published/unpublished to the public catalog.
- **FR-13**: Public users can view published services without authentication.

### Slot Management
- **FR-14**: Providers can create availability slots.
- **FR-15**: Slots track status (available, booked, etc.).

### Appointment Management
- **FR-16**: Patients can book appointments through a saga that validates and reserves slots.
- **FR-17**: Idempotency keys prevent duplicate booking requests.
- **FR-18**: Appointments can be cancelled and rescheduled.
- **FR-19**: Visit lifecycle supports check-in, start, and complete.
- **FR-20**: Appointment state is queryable at any time.

### Waitlist
- **FR-21**: Patients can join a waitlist when a slot is unavailable.

### Billing
- **FR-22**: Billing pre-check supports mock approve/fail based on configuration.
- **FR-23**: Billing records are linked to appointments.

### Notifications
- **FR-24**: Appointment reminders are scheduled via Celery Beat every 15 minutes.
- **FR-25**: Notification status is tracked (PENDING → SENT → CANCELLED).
- **FR-26**: External email/SMS delivery is out of scope.

### AI Assistant
- **FR-27**: Patients and staff can ask healthcare questions via the assistant.
- **FR-28**: Responses undergo medical advice refusal checks (no diagnosis/treatment advice).
- **FR-29**: PHI (Protected Health Information) is scoped in assistant responses.
- **FR-30**: AI interactions are logged with refusal status and latency.
- **FR-31**: Operational utilisation reports can be generated.

### Search
- **FR-32**: Semantic search for services using vector embeddings (pgvector).
- **FR-33**: Hybrid keyword + vector search.

### Observability
- **FR-34**: Structured JSON logs with correlation IDs across HTTP, Celery, and workflow boundaries.
- **FR-35**: Prometheus metrics endpoint with domain counters.
- **FR-36**: Audit trail across all operations.

### Workflow Orchestration
- **FR-37**: Temporal workflows handle multi-step sagas (booking, service publishing) with compensation.
- **FR-38**: Celery workers process background tasks (reminders, analytics, outbox).

### Event Streaming
- **FR-39**: Kafka publishes domain events with PHI sanitization.
- **FR-40**: Outbox pattern ensures reliable event delivery.

## Non-Functional Requirements

### Performance
- **NFR-01**: API responds to health checks within acceptable latency.
- **NFR-02**: Database connection pooling configurable for production load.
- **NFR-03**: Celery Beat processes reminders every 15 minutes.

### Scalability
- **NFR-04**: Horizontal scaling supported via containerized services.
- **NFR-05**: Redis-backed idempotency supports distributed retries.

### Reliability
- **NFR-06**: Degraded mode allows API startup without Kafka.
- **NFR-07**: Outbox pattern provides reliable event delivery.
- **NFR-08**: Temporal sagas support compensation on failure.

### Security
- **NFR-09**: JWT authentication with configurable expiry.
- **NFR-10**: Role-based authorization on all authenticated routes.
- **NFR-11**: Security headers middleware in production.
- **NFR-12**: HTTPS redirect in production.
- **NFR-13**: PHI sanitization in Kafka events and AI responses.
- **NFR-14**: Production requires strong JWT_SECRET (32+ chars) and LLM_API_KEY.

### Maintainability
- **NFR-15**: Alembic migrations version the database schema (29 migrations).
- **NFR-16**: Layered architecture separates API, service, repository, and workflow concerns.
- **NFR-17**: Pydantic schemas validate all request/response contracts.

### Testability
- **NFR-18**: FakeLLM enables offline AI testing without API keys.
- **NFR-19**: Unit and integration test suites cover core domains.
- **NFR-20**: Linting with ruff enforces code quality.

### Operability
- **NFR-21**: Docker Compose orchestrates all infrastructure services.
- **NFR-22**: Health checks on all major services.
- **NFR-23**: Prometheus scraping for metrics and alerting.
- **NFR-24**: Makefile provides common development and production targets.

### Compliance
- **NFR-25**: Audit logs capture administrative and operational actions.
- **NFR-26**: No real payment processing (mock billing pre-check only).
- **NFR-27**: No real email/SMS delivery (database tracking only).

## Out of Scope

The following are intentionally not implemented:

| Feature | Reason |
|---------|--------|
| Real email/SMS delivery | Out of scope; reminders tracked in DB only |
| Real payment processing | Out of scope; billing pre-check is mock |
| Calendar sync (Google/Outlook) | Out of scope; appointments local only |
| Document parsing / OCR | Out of scope; text-based content only |
| Multi-clinic tenancy | Out of scope; single clinic scope |
| Clinical records (EHR) | Out of scope; no medical record storage |
| Provider time-off / vacation | Out of scope |
| Production secrets management | Out of scope; use a vault in production |

## Acceptance Criteria

- All unit tests pass with `pytest -q`.
- Docker Compose stack starts and health checks pass.
- API documentation available at `/docs`.
- Migrations apply cleanly with `alembic upgrade head`.
- Linting passes with `ruff check app tests scripts`.
