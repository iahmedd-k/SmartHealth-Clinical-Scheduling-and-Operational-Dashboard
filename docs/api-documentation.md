# API Documentation

SmartHealth exposes a RESTful API built with FastAPI. All authenticated routes are versioned under `/api/v1`. Public catalog endpoints are available without authentication. Interactive documentation is available at `/docs` (Swagger UI) and `/redoc` when the API is running.

## Base URL

- Local development: `http://localhost:8000`
- Production: configured via `CORS_ALLOWED_ORIGINS`

## Authentication

SmartHealth uses JWT (HS256) access tokens. Supported roles:

| Role | Permissions |
|------|-------------|
| `patient` | Booking and personal profile access |
| `provider` | Provider profile, slot, and service access |
| `front_desk` | Operational catalogue and support workflows |
| `admin` | Full administrative control |

### Auth Endpoints

#### POST /auth/register
Creates a user account.

**Request:**
```json
{
  "email": "patient@example.com",
  "password": "secret123",
  "role": "patient"
}
```

**Response:**
```json
{
  "id": "uuid",
  "email": "patient@example.com",
  "role": "patient",
  "created_at": "2026-01-01T00:00:00Z"
}
```

#### POST /auth/login
Validates credentials and returns an access token.

**Request:**
```json
{
  "email": "patient@example.com",
  "password": "secret123"
}
```

**Response:**
```json
{
  "access_token": "eyJ...",
  "token_type": "bearer"
}
```

Use the token in the `Authorization: Bearer <token>` header for subsequent requests.

## Core Domain APIs

### Departments

#### POST /api/v1/departments
Create a department. Requires admin role.

#### GET /api/v1/departments
List all departments.

### Providers

#### POST /api/v1/providers
Create a provider profile.

#### GET /api/v1/providers
List providers.

#### GET /api/v1/providers/{provider_id}/slots
Get available slots for a provider.

### Services

#### POST /api/v1/services
Create a service offering.

#### POST /api/v1/services/{service_id}/publish
Publish a service to the public catalog.

#### POST /api/v1/services/{service_id}/unpublish
Unpublish a service.

#### GET /api/v1/services/{service_id}/publish-status
Get publishing status of a service.

#### GET /api/v1/services
List services.

### Slots

#### POST /api/v1/slots
Create availability slots.

#### GET /api/v1/slots
List available slots.

### Appointments

Patients book an available slot by calling `POST /api/v1/appointments`. The appointment saga validates and reserves the slot internally.

#### POST /api/v1/appointments
Create an appointment. Requires `Idempotency-Key` header to prevent duplicate bookings.

**Request:**
```json
{
  "slot_id": "uuid",
  "patient_id": "uuid",
  "service_id": "uuid"
}
```

#### GET /api/v1/appointments/{appointment_id}/state
Get current appointment state.

#### POST /api/v1/appointments/{appointment_id}/cancel
Cancel an appointment.

#### POST /api/v1/appointments/{appointment_id}/reschedule
Reschedule an appointment to a new slot.

#### POST /api/v1/appointments/{appointment_id}/billing/pre-check
Run a mock billing pre-check (approves/fails based on configuration).

#### POST /api/v1/appointments/{appointment_id}/visit/check-in
Check in for a visit.

#### POST /api/v1/appointments/{appointment_id}/visit/start
Start a visit.

#### POST /api/v1/appointments/{appointment_id}/visit/complete
Complete a visit.

### Waitlist

Patients may join the waitlist when a slot is unavailable.

### Public Catalog

#### GET /api/v1/public/services
Public listing of published services (no authentication required).

## AI Assistant

The assistant provides intelligent, safety-checked responses. All responses undergo medical advice refusal checks and PHI scoping.

#### POST /assistant/ask
Stream answers to healthcare questions.

**Request:**
```json
{
  "message": "How do I prepare for a blood test?",
  "conversation_id": "optional-uuid"
}
```

**Response:** Streamed text with safety-check metadata.

#### POST /assistant/report
Generate operational utilisation reports. Requires analytics read permission.

## Notifications

#### GET /api/v1/notifications
Query notification status (PENDING → SENT → CANCELLED).

## Health and Monitoring

#### GET /health
Liveness check for the API.

#### GET /health/ready
Readiness check used by Docker healthcheck.

#### GET /metrics
Prometheus-formatted metrics endpoint.

Exported counters include:
- `appointments_booked_total`
- `double_booking_prevented_total`
- `events_consumed_total`
- `events_failed_total`

#### GET /docs
Swagger UI for API inspection.

## Idempotency

The appointment flow uses the `Idempotency-Key` header to safely prevent duplicate booking requests from creating duplicate records.

**Header:**
```
Idempotency-Key: <unique-client-generated-uuid>
```

## Rate Limiting

Authentication endpoints are rate-limited:
- Login: 5/minute
- Register: 3/hour
- Password reset: 3/hour

AI assistant: 30/minute

## Error Responses

Standard error format:

```json
{
  "detail": "Error description",
  "error_code": "ERR_CODE",
  "correlation_id": "uuid"
}
```

Common HTTP status codes:
- `200` — Success
- `201` — Created
- `400` — Bad request / validation error
- `401` — Unauthorized
- `403` — Forbidden (insufficient role)
- `404` — Not found
- `409` — Conflict (e.g., double booking)
- `422` — Unprocessable entity
- `429` — Too many requests
- `500` — Internal server error

## SDK / Client

The frontend consumes the API via `frontend/src/api.js`. No official client SDK is published; generate one from `/openapi.json` if needed.
