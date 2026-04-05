# 🔌 REST API Standards

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Principles

Every API we expose is a product. It will be consumed by mobile apps, partner integrations, internal services, and tools we haven't built yet. APIs are versioned contracts - once published, they must not be broken.

**Non-negotiable rules:**
- APIs are designed **API-first** - the OpenAPI spec is written before implementation begins
- APIs are **versioned from day one**
- APIs never return stack traces or internal error details to callers
- APIs are **idempotent where they mutate state**

---

## 🌐 2. URL Structure

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

> **Per-environment hosts:** `api-dev.{company}.com`, `api-staging.{company}.com`, `api.{company}.com`

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

These are acceptable - they represent state transitions, not arbitrary RPC.

---

## 📡 3. HTTP Methods

| Method | Use | Idempotent | Body |
|--------|-----|-----------|------|
| `GET` | Retrieve resource(s) | Yes | No |
| `POST` | Create a new resource or trigger an action | No (use idempotency key) | Yes |
| `PUT` | Replace a resource entirely | Yes | Yes |
| `PATCH` | Partial update of a resource | No (use idempotency key) | Yes |
| `DELETE` | Remove a resource | Yes | No |

**`POST` idempotency:** All `POST` endpoints that create resources or trigger mutations must support an `Idempotency-Key` header. The server stores the response for a `POST` keyed on this value and returns the cached response if the same key is received again within 24 hours.

### 3.1 Idempotency Details

**Scope:** The `Idempotency-Key` is scoped per authenticated principal - specifically `tenant + user`. The same key submitted by different principals is treated as different operations. This prevents cross-tenant collisions in multi-tenant environments.

**Payload mismatch:** If the same `Idempotency-Key` is submitted with a **different request payload** than the original, the server returns `422 Unprocessable Entity` with error code `IDEMPOTENCY_KEY_PAYLOAD_MISMATCH`. The server does not execute the new payload - the original result stands.

**PATCH operations:** `PATCH` **can be** idempotent when using absolute field assignments (e.g., "set status to X") but is **not inherently so** - relative operations like "increment counter" or "append to list" are non-idempotent (see RFC 5789). For absolute-assignment PATCH endpoints, no `Idempotency-Key` is required (though clients may include one; the server will honour it if present). For PATCH endpoints that perform relative updates, require an `Idempotency-Key` header using the same semantics as `POST`.

**gRPC:** For mutation RPCs in internal gRPC services, include an `idempotency_key` field in the request message:

```protobuf
message CreatePaymentRequest {
  string idempotency_key = 1;
  string order_id = 2;
  int64 amount_cents = 3;
  string currency = 4;
}
```

The same storage and TTL semantics apply as for REST - the key is stored in the idempotency table with the serialised response, and replayed if the same key is received within 24 hours.

---

## 📏 4. Versioning

### 4.1 Strategy

We use **URL path versioning**: `/v1/`, `/v2/`. Header-based and query-param versioning are not used.

### 4.2 Rules

- Every API starts at `v1`
- A new version is required when making a **breaking change** (see below)
- Old versions must be **supported for a minimum of 12 months** after a new version is released
- Deprecation is communicated via the `Deprecation` and `Sunset` response headers

