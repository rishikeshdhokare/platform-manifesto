# {Company} Platform Engineering Manifesto

### The operating system for how we build software.

> **One manifesto. 83 documents. Every decision you shouldn't have to make twice.**
>
> Opinionated by design. When in doubt, follow it. When you disagree, raise a PR.

---

## Why This Exists

Every hour an engineer spends debating "which database should I use?" or "how do I structure this service?" is an hour not spent solving customer problems. This manifesto exists to eliminate that tax.

It is the **single source of truth** for our engineering platform — the standards, practices, tools, and architecture every team is expected to adopt. Consistency compounds: every shared decision here is one fewer decision each team makes alone.

This is a **living document**. It evolves as our platform matures, as industry practices shift, and as we learn from running systems in production. All changes require a PR with at least one Staff Engineer approval.

---

## Who Should Read What

| You are... | Start here | Then read |
|------------|-----------|-----------|
| **Day 1 new joiner** | [`ONBOARDING.md`](./ONBOARDING.md) | Tech stack, Golden Path, Git workflow |
| **Backend engineer** | [Spring Boot Standards](03-engineering-practices/09-spring-boot-standards.md) | Architecture, Kafka, Caching, Testing |
| **Frontend engineer** | [Web Frontend Standards](09-mobile-and-frontend/02-web-frontend-standards.md) | Frontend CI/CD, Design System |
| **Mobile engineer** | [Mobile Standards](09-mobile-and-frontend/01-mobile-standards.md) | Android / iOS / React Native guides |
| **Tech lead** | [System Architecture](02-architecture-and-api/01-system-architecture.md) | Service Decomposition, Team Topology |
| **Engineering manager** | [Engineering Management](07-ways-of-working/09-engineering-management.md) | Engineering Ladder, Metrics, Product Ops |
| **On-call and panicking** | [Incident Management](05-operational-excellence/04-incident-management.md) | Debugging Guide, DR Playbook |

> Don't know a term? Check the [`GLOSSARY.md`](./GLOSSARY.md).

---

## The 8 Principles

These aren't aspirations. They're constraints. Every technical decision in this manifesto traces back to one of these.

| # | Principle | In Practice |
|---|-----------|-------------|
| 1 | **Pave the golden path, don't police the wilderness** | Make the right way the easy way. Teams feel pulled toward standards, not pushed. |
| 2 | **Ship artifacts, not code** | The same container image built in CI is the one running in production. Never rebuild. |
| 3 | **Observable by default** | Every service ships with logs, metrics, and traces from day one. Not a retrofit. |
| 4 | **Own your data, respect boundaries** | Services own their data stores. Direct cross-service DB access is forbidden. No exceptions. |
| 5 | **Everything in Git** | Infrastructure, config, runbooks, ADRs. If it's not in Git, it doesn't exist. |
| 6 | **Automate the boring, document the interesting** | Pipelines handle repetition. Humans handle architecture. |
| 7 | **Design for failure** | Assume every dependency will fail. Circuit breakers, retries, fallbacks, graceful degradation. |
| 8 | **Security is not a phase** | It runs in every pipeline, in every environment, from the first commit. |

---

## Platform at a Glance

```mermaid
flowchart TB
    subgraph clients [Client Layer]
        CustomerApp[Customer App]
        ProviderApp[Provider App]
        OpsPortal[Ops Portal]
        PartnerAPI[Partner API]
    end

    subgraph gateway [API Gateway Layer]
        APIGW[Amazon API Gateway + WAF]
    end

    subgraph bff [BFF Layer]
        CustomerBFF[Customer BFF]
        ProviderBFF[Provider BFF]
        OpsBFF[Ops BFF]
        PartnerBFF[Partner BFF]
    end

    subgraph core [Core Domain Services]
        Fulfillment[Fulfillment Engine]
        Pricing[Pricing Service]
        Orders[Order Service]
        ProviderProfile[Provider Profile]
        CustomerProfile[Customer Profile]
    end

    subgraph supporting [Supporting Services]
        Payments[Payment Service]
        Notifications[Notifications]
        Geolocation[Geolocation Service]
        DynamicPricing[Dynamic Pricing]
        Fraud[Fraud Engine]
    end

    subgraph platform [Platform Layer]
        Kafka[Kafka / MSK]
        Aurora[(Aurora PostgreSQL)]
        Redis[(ElastiCache Redis)]
        OpenSearch[(OpenSearch)]
        S3[(S3)]
    end

    subgraph infra [Infrastructure]
        EKS[Amazon EKS]
        Istio[Istio Service Mesh]
        ArgoCD[ArgoCD GitOps]
        Prometheus[Prometheus + Grafana]
    end

    clients --> gateway
    gateway --> bff
    bff --> core
    bff --> supporting
    core --> platform
    supporting --> platform
    core --> Kafka
    supporting --> Kafka
    EKS --> core
    EKS --> supporting
    EKS --> bff
    Istio --> EKS
```

