# ADR 001: Layered Architecture with FastAPI

## Status

Accepted

## Context

SmartHealth needed a modular backend for healthcare scheduling and operations. The system required clear separation between HTTP handling, business logic, data access, and async workflow orchestration. Multiple infrastructure concerns (PostgreSQL, Redis, Kafka, Temporal, Celery) needed to be integrated without tightly coupling domain logic to infrastructure.

## Decision

Adopt a layered architecture using FastAPI with the following layers:

1. **API Layer** (`app/api/v1/endpoints/`) — HTTP routing, authentication, request/response handling
2. **Service Layer** (`app/services/`) — Business logic and workflow orchestration
3. **Repository Layer** (`app/repositories/`) — Data access via SQLAlchemy ORM
4. **Schema Layer** (`app/schemas/`) — Pydantic validation
5. **Workflow Layer** (`app/workers/temporal/`) — Temporal saga orchestration
6. **Worker Layer** (`app/workers/celery`, `app/workers/kafka`) — Background task execution
7. **Observability Layer** — Structured logging, correlation IDs, Prometheus metrics

## Consequences

### Positive
- Clear separation of concerns makes the codebase maintainable
- Domain logic is testable without infrastructure dependencies
- Infrastructure can be swapped or mocked in tests
- FastAPI provides automatic OpenAPI documentation

### Negative
- Additional abstraction layers increase code volume
- Onboarding requires understanding the layer boundaries

## Alternatives Considered

| Alternative | Reason for Rejection |
|-------------|---------------------|
| Monolithic single-module structure | Would become unmaintainable as domain grows |
| Direct ORM usage in API endpoints | Tightly couples HTTP layer to database, hard to test |
| Hexagonal architecture | Over-engineering for current scope; layered is sufficient |
