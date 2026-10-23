# Testing

SmartHealth includes unit tests, integration tests, and workflow/concurrency demo tasks. Tests use `pytest` with a deterministic `FakeLLM` implementation that requires no network access or API keys.

## Test Structure

```
backend/tests/
  conftest.py              # Shared pytest fixtures
  conftest_llm.py          # FakeLLM fixtures
  unit/                    # Unit tests (no external dependencies)
  integration/             # Integration tests (requires Docker infra)
  demo_tasks/              # Workflow and concurrency exercises
```

## Running Tests

### All tests

```bash
cd backend
pytest -q
```

Or via Make:

```bash
make test
```

### Unit tests only

```bash
pytest tests/unit/ -q
```

### Integration tests

Requires Docker infrastructure running:

```bash
pytest tests/integration/ -q
```

Or with the infra-test Make target:

```bash
make infra-test
```

### AI assistant tests

```bash
pytest tests/unit/test_ai_layer_comprehensive.py -v
```

### Safety and refusal tests

```bash
pytest tests/unit/test_assistant_safety.py -v
```

### Specific test file

```bash
pytest tests/unit/test_authorization.py -v
```

## Test Suites

### Unit Tests (`tests/unit/`)

| Test File | Coverage |
|-----------|----------|
| `test_ai_layer_comprehensive.py` | AI layer with FakeLLM |
| `test_ai_pipeline_regressions.py` | AI pipeline regressions |
| `test_ai_analytics_requirements.py` | AI analytics requirements |
| `test_assistant_safety.py` | Medical advice refusal and safety |
| `test_assistant_graph.py` | Assistant graph workflows |
| `test_authorization.py` | Role-based access control |
| `test_appointment_saga_service.py` | Appointment saga service |
| `test_content_chunks_repository.py` | Content chunk repository |
| `test_embedding_service.py` | Embedding service |
| `test_endpoint_contract_checklist.py` | Endpoint contract verification |
| `test_error_handling_refactor.py` | Error handling |
| `test_refactored_error_sanitization.py` | Error sanitization |
| `test_event_safety.py` | Event safety |
| `test_generation_scopes.py` | Content generation scopes |
| `test_idempotency.py` | Idempotency behavior |
| `test_kafka_event_publisher.py` | Kafka event publishing |
| `test_notification_lifecycle.py` | Notification lifecycle |
| `test_provider_onboarding.py` | Provider onboarding |
| `test_search_service.py` | Search service |
| `test_service_publish_service.py` | Service publishing |
| `test_sse.py` | Server-sent events |
| `test_app_factory_kafka_optional.py` | App factory with optional Kafka |

### Integration Tests (`tests/integration/`)

| Test File | Coverage |
|-----------|----------|
| `test_api.py` | Full API workflows |
| `test_assistant_api.py` | Assistant API end-to-end |
| `test_docker_infrastructure.py` | Docker infrastructure validation |
| `test_embedding_workflow.py` | Embedding workflow |
| `test_generation_endpoints.py` | Generation endpoints |
| `test_kafka_failure.py` | Kafka failure handling |
| `test_phi_scoping.py` | PHI scoping |
| `test_real_llm.py` | Real LLM integration |

### Demo Tasks (`tests/demo_tasks/`)

| Test File | Coverage |
|-----------|----------|
| `embedding_demo.py` | Embedding demonstration |
| `race_demo.py` | Race-condition and concurrency |
| `temporal_test.py` | Temporal workflow exercises |

## FakeLLM

Tests use a deterministic `FakeLLM` implementation that requires no network or API keys. It handles:

- Medical advice refusal (diagnosis, medication, treatment)
- Appointment status queries
- Service preparation information
- Availability checking
- General service navigation

This allows the AI test suite to run in isolated environments without external dependencies.

## Linting

```bash
make lint
```

Or directly:

```bash
ruff check app tests scripts
```

## Test Configuration

`pytest.ini`:

```ini
[pytest]
testpaths = tests
```

## Writing Tests

- Unit tests should not require external services (use FakeLLM, in-memory stores).
- Integration tests requiring Docker infra should be marked with `pytest.mark.integration`.
- Use `conftest.py` fixtures for shared setup.
- Follow the existing test naming convention: `test_<feature>.py`.