```
Deprecation: true
Sunset: Thu, 31 Dec 2026 23:59:59 GMT
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

## 📡 5. Request & Response Standards

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
- **Date/time fields:** ISO 8601 with timezone - `2026-11-15T14:30:00Z`
- **IDs:** String UUIDs - never expose sequential integers as public IDs
- **Enums:** `SCREAMING_SNAKE_CASE` - `ORDER_IN_PROGRESS`, `PAYMENT_FAILED`
- **Currency:** Integer cents/pence - never floating point. Include currency code separately.
  ```json
  { "amount": 1250, "currency": "USD" }
  ```
- **Coordinates:** GeoJSON-compatible - `{ "lat": 40.7128, "lng": -74.0060 }`

  > **Note:** JSON API payloads use `{"lat": N, "lng": N}` objects. GeoJSON endpoints use `[lng, lat]` per GeoJSON spec.

---

## 📋 6. Pagination

All list endpoints must be paginated. We use **cursor-based pagination** for all production APIs (offset pagination is only acceptable for internal/ops tools).

### 6.1 Request

```
GET /v1/orders?limit=20&cursor=eyJpZCI6IjEyMyJ9
```

| Parameter | Default | Max | Description |
|-----------|---------|-----|-------------|
| `limit` | 20 | 100 | Number of items per page |
| `cursor` | - | - | Opaque cursor from previous response |

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

### 6.3 Deep Pagination Limit

Maximum traversal depth is **10,000 items** (i.e., 500 pages at `limit=20`). Requests that attempt to paginate beyond 10,000 items receive a `400 Bad Request` with error code `PAGINATION_DEPTH_EXCEEDED` and a message instructing the caller to apply filters (date range, status, etc.) to narrow the result set.

This limit exists to protect database performance - deep cursor-based pagination still requires index traversal proportional to depth.

### 6.4 Cursor Encoding & Expiry

- **Encoding:** Cursors are **opaque, base64-encoded strings**. Clients must not parse, construct, or modify cursors - they are server-generated and server-interpreted.
- **Expiry:** Cursors expire after **1 hour**. A request with an expired cursor returns `400 Bad Request` with error code `CURSOR_EXPIRED`. The client must restart pagination from the beginning.
- **Contents:** Internally, cursors encode the last-seen sort key(s) and direction. The encoding format is an implementation detail and may change without notice.

### 6.5 Consistency

- Where the underlying data store supports it, cursors use **snapshot isolation** - the result set is consistent as of the time the first page was requested.
- For data stores without snapshot isolation (e.g., OpenSearch), results **may shift** between pages if the underlying data changes (inserts, deletes, re-sorting). This must be documented in the API spec for affected endpoints.
- APIs that cannot guarantee cursor consistency must include a `consistency: "best-effort"` field in the pagination response object.

---

## ⚠️ 7. Error Handling

### 7.1 Standard Error Shape

For the complete error response specification, see [09-error-catalog.md](./09-error-catalog.md).

All errors - validation, auth, not found, server errors - use this shape (fields and code format match the Error Catalog):

```json
{
  "error": {
    "code": "ORDERS.ORDER.NOT_FOUND",
    "message": "The requested order does not exist or you do not have access to it.",
    "path": "/api/v1/orders/ord-12345",
    "requestId": "req-abc-123",
    "traceId": "1-abc123-def456",
    "metadata": {
      "orderId": "ord-12345"
    },
    "timestamp": "2026-03-15T10:30:00Z"
  }
}
```

| Field | Description |
|-------|-------------|
| `code` | Machine-readable dotted code from the catalog (e.g. `DOMAIN.RESOURCE.REASON`), stable across versions |
| `message` | Human-readable, may change - do not parse in code |
| `path` | Request path that produced the error |
| `requestId` | Correlation ID (echo `X-Request-Id` where applicable) |
| `traceId` | Distributed tracing ID |
| `metadata` | Optional object with extra context (entity IDs, validation hints) |
| `timestamp` | When the error occurred (ISO-8601) |

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
| `409 Conflict` | State conflict - e.g. order already cancelled |
| `422 Unprocessable Entity` | Business rule violation (different from 400 validation) |
| `429 Too Many Requests` | Rate limit exceeded; include `Retry-After` header |
| `500 Internal Server Error` | Unhandled server error - never expose internals |
| `503 Service Unavailable` | Deliberate unavailability (circuit open, maintenance) |

**Never use:** `200` with an error payload body. Never return `500` with stack traces.

---

## 🔒 8. Authentication & Authorization

### 8.1 Authentication

- All external APIs require a JWT Bearer token in the `Authorization` header
- JWTs are issued by our identity provider (Auth0 / Cognito - see team decision)
- JWTs must be validated at the **BFF layer** - downstream services trust the claims forwarded by the BFF (internal JWT propagation)
- Token expiry: **15 minutes** for access tokens; refresh tokens are long-lived with rotation

### 8.2 Authorisation

- Authorisation is enforced **at the service level**, not just at the gateway
- Services must validate that the calling principal has rights to the requested resource
- Never use sequential IDs in URLs - UUID prevents enumeration attacks

### 8.3 API Keys (Partner APIs)

- Partner API access uses API keys with scopes
- API keys are issued via the developer portal, rotatable, and revocable
- API keys are hashed at rest - the platform stores the hash, not the key

---

## 🛡️ 9. Rate Limiting

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

### 9.1 Algorithm & Enforcement

| Algorithm | Use Case | Behaviour |
|-----------|----------|-----------|
| **Token bucket** (standard) | Most endpoints | Tokens refill at a steady rate; allows smoothed traffic with some burst capacity |
| **Sliding window** | Endpoints requiring strict per-second limits (e.g., payment initiation) | Exact count over a rolling time window; no burst allowed |

Token bucket is the default. Sliding window is used only when the downstream system cannot tolerate any burst (e.g., third-party payment processor rate limits).

### 9.2 Rate Limit Dimensions

| Dimension | Applied To | Key |
|-----------|-----------|-----|
| **Per-API-key** (default) | All authenticated endpoints | API key or JWT `sub` claim |
| **Per-IP** | Unauthenticated endpoints (login, registration, public APIs) | Client IP (respecting `X-Forwarded-For` from trusted proxies only) |
| **Per-tenant** | B2B / partner APIs | Tenant ID from JWT claims |

### 9.3 Burst Allowance

All rate-limited endpoints allow a **burst of 2× the sustained rate for up to 10 seconds**. For example, an endpoint with a sustained limit of 60 req/min (1 req/s) may burst to 2 req/s for 10 seconds before rate limiting applies.

Burst is implemented via the token bucket's bucket depth - set to `sustained_rate × 10`.

### 9.4 Fairness & Noisy Neighbour Prevention

- **Per-tenant quotas:** In multi-tenant (B2B) contexts, each tenant receives an independent quota. A single tenant cannot exhaust the shared pool.
- **Shared pool:** Unauthenticated traffic shares a global pool. If a single IP consumes a disproportionate share, per-IP limiting triggers before the global pool is exhausted.
- **Priority tiers:** Partner API keys can be assigned higher quotas via the developer portal. Default quotas apply unless explicitly overridden.

### 9.5 Enforcement Split

| Layer | Handles | Tool |
|-------|---------|------|
| **API Gateway** | Global rate limits per tier (unauthenticated, authenticated, partner) | AWS API Gateway usage plans + throttling |
| **BFF** | Per-user / per-session rate limits (e.g., max 5 order creations per minute per user) | Application-level rate limiter (Resilience4j `RateLimiter` or Redis-backed counter) |

The API Gateway is the first line of defence - it protects backend services from traffic floods. The BFF applies business-level rate limits that require awareness of the authenticated user context.

### 9.6 Response

All rate-limited responses return `429 Too Many Requests` with a `Retry-After` header indicating the number of seconds until the client may retry. Clients must respect this header - SDKs and BFFs should implement automatic backoff.

---

## 📜 10. OpenAPI Contract

### 10.1 API-First Process

1. Engineer drafts the OpenAPI 3.1 spec in `api/openapi.yaml`
2. PR raised for review - API contract review is separate from implementation review
3. Once approved, implementation begins
4. CI validates implementation against contract using `openapi-diff` (breaking changes in CI fail the build)

**Visual overview:**

```mermaid
flowchart LR
    Design[Design Spec] --> Review[API Review]
    Review --> Implement[Implement]
    Implement --> Contract[Contract Test]
    Contract --> Publish[Publish to Backstage]
