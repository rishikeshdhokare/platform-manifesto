# ⚙️ Tech Stack & Standards

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 🎯 1. Overview

This document defines the approved technology stack for all new and replatformed services. Deviation from this stack requires an ADR and explicit approval from the Platform team. The goal is not uniformity for its own sake — it is to reduce cognitive overhead, enable rotation of engineers across teams, and concentrate platform investment in a small number of well-understood tools.

---

## ☕ 2. Backend

### 2.1 Language

| Decision | Rationale |
|----------|-----------|
| **Java 21 (LTS)** — primary language | Virtual threads (Project Loom), strong ecosystem, excellent AWS SDK support, wide hiring pool |
| **Java 17** — minimum for existing services not yet on 21 | Still LTS, security-maintained |

**We do not accept new services in:** Scala, Groovy, Kotlin (unless a team has an existing Kotlin codebase — it may remain, but new services start in Java).

### 2.2 Build Tool

- **Gradle (Kotlin DSL)** — mandatory for all new Java services
- Maven is acceptable for existing services; new services must use Gradle
- All builds must be **reproducible** — no floating dependency versions (use `implementation("group:artifact:1.2.3")`, not `1.+`)
- Dependency locking via `gradle.lockfile` is required in all services

### 2.3 Framework

| Tier | Framework | Rationale |
|------|-----------|-----------|
| **REST APIs / HTTP services** | Spring Boot 3.x | Mature, well-supported, integrates with all AWS SDKs and observability tooling |
| **Async / event-driven workers** | Spring Boot 3.x + Spring Kafka | Consistent programming model; avoid mixing frameworks per service |
| **Batch jobs** | Spring Batch | Only for ETL-style workloads; most scheduled tasks should be event-driven |
| **Lightweight lambdas** | AWS Lambda + Spring Cloud Function | For genuine event-triggered, short-lived compute; not a default |

**We do not use:** Quarkus, Micronaut, Vert.x (these are not prohibited but will receive no platform support).

### 2.4 Runtime

- **JDK:** Amazon Corretto 21 — used in all container base images and CI
- **Container base image:** `amazoncorretto:21-alpine` for production; never `latest`
- All services run as **non-root** inside containers
- JVM flags are standardised via a shared `jvm.options` template (see Golden Path)

---

## 🌐 3. Frontend (Web)

### 3.1 Language & Framework

| Decision | Rationale |
|----------|-----------|
| **TypeScript** — mandatory for all web frontend | Type safety, better tooling, fewer runtime errors |
| **React 18+** | Component model, ecosystem maturity, hiring pool |
| **Next.js** — recommended for new web apps | SSR/SSG, routing, API routes, image optimization |

### 3.2 Build & Tooling

| Tool | Purpose | Mandatory? |
|------|---------|-----------|
| **Vite** or **Next.js built-in** | Build tool / dev server | Yes (one of) |
| **ESLint + Prettier** | Code quality and formatting | Yes |
| **Jest + React Testing Library** | Unit and component testing | Yes |
| **Playwright** | E2E browser testing | Yes |
| **Storybook** | Component development and documentation | Recommended |

### 3.3 State Management & Data Fetching

| Concern | Tool | Notes |
|---------|------|-------|
| Server state | **TanStack Query (React Query)** | Caching, invalidation, optimistic updates |
| Client state | **Zustand** or **React Context** | Keep it minimal; most state is server state |
| Forms | **React Hook Form + Zod** | Validation at the edge |
| API client | **OpenAPI-generated client** | Auto-generated from the BFF's OpenAPI spec |

### 3.4 Standards

- All web apps are SPA or SSR — no server-rendered templates (JSP, Thymeleaf, etc.)
- CSS: Tailwind CSS or CSS Modules — no CSS-in-JS runtime solutions in production
- Bundle size budget: < 200KB initial JS (gzipped). Monitored in CI
- Core Web Vitals (LCP < 2.5s, FID < 100ms, CLS < 0.1) monitored and alerted
- Accessibility: WCAG 2.1 AA compliance — tested with axe-core in CI

---

## 📱 4. Mobile

### 4.1 Language & Framework

