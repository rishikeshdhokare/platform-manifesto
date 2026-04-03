# REST API Standards

> **Status:** Mandated  
> **Owner:** Platform Engineering  
> **Last Updated:** 2025

---

## 1. Principles

Every API we expose is a product. It will be consumed by mobile apps, partner integrations, internal services, and tools we haven't built yet. APIs are versioned contracts — once published, they must not be broken.

**Non-negotiable rules:**
- APIs are designed **API-first** — the OpenAPI spec is written before implementation begins
- APIs are **versioned from day one**
- APIs never return stack traces or internal error details to callers
- APIs are **idempotent where they mutate state**

---

## 2. URL Structure

### 2.1 Base URL Pattern

```
https://api.{company}.com/{version}/{domain}/{resource}
```

Examples:
```
https://api.{company}.com/v1/orders
https://api.{company}.com/v1/orders/{orderId}
https://api.{company}.com/v1/providers/{providerId}/location
https://api.{company}.com/v2/pricing/estimates
```

### 2.2 Resource Naming Rules

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Use nouns, not verbs | `/orders` | `/getOrders`, `/createOrder` |
| Use plural for collections | `/orders` | `/order` |
| Use kebab-case | `/order-events` | `/orderEvents`, `/order_events` |
| Nest for clear ownership only | `/providers/{id}/documents` | `/documents?providerId={id}` (if document is owned by provider) |
| Max 2 levels of nesting | `/orders/{id}/events` | `/orders/{id}/legs/{legId}/events/{eventId}` |

### 2.3 Actions That Don't Map to CRUD

For actions that don't fit standard CRUD, use a verb sub-resource:

```
POST /orders/{orderId}/cancel
POST /orders/{orderId}/complete
POST /providers/{providerId}/suspend
POST /payments/{paymentId}/refund
```

These are acceptable — they represent state transitions, not arbitrary RPC.

---

## 3. HTTP Methods

| Method | Use | Idempotent | Body |
|--------|-----|-----------|------|
| `GET` | Retrieve resource(s) | Yes | No |
| `POST` | Create a new resource or trigger an action | No (use idempotency key) | Yes |
| `PUT` | Replace a resource entirely | Yes | Yes |
| `PATCH` | Partial update of a resource | No (use idempotency key) | Yes |
| `DELETE` | Remove a resource | Yes | No |

**`POST` idempotency:** All `POST` endpoints that create resources or trigger mutations must support an `Idempotency-Key` header. The server stores the response for a `POST` keyed on this value and returns the cached response if the same key is received again within 24 hours.

---

## 4. Versioning

### 4.1 Strategy

We use **URL path versioning**: `/v1/`, `/v2/`. Header-based and query-param versioning are not used.

### 4.2 Rules

- Every API starts at `v1`
- A new version is required when making a **breaking change** (see below)
- Old versions must be **supported for a minimum of 12 months** after a new version is released
- Deprecation is communicated via the `Deprecation` and `Sunset` response headers

```
Deprecation: true
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
```

### 4.3 Breaking vs Non-Breaking Changes

| Change | Breaking? | Requires New Version? |
|--------|-----------|-----------------------|
| Adding a new optional field to response | No | No |
| Adding a new optional request parameter | No | No |
| Removing a field from response | **Yes** | **Yes** |
| Changing a field type | **Yes** | **Yes** |
| Renaming a field | **Yes** | **Yes** |
| Adding a required request field | **Yes** | **Yes** |
| Changing status codes | **Yes** | **Yes** |
| Adding a new endpoint | No | No |

---

## 5. Request & Response Standards

### 5.1 Request Headers

| Header | Required | Description |
|--------|---------|-------------|
| `Authorization` | Yes (authenticated) | `Bearer {jwt_token}` |
| `Content-Type` | Yes (with body) | `application/json` |
| `Accept` | Recommended | `application/json` |
| `X-Request-ID` | Recommended | Client-generated UUID for tracing |
| `Idempotency-Key` | Required for mutating POST | UUID v4, client-generated |
| `Accept-Language` | Optional | e.g. `en`, `ar` for localisation |

### 5.2 Response Headers