---

## What's Inside

**83 documents** across 11 sections. Each section is a folder. Each document is a self-contained standard you can read in 10-15 minutes.

### [`01-platform-standards/`](01-platform-standards/) -- What We Build With
The approved tech stack. Languages, frameworks, cloud services, data stores. If it's not here, it's not approved.

| File | What You'll Learn |
|------|-------------------|
| `01-tech-stack.md` | Java 21, Spring Boot 3, React, React Native, AWS services, and why we chose them |

### [`02-architecture-and-api/`](02-architecture-and-api/) -- How The System Fits Together
Domain decomposition, communication patterns, API contracts, event schemas, and the error catalog.

| File | What You'll Learn |
|------|-------------------|
| `01-system-architecture.md` | The domain map, event backbone, BFF pattern, and resilience ownership matrix |
| `02-api-standards.md` | URL design, versioning, error shapes, pagination, rate limiting, idempotency |
| `03-hexagonal-architecture.md` | Ports & adapters with a full worked example you can copy |
| `04-real-time-architecture.md` | WebSocket, SSE, push notifications, and location streaming |
| `05-grpc-standards.md` | Proto conventions, code generation, load balancing, cancellation |
| `06-saga-patterns.md` | Distributed transactions, choreography, compensation patterns |
| `07-service-decomposition.md` | When to split or merge services, with a decision framework |
| `08-event-schema-evolution.md` | Avro compatibility rules, partition keys, and the breaking change playbook |
| `09-error-catalog.md` | Central error registry, exception handling, and frontend error boundaries |

### [`03-engineering-practices/`](03-engineering-practices/) -- How We Write and Ship Code
The day-to-day craft. Testing, CI/CD, code review, coding standards, and the Spring Boot platform.

| File | What You'll Learn |
|------|-------------------|
| `01-testing-pyramid.md` | Unit, integration, contract, E2E, load tests -- with worked examples |
| `02-ci-practices.md` | GitHub Actions pipelines, quality gates, DAST, Terraform testing |
| `03-cd-practices.md` | GitOps, canary deployments, feature flags, change risk rubric |
| `04-coding-standards.md` | Naming, error handling, null safety -- with before/after examples |
| `05-git-workflow.md` | Trunk-based dev in practice |
| `06-code-review-guide.md` | How to give and receive feedback that makes code better |
| `07-ab-testing.md` | Experiment design, statistical rigor, guardrails, and product analytics |
| `08-deprecation-lifecycle.md` | How we sunset APIs, events, services, and flags without breaking consumers |
| `09-spring-boot-standards.md` | Logging, config, health checks, LaunchDarkly, virtual threads, versioning |
| `10-frontend-ci-cd.md` | Frontend golden path, monorepo structure, preview environments |
| `11-qa-standards.md` | Test environments, test data, bug triage severity, device matrix |

### [`04-infrastructure-and-cloud/`](04-infrastructure-and-cloud/) -- The Platform Under The Platform
AWS architecture, security, FinOps, multi-tenancy, and everything Terraform.

| File | What You'll Learn |
|------|-------------------|
| `01-cloud-architecture.md` | AWS accounts, VPC topology, EKS, Aurora, MSK, backup & restore |
| `02-infra-components.md` | Istio, Backstage, ArgoCD, ECR -- and the self-service boundaries |
| `03-security.md` | Shift-left security, IAM/IRSA, PII handling, KMS, egress control |
| `04-configuration-management.md` | Secrets vs config vs flags -- what lives where |
| `05-finops.md` | Cost allocation, rightsizing, reservations, real-time cost visibility |
| `06-capacity-planning.md` | Demand modeling, pre-scaling, load test-driven validation |
| `07-api-gateway-strategy.md` | API Gateway configuration, routing, throttling, WAF |
| `08-privacy-engineering.md` | GDPR, PCI, SOC 2, ISO 27001, anonymization, consent |
| `09-multi-tenancy.md` | Data isolation patterns, noisy-neighbor controls, tenant-aware observability |
| `10-security-operations.md` | Threat modeling, SIEM, pen testing, SBOM, bug bounty, PAM |