```

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

## 📋 11. Filtering, Sorting & Searching

```
GET /v1/orders?status=COMPLETED&fromDate=2026-01-01&toDate=2026-12-31
GET /v1/providers?sort=rating&order=desc
GET /v1/orders?q=premium                     # full-text search (where supported)
```

| Parameter | Format | Example |
|-----------|--------|---------|
| Filter | `field=value` | `?status=COMPLETED` |
| Date range | `fromDate`, `toDate` | `?fromDate=2026-01-01` (ISO 8601 date) |
| Sort | `sort=field&order=asc\|desc` | `?sort=createdAt&order=desc` |
| Search | `q=term` | `?q=premium` |

---

## 📏 12. API Lifecycle Management

Publishing an API version is easy. Managing its lifecycle - tracking consumers, enforcing deprecation, and sunsetting - is harder. These rules ensure we don't accumulate zombie API versions.

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
3. **Monthly:** Deprecation dashboard reviewed - which consumers still use the old version?
4. **Month 6:** Warning emails to consumer teams still on old version.
5. **Month 11:** Final warning - old version will be removed in 30 days.
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

## 📏 13. GraphQL Policy

### 13.1 Default Position

**GraphQL is NOT approved for new services.** The standard API pattern is REST (external) + gRPC (internal). This is a deliberate decision, not an oversight.

### 13.2 Rationale

GraphQL adds significant operational complexity that our current platform tooling does not support at scale:

| Concern | REST/gRPC (current) | GraphQL (additional burden) |
|---------|---------------------|----------------------------|
| **Caching** | Standard HTTP caching (CDN, `Cache-Control`) | Requires custom cache strategies; POST-based queries defeat HTTP caches |
| **Monitoring** | Per-endpoint metrics (RED method) | Single `/graphql` endpoint - all queries look the same without custom instrumentation |
| **Authorization** | Per-endpoint middleware | Per-field authorization required; easy to accidentally expose data |
| **N+1 queries** | Explicit in code; visible in traces | Hidden in resolver chains; requires DataLoader discipline |
| **Rate limiting** | Per-endpoint limits | Query complexity varies wildly; simple rate limits are insufficient |

### 13.3 Exception Process

If a team believes GraphQL is the right solution for a **BFF aggregation** use case (e.g., a mobile BFF that aggregates 5+ downstream services into a single screen), they may propose it via the RFC process (see Ways of Working - RFC Process).

The RFC must address:
- Why REST aggregation in the BFF is insufficient
- How the team will handle each concern in the table above
- Who will own the ongoing operational complexity

### 13.4 If Approved: Mandatory Guardrails

GraphQL services approved via RFC must enforce **all** of the following:

| Guardrail | Configuration |
|-----------|--------------|
| **Query depth limit** | Maximum 10 levels of nesting |
| **Complexity scoring** | Maximum 1,000 points per query (fields = 1 point, nested objects = 10 points, connections = 50 points) |
| **Per-field authorization** | Every field resolver checks the caller's permissions; no implicit trust based on parent access |
| **DataLoader** | Mandatory for all resolvers that access a data source; prevents N+1 queries |
| **Persisted queries only** | In production, only pre-registered query hashes are accepted; arbitrary query strings are rejected |
| **Introspection disabled** | `introspection: false` in production; enabled only in development/staging |

### 13.5 Monitoring Requirements for GraphQL

If a GraphQL service is approved, it must implement custom observability to match the visibility we get from REST endpoints:

- Per-operation-name metrics (rate, errors, duration) - operation names are required on all queries
- Query complexity tracking as a Prometheus histogram
- Slow query logging (queries exceeding complexity threshold or duration > 1s)
- Depth violation alerts

---

## 🔒 14. Internal Authentication

### 14.1 BFF-to-Service Authentication

The BFF validates the external JWT and forwards identity claims to downstream services as HTTP headers:

| Header | Value | Description |
|--------|-------|-------------|
| `X-{Company}-User-Id` | UUID from JWT `sub` claim | Authenticated user identity |
| `X-{Company}-Roles` | Comma-separated role list from JWT | User roles for authorization |
| `X-{Company}-Tenant-Id` | Tenant identifier from JWT | Tenant context for multi-tenant isolation |

Downstream services trust these headers because the request arrives over the Istio mesh with mTLS - the BFF's SPIFFE identity is verified by the sidecar. Services do not re-validate the original JWT.

### 14.2 Service-to-Service Authentication

| Communication Path | Auth Mechanism | Token Required? |
|--------------------|---------------|-----------------|
| BFF → internal service | Istio mTLS + forwarded JWT claim headers | No additional token - mTLS identity is sufficient |
| Service → service (same mesh) | Istio mTLS (SPIFFE identity) | No - mesh identity verified automatically |
| Service → external API | OAuth2 client credentials or API key | Yes - managed via Secrets Manager |
| External partner → API Gateway | API key or OAuth2 Bearer token | Yes |

For service-to-service calls within the mesh, Istio's **SPIFFE-based mTLS** provides mutual authentication. No additional JWT or API key is needed - the `AuthorizationPolicy` controls which service identities may call which endpoints.

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
