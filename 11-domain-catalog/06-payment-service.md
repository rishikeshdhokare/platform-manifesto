# 💳 Payment Service

![Status: Active](https://img.shields.io/badge/status-Active-green?style=flat-square) ![Owner: Team Payments](https://img.shields.io/badge/owner-Team_Payments-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

**Service identifier:** `{company}.payments`

---

## 📋 1. Overview

The **Payment Service** is the bounded context responsible for **payment processing**, **settlements**, **customer wallets**, and **refunds** after an order's financial outcome is known. It integrates with external payment providers (e.g., Stripe, Adyen) under the `{company}.orders` ecosystem contracts.

### 1.1 Ownership

| Owns | Does not own |
|------|----------------|
| Payment transactions (authorize, capture, void, refund) | Price calculation (Pricing Service) |
| Provider payouts and payout scheduling | Order lifecycle state (Order Service) |
| Customer wallet balances and wallet ledger entries | Card network / issuer rules (provider) |

### 1.2 PCI-DSS scope boundaries

Cardholder data is **never** persisted in platform systems in raw form. **Tokenization and vaulting** are delegated to the payment provider; the platform stores only **provider tokens**, **last4**, **brand**, and **expiry metadata** where permitted. The **PCI-DSS assessment scope** for application infrastructure is limited to **token handling** and **provider API integration** - not full card data processing.

**Visual overview:**

```mermaid
flowchart LR
    subgraph platform_pci [{Company} - reduced PCI scope]
        PS[Payment Service]
        API[REST APIs]
        DB[(Aurora - tokens & ledger)]
        PS --> API
        PS --> DB
    end

    subgraph provider_pci [Payment Provider - full PCI scope]
        TOK[Tokenization / vault]
        CHD[Cardholder data zone]
        CHD --> TOK
    end

    Customer[Customer / Client] -->|card via provider SDK| provider_pci
    PS <-->|tokens only| provider_pci
```

---

## 🔄 2. Payment lifecycle

State transitions for a single payment intent tied to an order (`{company}.orders` correlation).

**Visual overview:**

```mermaid
stateDiagram-v2
    [*] --> AUTHORIZED: authorize OK
    AUTHORIZED --> CAPTURED: capture OK
    CAPTURED --> SETTLED: provider settlement confirmed
    SETTLED --> PAID_OUT: provider payout completed

    AUTHORIZED --> VOIDED: void before capture
    CAPTURED --> REFUNDED: full or partial refund

    note right of REFUNDED
        Partial refunds may
        remain CAPTURED for
        remaining captured amount
    end note
```

| State | Meaning |
|-------|---------|
| **AUTHORIZED** | Funds held; not yet transferred to settlement. |
| **CAPTURED** | Charge completed; awaiting settlement batch from provider. |
| **SETTLED** | Provider settlement reconciled to internal ledger. |
| **PAID_OUT** | Provider share paid via payout rail. |
| **VOIDED** | Authorization released; no capture. |
| **REFUNDED** | Full or partial money returned to customer (provider-side). |

---

## 🔀 3. Payment flow

End-to-end flow from order completion through payout, including failure and retry semantics.

**Visual overview:**

```mermaid
sequenceDiagram
    participant Orders as Order Service<br/>{company}.orders
    participant Bus as Event Bus
    participant Pay as Payment Service<br/>{company}.payments
    participant Prov as Payment Provider<br/>Stripe / Adyen
    participant Payout as Payout Rail

    Orders->>Bus: orders.order.completed
    Bus->>Pay: consume order.completed

    Pay->>Prov: Authorize payment (token, amount)
    alt authorize fails
        Prov-->>Pay: error / decline
        Pay->>Bus: payments.payment.failed
        Pay->>Pay: retry with backoff (idempotent)
        Note over Pay: max attempts / DLQ per policy
    else authorize OK
        Prov-->>Pay: authorization id
        Pay->>Bus: payments.payment.authorized
        Pay->>Prov: Capture payment
        alt capture fails
            Prov-->>Pay: error
            Pay->>Bus: payments.payment.failed
            Pay->>Pay: retry capture (idempotency key)
        else capture OK
            Prov-->>Pay: capture confirmation
            Pay->>Bus: payments.payment.captured
            Pay->>Prov: Confirm settlement / batch status
            alt settlement pending
                Prov-->>Pay: pending
                Pay->>Pay: poll / webhook reconcile
            end
            Prov-->>Pay: settled
            Pay->>Pay: Calculate provider payout (from captured amount)
            Pay->>Payout: Initiate payout
            alt payout fails
                Payout-->>Pay: error
                Pay->>Pay: retry payout (ledger idempotent)
            else payout OK
                Payout-->>Pay: completed
                Pay->>Bus: payments.payout.completed
            end
        end
    end
```

---

## 🧩 4. Domain model

Core aggregates and value objects for the Payment Service.

**Visual overview:**

```mermaid
classDiagram
    class Payment {
        +UUID id
        +String orderId
        +Money amountCents
        +PaymentStatus status
        +String providerPaymentId
        +String idempotencyKey
    }

    class PaymentStatus {
        <<enumeration>>
        AUTHORIZED
        CAPTURED
        SETTLED
        PAID_OUT
        VOIDED
        REFUNDED
        FAILED
    }

    class PaymentMethod {
        +UUID id
        +String customerId
        +String providerToken
        +String brand
        +String last4
        +Boolean isDefault
    }

    class Payout {
        +UUID id
        +String providerId
        +Money amountCents
        +String status
        +String providerPayoutId
        +UUID linkedPaymentId
    }

    class Wallet {
        +UUID id
        +String customerId
        +Money balanceCents
        +String currency
    }

    class Transaction {
        +UUID id
        +UUID walletId
        +Money deltaCents
        +String type
        +String referenceId
        +DateTime createdAt
    }

    class RefundRequest {
        +UUID id
        +UUID paymentId
        +Money amountCents
        +String status
        +String reason
    }

    Payment "1" --> "1" PaymentStatus
    Payment "n" --> "1" PaymentMethod : uses
    Payment "1" --> "0..n" Payout : funds
    Payment "1" --> "0..n" RefundRequest
    Wallet "1" --> "0..*" Transaction : ledger
```

---

## 🔌 5. API surface

Base path: `/v1` · All mutation endpoints require **`Idempotency-Key`** header (see section 9).

| Method | Path | Description |
|--------|------|---------------|
| `POST` | `/v1/payments/authorize` | Hold funds for an order; returns payment id and provider auth reference. |
| `POST` | `/v1/payments/{id}/capture` | Finalize charge for authorized payment. |
| `POST` | `/v1/payments/{id}/refund` | Full or partial refund (body: `amountCents`, optional `reason`). |
| `GET` | `/v1/payments/{id}` | Payment status, amounts (cents), timestamps, linked order id. |
| `GET` | `/v1/wallets/{customerId}` | Wallet balance and recent transaction summary (paginated). |

**Internal contract packages:** `{company}.payments.api` (REST), `{company}.payments.events` (async).

---

## 📤 6. Events published

| Event name | Payload highlights | Typical consumers |
|------------|-------------------|-------------------|
| `payments.payment.authorized` | `paymentId`, `orderId`, `amountCents`, `customerId` | Fraud Engine, Analytics Pipeline, Notifications |
| `payments.payment.captured` | `paymentId`, `orderId`, `capturedCents`, `providerReference` | Accounting / Finance feeds, Analytics Pipeline, Orders (read-model enrichment) |
| `payments.payment.failed` | `paymentId`, `orderId`, `reasonCode`, `retryable` | Notifications, Fraud Engine, Support tooling |
| `payments.payout.completed` | `payoutId`, `providerId`, `amountCents`, `paymentId` | Provider Profile, Notifications, Accounting |
| `payments.refund.processed` | `refundId`, `paymentId`, `amountCents`, `status` | Customer Profile, Notifications, Analytics Pipeline |

---

## 📥 7. Events consumed

| Event name | Producer | Purpose |
|------------|----------|---------|
| `orders.order.completed` | Order Service (`{company}.orders`) | Trigger authorize → capture → settlement → payout pipeline for the finalized price. |
| `orders.order.cancelled` | Order Service | Void uncaptured authorizations or apply cancellation fee policy via capture/refund rules. |

---

## 💾 8. Data store

| Aspect | Choice |
|--------|--------|
| **Primary database** | **Amazon Aurora PostgreSQL** - ACID guarantees for ledger, payouts, and wallet balances. |
| **Monetary amounts** | **Integer cents** everywhere (DB columns, APIs, events); single currency per row where applicable. |

### 8.1 Core tables

| Table | Role |
|-------|------|
| `payments` | Payment intents, status, provider ids, order correlation, idempotency keys. |
| `payouts` | Provider payouts, linkage to payments, rail references. |
| `wallets` | Per-customer wallet header (balance in cents). |
| `wallet_transactions` | Append-only ledger lines for wallet credits/debits. |
| `refunds` | Refund requests and outcomes tied to `payments`. |

**Visual overview:**

```mermaid
erDiagram
    payments ||--o{ payouts : "funds"
    payments ||--o{ refunds : "reverses"
    wallets ||--o{ wallet_transactions : "ledger"
    payments {
        uuid id PK
        uuid order_id
        bigint amount_cents
        text status
        text provider_payment_id
        text idempotency_key
    }
    payouts {
        uuid id PK
        uuid payment_id FK
        text provider_id
        bigint amount_cents
        text status
    }
    wallets {
        uuid id PK
        text customer_id
        bigint balance_cents
    }
    wallet_transactions {
        uuid id PK
        uuid wallet_id FK
        bigint delta_cents
        text type
    }
    refunds {
        uuid id PK
        uuid payment_id FK
        bigint amount_cents
        text status
    }
```

---

## 🔒 9. Security and compliance

| Topic | Payment Service policy |
|-------|------------------------------|
| **PCI-DSS** | No storage of raw PAN/CVV; **tokenization via Stripe/Adyen**; provider maintains PCI validation; platform handles tokens and non-sensitive metadata only. |
| **Idempotency** | **`Idempotency-Key` mandatory** on all mutating REST operations (`authorize`, `capture`, `refund`, wallet adjustments) to prevent duplicate side effects. |
| **Double-charge prevention** | **Distributed locking** (e.g., Redis Redlock or DB advisory locks) keyed by `orderId` + payment phase during authorize/capture windows. |
| **Secrets** | Provider API keys in AWS Secrets Manager; no keys in application images or repos. |

---

## 🔄 10. Reconciliation

**Visual overview:**

```mermaid
flowchart TB
    subgraph daily [Daily reconciliation job - {company}.payments]
        J[L internal ledger<br/>payments / payouts]
        P[Provider reports<br/>settlements / fees]
        J --> C[Compare totals & line items]
        P --> C
        C --> OK{match?}
        OK -->|yes| LOG[audit log]
        OK -->|no| ALERT[P2 severity alert<br/>Finance / Payments on-call]
    end
```

| Step | Description |
|------|-------------|
| **Schedule** | Automated **daily** reconciliation between internal Aurora ledger and payment provider settlement files / APIs. |
| **Scope** | Captured amounts, fees, refunds, payout batches. |
| **Discrepancies** | Raised as **P2 severity** alerts with ticket automation; no silent adjustment without human or policy-approved rules engine. |

---

## 📊 11. Key metrics

| Metric | Target |
|--------|--------|
| **Availability** | **99.99%** monthly (excluding provider outages attributed separately). |
| **Payment success rate** | **> 99.5%** for capture after successful authorization (platform-attributable failures). |
| **P99 latency** | **< 1s** for synchronous authorize/capture API paths (platform service + provider round-trip budget shared). |
| **Settlement lag** | **< 24h** from capture to internally reconciled **SETTLED** state under normal provider batching. |

---

## 👥 12. Team and ownership

| Role | Assignment |
|------|------------|
| **Owning team** | **Team Payments** |
| **Escalations** | P2 reconciliation, payout failures, PCI-related incidents |
| **Related domains** | Orders (`{company}.orders`), Pricing, Fraud Engine, Customer/Provider profiles |

---

## 📊 13. SLOs and Error Budgets

| Metric | Target | Error Budget Policy |
|--------|--------|-------------------|
| Availability | 99.99% | Freeze non-critical deploys when budget < 20% remaining |
| Latency (p99) | < 1000ms | Alert at 80% of budget consumption |
| Error rate | < 0.1% | Page on-call when rate exceeds target for > 5 minutes |

## ⚠️ 14. Failure Modes

| Failure | User Impact | Fallback | Blast Radius |
|---------|------------|----------|-------------|
| Aurora PostgreSQL unavailable | Cannot authorize, capture, or refund | Fail closed; queue idempotent retries at BFF; P1 incident | All payment and wallet operations |
| Payment provider API outage | Authorize/capture fail | Surface retryable errors; idempotency keys prevent double charge on retry | Provider rail `{providerName}` only |
| Redis lock / idempotency store unavailable | Risk of duplicate side effects if clients retry blindly | Fail closed on mutating paths until locks recover; alert on-call | Mutations only |
| Kafka lag on consumed order topics | Delayed post-order capture pipeline | Orders complete in domain; Payments catches up; monitor lag SLO | Settlement delay, not double capture if guards hold |

## 📏 15. Capacity Sizing

| Parameter | Value |
|-----------|-------|
| Min replicas | 6 |
| Max replicas | 50 |
| Target CPU | 70% |

## 🗄️ 16. Data Retention Matrix

| Store | Data | Retention | Mechanism |
|-------|------|-----------|-----------|
| PostgreSQL | `payments`, `payouts`, `wallets`, `wallet_transactions`, `refunds` | `{ledgerRetention}` per PCI and finance policy | Immutable ledger rows; anonymize after `{accountClosurePeriod}` |
| Kafka | `payments.payment.*`, `payments.payout.*` | 14 days (platform default) | Topic retention policy |
| CloudWatch Logs | Application logs (redacted) | 90 days | Log group retention policy |
| S3 | Provider settlement file drops | `{settlementFileRetention}` | Lifecycle policy |

## 🔐 17. Allowed Callers

| Caller | Protocol | Authorization |
|--------|----------|--------------|
| Order Service (`{company}.orders`) | Kafka (consume `orders.order.completed` / `cancelled`) | mTLS + consumer ACL |
| Customer BFF (`{company}.bff.customer`) | REST | mTLS + OAuth2 + scoped payment methods |
| Provider BFF (`{company}.bff.provider`) | REST (payout status reads) | mTLS + OAuth2 |
| Fraud Engine (`{company}.fraud`) | Kafka (consume `payments.payment.*`) | mTLS + consumer ACL |
| Finance reconciliation jobs (`{company}.payments-recon`) | REST internal | mTLS + service API key |

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