| Header | Always Present | Description |
|--------|---------------|-------------|
| `Content-Type` | Yes | `application/json` |
| `X-Request-ID` | Yes | Echoed back from request or server-generated |
| `X-Correlation-ID` | Yes | Internal trace ID for log correlation |
| `Cache-Control` | Yes | Explicit caching directive on all responses |
| `Deprecation` | When deprecated | As per section 4.2 |

### 5.3 Field Naming

- **JSON fields:** `camelCase`
- **Date/time fields:** ISO 8601 with timezone — `2024-11-15T14:30:00Z`
- **IDs:** String UUIDs — never expose sequential integers as public IDs
- **Enums:** `SCREAMING_SNAKE_CASE` — `ORDER_IN_PROGRESS`, `PAYMENT_FAILED`
- **Currency:** Integer cents/pence — never floating point. Include currency code separately.
  ```json
  { "amount": 1250, "currency": "USD" }
  ```
- **Coordinates:** GeoJSON-compatible — `{ "lat": 40.7128, "lng": -74.0060 }`

---

## 6. Pagination

All list endpoints must be paginated. We use **cursor-based pagination** for all production APIs (offset pagination is only acceptable for internal/ops tools).

### 6.1 Request

```
GET /v1/orders?limit=20&cursor=eyJpZCI6IjEyMyJ9
```

| Parameter | Default | Max | Description |
|-----------|---------|-----|-------------|
| `limit` | 20 | 100 | Number of items per page |
| `cursor` | — | — | Opaque cursor from previous response |

### 6.2 Response

```json
{
  "data": [...],
  "pagination": {
    "limit": 20,
    "nextCursor": "eyJpZCI6IjE0MyJ9",
    "hasMore": true
  }
}
```

When there are no more pages, `nextCursor` is `null` and `hasMore` is `false`.

---

## 7. Error Handling

### 7.1 Standard Error Shape

All errors — validation, auth, not found, server errors — use this shape:

```json
{
  "error": {
    "code": "ORDER_NOT_FOUND",
    "message": "The requested order does not exist or you do not have access to it.",
    "requestId": "req_01HXYZ...",
    "timestamp": "2024-11-15T14:30:00Z",
    "details": [
      {
        "field": "orderId",
        "issue": "No order found with id 'abc-123'"
      }
    ]
  }
}
```

| Field | Description |
|-------|-------------|
| `code` | Machine-readable, `SCREAMING_SNAKE_CASE`, stable across versions |
| `message` | Human-readable, may change — do not parse in code |
| `requestId` | For support — correlates to logs |
| `timestamp` | When the error occurred |
| `details` | Optional array of field-level issues (validation errors) |

### 7.2 HTTP Status Code Usage

| Code | When to Use |
|------|-------------|
| `200 OK` | Successful GET, PATCH, PUT, DELETE |
| `201 Created` | Successful POST that created a resource |
| `202 Accepted` | Request accepted for async processing |
| `204 No Content` | Successful DELETE with no response body |
| `400 Bad Request` | Validation failure, malformed request |
| `401 Unauthorized` | Missing or invalid authentication |
| `403 Forbidden` | Authenticated but not authorised |
| `404 Not Found` | Resource does not exist (or caller is not authorised to know it exists) |
| `409 Conflict` | State conflict — e.g. order already cancelled |
| `422 Unprocessable Entity` | Business rule violation (different from 400 validation) |
| `429 Too Many Requests` | Rate limit exceeded; include `Retry-After` header |
| `500 Internal Server Error` | Unhandled server error — never expose internals |
| `503 Service Unavailable` | Deliberate unavailability (circuit open, maintenance) |

**Never use:** `200` with an error payload body. Never return `500` with stack traces.

---

## 8. Authentication & Authorization

### 8.1 Authentication

- All external APIs require a JWT Bearer token in the `Authorization` header
- JWTs are issued by our identity provider (Auth0 / Cognito — see team decision)
- JWTs must be validated at the **BFF layer** — downstream services trust the claims forwarded by the BFF (internal JWT propagation)
- Token expiry: **15 minutes** for access tokens; refresh tokens are long-lived with rotation

### 8.2 Authorisation