### [`05-operational-excellence/`](05-operational-excellence/) -- Keeping It Running
Observability, resilience, incidents, chaos engineering, and what to do when things break.

| File | What You'll Learn |
|------|-------------------|
| `01-observability-standards.md` | Logging, metrics, tracing, SLOs, error budgets, alert hygiene |
| `02-observability-in-practice.md` | Step-by-step: structured logging, correlation IDs, Prometheus, tracing |
| `03-resilience-patterns.md` | Circuit breaker, retry, timeout, bulkhead -- Resilience4j worked examples |
| `04-incident-management.md` | Response playbook, on-call structure, PIR template, change management |
| `05-load-shedding.md` | Priority tiers, backpressure, graceful degradation, auto-remediation |
| `06-chaos-engineering.md` | Fault injection, game days, resilience validation |
| `07-disaster-recovery-playbook.md` | Region failover, failback, DNS strategy, rollback SLAs |
| `08-testing-in-production.md` | Synthetic monitoring, traffic mirroring, dark launches |
| `09-debugging-guide.md` | How to debug locally, in staging, and in production -- with real queries |

### [`06-developer-guides/`](06-developer-guides/) -- Hands-On Playbooks
The guides you keep open while writing code.

| File | What You'll Learn |
|------|-------------------|
| `01-developer-experience.md` | Local setup, onboarding milestones, buddy program, documentation standards |
| `02-golden-path.md` | Scaffold to production in one walkthrough |
| `03-database-migrations.md` | Flyway, expand-contract, large table migrations |
| `04-kafka-patterns.md` | Producers, consumers, DLQ, idempotency, topic creation, DLQ replay |
| `05-data-platform.md` | CDC, Redshift conventions, Airflow orchestration, data access controls |
| `06-multi-region-patterns.md` | Region config, i18n, localization, UX writing |
| `07-distributed-locking.md` | Optimistic locking, Redis locks, preventing double-assignment |
| `08-cache-patterns.md` | Cache-aside, TTL, event-driven invalidation, Caffeine, stampede prevention |
| `09-data-governance.md` | Data stewards, quality SLAs, retention matrices, analytics contracts |
| `10-local-development.md` | Docker Compose, database seeding, your first PR |

### [`07-ways-of-working/`](07-ways-of-working/) -- How We Work Together
Team structure, decision-making, hiring, career growth, and knowledge sharing.

| File | What You'll Learn |
|------|-------------------|
| `01-team-topology.md` | Stream-aligned teams, platform team, inner source, tech debt registry |
| `02-engineering-ladder.md` | Engineer I to Principal -- expectations at every level |
| `03-open-source-policy.md` | License compliance, contribution guidelines |
| `04-product-engineering.md` | PM-engineering collaboration, discovery, rituals, user stories |
| `05-rfc-process.md` | How to propose cross-cutting changes |
| `06-technology-radar.md` | How we evaluate and adopt new technology |
| `07-hiring-standards.md` | Interview loop, rubrics, bar-raiser, bias mitigation |
| `08-knowledge-sharing.md` | Guilds, tech talks, architecture clinics |
| `09-engineering-management.md` | 1:1s, reviews, team health, headcount, career development |
| `10-product-operations.md` | Roadmap, OKRs, launch management, customer feedback loops |

### [`08-program/`](08-program/) -- Where We Are and Where We're Going
Maturity assessment, migration roadmap, metrics, and vendor management.

| File | What You'll Learn |
|------|-------------------|
| `01-maturity-model.md` | Self-assessment rubric -- L0 to L4 across 11 dimensions |
| `02-migration-roadmap.md` | Phased migration program, DORA targets, risk register |
| `03-vendor-assessment.md` | AWS lock-in analysis, portability, exit costs |
| `04-engineering-metrics.md` | What we measure, what we don't, and why |
| `05-vendor-intake.md` | How to evaluate and onboard new SaaS tools |

### [`09-mobile-and-frontend/`](09-mobile-and-frontend/) -- Client Applications
Everything for the engineers building what users actually see and touch.

| File | What You'll Learn |
|------|-------------------|
| `01-mobile-standards.md` | Offline-first, push, performance budgets, auth lifecycle, cert pinning |
| `02-web-frontend-standards.md` | SPA/SSR, design system, a11y, Core Web Vitals, LaunchDarkly React |
| `03-react-native-guide.md` | Turbo Modules, navigation, Hermes, CodePush, versioning |
| `04-android-standards.md` | Gradle, Compose, Kotlin coroutines, Hilt, WorkManager |
| `05-ios-standards.md` | SwiftUI, SPM, privacy manifest, extensions, dSYM |
| `06-design-system.md` | Design-dev handoff, tokens, motion, dark mode, data visualization |