| Decision | Rationale |
|----------|-----------|
| **React Native** — primary framework | Cross-platform, shared codebase, JS/TS ecosystem |
| **TypeScript** — mandatory | Same benefits as web; consistent language across frontend |
| **Native modules** | Allowed when RN cannot meet performance or platform API requirements (camera, Bluetooth, etc.) |

### 4.2 Architecture

| Concern | Approach |
|---------|----------|
| Navigation | React Navigation |
| State management | TanStack Query (server state) + Zustand (local state) |
| Offline support | react-native-quick-sqlite for structured data (default), AsyncStorage for simple key-value; WatermelonDB only with ADR |
| Push notifications | Firebase Cloud Messaging (FCM) + APNs |
| Deep linking | `{company}://` scheme + universal links |
| OTA updates | CodePush (for JS bundle updates without app store release) |

### 4.3 Performance Budgets

| Metric | Target | Monitoring |
|--------|--------|-----------|
| Cold start (app launch to interactive) | < 3 seconds | Firebase Performance |
| JS bundle size | < 5MB (Hermes bytecode) | CI check |
| Memory footprint | < 200MB steady state | Automated profiling in CI |
| Crash-free rate | > 99.5% | Firebase Crashlytics |

### 4.4 BFF Pattern

Each mobile app has a dedicated Backend-for-Frontend (BFF) service:

| App | BFF | Owner |
|-----|-----|-------|
| Customer App | Customer BFF | Customers team |
| Provider App | Provider BFF | Providers team |
| Ops Portal | Ops BFF | Platform team |

BFFs are written in **Java (Spring Boot)**, owned by the respective product team, and follow the same backend standards as all other services.

### 4.5 Release Process

| Channel | Purpose | Approval |
|---------|---------|----------|
| **Internal** | Team testing | Automatic on merge to main |
| **Beta** (TestFlight / Google Play Internal) | Wider internal + beta testers | Manual promotion |
| **Staged rollout** (1% → 10% → 50% → 100%) | Gradual production release | Manual gates with crash-rate monitoring |
| **Hotfix** | Critical bug fix | Expedited review, skip staged rollout |

---

## ☕ 5. Language Strategy Beyond Java

Java is our primary language and receives full platform support (golden path, BOM, Backstage templates, CI/CD templates). However, certain workloads are better served by other languages:

| Language | Approved For | Platform Support Level | Requires ADR? |
|----------|-------------|----------------------|---------------|
| **Java 21** | All domain services, BFFs, event consumers | Full (golden path, BOM, templates) | No |
| **Python 3.11+** | ML model training, data engineering (Glue ETL), Jupyter notebooks, SageMaker pipelines | Partial (CI template, no BOM) | Yes |
| **Go** | High-performance edge proxies, CLI tooling, infrastructure utilities | Minimal (CI template only) | Yes |
| **TypeScript** | Frontend (React), mobile (React Native), BFF prototyping | Partial (CI template, Backstage template for frontend) | Yes (for BFF use) |

### Decision Criteria

A non-Java language is justified when:

1. **Ecosystem gap** — the required library ecosystem does not exist in Java (e.g., ML training with PyTorch, TensorFlow)
2. **Performance proof** — benchmarks demonstrate that Java cannot meet the latency or throughput requirement (rare with virtual threads)
3. **Team expertise** — the team has deep expertise in the alternative language AND the use case is isolated (no shared library dependencies)

### What "Partial Support" Means

- CI templates exist but are maintained on a best-effort basis
- No platform BOM — teams manage their own dependency versions
- No Backstage scaffolding template (except TypeScript frontend)
- Observability standards (structured logging, Prometheus metrics, OTel tracing) still apply — teams must implement them

### Anti-Patterns

- Starting a new domain service in Python or Go because "it would be faster" — without benchmarks
- Using TypeScript for a BFF because "the frontend team knows it" — without evaluating the operational cost of a second runtime in production
- Mixing languages within a single service (e.g., Java service calling Python scripts)

---

## 🗄️ 6. Data Stores

### 6.1 Relational Database

