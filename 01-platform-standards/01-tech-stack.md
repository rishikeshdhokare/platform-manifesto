# ⚙️ Tech Stack & Standards

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Overview

This document defines the approved technology stack for all new and replatformed services. Deviation from this stack requires an ADR and explicit approval from the Platform team. The goal is not uniformity for its own sake - it is to reduce cognitive overhead, enable rotation of engineers across teams, and concentrate platform investment in a small number of well-understood tools.

> **Note on technology choices.** The specific tools named in this document (languages, frameworks, managed services) are {Company}'s **reference implementation** of the principles below. Organizations adopting this manifesto should substitute their own approved stack while preserving the underlying principles: one primary backend language, one framework with full platform support, managed data stores with no shared databases, and infrastructure as code for everything.

---

## ☕ 2. Backend

**Principle:** Choose one primary backend language with full platform support (golden path, dependency management, CI/CD templates, Backstage scaffolding). Additional languages are permitted for specific use cases with an ADR.

### 2.1 Language (Reference Implementation)

| Decision | Rationale |
|----------|-----------|
| **Java 21 (LTS)** - primary language | Virtual threads (Project Loom), strong ecosystem, excellent cloud SDK support, wide hiring pool |
| **Java 17** - minimum for existing services not yet on 21 | Still LTS, security-maintained |

> **Substitution point:** Your primary backend language could be TypeScript (Node.js), Go, C#, Python, or Kotlin - choose one with long-term support, a mature ecosystem, and a strong hiring pipeline in your market.

### 2.2 Build Tool

**Principle:** All builds must be **reproducible** - no floating dependency versions. Use dependency locking.

- **Gradle (Kotlin DSL)** - mandatory for all new Java services
- Maven is acceptable for existing services; new services must use Gradle
- Dependency locking via `gradle.lockfile` is required in all services

> **Substitution point:** npm/pnpm with lockfiles, Go modules, NuGet with lock files, Poetry/uv with `pyproject.toml` - the principle is reproducible builds, not the specific tool.

### 2.3 Framework

**Principle:** Standardize on one backend framework with full platform investment. Avoid mixing frameworks within the same runtime.

| Tier | Framework | Rationale |
|------|-----------|-----------|
| **REST APIs / HTTP services** | Spring Boot 3.x | Mature, well-supported, integrates with cloud SDKs and observability tooling |
| **Async / event-driven workers** | Spring Boot 3.x + Spring Kafka | Consistent programming model; avoid mixing frameworks per service |
| **Batch jobs** | Spring Batch | Only for ETL-style workloads; most scheduled tasks should be event-driven |
| **Lightweight lambdas** | Cloud Functions (e.g., AWS Lambda) | For genuine event-triggered, short-lived compute; not a default |

> **Substitution point:** NestJS/Fastify, ASP.NET, Go stdlib + Chi/Echo, Django/FastAPI, Ktor - pick one framework per runtime that covers HTTP, async workers, and batch.

### 2.4 Runtime

**Principle:** Pin your runtime version in container images and CI. Use a vendor-supported distribution. Never use `latest` tags.

- **JDK:** Amazon Corretto 21 - used in all container base images and CI
- **Container base image:** `amazoncorretto:21-alpine` for production; never `latest`
- All services run as **non-root** inside containers
- Runtime flags are standardized via a shared options template (see Golden Path)

> **Substitution point:** `node:20-alpine`, `mcr.microsoft.com/dotnet/aspnet:8.0`, `golang:1.22-alpine`, `python:3.12-slim` - same principle, different runtime.

---

## 🌐 3. Frontend (Web)

### 3.1 Language & Framework

| Decision | Rationale |
|----------|-----------|
| **TypeScript** - mandatory for all web frontend | Type safety, better tooling, fewer runtime errors |
| **React 18+** | Component model, ecosystem maturity, hiring pool |
| **Next.js** - recommended for new web apps | SSR/SSG, routing, API routes, image optimization |

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
| Server state | **TanStack Query** | Caching, invalidation, optimistic updates |
| Client state | **Zustand** or **React Context** | Keep it minimal; most state is server state |
| Forms | **React Hook Form + Zod** | Validation at the edge |
| API client | **OpenAPI-generated client** | Auto-generated from the BFF's OpenAPI spec |

### 3.4 Standards

- All web apps are SPA or SSR - no server-rendered templates (JSP, Thymeleaf, etc.)
- CSS: Tailwind CSS or CSS Modules - no CSS-in-JS runtime solutions in production
- Bundle size budget: < 200 KB initial JS gzipped. Monitored in CI
- Core Web Vitals (LCP < 2.5s, INP < 200ms, CLS < 0.1) monitored and alerted
- Accessibility: WCAG 2.1 AA compliance - tested with axe-core in CI

