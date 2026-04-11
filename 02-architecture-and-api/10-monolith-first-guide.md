# 🏗️ Monolith-First Architecture

![Status: Guidance](https://img.shields.io/badge/status-Guidance-orange?style=flat-square) ![Owner: Architecture](https://img.shields.io/badge/owner-Architecture-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Overview

Starting with microservices is a common mistake. Teams adopt distributed systems before they understand their domain, then spend months fighting accidental complexity instead of shipping features.

**The default at {Company} for new products and small teams is a modular monolith.** You earn the right to extract services by proving you have distinct bounded contexts, independent scaling needs, and the operational maturity to run them.

This guide defines when to start with a monolith, when microservices are warranted from day one, and how to migrate safely when the time comes.

> **Related reading:** [System Architecture Blueprint](./01-system-architecture.md) for the target-state architecture and [Service Decomposition Criteria](./07-service-decomposition.md) for split/merge signals.

---

## 📐 2. When to Start with a Monolith

A monolith is the right starting point when most of the following are true:

| Condition | Why it matters |
|-----------|----------------|
| **Team size < 20 engineers** | A single team can own the entire codebase without coordination overhead |
| **Domain boundaries are unclear** | You have not yet discovered where the natural seams are - splitting prematurely creates wrong boundaries |
| **Speed to market is critical** | Monoliths have zero network overhead, simpler debugging, and faster iteration cycles |
| **Shared data model** | Most features read and write the same core entities |
| **Limited operational maturity** | You lack the CI/CD, observability, and on-call infrastructure that microservices demand |
| **Single deployment target** | One artifact, one pipeline, one rollback - dramatically simpler operations |

### 2.1 The Cost of Premature Decomposition

Splitting too early introduces problems that are harder to fix than a large monolith:

- **Wrong service boundaries** lock you into a distributed architecture around the wrong seams. Merging services back together is painful.
- **Distributed debugging** requires tracing, correlation IDs, and centralized logging before you can even reproduce a bug.
- **Data consistency** shifts from a database transaction to sagas and eventual consistency - unnecessary complexity when one database would suffice.
- **Operational overhead** multiplies: each service needs its own CI pipeline, deployment config, monitoring dashboards, and on-call runbook.

---

## 🚀 3. When to Start with Microservices

Skip the monolith and go distributed from day one only when **all** of the following hold:

| Condition | Example |
|-----------|---------|
| **Proven domain model** | You are rebuilding a system whose bounded contexts are well understood from years of production experience |
| **Multiple independent teams** | Three or more teams need to ship independently on day one, and ownership boundaries are clear |
| **Independent scaling needs** | One component handles 10x the traffic of others and must scale separately |
| **Polyglot requirements** | Different components genuinely need different runtimes (e.g., ML model serving in Python alongside a Java API) |
| **Regulatory isolation** | Compliance mandates strict process or data isolation between components (e.g., PCI scope separation) |

If fewer than three of these conditions are true, start with a monolith.

---

## 🧭 4. Decision Framework

Use this matrix to make the call. Score each criterion, then follow the recommendation for your total.

| # | Criterion | Monolith (0 pts) | Microservices (1 pt) |
|---|-----------|------------------|----------------------|
| 1 | Team size | < 20 engineers | 20+ across 3+ teams |
| 2 | Domain clarity | Exploring / unclear | Well-understood boundaries |
| 3 | Scaling profile | Uniform load | Components differ by > 5x |
| 4 | Deployment cadence | Single cadence is fine | Teams blocked by coupled releases |
| 5 | Operational maturity | Building foundations | CI/CD, observability, on-call mature |
| 6 | Compliance isolation | Shared compliance scope | Mandatory process separation |

| Score | Recommendation |
|-------|----------------|
| 0 - 2 | **Monolith.** Focus on modular internals and ship fast. |
| 3 - 4 | **Modular monolith.** Enforce strict module boundaries. Plan extraction within 6 - 12 months. |
| 5 - 6 | **Microservices.** You have the domain knowledge and operational maturity to justify the complexity. |

---

## 🧱 5. The Modular Monolith Pattern

A modular monolith gives you the deployment simplicity of a single artifact with the architectural clarity of well-defined boundaries. It is the recommended intermediate step before microservices.

### 5.1 Core Principles

1. **One deployable, many modules.** The application is a single binary or container, but internally organized into isolated modules aligned to bounded contexts.
2. **Module APIs only.** Modules communicate through explicit public interfaces (Java packages, Go packages, or internal API layers) - never by reaching into another module's internals.
3. **Separate data ownership.** Each module owns its database schema (separate schemas, not separate databases). No cross-module joins.
4. **Async where possible.** Use an in-process event bus (or a lightweight broker) for cross-module communication. This mirrors the event-driven patterns you will use after extraction.
5. **Independent testability.** Each module has its own test suite that runs without loading the full application.

### 5.2 Module Structure Example

```
src/
  orders/
    api/              # public module interface
    internal/          # private implementation
    persistence/       # schema owned by this module
    events/            # events this module publishes
  payments/
    api/
    internal/
    persistence/
    events/
  shared-kernel/      # truly shared types (keep minimal)
```

### 5.3 Boundary Rules

| Rule | Enforcement |
|------|-------------|
| No cross-module imports of `internal/` packages | ArchUnit or build-tool module boundaries |
| No cross-module database access | Separate schemas, checked in CI |
| All cross-module calls go through `api/` layer | Code review and static analysis |
| Shared kernel contains only value objects and enums | Size limit enforced (< 200 types) |

---

## 🗺️ 6. Migration Path

When the signals in the [Service Decomposition Criteria](./07-service-decomposition.md) fire, follow this graduated migration path.

### 6.1 Migration Stages

**Visual overview:**

```mermaid
flowchart LR
    mono["Monolith"] --> modMono["Modular Monolith"]
    modMono --> strangler["Strangler Fig"]
    strangler --> extract["Extract Service"]
    extract --> validate["Validate in Prod"]
    validate --> retire["Retire Old Module"]
    retire --> micro["Microservices"]
```

### 6.2 Stage Details

| Stage | What happens | Duration |
|-------|-------------|----------|
| **1. Monolith** | Ship features fast. Discover real domain boundaries from production usage patterns. | Months 0 - 12+ |
| **2. Modular monolith** | Refactor internals into modules with strict boundaries (Section 5). No architecture change visible externally. | 2 - 4 sprints per module |
| **3. Strangler fig** | Route traffic for the target module through a thin API gateway. Both old and new paths serve traffic. | 1 - 2 sprints |
| **4. Extract service** | Deploy the module as an independent service. Swap in-process calls for network calls (REST/gRPC) and replace in-process events with Kafka topics. | 2 - 4 sprints |
| **5. Validate in production** | Run both paths in parallel. Compare responses and latency. Roll back if SLOs degrade. | 1 - 2 weeks |
| **6. Retire old module** | Remove the module code from the monolith. Update routing. Clean up shared-kernel references. | 1 sprint |

### 6.3 Extraction Checklist

Before extracting any module into a standalone service, confirm:

- [ ] The module has zero imports from other modules' `internal/` packages
- [ ] All cross-module communication uses the `api/` layer or events
- [ ] The module has its own database schema with no cross-module joins
- [ ] CI runs the module's tests in isolation
- [ ] Observability (metrics, traces, logs) is instrumented inside the module
- [ ] An on-call runbook exists for the future service
- [ ] The [Service Decomposition Criteria](./07-service-decomposition.md) split signals justify the extraction

---

## ⚠️ 7. Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| **Distributed monolith** | Services are split but still deploy together and share a database | Enforce data ownership per service; use events for cross-service data |
| **Premature extraction** | Splitting before domain boundaries stabilize | Wait for production traffic patterns to confirm boundaries |
| **Shared database** | Multiple services read/write the same tables | Assign table ownership to one service; others subscribe to events |
| **Big bang migration** | Rewriting the monolith from scratch as microservices | Use the strangler fig pattern to migrate incrementally |
| **Over-engineered monolith** | Adding service mesh, API gateways, or Kafka to a 3-person team | Keep the stack simple until complexity is earned |

---

## 📏 8. Success Metrics

Track these to know whether your architecture choice is working:

| Metric | Monolith target | Microservices target |
|--------|----------------|----------------------|
| **Lead time** (commit to production) | < 1 day | < 1 day per service |
| **Deployment frequency** | Daily | Multiple per day per service |
| **Mean time to recovery** | < 1 hour | < 30 minutes per service |
| **Cross-team PRs per sprint** | N/A (single team) | < 2 (ownership is clear) |
| **Build time** | < 10 minutes | < 5 minutes per service |
| **Onboarding time** (new engineer productive) | < 1 week | < 1 week within owned services |

If your monolith's lead time exceeds one day or your build time exceeds 15 minutes, those are early signals that modularization or extraction is due.

---

## 🔗 9. Related Documents

| Document | Relevance |
|----------|-----------|
| [System Architecture Blueprint](./01-system-architecture.md) | Target-state architecture that extracted services converge toward |
| [Service Decomposition Criteria](./07-service-decomposition.md) | When to split or merge - the signals that trigger migration |
| [Hexagonal Architecture](./03-hexagonal-architecture.md) | Internal module structure follows hexagonal principles |
| [Saga Patterns](./06-saga-patterns.md) | Required reading before introducing distributed transactions post-extraction |

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