| Decision | Detail |
|----------|--------|
| **Engine** | PostgreSQL 15+ |
| **Managed service** | Amazon RDS (Multi-AZ) for standard workloads; Amazon Aurora PostgreSQL for high-throughput domains (fulfillment, pricing) |
| **Migrations** | Flyway — mandatory. No manual schema changes in any environment |
| **Connection pooling** | PgBouncer sidecar, or RDS Proxy for serverless workloads |

### 6.2 Caching

| Decision | Detail |
|----------|--------|
| **Engine** | Redis 7+ |
| **Managed service** | Amazon ElastiCache for Redis (cluster mode enabled for production) |
| **Use cases** | Session state, rate limiting, geospatial indexes (provider location), hot read caches |
| **Prohibited use** | Redis must not be used as a primary data store — data there must be reconstructible |

### 6.3 In-Process Caching

| Decision | Detail |
|----------|--------|
| **Library** | Caffeine — for config and reference data only |
| **TTL** | < 5 minutes maximum |
| **Max entries** | 10,000 per cache instance |
| **Use cases** | Configuration lookups, reference/master data, feature flag snapshots |

| Scenario | Allowed? | Rationale |
|----------|----------|-----------|
| Config / reference data (e.g., pricing rules, region definitions) | ✅ Yes | Low-frequency change, small dataset, tolerance for brief staleness |
| Feature flag snapshots (short TTL) | ✅ Yes | Reduces calls to LaunchDarkly; TTL ≤ 30 seconds |
| User profile or session data | ❌ Forbidden | Stale user data causes authorization and display bugs across replicas |
| Shared mutable state (e.g., rate-limit counters) | ❌ Forbidden | In-process caches are per-pod; use Redis for shared state |
| Data that must be immediately consistent after writes | ❌ Forbidden | TTL-based eviction cannot guarantee instant consistency |

### 6.4 Event Streaming

| Decision | Detail |
|----------|--------|
| **Platform** | Amazon MSK (Managed Streaming for Apache Kafka) |
| **Client library** | `spring-kafka` — no direct Kafka client usage |
| **Schema format** | Avro with Schema Registry (AWS Glue Schema Registry) |
| **Topic naming** | `{domain}.{entity}.{event}` e.g. `orders.order.completed` |

### 6.5 Search

- **Amazon OpenSearch Service** — for full-text and geo search workloads
- Not a general-purpose store; data in OpenSearch must be projected from a canonical source

### 6.6 Object Storage

- **Amazon S3** — for all binary/blob storage
- Versioning enabled on all production buckets
- No public buckets; all access via pre-signed URLs or CloudFront

---

## ☁️ 7. Cloud Platform

### 7.1 Provider

**AWS — primary and exclusive cloud provider.**

| Service Category | AWS Service |
|-----------------|-------------|
| Compute | Amazon EKS (Kubernetes) |
| Serverless compute | AWS Lambda |
| Container registry | Amazon ECR |
| Networking | Amazon VPC, ALB, NLB |
| DNS | Amazon Route 53 |
| CDN | Amazon CloudFront |
| Secrets | AWS Secrets Manager |
| Config | AWS Systems Manager Parameter Store |
| Certificates | AWS Certificate Manager (ACM) |
| IAM | AWS IAM + IRSA (IAM Roles for Service Accounts) |
| Observability | Amazon CloudWatch + AWS X-Ray (supplemented by Grafana stack) |
| Cost management | AWS Cost Explorer + tagging enforcement |

### 7.2 Infrastructure as Code

- **Terraform** — mandatory for all AWS infrastructure
- **Terraform version:** pinned via `.terraform-version` (tfenv)
- **Module strategy:** Internal module registry in GitHub; no direct use of community modules without Platform team review
- **State backend:** S3 + DynamoDB locking, per environment
- **No ClickOps:** Any infrastructure not defined in Terraform is out of compliance

---

## 📨 8. Messaging & Integration