- Authorisation is enforced **at the service level**, not just at the gateway
- Services must validate that the calling principal has rights to the requested resource
- Never use sequential IDs in URLs — UUID prevents enumeration attacks

### 8.3 API Keys (Partner APIs)

- Partner API access uses API keys with scopes
- API keys are issued via the developer portal, rotatable, and revocable
- API keys are hashed at rest — the platform stores the hash, not the key

---

## 9. Rate Limiting

| Tier | Default Limit | Header |
|------|-------------|--------|
| Unauthenticated | 10 req/min | `X-RateLimit-Limit` |
| Authenticated customer | 60 req/min | `X-RateLimit-Limit` |
| Authenticated provider | 120 req/min | `X-RateLimit-Limit` |
| Partner (API key) | Configurable per key | `X-RateLimit-Limit` |

All rate-limited responses return:
```
HTTP 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1700000000
```

---

## 10. OpenAPI Contract

### 10.1 API-First Process

1. Engineer drafts the OpenAPI 3.1 spec in `api/openapi.yaml`
2. PR raised for review — API contract review is separate from implementation review
3. Once approved, implementation begins
4. CI validates implementation against contract using `openapi-diff` (breaking changes in CI fail the build)

### 10.2 Contract Location

- Each service owns its OpenAPI spec at `api/openapi.yaml` in the repo root
- Specs are published automatically to the internal API catalog (Backstage) on merge to main
- Consumer-driven contract tests (Pact) are run against the published spec

### 10.3 Required Spec Elements

Every endpoint in the spec must have:
- `summary` and `description`
- All request parameters and body fields documented with types and examples
- All response codes documented (including 400, 401, 404, 500 at minimum)
- `x-internal: true` tag on any endpoint not intended for external consumers

---

## 11. Filtering, Sorting & Searching

```
GET /v1/orders?status=COMPLETED&fromDate=2024-01-01&toDate=2024-12-31
GET /v1/providers?sort=rating&order=desc
GET /v1/orders?q=premium                     # full-text search (where supported)
```

| Parameter | Format | Example |
|-----------|--------|---------|
| Filter | `field=value` | `?status=COMPLETED` |
| Date range | `fromDate`, `toDate` | `?fromDate=2024-01-01` (ISO 8601 date) |
| Sort | `sort=field&order=asc\|desc` | `?sort=createdAt&order=desc` |
| Search | `q=term` | `?q=premium` |

---

## 12. API Lifecycle Management

Publishing an API version is easy. Managing its lifecycle — tracking consumers, enforcing deprecation, and sunsetting — is harder. These rules ensure we don't accumulate zombie API versions.

### 12.1 Version Usage Tracking

Every API response includes the version in the path (`/v1/`, `/v2/`). API Gateway logs capture the version per request. Metrics are available in Grafana:

```promql
# Requests per API version (last 7 days)
sum by (version) (
  increase(http_server_requests_seconds_count{service="orders-service"}[7d])
)
```

The API catalog in Backstage shows per-version request volume, updated daily.

### 12.2 Deprecation Process

When a new version is released:

1. **Day 0:** New version published. Old version marked with `Deprecation: true` and `Sunset` response headers.
2. **Week 1:** Backstage sends automated notification to all registered consumers of the old version.
3. **Monthly:** Deprecation dashboard reviewed — which consumers still use the old version?
4. **Month 6:** Warning emails to consumer teams still on old version.
5. **Month 11:** Final warning — old version will be removed in 30 days.
6. **Month 12:** Old version removed. Requests return `410 Gone`.

### 12.3 Consumer Migration Tracking

| Metric | Source | Dashboard |
|--------|--------|-----------|
| Request volume per version | API Gateway logs | Grafana |
| Consumer count per version | Backstage API catalog | Backstage |
| Days since deprecation | Backstage annotations | Backstage |
| Migration completion % | Consumer request ratio v1/v2 | Grafana |

### 12.4 Enforcement

- CI check: new services cannot integrate against a deprecated API version (enforced via Pact broker tags)
- Backstage: deprecated APIs are visually flagged in the catalog
- Quarterly review: API versions older than 18 months with <1% traffic are candidates for forced sunset

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
