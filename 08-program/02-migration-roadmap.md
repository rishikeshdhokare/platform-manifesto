# 🗺️ Migration Roadmap

![Status: Active Program](https://img.shields.io/badge/status-Active_Program-blue?style=flat-square) ![Owner: CTO / Platform Engineering](https://img.shields.io/badge/owner-CTO_%2F_Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 📋 Overview

This document defines the **prioritised, phased program** to migrate the organisation from the current state to the target platform defined in this manifesto.

The migration follows a **strangler fig pattern** - we do not rewrite everything at once. We build the new platform alongside the existing system, migrate services incrementally, and decommission the old as we go.

**The golden rule:** Production reliability is non-negotiable. The migration must not increase operational risk.

Phases and exit criteria are **universal**. Checklists that name **AWS**, **EKS**, **Java**, **Gradle**, or **Spring** are **reference implementation** details - substitute your cloud accounts, Kubernetes distribution, language stack, and CI templates.

---

## 🧭 Principles

1. **Platform before services** - build the target platform first; migrate services second
2. **Prove it with one service** - the golden path service validates the full platform before broad rollout
3. **Automate the migration path** - manual migrations don't scale; build the tooling
4. **No big bangs** - every phase delivers working software to production
5. **Measure everything** - track migration progress, platform stability, and developer velocity

**Visual overview:**

```mermaid
gantt
    title Migration Roadmap
    dateFormat YYYY-Q
    axisFormat %Y-Q%q
    section Foundation
        Platform setup          :done, p0, 2026-Q1, 90d
        Golden path             :done, p1, 2026-Q1, 90d
    section Migrate
        Tier 1 services         :active, m1, 2026-Q2, 90d
        Tier 2 services         :m2, 2026-Q3, 90d
    section Optimize
        Performance tuning      :o1, 2026-Q4, 90d
        Cost optimization       :o2, 2026-Q4, 90d
    section Scale
        Multi-region            :s1, 2027-Q1, 90d
        Platform maturity       :s2, 2027-Q1, 90d
```

---

## 🏗️ Phase 0: Foundation (Weeks 1-6)

**Goal:** The new platform exists and can run one service end-to-end.

**Platform Engineering delivers:**

### Infrastructure (Weeks 1-3)

**Reference implementation (AWS):**

- [ ] AWS account structure created (Organisation, dev/staging/production accounts)
- [ ] VPC and networking in all accounts (Terraform)
- [ ] EKS clusters provisioned in all three environments
- [ ] ECR repository structure
- [ ] MSK (Kafka) clusters in all environments
- [ ] Amazon Aurora instances in all environments
- [ ] ElastiCache Redis clusters
- [ ] IAM roles and IRSA configuration
- [ ] AWS Secrets Manager + External Secrets Operator installed on EKS
- [ ] AWS GuardDuty, CloudTrail, Config enabled on all accounts

### CI/CD Platform (Weeks 2-4)
- [ ] GitHub organisation configuration (branch protection templates, CODEOWNERS tooling)
- [ ] Shared GitHub Actions workflow templates (e.g. `java-pr.yml`, `java-main.yml` for **reference stack Java**; add templates per primary language)
- [ ] ArgoCD installed and configured on all EKS clusters
- [ ] Pact Broker deployed
- [ ] SonarCloud organisation configured
- [ ] Snyk organisation configured
- [ ] Gitleaks pre-commit hook template

### Observability Stack (Weeks 3-5)
- [ ] Prometheus Operator + kube-prometheus-stack installed
- [ ] Grafana deployed with SSO
- [ ] Fluent Bit DaemonSet + OpenSearch deployed
- [ ] Distributed tracing (e.g. OpenTelemetry Collector; **reference:** AWS X-Ray alongside OTel)
- [ ] PagerDuty integration configured
- [ ] Platform standard alert rules defined

### Developer Tooling (Weeks 4-6)
- [ ] Backstage deployed with SSO
- [ ] Primary-language microservice template in Backstage (**reference:** Java)
- [ ] Platform BOM or dependency baseline (first version) published to your artifact registry (**reference:** GitHub Packages for Java BOM)
- [ ] `setup.sh` engineer onboarding script
- [ ] Local cloud emulator integration guide (**reference:** LocalStack for AWS-shaped APIs)
- [ ] `platform-config` GitOps repository structure

### Phase 0 Exit Criteria
- A test service can be scaffolded, built, and deployed to production via the full pipeline
- All observability components visible and collecting data
- At least one engineer outside Platform team successfully used the golden path

---

## ✨ Phase 1: Golden Path (Weeks 5-10)

> **Note:** The service name (`notifications-service`) and source VCS (`Bitbucket`) below are fictional exemplars. Replace them with your actual migration candidates.

**Goal:** Migrate **one real, non-critical service** end-to-end through the new platform. This is the proof point.

**Candidate service:** `notifications-service` (lower risk, no payment or fulfillment criticality)

### Tasks
- [ ] Migrate `notifications-service` repository from Bitbucket to GitHub
  - [ ] Full Git history preserved (`git clone --mirror`)
  - [ ] Branch protection rules applied
  - [ ] CI pipeline configured (PR + main)
- [ ] Containerise service with standard `Dockerfile` and non-root image
- [ ] Add structured JSON logging (**reference:** Logback for Java)
- [ ] Add Prometheus metrics (**reference:** Spring Actuator + Micrometer)
- [ ] Add OpenTelemetry tracing (language-appropriate agent or SDK; **reference:** Java agent)
- [ ] Write/migrate unit tests to your standard framework (**reference:** JUnit 5)
- [ ] Add integration tests with real dependencies in CI (**reference:** Testcontainers for Java)
- [ ] Define Pact consumer contracts for dependencies
- [ ] Add architecture tests (**reference:** ArchUnit for Java)
- [ ] Deploy to all environments via GitOps (ArgoCD)
- [ ] Create Grafana dashboard
- [ ] Configure P1/P2 alerts with runbooks
- [ ] Register in Backstage
- [ ] Write golden path documentation (the guide in `../06-developer-guides/02-golden-path.md`)

### Phase 1 Exit Criteria
- `notifications-service` live in production on new platform
- Full CI/CD pipeline working (merge → production in < 20 min)
- Service observable (logs searchable, metrics visible, traces working)
- All maturity model dimensions at L3 for this service
- One other team has reviewed the golden path and found no blockers
- Post-migration retro completed; lessons fed back into platform

---

## ⚡ Phase 2: Critical Path Services (Weeks 8-20)

**Goal:** Migrate the core services that form the critical path for a completed order.

**Sequence (order matters - lower risk / fewer dependencies first):**

| Order | Service | Risk | Key Dependencies | Est. Duration |
|-------|---------|------|-----------------|--------------|
| 1 | `customer-profile-service` | Medium | RDS, Redis | 3 weeks |
| 2 | `provider-profile-service` | Medium | RDS, S3 | 3 weeks |
| 3 | `pricing-service` | Medium | Aurora, Kafka | 2 weeks |
| 4 | `orders-service` | High | Aurora, Kafka, Fulfillment | 4 weeks |
| 5 | `fulfillment-engine` | High | Redis (geospatial), Kafka | 4 weeks |

Dependency column uses **reference implementation (AWS)** names (RDS, Aurora, S3) where applicable; map to your managed SQL, object store, and cache.

### Migration Pattern Per Service

Each service migration follows the same pattern:

```
Week 1: Codebase prep
  - Repository migration (Bitbucket → GitHub)
  - **Reference stack:** dependency upgrade to Java 21 + Spring Boot 3.x; build tool migration to Gradle (if on Maven); import platform BOM
  - **Other stacks:** align runtime and framework to platform baseline via equivalent upgrades
  - Unit test health check

Week 2: Platform integration
  - Containerisation + security hardening
  - Observability instrumentation
  - CI pipeline (PR + main)
  - Integration tests + Testcontainers
  - Pact contract tests

Week 3: Deployment
  - ArgoCD GitOps setup (all environments)
  - Feature flag migration (to LaunchDarkly)
  - Staging validation
  - Production canary rollout (5% → 25% → 100% over 3 days)
  - Old service traffic drained and decommissioned
```

### Parallel Running Strategy

For Tier 1 services (Orders, Fulfillment), we run **old and new in parallel** behind a feature flag before full cutover:

```
Production traffic routing (LaunchDarkly)
├── 100% → old-fulfillment-engine    (Phase 2 start)
├── 5%   → new-fulfillment-engine    (Week 1 canary)
├── 50%  → new-fulfillment-engine    (Week 2)
└── 100% → new-fulfillment-engine    (Week 3 cutover)
```

Shadow mode testing (write to both, read from old) where safe.

### Phase 2 Exit Criteria
- All five services live in production on new platform
- Old Bitbucket pipelines decommissioned for migrated services
- Full observability for all services
- Incident data shows no regression in MTTR or incident rate

---

## 📦 Phase 3: Remaining Services (Weeks 18-36)

**Goal:** Migrate all remaining services to the new platform.

At this point, the migration pattern is proven and the platform is mature. Teams can self-serve their migrations with Platform team as support.

**Migration velocity target:** 2 services per team per 4-week cycle.

### Remaining Service Categories

- [ ] **BFF services** (Customer App BFF, Provider App BFF, Ops BFF)
- [ ] **Payment services** (Payment Service, Payout Service) - requires extra care: PCI-DSS scope
- [ ] **Analytics services** (Reporting service, data pipeline components)
- [ ] **Supporting services** (Maps proxy, SMS gateway, push notification workers)
- [ ] **Admin tooling** (Ops portal backend)

### Self-Service Migration Guide

At this phase, each team owns their migration with a checklist:

```
[ ] Migration kickoff: team completes pre-migration assessment
[ ] Platform team reviews: 1-hour review session, identify risks
[ ] Migration execution: team follows golden path pattern
[ ] Platform team sign-off: golden path checklist review
[ ] Decommission: old service drained and removed from Bitbucket
```

### Phase 3 Exit Criteria
- 100% of services on new platform
- Bitbucket decommissioned
- Old CI system (Jenkins / Bitbucket Pipelines) decommissioned
- All teams at L3 on maturity model
- New hire can be productive within their first week without platform hand-holding

---

## 🔧 Phase 4: Optimise (Month 10+)

**Goal:** Continuous improvement - push teams from L3 to L4.

- [ ] Load testing program: weekly automated load tests for all Tier 1 services
- [ ] SLO program: all services have defined and tracked SLOs
- [ ] Error budget policy: formal freeze process when budget exhausted
- [ ] Cost optimisation: Karpenter tuning, Reserved capacity planning
- [ ] Platform self-service improvements: Backstage template updates, richer scaffolding
- [ ] Developer productivity metrics: DORA metrics baseline and improvement targets
- [ ] DR exercises: quarterly failover tests

---

## 📊 DORA Metrics - Baseline & Target

We will track **DORA (DevOps Research and Assessment) metrics** as the primary measure of platform health.

| Metric | Current State (Estimated) | Phase 1 Target | Phase 3 Target |
|--------|--------------------------|----------------|----------------|
| **Deployment Frequency** | Weekly or less | Daily | Multiple times/day |
| **Lead Time for Changes** | Weeks | < 3 days | < 1 day |
| **Change Failure Rate** | Unknown | < 10% | < 5% |
| **MTTR (Mean Time to Restore)** | Hours | < 1 hour | < 30 minutes |

DORA metrics are tracked in the engineering health dashboard (Grafana) and reviewed monthly by the CTO.

---

## ⚠️ Migration Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Data loss during DB migration | Low | Critical | Parallel running; point-in-time recovery tested |
| Team resistance to new practices | Medium | High | Involve teams early; golden path makes it easy |
| Platform instability during migration | Medium | High | Migrate non-critical services first; gradual rollout |
| Skill gaps (Kubernetes, Terraform) | Medium | Medium | Training program; pairing with Platform team |
| Scope creep (rewrites during migration) | High | Medium | Strict rule: migrate, don't rewrite. Rewrites are separate projects. |
| Underestimating service complexity | Medium | Medium | Pre-migration assessment for every service |
| Bitbucket/old system dependency | Low | Medium | Map all dependencies before migrating any service |

---

## 🏛️ Governance & Reporting

- **Weekly:** Platform Engineering standup (internal) - migration progress, blockers
- **Bi-weekly:** Migration status report to CTO - services migrated, DORA metrics, risks
- **Monthly:** Engineering all-hands - maturity model scores, platform wins, what's next
- **Quarterly:** Maturity model review with all Tech Leads

---

## 👥 Contacts

| Role | Person | Responsibility |
|------|--------|---------------|
| Program Sponsor | CTO | Priority decisions, resource allocation |
| Platform Lead | Head of Platform Engineering | Technical delivery of platform |
| Migration Coordinator | Staff Engineer | Cross-team migration tracking |
| Security Lead | Security Engineer | Security standards and compliance |

---

*This is a living document. Update migration progress weekly. When a service is migrated, mark it complete and capture lessons learned.*

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
