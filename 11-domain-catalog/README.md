# 🗂️ 11 - Domain Catalog

> Detailed documentation for each bounded context in the platform - responsibilities, APIs, events, data models, dependencies, and team ownership.

> **Reference implementation:** The domains below represent a **reference domain model** for an order-and-fulfillment platform. Organizations adopting this manifesto should replace these domains with their own bounded contexts. The value is in the **patterns** (event flows, SLO tables, failure modes, data ownership) rather than the specific service boundaries.

## 🗺️ Domain Landscape

**Visual overview:**

```mermaid
flowchart TB
    subgraph customerJourney ["Customer Journey"]
        direction LR
        R1["Place Order"] --> R2["Get Estimate"]
        R2 --> R3["Confirm Order"]
        R3 --> R4["Get Assigned"]
        R4 --> R5["Track Provider"]
        R5 --> R6["Complete Order"]
        R6 --> R7["Pay & Rate"]
    end

    subgraph servicesInvolved ["Services Involved"]
        R1 -.- Orders
        R2 -.- Pricing
        R3 -.- Orders
        R4 -.- Fulfillment
        R5 -.- Geolocation
        R6 -.- Orders
        R7 -.- Payments
    end

    Orders["Order Service"]
    Pricing["Pricing Service"]
    Fulfillment["Fulfillment Engine"]
    Geolocation["Geolocation Service"]
    Payments["Payment Service"]
```

## 👥 Domain Ownership Matrix

| Domain | Team | Data Store | Key Events | Tier |
|--------|------|-----------|------------|------|
| [Order Service](./01-order-service.md) | Orders | Aurora PostgreSQL | `orders.order.*` | Tier 1 - Critical |
| [Fulfillment Engine](./02-fulfillment-engine.md) | Orders | Redis (geospatial) | `fulfillment.assignment.*` | Tier 1 - Critical |
| [Pricing Service](./03-pricing-service.md) | Commercial | Aurora PostgreSQL | `pricing.price.*` | Tier 1 - Critical |
| [Provider Profile](./04-provider-profile.md) | Providers | RDS PostgreSQL | `providers.provider.*` | Tier 2 - Important |
| [Customer Profile](./05-customer-profile.md) | Customers | RDS PostgreSQL | `customers.customer.*` | Tier 2 - Important |
| [Payment Service](./06-payment-service.md) | Payments | Aurora PostgreSQL | `payments.payment.*` | Tier 1 - Critical |
| [Notifications](./07-notifications.md) | Platform | RDS PostgreSQL | `notifications.*` | Tier 3 - Standard |
| [Geolocation Service](./08-geolocation-service.md) | Orders | (proxies external) | - | Tier 2 - Important |
| [Dynamic Pricing](./09-dynamic-pricing.md) | Commercial | Redis + RDS | `dynamicpricing.*` | Tier 2 - Important |
| [Fraud Engine](./10-fraud-engine.md) | Trust & Safety | Aurora PostgreSQL | `fraud.signal.*` | Tier 1 - Critical |

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

**Visual overview:**

```mermaid
flowchart TB
    subgraph ordersDB ["Orders DB - Aurora"]
        ordersTable["orders"]
        orderEvents["order_events"]
        outbox["outbox_events"]
    end

    subgraph fulfillmentStore ["Fulfillment - Redis"]
        providerLocations["provider:locations GEO"]
        activeAssignments["active_assignments"]
    end

    subgraph pricingDB ["Pricing DB - Aurora"]
        pricingRules["pricing_rules"]
        priceEstimates["price_estimates"]
        pricingHistory["pricing_history"]
    end

    orderService["Order Service"] --> ordersDB
    fulfillmentEngine["Fulfillment Engine"] --> fulfillmentStore
    pricingService["Pricing Service"] --> pricingDB
```

Profile and payment services each own an isolated data store with no shared tables.

**Visual overview:**