---

## 📱 4. Mobile

### 4.1 Language & Framework

| Decision | Rationale |
|----------|-----------|
| **React Native** - primary framework | Cross-platform, shared codebase, JS/TS ecosystem |
| **TypeScript** - mandatory | Same benefits as web; consistent language across frontend |
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

BFFs are written in the primary backend language and framework, owned by the respective product team, and follow the same backend standards as all other services.

### 4.5 Release Process

| Channel | Purpose | Approval |
|---------|---------|----------|
| **Internal** | Team testing | Automatic on merge to main |
| **Beta** (TestFlight / Google Play Internal) | Wider internal + beta testers | Manual promotion |
| **Staged rollout** (1% → 10% → 50% → 100%) | Gradual production release | Manual gates with crash-rate monitoring |
| **Hotfix** | Critical bug fix | Expedited review, skip staged rollout |

---

## 🗣️ 5. Multi-Language Strategy

**Principle:** One primary language receives full platform support. Secondary languages are permitted for specific use cases with an ADR and acceptance of reduced platform support.

The primary language receives full platform support (golden path, dependency management, Backstage templates, CI/CD templates). However, certain workloads are better served by other languages:

| Language | Approved For | Platform Support Level | Requires ADR? |
|----------|-------------|----------------------|---------------|
| **Primary language** | All domain services, BFFs, event consumers | Full (golden path, dependency management, templates) | No |
| **ML/Data language** (e.g., Python) | ML model training, data engineering, notebooks | Partial (CI template, no shared dependency management) | Yes |
| **Systems language** (e.g., Go, Rust) | High-performance edge proxies, CLI tooling, infrastructure utilities | Minimal (CI template only) | Yes |
| **Frontend language** (e.g., TypeScript) | Frontend apps, mobile, BFF prototyping | Partial (CI template, Backstage template for frontend) | Yes (for BFF use) |

### Decision Criteria

A secondary language is justified when:

1. **Ecosystem gap** - the required library ecosystem does not exist in the primary language (e.g., ML training with PyTorch, TensorFlow)
2. **Performance proof** - benchmarks demonstrate that the primary language cannot meet the latency or throughput requirement
3. **Team expertise** - the team has deep expertise in the alternative language AND the use case is isolated (no shared library dependencies)

### What "Partial Support" Means

- CI templates exist but are maintained on a best-effort basis
- No platform-wide dependency management - teams manage their own dependency versions
- No Backstage scaffolding template (except frontend)
- Observability standards (structured logging, metrics, distributed tracing) still apply - teams must implement them

### Anti-Patterns

- Starting a new domain service in a secondary language because "it would be faster" - without benchmarks
- Using a different language for a BFF because "the frontend team knows it" - without evaluating the operational cost of a second runtime in production
- Mixing languages within a single service

---

## 🗄️ 6. Data Stores

**Principle:** Each service owns its data store. Cross-service database access is forbidden. Use managed services over self-hosted databases. Automate all schema changes.

### 6.1 Relational Database

| Decision | Detail |
|----------|--------|
| **Engine** | PostgreSQL 15+ |
| **Managed service** | Amazon RDS (Multi-AZ) for standard workloads; Amazon Aurora PostgreSQL for high-throughput domains |
| **Migrations** | Flyway - mandatory. No manual schema changes in any environment |
| **Connection pooling** | PgBouncer sidecar, or RDS Proxy for serverless workloads |

> **Substitution point:** Cloud SQL (GCP), Azure Database for PostgreSQL, PlanetScale (MySQL), or CockroachDB. The principle is managed, Multi-AZ, with automated migrations.

### 6.2 Caching

| Decision | Detail |
|----------|--------|
| **Engine** | Redis 7+ |
| **Managed service** | Amazon ElastiCache for Redis (cluster mode enabled for production) |
| **Use cases** | Session state, rate limiting, geospatial indexes, hot read caches |
| **Prohibited use** | Redis must not be used as a primary data store - data there must be reconstructible |

> **Substitution point:** Memorystore (GCP), Azure Cache for Redis, Valkey, or KeyDB. The principle is a managed distributed cache that is reconstructible from the primary store.

### 6.3 In-Process Caching

**Principle:** In-process caches are for read-only reference data with short TTLs. Never cache mutable user state or shared counters in process memory.

| Decision | Detail |
|----------|--------|
| **Library** | Use your runtime's standard caching library (e.g., Caffeine for JVM, node-cache for Node.js, lru-cache) |
| **TTL** | < 5 minutes maximum |
| **Max entries** | 10,000 per cache instance |
| **Use cases** | Configuration lookups, reference/master data, feature flag snapshots |

