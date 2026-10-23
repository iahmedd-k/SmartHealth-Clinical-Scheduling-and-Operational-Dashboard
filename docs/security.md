# Security

SmartHealth implements multiple layers of security controls to protect healthcare data and ensure authorized access.

## Authentication

SmartHealth uses JWT (HS256) bearer tokens for stateless authentication.

| Control | Implementation |
|---------|---------------|
| Token issuance | `POST /auth/login` returns a signed JWT |
| Token validation | Middleware validates JWT on each authenticated request |
| Token expiry | Access tokens expire after `ACCESS_TOKEN_EXPIRE_MINUTES` (default 60) |
| Refresh tokens | `REFRESH_TOKEN_EXPIRE_DAYS` (default 7) |
| Password hashing | `passlib[bcrypt]` / `bcrypt` |

## Authorization

Role-based access control (RBAC) is enforced on all authenticated routes:

| Role | Scope |
|------|-------|
| `patient` | Booking and personal profile access |
| `provider` | Provider profile, slot, and service access |
| `front_desk` | Operational catalogue and support workflows |
| `admin` | Full administrative control |

Resource ownership guards prevent users from accessing data belonging to other users.

## Secrets Management

| Secret | Storage | Production Requirement |
|--------|---------|----------------------|
| `JWT_SECRET` | Environment variable | 32+ characters, not a known demo secret |
| `LLM_API_KEY` | Environment variable | Required in production |
| `EMBEDDING_API_KEY` | Environment variable | Optional |
| Database credentials | Environment variables | Required for PostgreSQL |

**Never commit `.env` or `.env.production` with real secrets to version control.** Use a secrets manager (HashiCorp Vault, AWS Secrets Manager) in production.

## Security Controls

### Security Headers

`SecurityHeadersMiddleware` applies protective HTTP headers on all responses.

### HTTPS

`HTTPSRedirectMiddleware` is enabled in production to enforce TLS.

### CORS

`CORS_ALLOWED_ORIGINS` restricts cross-origin requests. In production, only explicitly listed origins are allowed.

### Rate Limiting

Authentication endpoints are rate-limited using `slowapi`:

| Endpoint | Limit |
|----------|-------|
| Login | 5/minute |
| Register | 3/hour |
| Password reset | 3/hour |

AI assistant: 30 requests/minute.

### Idempotency

`Idempotency-Key` header prevents duplicate booking requests from creating duplicate records. Keys are stored in Redis.

### PHI Protection

- Kafka events undergo PHI sanitization before publishing.
- AI assistant responses are scoped to prevent leaking Protected Health Information.
- Medical advice refusal checks prevent diagnosis and treatment advice.

### Audit Logging

`AuditLog` records administrative and operational actions for traceability.

### Error Handling

Errors are sanitized to avoid leaking internal implementation details. Sanitization tests verify error messages do not expose sensitive data.

## Security Recommendations for Production

1. Rotate `JWT_SECRET` regularly.
2. Store secrets in a vault, not `.env` files.
3. Enable SASL/TLS on Kafka (currently PLAINTEXT by default).
4. Use a WAF or API gateway in front of the API.
5. Enable TLS termination at the ingress/load balancer.
6. Restrict `CORS_ALLOWED_ORIGINS` to known production domains.
7. Rotate database credentials periodically.
8. Monitor `events_failed_total` and `appointments_booked_total` Prometheus counters for anomalies.

## Security Testing

- `tests/unit/test_assistant_safety.py` — verifies medical advice refusal.
- `tests/unit/test_refactored_error_sanitization.py` — verifies error sanitization.
- `tests/unit/test_phi_scoping.py` — verifies PHI scoping.
- `tests/unit/test_authorization.py` — verifies RBAC enforcement.
