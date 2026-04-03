# System Architecture Blueprint

> **Status:** Mandated  
> **Owner:** Principal Engineering / Platform  
> **Last Updated:** 2025

---

## 1. Architectural Philosophy

Our architecture is **domain-driven, event-native, and designed for independent deployability**. Every architectural decision should be evaluated against three questions:

1. Can this service be deployed independently without coordinating with another team?
2. Can this service fail without cascading failures across the platform?
3. Can an engineer understand what this service does by reading its code alone?

If the answer to any of these is "no", the design needs to change.

---

## 2. Domain Decomposition

The platform naturally decomposes into the following bounded contexts. Each domain owns its data, its events, and its API contracts. **No domain may directly access another domain's database.**

```
┌─────────────────────────────────────────────────────────────────┐
│                         API Gateway Layer                        │
│              (Amazon API Gateway + ALB per surface)             │
└────────┬──────────┬──────────┬───────────┬──────────────────────┘
         │          │          │           │
    ┌────▼───┐ ┌────▼───┐ ┌───▼────┐ ┌────▼────┐
    │Customer│ │Provider│ │  Ops   │ │ Partner │
    │  BFF   │ │  BFF   │ │  BFF   │ │   BFF   │
    └────┬───┘ └────┬───┘ └───┬────┘ └────┬────┘
         │          │          │           │
         └──────────┴──────────┴───────────┘
                          │
              ┌───────────▼────────────┐
              │     Internal Services  │
              │                        │
  ┌───────────┼────────────────────────┼───────────┐
  │           │                        │           │
┌─▼──────┐ ┌─▼──────┐ ┌──────────┐ ┌──▼─────┐ ┌──▼─────┐
│Fulfill-│ │Pricing │ │  Orders  │ │Provider│ │Customer│
│ment    │ │Service │ │ Service  │ │Profile │ │Profile │
│ Engine │ │        │ │          │ │        │ │        │
└─┬──────┘ └─┬──────┘ └──┬───────┘ └─┬──────┘ └─┬──────┘
  │          │            │            │           │
  └──────────┴────────────┴────────────┴───────────┘
                          │
               ┌──────────▼──────────┐
               │   Supporting Domains │
               │                      │
  ┌────────────┼──────────────────────┼────────────┐
  │            │                      │            │
┌─▼──────┐ ┌──▼─────┐ ┌──────────┐ ┌──▼─────┐ ┌──▼─────┐
│Payment │ │Notific-│ │Geoloc. / │ │Dynamic │ │ Fraud  │
│Service │ │ations  │ │ Routing  │ │Pricing │ │ Engine │
└────────┘ └────────┘ └──────────┘ └────────┘ └────────┘
                          │
               ┌──────────▼──────────┐
               │   Event Backbone     │
               │   (Kafka / MSK)      │
               └─────────────────────┘
```

---

## 3. Domain Ownership Map

| Domain | Core Responsibility | Key Entities | Owned Data Store |
|--------|--------------------|--------------|--------------------|
| **Fulfillment Engine** | Real-time provider-customer assignment | FulfillmentResult, Assignment | Redis (geospatial) |
| **Pricing Service** | Price calculation, dynamic pricing | Price, PriceEstimate | Aurora PostgreSQL |
| **Orders Service** | Order lifecycle management | Order, OrderEvent | Aurora PostgreSQL |
| **Provider Profile** | Provider onboarding, documents, ratings | Provider, Document, Rating | RDS PostgreSQL |
| **Customer Profile** | Customer account, preferences, history | Customer, PaymentMethod | RDS PostgreSQL |
| **Payment Service** | Payment processing, settlements | Payment, Payout, Wallet | Aurora PostgreSQL |
| **Notifications** | Push, SMS, email dispatch | Notification, Template | RDS PostgreSQL |
| **Geolocation / Routing** | ETA, route calculation, geocoding | Route, ETA | (proxies external APIs) |
| **Dynamic Pricing** | Demand/supply signals, multiplier | PricingZone, PricingMultiplier | Redis + RDS |
| **Fraud Engine** | Real-time fraud signals, blocking | Signal, RiskScore | Aurora PostgreSQL |

---

## 4. Communication Patterns

### 4.1 Synchronous (Request/Response)

Use for: **user-facing requests where a real-time response is required.**

- **External (client → BFF → service):** REST over HTTPS via API Gateway
- **Internal (service → service):** gRPC with mutual TLS
- **Timeout policy:** All synchronous calls must have explicit timeouts (default: 2s for internal, 5s for external dependencies)
- **Retry policy:** Exponential backoff with jitter, max 3 retries, only on idempotent operations
- **Circuit breaker:** Resilience4j — mandatory on all synchronous outbound calls

### 4.2 Asynchronous (Event-Driven)

Use for: **state changes that other domains need to react to, but not in the critical path of a user request.**

- **Platform:** Kafka on Amazon MSK
- **Producer rule:** A service publishes events about its own domain — it does not publish commands to other services
- **Consumer rule:** Consumers are responsible for idempotency — the same event may be delivered more than once
- **Schema:** Avro, registered in AWS Glue Schema Registry — breaking schema changes are forbidden; use schema evolution rules