| Scenario | Allowed? | Rationale |
|----------|----------|-----------|
| Config / reference data (e.g., pricing rules, region definitions) | Yes | Low-frequency change, small dataset, tolerance for brief staleness |
| Feature flag snapshots (short TTL) | Yes | Reduces calls to flag provider; TTL of 30 seconds or less |
| User profile or session data | Forbidden | Stale user data causes authorization and display bugs across replicas |
| Shared mutable state (e.g., rate-limit counters) | Forbidden | In-process caches are per-pod; use distributed cache for shared state |
| Data that must be immediately consistent after writes | Forbidden | TTL-based eviction cannot guarantee instant consistency |

### 6.4 Event Streaming

**Principle:** Use a managed event streaming platform for async cross-service communication. Enforce schema validation on all events.

| Decision | Detail |
|----------|--------|
| **Platform** | Apache Kafka (e.g., Amazon MSK, Confluent Cloud, self-managed) |
| **Client library** | Use your framework's idiomatic Kafka integration |
| **Schema format** | Avro with Schema Registry |
| **Topic naming** | `{domain}.{entity}.{event}` e.g. `orders.order.completed` |

> **Substitution point:** Kafka, Pulsar, NATS JetStream, or cloud-native alternatives (Amazon Kinesis, Google Pub/Sub, Azure Event Hubs). The principle is durable, ordered, schema-validated event streaming.

### 6.5 Search

- Use a managed search engine (e.g., OpenSearch, Elasticsearch, Typesense) for full-text and geo search workloads
- Not a general-purpose store; data in the search engine must be projected from a canonical source

### 6.6 Object Storage

- Use managed object storage (e.g., Amazon S3, Google Cloud Storage, Azure Blob Storage) for all binary/blob storage
- Versioning enabled on all production buckets
- No public buckets; all access via pre-signed URLs or CDN

---

## ☁️ 7. Cloud Platform

**Principle:** Standardize on one cloud provider. Use managed services over self-hosted. Define all infrastructure as code - no ClickOps.

### 7.1 Provider (Reference Implementation: AWS)

| Service Category | Reference Choice | Equivalent (GCP / Azure) |
|-----------------|-------------|--------------------------|
| Compute | Amazon EKS (Kubernetes) | GKE / AKS |
| Serverless compute | AWS Lambda | Cloud Functions / Azure Functions |
| Container registry | Amazon ECR | Artifact Registry / ACR |
| Networking | Amazon VPC, ALB, NLB | VPC, Cloud Load Balancing / VNet, Azure LB |
| DNS | Amazon Route 53 | Cloud DNS / Azure DNS |
| CDN | Amazon CloudFront | Cloud CDN / Azure Front Door |
| Secrets | AWS Secrets Manager | Secret Manager / Azure Key Vault |
| Config | AWS Systems Manager Parameter Store | Runtime Configurator / App Configuration |
| Certificates | AWS Certificate Manager (ACM) | Certificate Authority Service / App Service Certificates |
| IAM | AWS IAM + IRSA | Workload Identity / Workload Identity Federation |
| Observability | CloudWatch + X-Ray (supplemented by Grafana stack) | Cloud Monitoring + Cloud Trace / Azure Monitor |
| Cost management | AWS Cost Explorer + tagging enforcement | Billing / Cost Management |

### 7.2 Infrastructure as Code

**Principle:** All infrastructure must be defined in code and version-controlled. No manual changes in any environment.

- **Terraform** - mandatory for all infrastructure
- **Version:** pinned via `.terraform-version` (tfenv)
- **Module strategy:** Internal module registry in GitHub; no direct use of community modules without Platform team review
- **State backend:** Remote state with locking (e.g., S3 + DynamoDB, GCS, Azure Blob)
- **No ClickOps:** Any infrastructure not defined in code is out of compliance

> **Substitution point:** Pulumi, CDK, Crossplane, or OpenTofu. The principle is declarative, version-controlled, peer-reviewed infrastructure.

---

## 📨 8. Messaging & Integration

**Principle:** Use async events for cross-service communication by default. Reserve synchronous calls for latency-sensitive paths. Define all scheduled tasks in infrastructure as code.

| Pattern | Reference Tool | When to Use |
|---------|------|-------------|
| Async domain events | Kafka | Cross-service communication where eventual consistency is acceptable |
| Internal synchronous calls | gRPC (service-to-service) | Latency-sensitive internal paths |
| External APIs | REST over HTTPS | Public-facing and partner APIs |
| Scheduled tasks | Cloud scheduler (e.g., EventBridge, Cloud Scheduler) | Replaces cron jobs; all schedules in IaC |
| Webhooks inbound | API Gateway + serverless function | Ingestion from third parties (payment providers, etc.) |

---

## 👁️ 9. Observability Toolchain