### [`10-ai-ml-platform/`](10-ai-ml-platform/) -- AI & Machine Learning
Model infrastructure, governance, and responsible AI.

| File | What You'll Learn |
|------|-------------------|
| `01-ml-platform.md` | Model serving, feature stores, training pipelines, GPU governance |
| `02-ai-governance.md` | Bias auditing, LLM guardrails, red-teaming, generative AI patterns |

### [`11-domain-catalog/`](11-domain-catalog/) -- The Bounded Contexts
Detailed documentation for every domain service -- APIs, events, data models, SLOs, and failure modes.

| File | What You'll Learn |
|------|-------------------|
| `01-order-service.md` | Order lifecycle -- the central orchestrating domain |
| `02-fulfillment-engine.md` | Provider-order assignment and geospatial algorithms |
| `03-pricing-service.md` | Price calculation, estimates, dynamic pricing integration |
| `04-provider-profile.md` | Provider onboarding, documents, ratings, availability |
| `05-customer-profile.md` | Customer account, preferences, payment methods |
| `06-payment-service.md` | Payments, settlements, wallets, refunds |
| `07-notifications.md` | Push, SMS, email dispatch across all channels |
| `08-geolocation-service.md` | ETA calculation, route planning, geocoding |
| `09-dynamic-pricing.md` | Supply/demand signals, zone management, multipliers |
| `10-fraud-engine.md` | Real-time risk scoring, blocking rules, ML fallbacks |

---

## Domain Map

```mermaid
flowchart TB
    subgraph core [Core Domains]
        Orders[Order Service<br/>Order lifecycle]
        Fulfillment[Fulfillment Engine<br/>Provider-order assignment]
        Pricing[Pricing Service<br/>Price calculation]
    end

    subgraph identity [Identity Domains]
        ProviderProfile[Provider Profile<br/>Onboarding, docs, ratings]
        CustomerProfile[Customer Profile<br/>Account, preferences]
    end

    subgraph support [Supporting Domains]
        Payments[Payment Service<br/>Capture, settle, refund]
        Notifications[Notifications<br/>Push, SMS, email]
        Geolocation[Geolocation Service<br/>ETA, routes, geocoding]
        DynamicPricing[Dynamic Pricing<br/>Supply/demand signals]
        Fraud[Fraud Engine<br/>Risk scoring, blocking]
    end

    Orders -->|"gRPC"| Pricing
    Orders -->|"gRPC"| Fulfillment
    Orders -.->|"orders.order.completed"| Payments
    Orders -.->|"orders.order.completed"| Notifications
    Fulfillment -->|"gRPC"| Geolocation
    Fulfillment -->|Redis GEO| ProviderLocations[(Provider Locations)]
    Pricing -->|"gRPC"| DynamicPricing
    Payments -.->|"payments.payment.captured"| Orders
    Payments -.->|"payments.payment.captured"| Notifications
    Fraud -.->|"fraud.signal.raised"| Fulfillment
    Fraud -.->|"fraud.signal.raised"| Orders
```

*Solid arrows = synchronous (gRPC). Dashed arrows = asynchronous (Kafka events).*

---

## Governance

| Aspect | Rule |
|--------|------|
| **Owner** | Platform Engineering team |
| **How to change it** | PR required; at least one Staff Engineer or Principal approval |
| **How to deviate** | Any exception requires an ADR documenting the rationale |
| **Review cadence** | Maturity model quarterly with Tech Leads; full manifesto bi-annually |
| **How to disagree** | Start a discussion in `#engineering-discussions` or raise it at Architecture Clinic. Then open a PR. |

---

## By The Numbers

| Section | Documents |
|---------|-----------|
| Platform Standards | 1 |
| Architecture & API | 9 |
| Engineering Practices | 11 |
| Infrastructure & Cloud | 10 |
| Operational Excellence | 9 |
| Developer Guides | 10 |
| Ways of Working | 10 |
| Program | 5 |
| Mobile & Frontend | 6 |
| AI/ML Platform | 2 |
| Domain Catalog | 10 |
| **Total** | **83 documents** |

Plus [`ONBOARDING.md`](./ONBOARDING.md) and [`GLOSSARY.md`](./GLOSSARY.md) at the root.

---

*{Company} Platform Engineering — last updated 2026*