### 4.3 Decision Guide

```
Is a real-time response required by the user?
  └─ YES → REST (external) or gRPC (internal)
  └─ NO → Is it a state change other domains care about?
            └─ YES → Kafka event
            └─ NO → Is it fire-and-forget within one service?
                       └─ YES → Internal async (CompletableFuture / virtual thread)
```

---

## 5. Event Backbone — Kafka Topics

### 5.1 Topic Naming Convention

```
{domain}.{entity}.{event-verb}

Examples:
  orders.order.started
  orders.order.completed
  orders.order.cancelled
  providers.provider.location-updated
  payments.payment.captured
  payments.payment.failed
  customers.customer.registered
  fraud.signal.raised
```

### 5.2 Core Domain Events (Non-Exhaustive)

| Topic | Producer | Key Consumers | Retention |
|-------|----------|---------------|-----------|
| `orders.order.requested` | Orders Service | Fulfillment Engine, Fraud Engine, Analytics | 7 days |
| `orders.order.matched` | Orders Service | Notifications, Analytics | 7 days |
| `orders.order.started` | Orders Service | Dynamic Pricing, Notifications, Analytics | 7 days |
| `orders.order.completed` | Orders Service | Payments, Notifications, Analytics | 30 days |
| `orders.order.cancelled` | Orders Service | Payments, Fulfillment, Notifications | 7 days |
| `fulfillment.assignment.found` | Fulfillment Engine | Orders Service | 7 days |
| `providers.provider.location-updated` | Provider App → Location Service | Fulfillment Engine, Dynamic Pricing | 1 hour |
| `payments.payment.captured` | Payment Service | Orders Service, Notifications | 30 days |
| `providers.provider.status-changed` | Provider Profile | Fulfillment Engine | 7 days |

### 5.3 Consumer Group Naming

```
{consuming-service}.{topic-short-name}.consumer

Example: notifications.order-completed.consumer
```

---

## 6. API Gateway & BFF Pattern

### 6.1 Why BFF

Each client surface has different data needs. A single API layer serving all clients leads to over-fetching, versioning nightmares, and tight coupling between the platform and app release cycles.

### 6.2 BFF Responsibilities

- Authentication token validation (JWT verification)
- Request aggregation (one BFF call → multiple internal service calls)
- Response shaping per client (mobile needs different payloads than web)
- Rate limiting per client
- Client-specific caching

### 6.3 BFF Anti-Patterns (Forbidden)

- Business logic in BFFs — BFFs orchestrate, they do not calculate
- Direct database access from BFFs
- BFFs calling other BFFs
- Shared BFF across two different client surfaces

---

## 7. Data Architecture Principles

### 7.1 Database-per-Service

Every service owns exactly one logical database. No exceptions.

- Services communicate via APIs or events, never via shared database tables
- Reporting and analytics data is projected into a separate data warehouse (Amazon Redshift) via change data capture (CDC) using Debezium → MSK → Redshift

### 7.2 CQRS Where Applicable

For read-heavy domains (order history, provider listings), implement CQRS:
- **Write model:** Normalised relational store (Aurora)
- **Read model:** Denormalised projection (OpenSearch or a read replica)
- Projections are rebuilt from events — they are not the source of truth

### 7.3 Eventual Consistency Rules

- Services must explicitly document which of their operations are eventually consistent
- UI must handle eventual consistency gracefully (optimistic updates, polling, or websocket push)
- SLAs for eventual consistency propagation must be defined per event type

---

## 8. Resilience Patterns — Mandatory

Every service must implement the following:

| Pattern | Library | Applied To |
|---------|---------|-----------|
| **Circuit Breaker** | Resilience4j | All synchronous outbound calls |
| **Retry with backoff** | Resilience4j | Idempotent outbound calls |
| **Bulkhead** | Resilience4j | Isolate thread pools per downstream |
| **Timeout** | Resilience4j / Spring | All outbound calls — no unbounded waits |
| **Dead Letter Queue** | Kafka DLQ topic | All Kafka consumers |
| **Idempotency** | Custom (idempotency key in DB) | All payment and mutation operations |

---

## 9. Security Architecture

- **Zero-trust networking:** No service trusts another based on network location alone
- **mTLS:** All service-to-service gRPC calls use mutual TLS (Istio handles this)
- **JWT:** All external API calls carry a signed JWT; BFFs validate and strip before forwarding
- **IRSA:** All AWS API calls from pods use IAM Roles for Service Accounts — no static credentials
- **No secrets in environment variables** — all secrets from AWS Secrets Manager at startup

---

## 10. Architecture Decision Records (ADRs)

Any deviation from this blueprint, or any significant architectural decision, must be recorded as an ADR in `docs/adr/` in the relevant repository.

ADR template:
```
# ADR-NNN: Title

**Date:** YYYY-MM-DD  
**Status:** Proposed | Accepted | Deprecated  
**Deciders:** [names]

## Context
What problem are we solving?

## Decision
What did we decide?

## Consequences
What are the trade-offs?

## Alternatives Considered
What did we not pick, and why?
```

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
