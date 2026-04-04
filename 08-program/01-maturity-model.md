# 📊 Maturity Model

![Status: Reference](https://img.shields.io/badge/status-Reference-blue?style=flat-square) ![Owner: Platform Engineering + CTO](https://img.shields.io/badge/owner-Platform_Engineering_%2B_CTO-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 📋 Overview

This maturity model allows teams to **self-assess their engineering practices** against the platform standard. It is not a performance review tool - it is a diagnostic tool to identify where to invest next.

Each capability has four levels:

| Level | Label | Meaning |
|-------|-------|---------|
| **0** | None | Not in place |
| **1** | Initiating | Started but inconsistent |
| **2** | Developing | In place, not fully automated |
| **3** | Defined | Consistently applied, partially automated |
| **4** | Optimising | Fully automated, measured, continuously improving |

**Target state:** All teams at Level 3 or above within 12 months of platform launch. Level 4 is the long-term aspiration.

---

## 🔀 Dimension 1: Source Control & Branching

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Branch strategy | Long-lived feature branches | Feature branches < 1 week | Trunk-based; branches < 2 days | Trunk-based; most devs commit directly to main with feature flags |
| Commit standards | Freeform commit messages | Some conventions applied | Conventional Commits enforced by CI linter | Commits auto-link to tickets; changelogs auto-generated |
| Branch protection | None | Basic PR required | PR required + status checks + 1 reviewer | Automated reviewer assignment; CODEOWNERS enforced |
| PR size | PRs regularly > 1000 lines | PRs often > 500 lines | PRs consistently < 400 lines | Team tracks and trends PR size as a health metric |

---

## 🧪 Dimension 2: Testing

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Unit tests | Few or no unit tests | Unit tests exist but coverage < 60% | ≥ 80% coverage; tests are meaningful | Coverage ≥ 90% on domain logic; mutation testing |
| Integration tests | None or manual | Integration tests exist but use mocks/H2 | Testcontainers with real dependencies | Full integration suite < 3 min; reliable and fast |
| Contract tests | None | Consumer-driven contracts exist | Pact contracts in CI; blocks provider deployment | Pact Broker tracks compatibility across all consumers |
| E2E tests | None or manual QA | Some E2E automation | E2E covers all critical journeys; runs nightly | E2E runs pre-production; <20 min; parallelised |
| Performance tests | Ad-hoc load tests | Load tests exist | Weekly automated load tests; baselines defined | Performance regressions detected automatically in CI |
| Flaky tests | Flaky tests accepted | Some flaky tests quarantined | Flaky tests quarantined within 24h; 0 tolerance | Flakiness tracked as team metric; trend to zero |
| Architecture tests | None | Some ArchUnit rules | Standard platform ArchUnit rules in all services | Team adds custom rules for their domain |

---

## 🔨 Dimension 3: CI / Build

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| CI pipeline exists | No pipeline | Pipeline runs tests only | Full pipeline: lint, test, coverage, security scan | Pipeline is shared template; maintained by platform |
| Pipeline speed | > 20 min | 15–20 min | < 10 min | < 7 min; measured and trending |
| Quality gates | No gates | Some gates, not enforced | All gates enforced; PRs blocked on failure | Gate history tracked; failures trend to zero |
| Secret detection | No scanning | Manual review | Gitleaks in pre-commit and CI | Zero tolerance; alert on any detection event |
| Dependency scanning | Quarterly manual | Monthly Snyk | Daily Snyk scan; CI blocks on critical/high | Auto-PRs for patch versions; < 5 open vulnerabilities at any time |
| Container scanning | Not done | Manual periodically | Snyk on every build; blocks on critical | ECR enhanced scanning + weekly rebuild of base images |

---

## 🚀 Dimension 4: Continuous Delivery

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Deployment frequency | Monthly or less | Weekly | Multiple times per week | Multiple times per day (on demand) |
| Lead time to production | Weeks | Days | < 1 day | < 2 hours from merge to production |
| Deployment process | Manual | Semi-automated | Fully automated via GitOps | Self-service with zero-touch production deploys |
| Feature flags | None | Basic env-var flags | LaunchDarkly for all unreleased features | Flags used for experiments, ops kill switches, and progressive rollout |
| Canary / progressive delivery | Full rollout only | Manual canary | Automated canary with Argo Rollouts | Automated canary analysis; auto-rollback on threshold breach |
| Rollback | Manual, slow | Manual ArgoCD; rollback SLA < 30 min | Automated rollback for canary failures; rollback SLA < 15 min | Fully automated rollback < 5 min; zero-touch |
| Environment parity | Dev ≠ Staging ≠ Production | Partial parity | All environments structurally identical | Drift detection alerts on any deviation |

---

## 👁️ Dimension 5: Observability

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Logging | Unstructured logs or no logs | Structured logs, inconsistent fields | Structured JSON with all required fields; correlation IDs | Logs searchable in OpenSearch; traceId → log query from trace |
| Metrics | No metrics | Basic Spring Actuator metrics | RED metrics + business metrics + Grafana dashboard | Custom SLO dashboard; error budget tracked |
| Distributed tracing | No tracing | Some manual instrumentation | OTel agent; full trace from BFF to DB | Trace sampling optimised; tail-based sampling for errors |
| Alerting | No alerts | Some alerts, no runbooks | P1/P2 alerts with runbooks for all services | Alerts reviewed quarterly; false positive rate < 5% |
| SLOs | Not defined | SLOs defined; error budget policy defined and enforced | SLOs tracked in Grafana; burn-rate alerting configured | Error budget automated; freeze on budget exhaustion with self-service override |
| On-call runbooks | None | Exists but outdated | Runbooks up to date; linked from every alert | Runbooks tested in DR exercises; measurable MTTR |
| Service dependency map | None | Informal knowledge only | Service dependencies mapped in Backstage; blast radius documented | Live dependency graph auto-generated; impact analysis automated |

---

## 🔐 Dimension 6: Security

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Secrets management | Secrets in code or env vars | Secrets in Secrets Manager but manually managed | All secrets via External Secrets Operator; auto-rotated | Rotation tested; zero manual secret handling |
| Auth & authz | API key or no auth | JWT auth at gateway only | JWT validated + resource-level authorisation in services | RBAC model documented; access reviews quarterly |
| Dependency vulnerabilities | Not tracked | Tracked, slow remediation | Critical/high fixed within SLA | Patch PRs merged same day; zero high CVEs at any time |
| IaC security | No scanning | Manual review | tfsec + Checkov in CI; blocks on high | Security posture score tracked; improving trend |
| Container security | No scanning | Scanned periodically | Scanned on every build; non-root enforced | CIS benchmark compliance for all images |
| DAST | None | Manual pen test only | OWASP ZAP weekly in staging | Findings auto-tracked; mean time to remediation < 14 days |

---

## 🧑‍💻 Dimension 7: Developer Experience

| Practice | L1 | L2 | L3 ✅ Target | L4 |
|----------|----|----|-------------|-----|
| Local dev setup | No docs; complex setup | README exists; setup takes > 1 day | `docker compose up` + `./gradlew bootRun` works; < 1 hour to running | Setup script installs everything; < 30 min from zero |
| Onboarding | No structured onboarding | Basic README | New engineer ships to production in first week | NPS survey; onboarding continuously improved |
| Service scaffolding | Copy-paste from another service | Template exists but outdated | Backstage template generates fully working service | Template generates CI pipeline, ArgoCD app, Backstage entry automatically |
| Inner loop speed | Build > 2 minutes | Build 1–2 minutes | Build < 60s; test < 30s; startup < 15s | Incremental build < 10s; hot reload < 3s |
| Documentation | No docs or outdated | README exists | README + runbook + ADRs up to date | TechDocs in Backstage; searchable; review reminders |
| Platform BOM | No central dependency management | BOM exists but teams ignore it | All services on platform BOM; versions aligned | BOM updated quarterly; breaking changes communicated 4 weeks ahead |

---

## 📝 How to Self-Assess

### Quarterly Process

1. **Before the quarter review:** Tech Lead fills in the self-assessment worksheet (link in team wiki)
2. **In the review (1 hour, all Tech Leads + Platform team):**
   - Each team presents their self-assessment (10 min each)
   - Peer calibration - teams challenge each other's scores
   - Platform team highlights patterns across teams
3. **Output:** Each team commits to 2-3 improvement goals for the next quarter
4. **Tracking:** Improvement goals tracked as Jira epics; reviewed at next quarterly

### Scoring Guide

When assessing a level, **be honest and conservative**:
- If it works sometimes or for some services → one level lower
- If it requires manual steps → one level lower than "automated"
- If it exists but nobody uses it → L1 at best

The goal is an accurate picture, not a high score.

---

## 📱 Dimension 8: Real-Time & Mobile

| Practice | L1 | L2 | L3 Target | L4 |
|----------|----|----|-----------|-----|
| WebSocket / live updates | No real-time; polling only | WebSocket exists but unreliable | WebSocket with Redis pub/sub fan-out; graceful polling fallback | Sub-second location updates; connection monitoring; auto-reconnect |
| Push notifications | No push | Basic push; no channel management | FCM/APNs with notification channels; silent push for sync | Delivery tracking; A/B tested notification copy; per-user preferences |
| Offline handling | App crashes offline | Basic error screens offline | Optimistic updates; local cache; sync on reconnect | Conflict resolution; offline-first architecture; seamless transition |
| Mobile performance | No measurement | Some performance tracking | Budgets defined (startup <3s, TTI <5s); monitored in CI | Regression detection in CI; real-device testing; battery profiling |

---

## 🛡️ Dimension 9: Resilience Validation

| Practice | L1 | L2 | L3 Target | L4 |
|----------|----|----|-----------|-----|
| Chaos engineering | None | Ad-hoc manual fault injection | Quarterly game days; experiment catalog; staging chaos | Monthly production chaos; automated steady-state validation |
| DR exercises | None | DR plan exists on paper | Quarterly DR exercises in staging; documented results | DR exercises in production; measured RTO/RPO matches targets |
| Circuit breaker validation | Configured but never tested | Manually tested once | Chaos experiments validate CB triggers correctly | Automated CB validation in CI; drift detection |
| Failure mode documentation | None | Some failure modes documented | All critical failure modes documented with expected behavior | Failure modes tested; runbooks validated via chaos |

---

## 💰 Dimension 10: Cost Management

| Practice | L1 | L2 | L3 Target | L4 |
|----------|----|----|-----------|-----|
| Cost visibility | No tagging | Some resources tagged | All resources tagged; per-team cost dashboards | Cost-per-transaction tracked; anomaly detection active |
| Budget management | No budgets | AWS budget exists but not monitored | Budget alerts at 80%/100%; monthly team review | Quarterly rightsizing; Savings Plans optimized |
| Dev/staging optimization | Always-on everywhere | Manual scale-down | Automated scale-to-zero outside hours; staging at 50% | Spot instances for non-critical; Karpenter optimized |
| FinOps cadence | None | Annual review | Monthly per-team review | Weekly cost anomaly triage; cost part of PR review for infra changes |

---

## 🤖 Dimension 11: AI/ML Maturity

| Practice | L1 | L2 | L3 Target | L4 |
|----------|----|----|-----------|-----|
| Model serving | No ML; rule-based only | ML models exist but deployed ad-hoc | SageMaker endpoints; canary for model updates; fallback to rules | Feature store; A/B testing; automated retraining |
| ML observability | No monitoring | Basic accuracy tracking | Drift detection; prediction quality alerts; data quality checks | Automated retraining triggers; model performance SLOs |
| AI governance | No governance | Informal bias awareness | Quarterly bias audits; explainability for customer-facing decisions | AI Ethics Review Board; automated fairness metrics in CI |
| AI-assisted development | Ad-hoc tool usage | Approved tools list | Guardrails enforced; code review standards for AI PRs | AI productivity metrics tracked; contribution to platform tooling |

---

## 🎯 Platform Target State

The platform is designed to make **Level 3 the default** - if you follow the golden path, you hit Level 3 automatically. Level 4 requires deliberate investment by teams.

| Timeline | Target |
|----------|--------|
| Month 3 | All teams at L2 across all dimensions |
| Month 6 | All teams at L3 in CI, CD, and Security |
| Month 12 | All teams at L3 across all dimensions |
| Month 18 | Lead teams at L4 in 3+ dimensions |

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