**Principle:** Every service emits structured logs, metrics, and traces from day one. Use OpenTelemetry as the vendor-neutral collection layer. Centralize dashboards and alerting.

| Concern | Reference Tool | Alternatives |
|---------|------|-------------|
| Metrics | Prometheus + Grafana | Datadog, New Relic, Cloud Monitoring |
| Logs | Fluent Bit to centralized log store | ELK, Loki, Datadog Logs, Cloud Logging |
| Traces | OpenTelemetry SDK to trace backend | Jaeger, Zipkin, Datadog APM, Cloud Trace |
| Alerting | Grafana Alerting to PagerDuty | OpsGenie, VictorOps, Datadog Monitors |
| Uptime / synthetic | Synthetic monitoring | Checkly, Datadog Synthetics, cloud-native synthetics |
| Dashboards | Grafana (team dashboards) | Datadog, New Relic, cloud-native dashboards |

---

## 🧰 10. Developer Toolchain

**Principle:** Standardize the CI/CD pipeline, developer portal, static analysis, vulnerability scanning, and feature flag system. Every engineer uses the same workflow.

| Capability | Reference Tool | Alternatives | Mandatory? |
|------------|------|-------------|-----------|
| Source control + code review | GitHub | GitLab, Bitbucket | Yes |
| CI/CD pipelines | GitHub Actions | GitLab CI, CircleCI, Jenkins | Yes |
| GitOps / continuous deployment | ArgoCD | Flux, Spinnaker | Yes |
| Developer portal / service catalog | Backstage | Port, Cortex, custom | Yes |
| Static analysis, code quality | SonarCloud | SonarQube, CodeClimate | Yes |
| Vulnerability scanning | Snyk | Dependabot, Trivy, Grype | Yes |
| Feature flags | LaunchDarkly | Unleash, Flagsmith, Split | Yes |
| Local cloud emulation | LocalStack (for AWS) | Cloud-specific emulators, docker-compose | Recommended |
| Integration test infrastructure | Testcontainers | docker-compose, in-memory fakes | Yes (for DB/messaging tests) |

---

## 📋 11. Reference Implementation Summary

> The table below shows {Company}'s current stack choices. Organizations adopting this manifesto should substitute their own approved tools while keeping the same categories covered.

| Category | Reference Choice | Principle |
|----------|------------|----------------|
| Primary backend language | Java 21 | One language with full platform support |
| Backend framework | Spring Boot 3.x | One framework covering HTTP, async, and batch |
| Relational database | PostgreSQL / Aurora | Managed, Multi-AZ, automated migrations |
| Distributed cache | Redis (ElastiCache) | Managed cache, reconstructible data only |
| Event streaming | Kafka (MSK) | Durable, ordered, schema-validated events |
| Infrastructure as code | Terraform | Declarative, version-controlled, peer-reviewed |
| CI/CD | GitHub Actions + ArgoCD | Automated pipelines with quality gates + GitOps |
| Secrets management | AWS Secrets Manager | Centralized, audited, rotated - never in repos |
| Feature flags | LaunchDarkly | Managed flag system with targeting and audit trail |

---

## 🔒 12. Web Authentication Patterns

### 12.1 SPA Authentication (PKCE)

Single-page applications use **Authorization Code with PKCE**:

| Aspect | Standard |
|--------|----------|
| **Flow** | Authorization Code + PKCE (RFC 7636) |
| **Token storage** | In-memory only - never `localStorage` or `sessionStorage` |
| **Token lifetime** | Access token: 15 minutes; refresh token: rotating, httpOnly cookie |
| **Silent refresh** | Hidden iframe or service worker - no full-page redirect |

Tokens stored in `localStorage` are accessible to any JavaScript running on the page (XSS risk). In-memory storage is lost on page refresh - the silent refresh mechanism re-obtains tokens without user interaction.

### 12.2 SSR Authentication (BFF-Managed Session)

Server-side rendered applications use **httpOnly cookies** managed by the BFF:

| Aspect | Standard |
|--------|----------|
| **Session storage** | httpOnly, Secure, SameSite=Strict cookie |
| **Session backend** | BFF manages the session; tokens never reach the browser |
| **CSRF protection** | Double-submit cookie pattern or synchronizer token |

The BFF acts as a confidential client - it holds the client secret and exchanges tokens with the identity provider on behalf of the browser.

### 12.3 Silent Refresh Mechanisms

| Mechanism | When to Use |
|-----------|-------------|
| **Hidden iframe** | Default for SPAs where the IdP supports `prompt=none` |
| **Service worker** | When iframe-based refresh is blocked by third-party cookie policies |
| **Refresh token rotation** | Always enabled - each refresh token is single-use; a reused token revokes the entire session |

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