```mermaid
flowchart TB
    subgraph providerDB ["Provider DB - RDS"]
        providers["providers"]
        documents["documents"]
        ratings["provider_ratings"]
    end

    subgraph customerDB ["Customer DB - RDS"]
        customers["customers"]
        paymentMethods["payment_methods"]
        orderHistory["order_history - projection"]
    end

    subgraph paymentsDB ["Payments DB - Aurora"]
        payments["payments"]
        payouts["payouts"]
        wallets["wallets"]
    end

    providerProfile["Provider Profile"] --> providerDB
    customerProfile["Customer Profile"] --> customerDB
    paymentService["Payment Service"] --> paymentsDB
```

The following diagram shows forbidden direct database access between services.

**Visual overview:**

```mermaid
flowchart TB
    orderService["Order Service"] -.->|"never"| providerDB[("Provider DB")]
    orderService -.->|"never"| paymentsDB[("Payments DB")]
    paymentService["Payment Service"] -.->|"never"| ordersDB[("Orders DB")]
```

*Dashed lines with "never" = forbidden direct database access. Services communicate via APIs and events only. The `order_history` table in the Customer DB is a **read-model projection** maintained via Kafka events from the Order Service - the Order Service remains the source of truth for all order lifecycle data. Cross-domain writes to this projection are forbidden; it is updated exclusively through event consumption.*

## 📝 Domain Documentation Standards

Every domain document in this catalog must include the following subsections. When creating or updating a domain document, verify that each section is present and populated. Missing sections should be added during the next domain review cycle.

### Required Subsections

1. **SLOs and Error Budgets** - availability target, latency p99, error rate ceiling, and error budget policy (what happens when the budget is exhausted)
2. **Failure Modes** - degraded behavior description, user-facing impact, fallback strategy, and blast radius
3. **Capacity Sizing** - instance count (min/max replicas), database connection pool size, peak QPS, and HPA configuration (target CPU/memory, scale-up/scale-down behavior)
4. **Data Retention Matrix** - per data store (DB tables, Kafka topics, S3 buckets, log groups): what is retained, for how long, and the deletion/archival mechanism
5. **Allowed Callers** - which services may call this service, via which protocol (gRPC, REST, Kafka), and the authorization mechanism (mTLS identity, RBAC role, API key)

### Domain-to-Domain Access Matrix

Allowed synchronous (gRPC) and asynchronous (Kafka) communication paths between domains. Any path not listed here is **forbidden** without an ADR and cross-team review.

| Source ↓ / Target → | Order Service | Fulfillment | Pricing | Provider Profile | Customer Profile | Payments | Notifications | Geolocation | Dynamic Pricing | Fraud Engine |
|----------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Order Service** | - | gRPC | gRPC | gRPC (read) | gRPC (read) | Kafka | Kafka | gRPC | - | gRPC |
| **Fulfillment** | Kafka | - | - | gRPC (read) | - | - | Kafka | gRPC | - | gRPC |
| **Pricing** | - | - | - | - | - | - | - | gRPC | Kafka (consume) | - |
| **Provider Profile** | Kafka (consume) | Kafka | - | - | - | - | Kafka | - | - | Kafka (consume) |
| **Customer Profile** | Kafka (consume) | - | - | - | - | Kafka (consume) | - | - | - | - |
| **Payments** | Kafka (consume) | - | - | - | - | - | Kafka | - | - | Kafka (consume) |
| **Notifications** | Kafka (consume) | - | - | Kafka (consume) | - | Kafka (consume) | - | - | - | - |
| **Geolocation** | - | - | - | - | - | - | - | - | - | - |
| **Dynamic Pricing** | Kafka (consume) | - | - | Kafka (consume) | - | - | Kafka | - | - | - |
| **Fraud Engine** | Kafka (consume) | - | - | Kafka (consume) | Kafka (consume) | Kafka (consume) | Kafka | - | - | - |

### Domain Sunset Process

When a domain is deprecated or merged, follow the deprecation lifecycle process documented in [03-engineering-practices/08-deprecation-lifecycle.md](../03-engineering-practices/08-deprecation-lifecycle.md). This includes consumer notification timelines, traffic migration plans, and data ownership transfer procedures.

---
<div align="center">

🏠 [Back to root](../README.md)

</div>
