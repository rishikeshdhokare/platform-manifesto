# 💲 Pricing Service

![Status: Active](https://img.shields.io/badge/Status-Active-green?style=flat-square)
![Owner](https://img.shields.io/badge/Owner-Platform_Engineering_--_Pricing-grey?style=flat-square)
![Last Updated](https://img.shields.io/badge/Last_Updated-2026--03--31-grey?style=flat-square)

---

## 📋 1. Overview

The **Pricing Service** (`com.{company}.pricing`) is the system of record for **how much an order costs** before and after fulfillment. It turns route geometry, time, service class, and applicable rules into transparent price numbers that customers see in the app and that downstream systems use for receipts and analytics.

**Core responsibilities**

- **Price calculation** — Combine base price, distance, and time according to published rules for the customer's market and service type.
- **Estimates** — Produce pre-order price ranges and point estimates for order planning and customer consent.
- **Final price computation** — Reconcile estimated logic with actual distance/duration (and policy) after order completion to produce the billed amount consumed by billing and receipts.

**This domain owns**

| Concern | Description |
| --- | --- |
| Pricing rules | Per-market, per-service-type parameters (base, per-km, per-minute, minimums, cancellation fees). |
| Pricing models | How components (base + distance + time) compose into a subtotal before external multipliers. |
| Estimates | Creation, storage, and retrieval of price estimates tied to order intent or quotes. |

**This domain does not own**

| Concern | Owning domain |
| --- | --- |
| Dynamic pricing multipliers | **Dynamic Pricing** — defines and publishes current multipliers; Pricing **consumes** them. |
| Payment collection | **Payments / Billing** — charges cards, wallets, and settlement. |

---

## 🔄 2. Price Calculation Flow

The following flow describes how the platform produces a **price estimate** (final price follows the same structural steps with order-completed inputs and reconciliation rules).

```mermaid
flowchart TD
    A[Receive PriceCalculationRequest] --> B[Get route distance and duration from Geolocation]
    B --> C[Look up base pricing rules for market + service type]
    C --> D[Apply distance rate + time rate]
    D --> E[Query pricing multiplier from Dynamic Pricing]
    E --> F[Apply multiplier to subtotal]
    F --> G[Apply promotions / discounts]
    G --> H[Return PriceEstimate with PriceBreakdown]
```

**Notes**

- Geolocation integration uses the platform's **Geolocation Service** (`com.{company}.geolocation`) over gRPC for canonical distance and duration.
- Dynamic Pricing is **read-only** from Pricing's perspective; multiplier source of truth remains the Dynamic Pricing domain.

---

## 🧩 3. Domain Model

```mermaid
classDiagram
    class PriceCalculationRequest {
        +UUID requestId
        +String marketCode
        +ServiceType serviceType
        +GeoRoute route
        +Instant requestedAt
        +Optional~UUID~ orderId
        +Optional~UUID~ customerId
    }

    class PriceEstimate {
        +UUID estimateId
        +MoneyRange estimateRange
        +PriceBreakdown breakdown
        +Instant validUntil
        +String currency
    }

    class PricingRule {
        +String marketCode
        +ServiceType serviceType
        +int basePriceCents
        +int perKmCents
        +int perMinuteCents
        +int minimumPriceCents
        +int cancellationFeeCents
    }

    class PriceBreakdown {
        +int basePriceCents
        +int distanceChargeCents
        +int timeChargeCents
        +BigDecimal pricingMultiplier
        +int multiplierAmountCents
        +int discountCents
        +int totalAmountCents
        +String currency
    }

    class ServiceType {
        <<enumeration>>
        ECONOMY
        STANDARD
        PREMIUM
        XL
        ACCESSIBLE
    }

    PriceCalculationRequest --> ServiceType
    PricingRule --> ServiceType
    PriceEstimate --> PriceBreakdown : contains
    PriceCalculationRequest ..> PriceEstimate : produces
```

---

## 🔌 4. API Surface

### 4.1 gRPC (`com.{company}.pricing.v1`)

| RPC | Purpose |
| --- | --- |
| `CalculatePriceEstimate` | Pre-order quote from route (or origin/destination) and context; persists estimate for idempotency and audit. |
| `CalculateFinalPrice` | Post-order final amount from actual distance/duration and linked estimate or order. |
| `GetPriceBreakdown` | Retrieve a stored breakdown by `estimateId` or `orderId` for support, receipts, and BFF hydration. |

gRPC API package: **`com.{company}.pricing.v1`**. Shared order identifiers and enums align with **`com.{company}.orders`** types where applicable.

### 4.2 REST (BFF-facing)

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/pricing/estimates` | Query-string parameters for mobile/web BFFs (e.g. `origin`, `destination`, `service_type`, `market`) returning estimate payload aligned with gRPC `PriceEstimate`. |

REST is **read-optimized** for estimates; authoritative writes and final price triggers remain on gRPC and events.

### 4.3 Request lifecycle (API ↔ domain)

```mermaid
sequenceDiagram
    participant BFF as {Company} BFF
    participant PS as Pricing Service
    participant Geo as Geolocation Service
    participant DP as Dynamic Pricing

    BFF->>PS: GET /v1/pricing/estimates
    PS->>Geo: RouteMetrics(distance, duration)
    Geo-->>PS: metrics
    PS->>DP: GetCurrentMultiplier(market, serviceType)
    DP-->>PS: multiplier
    PS-->>BFF: PriceEstimate + PriceBreakdown
```

---

## 📤 5. Events Published

All topics use the platform naming prefix `com.{company}.events`.

| Event | Topic / type | Payload highlights | Typical consumers | Retention |
| --- | --- | --- | --- | --- |
| `pricing.price.estimated` | `com.{company}.events.pricing.price.estimated` | `estimateId`, `orderId?`, `breakdown`, `validUntil` | Analytics, Search ranking, Notifications | 7 days |
| `pricing.price.calculated` | `com.{company}.events.pricing.price.calculated` | `orderId`, `finalTotalCents`, `estimateId`, `deltaFromEstimate` | Billing, Orders state, Data warehouse | 30 days |

---

## 📥 6. Events Consumed

| Event | Source domain | Purpose in Pricing |
| --- | --- | --- |
| `dynamicpricing.multiplier.updated` | Dynamic Pricing | Invalidate or refresh cached multipliers per market/service slice; ensures new estimates use current pricing. |
| `orders.order.completed` | Order Service (`com.{company}.orders`) | Trigger **final price reconciliation**: pull actual distance/duration, re-run rule engine, emit `pricing.price.calculated`, persist `price_calculations`. |

```mermaid
flowchart LR
    subgraph Published_by_Pricing
        E1[pricing.price.estimated]
        E2[pricing.price.calculated]
    end
    subgraph Consumed_by_Pricing
        I1[dynamicpricing.multiplier.updated]
        I2[orders.order.completed]
    end
    DP[Dynamic Pricing] --> I1
    Orders[Order Service] --> I2
    E1 --> Analytics[Analytics]
    E2 --> Billing[Billing]
```

---

## ⚙️ 7. Pricing Rules Configuration

Pricing parameters are **authoritative in Aurora PostgreSQL**, keyed by **market** and **service type**. Operational changes go through the platform's config pipeline (validation, audit, gradual rollout).

**Example logical table: `pricing_rules` (excerpt)**

| market | service_type | base_price_cents | per_km_cents | per_minute_cents | minimum_price_cents | cancellation_fee_cents |
| --- | --- | --- | --- | --- | --- | --- |
| `US-NYC` | `ECONOMY` | 800 | 120 | 35 | 1200 | 1500 |
| `US-NYC` | `STANDARD` | 1200 | 180 | 50 | 2000 | 2000 |
| `US-LAX` | `ECONOMY` | 750 | 110 | 32 | 1100 | 1500 |

**Constraints (recommended)**

- Unique `(market, service_type)` per effective dating row if versioning is used.
- Currency implied by market or explicit `currency` column in production schemas.

---

## 💾 8. Data Store

**Primary store:** Aurora PostgreSQL (provisioned for Pricing).

| Table | Role |
| --- | --- |
| `pricing_rules` | Versioned or current effective pricing parameters per market and service type. |
| `price_estimates` | Immutable records of estimates (ids, inputs hash, breakdown JSON, `valid_until`). |
| `price_calculations` | Final price outcomes linked to `order_id` / `estimate_id`, actual metrics, and audit fields. |

```mermaid
erDiagram
    PRICING_RULES ||--o{ PRICE_ESTIMATES : "rules_version_ref"
    PRICE_ESTIMATES ||--o| PRICE_CALCULATIONS : "reconciles_to"
    PRICING_RULES {
        string market PK
        string service_type PK
        int base_price_cents
        int per_km_cents
        int per_minute_cents
    }
    PRICE_ESTIMATES {
        uuid estimate_id PK
        uuid order_id FK
        jsonb breakdown
        timestamptz valid_until
    }
    PRICE_CALCULATIONS {
        uuid calculation_id PK
        uuid order_id FK
        uuid estimate_id FK
        int final_total_cents
    }
```

---

## 📊 9. Key Metrics

### 9.1 SLOs and quality

| Metric | Target | Notes |
| --- | --- | --- |
| Estimate path P99 latency | **< 100 ms** | End-to-end Pricing handler after Geolocation + Dynamic Pricing calls; excludes extreme cold starts. |
| Price accuracy | **≥ 85%** of orders within **15%** of pre-order estimate | Measured as `abs(final - estimate) / estimate` capped distribution; excludes customer-caused route changes per policy. |

### 9.2 Business metrics

| Metric | Dimensions |
| --- | --- |
| `estimates_calculated` | `market`, `service_type`, `source` (app, BFF, API) |
| `final_prices_computed` | `market`, `service_type`, success/failure |

---

## 🔗 10. Dependencies

```mermaid
flowchart TB
    subgraph Pricing_Service["Pricing Service com.{company}.pricing"]
        PC[Price engine]
    end
    Geo["Geolocation Service\ncom.{company}.geolocation\ngRPC: distance, duration"]
    DP["Dynamic Pricing\ncom.{company}.dynamicpricing\ngRPC: current multiplier"]
    Orders["Order Service\ncom.{company}.orders\nKafka: orders.order.completed"]

    Geo -->|gRPC| PC
    DP -->|gRPC| PC
    Orders -->|Kafka consume| PC
```

| Dependency | Integration | Used for |
| --- | --- | --- |
| Geolocation Service | gRPC | Route distance and duration for estimates and final price. |
| Dynamic Pricing | gRPC | Current pricing multiplier (read-only). |
| Order Service | Kafka (`orders.order.completed`) | Final price reconciliation trigger. |

---

## 👥 11. Team & Ownership

| Role | Team |
| --- | --- |
| Service owner | **Team Commercial** |
| Escalations | Team Commercial on-call → Platform Engineering |

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