| Pattern | Tool | When to Use |
|---------|------|-------------|
| Async domain events | Kafka (MSK) | Cross-service communication where eventual consistency is acceptable |
| Internal synchronous calls | gRPC (service-to-service) | Latency-sensitive internal paths (fulfillment engine, pricing) |
| External APIs | REST over HTTPS | Public-facing and partner APIs |
| Scheduled tasks | Amazon EventBridge Scheduler | Replaces cron jobs; all schedules in IaC |
| Webhooks inbound | Amazon API Gateway + Lambda | Ingestion from third parties (payment providers, geolocation) |

---

## 👁️ 9. Observability Toolchain

| Concern | Tool |
|---------|------|
| Metrics | Prometheus + Grafana (running on EKS) |
| Logs | Fluent Bit → Amazon OpenSearch (or CloudWatch Logs for Lambda) |
| Traces | OpenTelemetry SDK → AWS X-Ray |
| Alerting | Grafana Alerting → PagerDuty |
| Uptime / synthetic | Amazon CloudWatch Synthetics |
| Dashboards | Grafana (team dashboards) + CloudWatch (infra dashboards) |

---

## 🧰 10. Developer Toolchain

| Tool | Purpose | Mandatory? |
|------|---------|-----------|
| GitHub | Source control, CI/CD, code review | Yes |
| GitHub Actions | CI/CD pipelines | Yes |
| ArgoCD | GitOps / continuous deployment to EKS | Yes |
| Backstage | Internal developer portal / service catalog | Yes |
| SonarCloud | Static analysis, code quality | Yes |
| Snyk | Dependency and container vulnerability scanning | Yes |
| LaunchDarkly | Feature flags | Yes |
| LocalStack | Local AWS emulation for development | Recommended |
| Testcontainers | Integration test infrastructure | Yes (for DB/Kafka tests) |

---

## 📋 11. Approved vs Unapproved — Quick Reference

| Category | ✅ Approved | ❌ Not Approved |
|----------|------------|----------------|
| Language | Java 21 | Scala, Ruby, PHP, C++ |
| Framework | Spring Boot 3.x | Quarkus, Micronaut (no platform support) |
| DB (relational) | PostgreSQL / Aurora | MySQL, Oracle, MSSQL |
| DB (document) | — (use RDS) | MongoDB (not on approved list) |
| Messaging | Kafka (MSK) | RabbitMQ, SQS for domain events |
| IaC | Terraform | CDK, CloudFormation directly |
| CI/CD | GitHub Actions + ArgoCD | Jenkins, CircleCI, Bitbucket Pipelines |
| Secrets | AWS Secrets Manager | Hardcoded env vars, `.env` files in repos |
| Feature flags | LaunchDarkly | Custom flag systems, env var toggles |

---

## 🔒 12. Web Authentication Patterns

### 12.1 SPA Authentication (PKCE)

Single-page applications use **Authorization Code with PKCE**:

| Aspect | Standard |
|--------|----------|
| **Flow** | Authorization Code + PKCE (RFC 7636) |
| **Token storage** | In-memory only — never `localStorage` or `sessionStorage` |
| **Token lifetime** | Access token: 15 minutes; refresh token: rotating, httpOnly cookie |
| **Silent refresh** | Hidden iframe or service worker — no full-page redirect |

Tokens stored in `localStorage` are accessible to any JavaScript running on the page (XSS risk). In-memory storage is lost on page refresh — the silent refresh mechanism re-obtains tokens without user interaction.

### 12.2 SSR Authentication (BFF-Managed Session)

Server-side rendered applications use **httpOnly cookies** managed by the BFF:

| Aspect | Standard |
|--------|----------|
| **Session storage** | httpOnly, Secure, SameSite=Strict cookie |
| **Session backend** | BFF manages the session; tokens never reach the browser |
| **CSRF protection** | Double-submit cookie pattern or synchronizer token |

The BFF acts as a confidential client — it holds the client secret and exchanges tokens with the identity provider on behalf of the browser.

### 12.3 Silent Refresh Mechanisms

| Mechanism | When to Use |
|-----------|-------------|
| **Hidden iframe** | Default for SPAs where the IdP supports `prompt=none` |
| **Service worker** | When iframe-based refresh is blocked by third-party cookie policies |
| **Refresh token rotation** | Always enabled — each refresh token is single-use; a reused token revokes the entire session |

---

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
