# 🗂️ 11 — Domain Catalog

![Docs](https://img.shields.io/badge/docs-10-blue?style=flat-square)

Detailed documentation for each bounded context in the platform — responsibilities, APIs, events, data models, dependencies, and team ownership.

## 🗺️ Domain Landscape

```mermaid
flowchart TB
    subgraph customer_journey [Customer Journey]
        direction LR
        R1[Open App] --> R2[Place Order]
        R2 --> R3[Get Price Estimate]
        R3 --> R4[Confirm Order]
        R4 --> R5[Get Assigned]
        R5 --> R6[Track Provider]
        R6 --> R7[Start Fulfillment]
        R7 --> R8[Complete Order]
        R8 --> R9[Pay & Rate]
    end

    subgraph services_involved [Services Involved at Each Step]
        R2 -.- Orders
        R3 -.- Pricing
        R4 -.- Orders
        R5 -.- Fulfillment
        R6 -.- Geolocation
        R7 -.- Orders
        R8 -.- Orders
        R9 -.- Payments
    end

    Orders[Order Service]
    Pricing[Pricing Service]
    Fulfillment[Fulfillment Engine]
    Geolocation[Geolocation Service]
    Payments[Payment Service]
```

## 👥 Domain Ownership Matrix

| Domain | Team | Data Store | Key Events | Tier |
|--------|------|-----------|------------|------|
| [Order Service](./01-order-service.md) | Orders | Aurora PostgreSQL | `orders.order.*` | Tier 1 — Critical |
| [Fulfillment Engine](./02-fulfillment-engine.md) | Orders | Redis (geospatial) | `fulfillment.assignment.*` | Tier 1 — Critical |
| [Pricing Service](./03-pricing-service.md) | Commercial | Aurora PostgreSQL | `pricing.price.*` | Tier 1 — Critical |
| [Provider Profile](./04-provider-profile.md) | Providers | RDS PostgreSQL | `providers.provider.*` | Tier 2 — Important |
| [Customer Profile](./05-customer-profile.md) | Customers | RDS PostgreSQL | `customers.customer.*` | Tier 2 — Important |
| [Payment Service](./06-payment-service.md) | Payments | Aurora PostgreSQL | `payments.payment.*` | Tier 1 — Critical |
| [Notifications](./07-notifications.md) | Platform | RDS PostgreSQL | `notifications.*` | Tier 3 — Standard |
| [Geolocation Service](./08-geolocation-service.md) | Orders | (proxies external) | — | Tier 2 — Important |
| [Dynamic Pricing](./09-dynamic-pricing.md) | Commercial | Redis + RDS | `dynamicpricing.*` | Tier 2 — Important |
| [Fraud Engine](./10-fraud-engine.md) | Trust & Safety | Aurora PostgreSQL | `fraud.signal.*` | Tier 1 — Critical |

## 🔀 Cross-Domain Event Flow

```mermaid
flowchart LR
    subgraph order_lifecycle [Order Lifecycle Events]
        T1[orders.order.requested]
        T2[orders.order.assigned]
        T3[orders.order.started]
        T4[orders.order.completed]
        T5[orders.order.cancelled]
    end

    subgraph consumers [Downstream Consumers]
        Fulfillment[Fulfillment Engine]
        Payments[Payment Service]
        Notifications[Notifications]
        DynamicPricing[Dynamic Pricing]
        Fraud[Fraud Engine]
        Analytics[Analytics Pipeline]
    end

    T1 --> Fulfillment
    T1 --> Fraud
    T2 --> Notifications
    T3 --> DynamicPricing
    T4 --> Payments
    T4 --> Notifications
    T4 --> Analytics
    T5 --> Payments
    T5 --> Fulfillment
    T5 --> Notifications
```

## 🔐 Data Ownership Boundaries

```mermaid
flowchart TB
    subgraph orders_db [Orders DB — Aurora]
        orders_table[orders]
        order_events[order_events]
        outbox[outbox_events]
    end

    subgraph fulfillment_store [Fulfillment — Redis]
        provider_locations[provider:locations GEO]
        active_assignments[active_assignments]
    end

    subgraph pricing_db [Pricing DB — Aurora]
        pricing_rules[pricing_rules]
        price_estimates[price_estimates]
        pricing_history[pricing_history]
    end

    subgraph provider_db [Provider DB — RDS]
        providers[providers]
        documents[documents]
        ratings[provider_ratings]
    end

    subgraph customer_db [Customer DB — RDS]
        customers[customers]
        payment_methods[payment_methods]
        order_history[order_history]
    end

    subgraph payments_db [Payments DB — Aurora]
        payments[payments]
        payouts[payouts]
        wallets[wallets]
    end

    OrderService[Order Service] --> orders_db
    FulfillmentEngine[Fulfillment Engine] --> fulfillment_store
    PricingService[Pricing Service] --> pricing_db
    ProviderProfile[Provider Profile] --> provider_db
    CustomerProfile[Customer Profile] --> customer_db
    PaymentService[Payment Service] --> payments_db

    OrderService -.->|"never"| provider_db
    OrderService -.->|"never"| payments_db
    PaymentService -.->|"never"| orders_db
```

*Dashed lines with "never" = forbidden direct database access. Services communicate via APIs and events only.*

## 📝 Domain Documentation Standards

Every domain document in this catalog must include the following subsections. When creating or updating a domain document, verify that each section is present and populated. Missing sections should be added during the next domain review cycle.

### Required Subsections

1. **SLOs and Error Budgets** — availability target, latency p99, error rate ceiling, and error budget policy (what happens when the budget is exhausted)
2. **Failure Modes** — degraded behaviour description, user-facing impact, fallback strategy, and blast radius
3. **Capacity Sizing** — instance count (min/max replicas), database connection pool size, peak QPS, and HPA configuration (target CPU/memory, scale-up/scale-down behaviour)
4. **Data Retention Matrix** — per data store (DB tables, Kafka topics, S3 buckets, log groups): what is retained, for how long, and the deletion/archival mechanism
5. **Allowed Callers** — which services may call this service, via which protocol (gRPC, REST, Kafka), and the authorization mechanism (mTLS identity, RBAC role, API key)

### Domain-to-Domain Access Matrix

Allowed synchronous (gRPC) and asynchronous (Kafka) communication paths between domains. Any path not listed here is **forbidden** without an ADR and cross-team review.

| Source ↓ / Target → | Order Service | Fulfillment | Pricing | Provider Profile | Customer Profile | Payments | Notifications | Geolocation | Dynamic Pricing | Fraud Engine |
|----------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Order Service** | — | gRPC | gRPC | gRPC (read) | gRPC (read) | Kafka | Kafka | gRPC | — | gRPC |
| **Fulfillment** | Kafka | — | — | gRPC (read) | — | — | Kafka | gRPC | — | gRPC |
| **Pricing** | — | — | — | — | — | — | — | gRPC | Kafka (consume) | — |
| **Provider Profile** | Kafka (consume) | Kafka | — | — | — | — | Kafka | — | — | Kafka (consume) |
| **Customer Profile** | Kafka (consume) | — | — | — | — | Kafka (consume) | — | — | — | — |
| **Payments** | Kafka (consume) | — | — | — | — | — | Kafka | — | — | Kafka (consume) |
| **Notifications** | Kafka (consume) | — | — | Kafka (consume) | — | Kafka (consume) | — | — | — | — |
| **Geolocation** | — | — | — | — | — | — | — | — | — | — |
| **Dynamic Pricing** | Kafka (consume) | — | — | Kafka (consume) | — | — | Kafka | — | — | — |
| **Fraud Engine** | Kafka (consume) | — | — | Kafka (consume) | Kafka (consume) | Kafka (consume) | Kafka | — | — | — |

### Domain Sunset Process

When a domain is deprecated or merged, follow the deprecation lifecycle process documented in [03-engineering-practices/08-deprecation-lifecycle.md](../03-engineering-practices/08-deprecation-lifecycle.md). This includes consumer notification timelines, traffic migration plans, and data ownership transfer procedures.

---
<div align="center">

🏠 [Back to root](../README.md)

</div>
