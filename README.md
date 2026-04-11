<div align="center">

<br/>

# 🏗️ {Company} Platform Engineering Manifesto

*The engineering playbook for teams that ship fast and sleep well.*

<br/>

[![Sections](https://img.shields.io/badge/13_sections-6C3FC5?style=for-the-badge&logo=bookstack&logoColor=white)](.)
[![Updated](https://img.shields.io/badge/2026-00A86B?style=for-the-badge&logo=calendar&logoColor=white)](.)
[![PRs Welcome](https://img.shields.io/badge/PRs_welcome-FF1493?style=for-the-badge&logo=github&logoColor=white)](.)
[![Agent-Ready](https://img.shields.io/badge/Agent--Native_Ready-00C853?style=for-the-badge&logo=probot&logoColor=white)](./12-ai-engineering/01-context-engineering.md)

<br/>

Every *"how should I do this?"* answered. Every *"which tool should I use?"* decided.<br/>
**Opinionated. Zero meetings required.**

<br/>

`Opinionated by design` &nbsp; `Follow by default` &nbsp; `Challenge by PR` &nbsp; `Agent-native ready`

---

[🆕 Onboarding](./ONBOARDING.md) &nbsp;·&nbsp; [📖 Glossary](./GLOSSARY.md) &nbsp;·&nbsp; [📊 Executive summary](./EXECUTIVE-SUMMARY.md) &nbsp;·&nbsp; [📋 Meta](./META.md) &nbsp;·&nbsp; [🏛️ Architecture](02-architecture-and-api/01-system-architecture.md) &nbsp;·&nbsp; [⚙️ Backend Standards](03-engineering-practices/09-backend-framework-standards.md) &nbsp;·&nbsp; [🛤️ Golden Path](06-developer-guides/02-golden-path.md) &nbsp;·&nbsp; [🚨 Incidents](05-operational-excellence/04-incident-management.md)

</div>

---

## 💡 What This Is (And What It Isn't)

| ✅ This is | ❌ This is not |
|:-----------|:---------------|
| A decision record that prevents re-debates | A suggestion box you can ignore |
| A living playbook, updated from production lessons | A dusty wiki page someone wrote two years ago |
| Opinionated and prescriptive by design | A buffet of options for every preference |
| The fastest path from idea to production | The *only* path - deviations just need an ADR |
| A shared operating system for human and AI agent teams alike | A human-only playbook that agents cannot parse or follow |

---

## 🤔 Why This Exists

You've seen it before. A new service gets built. The team picks a different database. Invents their own error format. Structures logs differently. Deploys with a custom pipeline. Six months later, nobody can debug across services, onboarding takes weeks, and every team is solving the same problems in isolation.

**This manifesto exists to make that impossible.**

One source of truth. One set of standards. Every team, every service, every environment. Consistency compounds - every shared decision here is one fewer decision each team makes alone.

> 📖 **Living document.** It evolves as our platform matures and as we learn from production. Changes require a PR with at least one Staff Engineer approval.

> 🔧 **Adopting this for your organization?** The principles are universal; the specific tools are our reference implementation. See [Customize This Manifesto](#-customize-this-manifesto) for how to swap in your own stack.

---

## 🤖 Agent-Native by Design

This manifesto is built for **human-led and agent-operated organizations alike**. If you are building an agentic software company - where AI agents plan, code, review, test, deploy, and operate alongside humans - this manifesto serves as the shared operating system that both humans and agents follow.

### Why it works for agents

| Design Choice | Agent Benefit |
|:--------------|:-------------|
| **Opinionated rules, not suggestions** | Agents need constraints, not options. "All services must use RFC 7807 errors" is a rule an agent can enforce. "Consider using RFC 7807" is not |
| **Structured, consistent formatting** | Numbered files (`NN-kebab-case.md`), emoji-prefixed sections, and consistent document structure make the manifesto machine-parseable without special tooling |
| **Explicit contracts everywhere** | API standards, event schemas, error catalogs, and naming conventions give agents the specs they need to generate compliant code autonomously |
| **Context engineering built in** | [Section 12](./12-ai-engineering/) defines how to structure knowledge so AI tools can access and apply it - from AGENTS.md to cursor rules to RAG pipelines |
| **Golden path templates** | Scaffold templates and standardized patterns let agents bootstrap new services that are compliant from the first commit |
| **Machine-readable glossary** | The [Glossary](./GLOSSARY.md) provides shared terminology so agents use the same domain language as your teams |

### For agentic organizations

If your engineering organization uses AI agents as first-class participants - not just assistants - this manifesto provides:

- **Agent-to-agent contracts** - The same API standards, event schemas, and error formats that govern human-built services govern agent-built services. No separate "agent API" layer needed.
- **Autonomous compliance** - Agents can read the manifesto, extract rules, and enforce them in CI/CD pipelines, code generation, architecture reviews, and incident response without human intervention.
- **Human-in-the-loop where it matters** - The manifesto is explicit about where human judgment is required (architecture decisions, security trade-offs, ethical considerations) and where agents can operate autonomously.
- **Onboarding for agents, not just humans** - The same AGENTS.md and context files that help a new human engineer ramp up help a new AI agent operate correctly in your codebase from its first interaction.

> 🤖 **Building an agent-native org?** Start with [Context Engineering](./12-ai-engineering/01-context-engineering.md) to wire the manifesto into your agent orchestration layer. Then follow the [Golden Path](./06-developer-guides/02-golden-path.md) to set up your first agent-operated service.

---

## 🧭 Pick Your Path

| You are... | Start here | Then explore |
|:-----------|:-----------|:-------------|
| 📊 **CTO, COO, or procurement** | [`EXECUTIVE-SUMMARY.md`](./EXECUTIVE-SUMMARY.md) | Why this exists → adoption phases → migration roadmap |
| 🆕 **Day 1 new joiner** | [`ONBOARDING.md`](./ONBOARDING.md) | Tech stack → Golden Path → Git workflow |
| ⚙️ **Backend engineer** | [Backend Framework Standards](03-engineering-practices/09-backend-framework-standards.md) | Architecture, Kafka, Caching, Testing |
| 🌐 **Frontend engineer** | [Web Frontend Standards](09-mobile-and-frontend/02-web-frontend-standards.md) | Frontend CI/CD, Design System |
| 📱 **Mobile engineer** | [Mobile Standards](09-mobile-and-frontend/01-mobile-standards.md) | Android / iOS / React Native guides |
| 🏛️ **Tech lead** | [System Architecture](02-architecture-and-api/01-system-architecture.md) | Service Decomposition, Team Topology |
| 👥 **Engineering manager** | [Engineering Management](07-ways-of-working/09-engineering-management.md) | Ladder, Metrics, Product Ops |
| 🔥 **On-call and panicking** | [Incident Management](05-operational-excellence/04-incident-management.md) | Debugging Guide, DR Playbook |
| 🎨 **Designer** | [Design System](09-mobile-and-frontend/06-design-system.md) | Web Frontend, Mobile Standards |
| 📊 **Product manager** | [Product Operations](07-ways-of-working/10-product-operations.md) | A/B Testing, Engineering Metrics |
| 🔒 **Security engineer** | [Security Operations](04-infrastructure-and-cloud/10-security-operations.md) | Security Standards, Privacy Engineering |
| 🧬 **AI-first engineer** | [Context Engineering](12-ai-engineering/01-context-engineering.md) | AI-Assisted SDLC, Adoption Metrics |
| 🤖 **Agentic org builder** | [Agent-Native by Design](#-agent-native-by-design) · [Context Engineering](12-ai-engineering/01-context-engineering.md) | AI-Assisted SDLC, API Standards, Golden Path, Service Catalog |
| 📝 **Technical Writer / Documentation** | [Developer Experience](06-developer-guides/01-developer-experience.md) · [Knowledge Sharing](07-ways-of-working/08-knowledge-sharing.md) | Content standards, documentation governance |
| 🧪 **QA / Test Engineer** | [Testing Pyramid](03-engineering-practices/01-testing-pyramid.md) · [QA Standards](03-engineering-practices/11-qa-standards.md) | Test strategy, environments, automation |
| ⚖️ **Procurement / Legal** | [Vendor Intake](08-program/05-vendor-intake.md) · [Vendor Assessment](08-program/03-vendor-assessment.md) | Vendor intake, assessment, compliance |
| 🎧 **Support / Customer Success** | [Incident Management](05-operational-excellence/04-incident-management.md) · [Debugging Guide](05-operational-excellence/09-debugging-guide.md) | Incident response, debugging, SLOs |
| 🔢 **Data Analyst / BI** | [Data Platform](06-developer-guides/05-data-platform.md) · [Data Governance](06-developer-guides/09-data-governance.md) | Data platform, governance, analytics |

> 💡 Lost on a term? The [**Glossary**](./GLOSSARY.md) has you covered.

---

## 🎯 The Non-Negotiables

Eight principles. Not aspirations - **constraints**. Every technical decision in this manifesto traces back to one of these.

<table>
<tr>
<td align="center" width="50%">

🛤️<br/><b>Pave the golden path</b><br/><sub>Make the right way the easy way. Teams feel <i>pulled</i> toward standards, not pushed.</sub>

</td>
<td align="center" width="50%">

📦<br/><b>Ship artifacts, not code</b><br/><sub>The same container image built in CI runs in production. Never rebuild between environments.</sub>

</td>
</tr>
<tr>
<td align="center">

👁️<br/><b>Observable by default</b><br/><sub>Logs, metrics, and traces ship from day one. Observability is not a retrofit.</sub>

</td>
<td align="center">

🔒<br/><b>Own your data, respect boundaries</b><br/><sub>Services own their data stores. Cross-service DB access is forbidden. No exceptions.</sub>

</td>
</tr>
<tr>
<td align="center">

📝<br/><b>Everything in Git</b><br/><sub>Infrastructure, config, runbooks, ADRs. If it's not in Git, it doesn't exist.</sub>

</td>
<td align="center">

🤖<br/><b>Automate the boring</b><br/><sub>Pipelines handle repetition. Humans handle architecture and product problems.</sub>

</td>
</tr>
<tr>
<td align="center">

💥<br/><b>Design for failure</b><br/><sub>Every dependency will fail. Circuit breakers, retries, fallbacks, graceful degradation.</sub>

</td>
<td align="center">

🛡️<br/><b>Security is not a phase</b><br/><sub>It runs in every pipeline, in every environment, from the first commit.</sub>

</td>
</tr>
</table>

<br/>

<div align="center"><sub><i>Those are the beliefs. Here's the system they built. ↓</i></sub></div>

---

## 🗺️ The Big Picture

From client apps to infrastructure, here's how the pieces fit together.

```mermaid
flowchart TB
    subgraph clients [Client Layer]
        CustomerApp[Customer App]
        ProviderApp[Provider App]
        OpsPortal[Ops Portal]
        PartnerAPI[Partner API]
    end

    subgraph gateway [API Gateway Layer]
        APIGW[API Gateway + WAF]
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
        Kafka[Event Streaming]
        Aurora[(Relational DB)]
        Redis[(Cache Layer)]
        OpenSearch[(Search Engine)]
        S3[(Object Storage)]
    end

    subgraph infra [Infrastructure]
        K8s[Kubernetes]
        Mesh[Service Mesh]
        GitOps[GitOps Engine]
        Observability[Observability Stack]
    end

    clients --> gateway
    gateway --> bff
    bff --> core
    bff --> supporting
    core --> platform
    supporting --> platform
    core --> Kafka
    supporting --> Kafka
    K8s --> core
    K8s --> supporting
    K8s --> bff
    Mesh --> K8s
```

<br/>

<div align="center"><sub><i>That's the 30,000-foot view. Now here's every document, organized by domain. ↓</i></sub></div>

---

## 📚 The Full Library

Click any section to explore.

<details>
<summary><b>📐 01 - Platform Standards</b> &nbsp;·&nbsp; <i>The foundation layer - touch this, everything changes</i></summary>

<br/>

The approved tech stack, naming conventions, repository structure, service catalog, and container standards. The foundation everything else builds on.

| File | What You'll Learn |
|------|-------------------|
| [`01-tech-stack.md`](01-platform-standards/01-tech-stack.md) | Approved languages, frameworks, cloud services, data stores, and the principles behind each choice |
| [`02-naming-conventions.md`](01-platform-standards/02-naming-conventions.md) | How to name everything - services, repos, packages, topics, buckets, metrics, flags |
| [`03-repository-standards.md`](01-platform-standards/03-repository-standards.md) | Required files, README template, branch protection, PR template, repo lifecycle |
| [`04-service-catalog.md`](01-platform-standards/04-service-catalog.md) | Backstage catalog-info.yaml spec, lifecycle states, scorecards, ownership |
| [`05-container-standards.md`](01-platform-standards/05-container-standards.md) | Base images, Dockerfile standards, tagging, size limits, signing, registry |

</details>

<details>
<summary><b>🏛️ 02 - Architecture & API</b> &nbsp;·&nbsp; <i>Domains, contracts, and the rules between services</i></summary>

<br/>

Domain decomposition, communication patterns, API contracts, event schemas, and the error catalog.

| File | What You'll Learn |
|------|-------------------|
| [`01-system-architecture.md`](02-architecture-and-api/01-system-architecture.md) | Domain map, event backbone, BFF pattern, resilience ownership matrix |
| [`02-api-standards.md`](02-architecture-and-api/02-api-standards.md) | URL design, versioning, error shapes, pagination, rate limiting, idempotency |
| [`03-hexagonal-architecture.md`](02-architecture-and-api/03-hexagonal-architecture.md) | Ports & adapters with a full worked example you can copy |
| [`04-real-time-architecture.md`](02-architecture-and-api/04-real-time-architecture.md) | WebSocket, SSE, push notifications, and location streaming |
| [`05-grpc-standards.md`](02-architecture-and-api/05-grpc-standards.md) | Proto conventions, code generation, load balancing, cancellation |
| [`06-saga-patterns.md`](02-architecture-and-api/06-saga-patterns.md) | Distributed transactions, choreography, compensation patterns |
| [`07-service-decomposition.md`](02-architecture-and-api/07-service-decomposition.md) | When to split or merge services, with a decision framework |
| [`08-event-schema-evolution.md`](02-architecture-and-api/08-event-schema-evolution.md) | Avro compatibility rules, partition keys, and the breaking change playbook |
| [`09-error-catalog.md`](02-architecture-and-api/09-error-catalog.md) | Central error registry, exception handling, and frontend error boundaries |
| [`10-monolith-first-guide.md`](02-architecture-and-api/10-monolith-first-guide.md) | When to start with a monolith, modular monolith pattern, and migration path |
| [`11-api-versioning.md`](02-architecture-and-api/11-api-versioning.md) | API versioning strategy, breaking change policy, deprecation |
| [`12-rate-limiting.md`](02-architecture-and-api/12-rate-limiting.md) | Rate limiting and throttling patterns, per-tenant limits |
| [`13-webhook-standards.md`](02-architecture-and-api/13-webhook-standards.md) | Webhook design, payload signing, delivery guarantees |
| [`14-batch-processing.md`](02-architecture-and-api/14-batch-processing.md) | Batch processing, CRON jobs, workflow orchestration |
| [`15-contract-testing.md`](02-architecture-and-api/15-contract-testing.md) | Consumer-driven contracts, Pact, CI integration |
| [`16-messaging-patterns.md`](02-architecture-and-api/16-messaging-patterns.md) | Messaging patterns beyond Kafka, queues, outbox pattern |
| [`17-graphql-standards.md`](02-architecture-and-api/17-graphql-standards.md) | GraphQL standards, federation, BFF integration |
| [`18-event-sourcing-cqrs.md`](02-architecture-and-api/18-event-sourcing-cqrs.md) | Event sourcing and CQRS patterns |
| [`19-api-lifecycle.md`](02-architecture-and-api/19-api-lifecycle.md) | API lifecycle management, design through sunset |

</details>

<details>
<summary><b>⚙️ 03 - Engineering Practices</b> &nbsp;·&nbsp; <i>Testing, CI/CD, code review - the daily craft</i></summary>

<br/>

The day-to-day craft. Testing, CI/CD, code review, coding standards, and backend framework standards.

| File | What You'll Learn |
|------|-------------------|
| [`01-testing-pyramid.md`](03-engineering-practices/01-testing-pyramid.md) | Unit, integration, contract, E2E, load tests - with worked examples |
| [`02-ci-practices.md`](03-engineering-practices/02-ci-practices.md) | GitHub Actions pipelines, quality gates, DAST, Terraform testing |
| [`03-cd-practices.md`](03-engineering-practices/03-cd-practices.md) | GitOps, canary deployments, feature flags, change risk rubric |
| [`04-coding-standards.md`](03-engineering-practices/04-coding-standards.md) | Naming, error handling, null safety - with before/after examples |
| [`05-git-workflow.md`](03-engineering-practices/05-git-workflow.md) | Trunk-based dev in practice |
| [`06-code-review-guide.md`](03-engineering-practices/06-code-review-guide.md) | How to give and receive feedback that makes code better |
| [`07-ab-testing.md`](03-engineering-practices/07-ab-testing.md) | Experiment design, statistical rigor, guardrails, product analytics |
| [`08-deprecation-lifecycle.md`](03-engineering-practices/08-deprecation-lifecycle.md) | How we sunset APIs, events, services, and flags |
| [`09-backend-framework-standards.md`](03-engineering-practices/09-backend-framework-standards.md) | Structured logging, config layering, health checks, feature flags, graceful shutdown, versioning |
| [`10-frontend-ci-cd.md`](03-engineering-practices/10-frontend-ci-cd.md) | Frontend golden path, monorepo structure, preview environments |
| [`11-qa-standards.md`](03-engineering-practices/11-qa-standards.md) | Test environments, test data, bug triage severity, device matrix |
| [`12-engineering-principles.md`](03-engineering-practices/12-engineering-principles.md) | Core engineering principles for the AI era with agent-native context |
| [`13-specification-driven-development.md`](03-engineering-practices/13-specification-driven-development.md) | Specification-driven development as the norm for agentic organizations |
| [`14-supply-chain-security.md`](03-engineering-practices/14-supply-chain-security.md) | SBOM generation, dependency scanning, license compliance, trusted registries |
| [`15-test-data-management.md`](03-engineering-practices/15-test-data-management.md) | Test data strategies, synthetic data generation, PII masking |
| [`16-progressive-delivery.md`](03-engineering-practices/16-progressive-delivery.md) | Canary, blue-green, shadow, dark launch, traffic splitting |
| [`17-monorepo-polyrepo.md`](03-engineering-practices/17-monorepo-polyrepo.md) | Monorepo vs polyrepo decision framework |
| [`18-cross-team-dependencies.md`](03-engineering-practices/18-cross-team-dependencies.md) | Interface contracts, API compatibility, breaking changes |
| [`19-performance-engineering.md`](03-engineering-practices/19-performance-engineering.md) | Performance engineering and backend budgets |

</details>

<details>
<summary><b>☁️ 04 - Infrastructure & Cloud</b> &nbsp;·&nbsp; <i>The invisible floor you stand on</i></summary>

<br/>

Cloud architecture, security, FinOps, multi-tenancy, and infrastructure as code.

| File | What You'll Learn |
|------|-------------------|
| [`01-cloud-architecture.md`](04-infrastructure-and-cloud/01-cloud-architecture.md) | Account structure, network topology, compute, data tier, backup & restore |
| [`02-infra-components.md`](04-infrastructure-and-cloud/02-infra-components.md) | Service mesh, developer portal, GitOps, container registry, and self-service boundaries |
| [`03-security.md`](04-infrastructure-and-cloud/03-security.md) | Shift-left security, identity and access, PII handling, encryption, egress control |
| [`04-configuration-management.md`](04-infrastructure-and-cloud/04-configuration-management.md) | Secrets vs config vs flags - what lives where |
| [`05-finops.md`](04-infrastructure-and-cloud/05-finops.md) | Cost allocation, rightsizing, reservations, real-time cost visibility, sustainability |
| [`06-capacity-planning.md`](04-infrastructure-and-cloud/06-capacity-planning.md) | Demand modeling, pre-scaling, load test-driven validation |
| [`07-api-gateway-strategy.md`](04-infrastructure-and-cloud/07-api-gateway-strategy.md) | API Gateway configuration, routing, throttling, WAF |
| [`08-privacy-engineering.md`](04-infrastructure-and-cloud/08-privacy-engineering.md) | GDPR, PCI, SOC 2, ISO 27001, anonymization, consent |
| [`09-multi-tenancy.md`](04-infrastructure-and-cloud/09-multi-tenancy.md) | Data isolation patterns, noisy-neighbor controls, tenant-aware observability |
| [`10-security-operations.md`](04-infrastructure-and-cloud/10-security-operations.md) | Threat modeling, SIEM, pen testing, SBOM, bug bounty, PAM |
| [`11-deployment-architecture.md`](04-infrastructure-and-cloud/11-deployment-architecture.md) | Environment landscape, artifact promotion, production gates, multi-region deployment |
| [`12-identity-management.md`](04-infrastructure-and-cloud/12-identity-management.md) | Identity provider architecture, OAuth2 flows, OIDC, token lifecycle |
| [`13-authorization-patterns.md`](04-infrastructure-and-cloud/13-authorization-patterns.md) | RBAC vs ABAC framework, role hierarchy, OPA/Cedar policy engines |
| [`14-secrets-management.md`](04-infrastructure-and-cloud/14-secrets-management.md) | Secret classification, rotation policies, injection patterns |
| [`15-certificate-lifecycle.md`](04-infrastructure-and-cloud/15-certificate-lifecycle.md) | Certificate types, automated renewal, mTLS, PKI hierarchy |
| [`16-compliance-engineering.md`](04-infrastructure-and-cloud/16-compliance-engineering.md) | Compliance as code, automated evidence collection, continuous compliance |
| [`17-zero-trust-networking.md`](04-infrastructure-and-cloud/17-zero-trust-networking.md) | Zero-trust principles, microsegmentation, mTLS, identity-based access |
| [`18-agent-data-governance.md`](04-infrastructure-and-cloud/18-agent-data-governance.md) | Agent PII access restrictions, compliance audit trail, dev-time governance |
| [`19-service-mesh.md`](04-infrastructure-and-cloud/19-service-mesh.md) | Service mesh patterns, traffic management, observability integration |
| [`20-serverless-patterns.md`](04-infrastructure-and-cloud/20-serverless-patterns.md) | Serverless and FaaS patterns, cold start mitigation, guardrails |
| [`21-multi-cloud-strategy.md`](04-infrastructure-and-cloud/21-multi-cloud-strategy.md) | Cloud-exit planning, portability, vendor lock-in mitigation |
| [`22-self-service-infra.md`](04-infrastructure-and-cloud/22-self-service-infra.md) | Self-service infrastructure provisioning patterns |
| [`23-infra-testing.md`](04-infrastructure-and-cloud/23-infra-testing.md) | IaC testing, policy-as-code, drift detection |
| [`24-cdn-edge-caching.md`](04-infrastructure-and-cloud/24-cdn-edge-caching.md) | CDN and edge caching strategy, multi-CDN patterns |
| [`25-kubernetes-operations.md`](04-infrastructure-and-cloud/25-kubernetes-operations.md) | Kubernetes cluster operations and namespace strategy |
| [`26-search-infrastructure.md`](04-infrastructure-and-cloud/26-search-infrastructure.md) | Search infrastructure patterns and OpenSearch operations |

</details>

<details>
<summary><b>🚨 05 - Operational Excellence</b> &nbsp;·&nbsp; <i>When things break - and they will</i></summary>

<br/>

Observability, resilience, incidents, chaos engineering, and what to do when things break.

| File | What You'll Learn |
|------|-------------------|
| [`01-observability-standards.md`](05-operational-excellence/01-observability-standards.md) | Logging, metrics, tracing, SLOs, error budgets, alert hygiene |
| [`02-observability-in-practice.md`](05-operational-excellence/02-observability-in-practice.md) | Step-by-step: structured logging, correlation IDs, Prometheus, tracing |
| [`03-resilience-patterns.md`](05-operational-excellence/03-resilience-patterns.md) | Circuit breaker, retry, timeout, bulkhead - with worked examples |
| [`04-incident-management.md`](05-operational-excellence/04-incident-management.md) | Response playbook, on-call structure, PIR template, change management |
| [`05-load-shedding.md`](05-operational-excellence/05-load-shedding.md) | Priority tiers, backpressure, graceful degradation, auto-remediation |
| [`06-chaos-engineering.md`](05-operational-excellence/06-chaos-engineering.md) | Fault injection, game days, resilience validation |
| [`07-disaster-recovery-playbook.md`](05-operational-excellence/07-disaster-recovery-playbook.md) | Region failover, failback, DNS strategy, rollback SLAs |
| [`08-testing-in-production.md`](05-operational-excellence/08-testing-in-production.md) | Synthetic monitoring, traffic mirroring, dark launches |
| [`09-debugging-guide.md`](05-operational-excellence/09-debugging-guide.md) | How to debug locally, in staging, and in production - with real queries |
| [`10-slo-framework.md`](05-operational-excellence/10-slo-framework.md) | SLO framework, SLI types, error budget, burn rate alerts |
| [`11-service-scorecards.md`](05-operational-excellence/11-service-scorecards.md) | Service scorecards, scoring dimensions, readiness gates |
| [`12-alerting-design.md`](05-operational-excellence/12-alerting-design.md) | Alert severity levels, routing rules, fatigue prevention |
| [`13-incident-severity.md`](05-operational-excellence/13-incident-severity.md) | SEV1-SEV4 definitions, escalation paths, communication protocols |
| [`14-on-call-standards.md`](05-operational-excellence/14-on-call-standards.md) | Rotation design, compensation, handoff, escalation SLAs |
| [`15-pir-process.md`](05-operational-excellence/15-pir-process.md) | PIR template, blameless retrospective, action tracking |
| [`16-external-comms.md`](05-operational-excellence/16-external-comms.md) | Status page, customer notifications, stakeholder matrix |
| [`17-production-readiness.md`](05-operational-excellence/17-production-readiness.md) | Pre-launch gates, observability, security, load testing |
| [`18-health-checks.md`](05-operational-excellence/18-health-checks.md) | Liveness vs readiness probes, circuit breaker integration |
| [`19-backup-testing.md`](05-operational-excellence/19-backup-testing.md) | Backup types, testing cadence, RTO/RPO validation |
| [`20-logging-taxonomy.md`](05-operational-excellence/20-logging-taxonomy.md) | Log levels, structured logging standards, PII redaction |
| [`21-runbook-standards.md`](05-operational-excellence/21-runbook-standards.md) | Runbook authoring standards, template, review process |
| [`22-feature-flags.md`](05-operational-excellence/22-feature-flags.md) | Flag types, lifecycle management, stale flag cleanup |
| [`23-release-management.md`](05-operational-excellence/23-release-management.md) | SemVer, release trains, changelog, rollback procedures |
| [`24-review-engineering.md`](05-operational-excellence/24-review-engineering.md) | Review standards for validating agent-generated output |
| [`25-agent-observability.md`](05-operational-excellence/25-agent-observability.md) | Agent observability, LLM tracing, token monitoring |
| [`26-agent-operational-safety.md`](05-operational-excellence/26-agent-operational-safety.md) | Agent deployment gates, rollback, chaos testing for agents |

</details>

<details>
<summary><b>🛠️ 06 - Developer Guides</b> &nbsp;·&nbsp; <i>The tabs you keep open while coding</i></summary>

<br/>

The guides you keep open while writing code.

| File | What You'll Learn |
|------|-------------------|
| [`01-developer-experience.md`](06-developer-guides/01-developer-experience.md) | Local setup, onboarding milestones, buddy program, documentation standards |
| [`02-golden-path.md`](06-developer-guides/02-golden-path.md) | Scaffold to production in one walkthrough |
| [`03-database-migrations.md`](06-developer-guides/03-database-migrations.md) | Automated schema migrations, expand-contract, large table strategies |
| [`04-kafka-patterns.md`](06-developer-guides/04-kafka-patterns.md) | Producers, consumers, DLQ, idempotency, topic creation, DLQ replay |
| [`05-data-platform.md`](06-developer-guides/05-data-platform.md) | CDC, data warehouse conventions, pipeline orchestration, data access controls |
| [`06-multi-region-patterns.md`](06-developer-guides/06-multi-region-patterns.md) | Region config, i18n, localization, UX writing |
| [`07-distributed-locking.md`](06-developer-guides/07-distributed-locking.md) | Optimistic locking, Redis locks, preventing double-assignment |
| [`08-cache-patterns.md`](06-developer-guides/08-cache-patterns.md) | Cache-aside, TTL, event-driven invalidation, in-process caching, stampede prevention |
| [`09-data-governance.md`](06-developer-guides/09-data-governance.md) | Data stewards, quality SLAs, retention matrices, analytics contracts |
| [`10-local-development.md`](06-developer-guides/10-local-development.md) | Docker Compose, database seeding, your first PR |
| [`11-database-selection.md`](06-developer-guides/11-database-selection.md) | Database selection, polyglot persistence, decision matrix |
| [`12-data-migration-patterns.md`](06-developer-guides/12-data-migration-patterns.md) | Schema evolution, zero-downtime migration strategies |
| [`13-onboarding-automation.md`](06-developer-guides/13-onboarding-automation.md) | Developer onboarding automation and knowledge base |
| [`14-sdk-versioning.md`](06-developer-guides/14-sdk-versioning.md) | SDK and client library versioning standards |
| [`15-api-documentation.md`](06-developer-guides/15-api-documentation.md) | API documentation and developer portal standards |
| [`16-object-storage.md`](06-developer-guides/16-object-storage.md) | Object and file storage patterns |
| [`17-database-performance.md`](06-developer-guides/17-database-performance.md) | Database performance tuning and connection management |
| [`18-i18n-localization.md`](06-developer-guides/18-i18n-localization.md) | Internationalization and localization engineering |
| [`19-api-mocking.md`](06-developer-guides/19-api-mocking.md) | API mocking and service virtualization standards |
| [`20-agentic-developer-guides.md`](06-developer-guides/20-agentic-developer-guides.md) | Developer guides reframed for agentic workflow |
| [`21-data-lakehouse.md`](06-developer-guides/21-data-lakehouse.md) | Data lakehouse standards, ETL vs ELT guide |

</details>

<details>
<summary><b>🤝 07 - Ways of Working</b> &nbsp;·&nbsp; <i>Teams, hiring, growth, and how we decide</i></summary>

<br/>

Team structure, decision-making, hiring, career growth, and knowledge sharing.

| File | What You'll Learn |
|------|-------------------|
| [`01-team-topology.md`](07-ways-of-working/01-team-topology.md) | Stream-aligned teams, platform team, inner source, tech debt registry |
| [`02-engineering-ladder.md`](07-ways-of-working/02-engineering-ladder.md) | Engineer I to Principal - expectations at every level |
| [`03-open-source-policy.md`](07-ways-of-working/03-open-source-policy.md) | License compliance, contribution guidelines |
| [`04-product-engineering.md`](07-ways-of-working/04-product-engineering.md) | PM-engineering collaboration, discovery, rituals, user stories |
| [`05-rfc-process.md`](07-ways-of-working/05-rfc-process.md) | How to propose cross-cutting changes |
| [`06-technology-radar.md`](07-ways-of-working/06-technology-radar.md) | How we evaluate and adopt new technology |
| [`07-hiring-standards.md`](07-ways-of-working/07-hiring-standards.md) | Interview loop, rubrics, bar-raiser, bias mitigation |
| [`08-knowledge-sharing.md`](07-ways-of-working/08-knowledge-sharing.md) | Guilds, tech talks, architecture clinics |
| [`09-engineering-management.md`](07-ways-of-working/09-engineering-management.md) | 1:1s, reviews, team health, headcount, career development |
| [`10-product-operations.md`](07-ways-of-working/10-product-operations.md) | Roadmap, OKRs, launch management, customer feedback loops |
| [`11-engineering-culture.md`](07-ways-of-working/11-engineering-culture.md) | Cultural values, blameless postmortems, innovation time, knowledge sharing |
| [`12-document-templates.md`](07-ways-of-working/12-document-templates.md) | Standalone ADR, RFC, PIR, PRD, and design doc templates |
| [`13-technical-writing.md`](07-ways-of-working/13-technical-writing.md) | Internal technical documentation and writing guide |
| [`14-async-culture.md`](07-ways-of-working/14-async-culture.md) | Engineering communication norms, async-first culture |
| [`15-agentic-ways-of-working.md`](07-ways-of-working/15-agentic-ways-of-working.md) | Ways of working for agentic organizations |

</details>

<details>
<summary><b>📈 08 - Program</b> &nbsp;·&nbsp; <i>The honest scorecard - where we are and where we're headed</i></summary>

<br/>

Maturity assessment, migration roadmap, metrics, and vendor management.

| File | What You'll Learn |
|------|-------------------|
| [`01-maturity-model.md`](08-program/01-maturity-model.md) | Self-assessment rubric - L0 to L4 across 11 dimensions (Dimension 11: ML Platform Maturity) |
| [`02-migration-roadmap.md`](08-program/02-migration-roadmap.md) | Phased migration program, DORA targets, risk register |
| [`03-vendor-assessment.md`](08-program/03-vendor-assessment.md) | AWS lock-in analysis, portability, exit costs |
| [`04-engineering-metrics.md`](08-program/04-engineering-metrics.md) | What we measure, what we don't, and why |
| [`05-vendor-intake.md`](08-program/05-vendor-intake.md) | How to evaluate and onboard new SaaS tools |
| [`06-adoption-guide.md`](08-program/06-adoption-guide.md) | Scale-based adoption guide for 10, 100, and 1000+ engineers |
| [`07-engineering-kpis.md`](08-program/07-engineering-kpis.md) | DORA metrics, platform adoption, developer satisfaction |
| [`08-agentic-maturity.md`](08-program/08-agentic-maturity.md) | Agentic maturity model for AI-native organizations |
| [`09-idp-architecture.md`](08-program/09-idp-architecture.md) | Internal developer platform architecture and golden path |
| [`10-tech-evaluation.md`](08-program/10-tech-evaluation.md) | Technology evaluation framework and scoring rubric |
| [`11-tech-debt.md`](08-program/11-tech-debt.md) | Technical debt tracking, quantification, and remediation |
| [`12-cost-allocation.md`](08-program/12-cost-allocation.md) | Cost allocation and chargeback model |
| [`13-change-management.md`](08-program/13-change-management.md) | Change management for platform-wide changes |
| [`14-okr-framework.md`](08-program/14-okr-framework.md) | Engineering OKR framework and goal-setting |
| [`15-budget-allocation.md`](08-program/15-budget-allocation.md) | Engineering budget and capacity planning |
| [`16-architecture-governance.md`](08-program/16-architecture-governance.md) | Architecture governance, ADR process, review cadence |
| [`17-build-vs-buy.md`](08-program/17-build-vs-buy.md) | Build vs buy vs open-source decision framework |
| [`18-vendor-mcp-compatibility.md`](08-program/18-vendor-mcp-compatibility.md) | Vendor MCP and agent compatibility assessment |

</details>

<details>
<summary><b>📱 09 - Mobile & Frontend</b> &nbsp;·&nbsp; <i>Apps, browsers, pixels - the user-facing layer</i></summary>

<br/>

Everything for the engineers building client applications.

| File | What You'll Learn |
|------|-------------------|
| [`01-mobile-standards.md`](09-mobile-and-frontend/01-mobile-standards.md) | Offline-first, push, performance budgets, auth lifecycle, cert pinning |
| [`02-web-frontend-standards.md`](09-mobile-and-frontend/02-web-frontend-standards.md) | SPA/SSR, design system, a11y, Core Web Vitals, LaunchDarkly React |
| [`03-react-native-guide.md`](09-mobile-and-frontend/03-react-native-guide.md) | Turbo Modules, navigation, Hermes, CodePush, versioning |
| [`04-android-standards.md`](09-mobile-and-frontend/04-android-standards.md) | Gradle, Compose, Kotlin coroutines, Hilt, WorkManager |
| [`05-ios-standards.md`](09-mobile-and-frontend/05-ios-standards.md) | SwiftUI, SPM, privacy manifest, extensions, dSYM |
| [`06-design-system.md`](09-mobile-and-frontend/06-design-system.md) | Design-dev handoff, tokens, motion, dark mode, data visualization |
| [`07-accessibility-standards.md`](09-mobile-and-frontend/07-accessibility-standards.md) | Accessibility standards for web and mobile (WCAG 2.2) |
| [`08-minimum-version-policy.md`](09-mobile-and-frontend/08-minimum-version-policy.md) | Mobile minimum version and forced upgrade strategy |
| [`09-agent-ui-constraints.md`](09-mobile-and-frontend/09-agent-ui-constraints.md) | Agent-generated UI constraints for design system compliance |

</details>

<details>
<summary><b>🤖 10 - AI/ML Platform</b> &nbsp;·&nbsp; <i>ML pipelines, model governance, and AI guardrails</i></summary>

<br/>

| File | What You'll Learn |
|------|-------------------|
| [`01-ml-platform.md`](10-ai-ml-platform/01-ml-platform.md) | Model serving, feature stores, training pipelines, GPU governance |
| [`02-ai-governance.md`](10-ai-ml-platform/02-ai-governance.md) | Bias auditing, LLM guardrails, red-teaming, generative AI patterns |
| [`03-model-selection.md`](10-ai-ml-platform/03-model-selection.md) | AI/ML model selection framework by task type and cost |

</details>

<details>
<summary><b>🗂️ 11 - Domain Catalog</b> &nbsp;·&nbsp; <i>Service-by-service deep dives with APIs, events, and SLOs</i></summary>

<br/>

Detailed documentation for every domain service - APIs, events, data models, SLOs, and failure modes.

| File | What You'll Learn |
|------|-------------------|
| [`01-order-service.md`](11-domain-catalog/01-order-service.md) | Order lifecycle - the central orchestrating domain |
| [`02-fulfillment-engine.md`](11-domain-catalog/02-fulfillment-engine.md) | Provider-order assignment and geospatial algorithms |
| [`03-pricing-service.md`](11-domain-catalog/03-pricing-service.md) | Price calculation, estimates, dynamic pricing integration |
| [`04-provider-profile.md`](11-domain-catalog/04-provider-profile.md) | Provider onboarding, documents, ratings, availability |
| [`05-customer-profile.md`](11-domain-catalog/05-customer-profile.md) | Customer account, preferences, payment methods |
| [`06-payment-service.md`](11-domain-catalog/06-payment-service.md) | Payments, settlements, wallets, refunds |
| [`07-notifications.md`](11-domain-catalog/07-notifications.md) | Push, SMS, email dispatch across all channels |
| [`08-geolocation-service.md`](11-domain-catalog/08-geolocation-service.md) | ETA calculation, route planning, geocoding |
| [`09-dynamic-pricing.md`](11-domain-catalog/09-dynamic-pricing.md) | Supply/demand signals, zone management, multipliers |
| [`10-fraud-engine.md`](11-domain-catalog/10-fraud-engine.md) | Real-time risk scoring, blocking rules, ML fallbacks |

</details>

<details>
<summary><b>🧬 12 - AI Engineering</b> &nbsp;·&nbsp; <i>Context engineering, AI-assisted workflows, and adoption metrics</i></summary>

<br/>

How we work with AI across the entire software development lifecycle, including guidance for agent-native organizations. Distinct from Section 10 (building AI products) - this section is about using AI to build all products.

| File | What You'll Learn |
|------|-------------------|
| [`01-context-engineering.md`](12-ai-engineering/01-context-engineering.md) | Context stack, AGENTS.md, cursor rules, architecture-as-context, knowledge retrieval, maintenance |
| [`02-ai-assisted-sdlc.md`](12-ai-engineering/02-ai-assisted-sdlc.md) | AI integration across discovery, design, development, testing, review, deployment, operations, documentation |
| [`03-ai-adoption-metrics.md`](12-ai-engineering/03-ai-adoption-metrics.md) | Adoption metrics, effectiveness metrics, quality guardrails, AI Engineering Maturity (L0-L4), AI Champions Guild |
| [`04-trust-tiered-autonomy.md`](12-ai-engineering/04-trust-tiered-autonomy.md) | Trust-tiered autonomy model for agent code changes, promotion criteria, guardrails |
| [`05-agent-security-model.md`](12-ai-engineering/05-agent-security-model.md) | Agent identity standards, least privilege, audit trails, threat model |
| [`06-ai-native-sdlc.md`](12-ai-engineering/06-ai-native-sdlc.md) | AI-native SDLC strategy based on Three Pillars |
| [`07-prompt-engineering.md`](12-ai-engineering/07-prompt-engineering.md) | Prompt engineering and LLM integration standards |
| [`08-rag-patterns.md`](12-ai-engineering/08-rag-patterns.md) | RAG patterns, vector database standards, LLM architecture |
| [`09-mcp-tool-use.md`](12-ai-engineering/09-mcp-tool-use.md) | MCP and tool-use integration standards |
| [`10-multi-agent-patterns.md`](12-ai-engineering/10-multi-agent-patterns.md) | AI agent orchestration and multi-agent patterns |
| [`11-ai-finops.md`](12-ai-engineering/11-ai-finops.md) | AI cost optimization and FinOps for AI workloads |
| [`12-sdlc-agent-blueprint.md`](12-ai-engineering/12-sdlc-agent-blueprint.md) | SDLC agent blueprint and orchestration |

</details>

<details>
<summary><b>🔍 13 - Technology Selection</b> &nbsp;·&nbsp; <i>Structured decision guides for the entire stack</i></summary>

<br/>

| Document | Description |
|----------|-------------|
| [`01-ai-coding-assistants.md`](13-technology-selection/01-ai-coding-assistants.md) | AI coding assistants |
| [`02-ai-code-review.md`](13-technology-selection/02-ai-code-review.md) | AI code review tools |
| [`03-llm-providers.md`](13-technology-selection/03-llm-providers.md) | LLM providers |
| [`04-backend-languages.md`](13-technology-selection/04-backend-languages.md) | Backend languages |
| [`05-backend-frameworks.md`](13-technology-selection/05-backend-frameworks.md) | Backend frameworks |
| [`06-frontend-frameworks.md`](13-technology-selection/06-frontend-frameworks.md) | Frontend frameworks |
| [`07-mobile-development.md`](13-technology-selection/07-mobile-development.md) | Mobile development |
| [`08-relational-databases.md`](13-technology-selection/08-relational-databases.md) | Relational databases |
| [`09-nosql-databases.md`](13-technology-selection/09-nosql-databases.md) | NoSQL databases |
| [`10-cache-stores.md`](13-technology-selection/10-cache-stores.md) | Cache stores |
| [`11-vector-databases.md`](13-technology-selection/11-vector-databases.md) | Vector databases |
| [`12-search-engines.md`](13-technology-selection/12-search-engines.md) | Search engines |
| [`13-cloud-providers.md`](13-technology-selection/13-cloud-providers.md) | Cloud providers |
| [`14-container-orchestration.md`](13-technology-selection/14-container-orchestration.md) | Container orchestration |
| [`15-iac-tools.md`](13-technology-selection/15-iac-tools.md) | IaC tools |
| [`16-api-gateways.md`](13-technology-selection/16-api-gateways.md) | API gateways |
| [`17-service-meshes.md`](13-technology-selection/17-service-meshes.md) | Service meshes |
| [`18-cdn-platforms.md`](13-technology-selection/18-cdn-platforms.md) | CDN platforms |
| [`19-cicd-platforms.md`](13-technology-selection/19-cicd-platforms.md) | CI/CD platforms |
| [`20-gitops-tools.md`](13-technology-selection/20-gitops-tools.md) | GitOps tools |
| [`21-feature-flags.md`](13-technology-selection/21-feature-flags.md) | Feature flags |
| [`22-artifact-registries.md`](13-technology-selection/22-artifact-registries.md) | Artifact registries |
| [`23-observability-platforms.md`](13-technology-selection/23-observability-platforms.md) | Observability platforms |
| [`24-incident-management.md`](13-technology-selection/24-incident-management.md) | Incident management |
| [`25-error-tracking.md`](13-technology-selection/25-error-tracking.md) | Error tracking |
| [`26-security-scanning.md`](13-technology-selection/26-security-scanning.md) | Security scanning |
| [`27-identity-providers.md`](13-technology-selection/27-identity-providers.md) | Identity providers |
| [`28-developer-portals.md`](13-technology-selection/28-developer-portals.md) | Developer portals |
| [`29-testing-frameworks.md`](13-technology-selection/29-testing-frameworks.md) | Testing frameworks |
| [`30-documentation-platforms.md`](13-technology-selection/30-documentation-platforms.md) | Documentation platforms |
| [`31-project-management.md`](13-technology-selection/31-project-management.md) | Project management |
| [`32-data-warehouses.md`](13-technology-selection/32-data-warehouses.md) | Data warehouses |
| [`33-data-pipelines.md`](13-technology-selection/33-data-pipelines.md) | Data pipelines |
| [`34-message-brokers.md`](13-technology-selection/34-message-brokers.md) | Message brokers |
| [`35-source-control.md`](13-technology-selection/35-source-control.md) | Source control |
| [`36-secret-management.md`](13-technology-selection/36-secret-management.md) | Secret management |
| [`37-cloud-dev-environments.md`](13-technology-selection/37-cloud-dev-environments.md) | Cloud dev environments |
| [`38-paas-platforms.md`](13-technology-selection/38-paas-platforms.md) | PaaS platforms |
| [`39-product-analytics.md`](13-technology-selection/39-product-analytics.md) | Product analytics |
| [`40-notification-platforms.md`](13-technology-selection/40-notification-platforms.md) | Notification platforms |
| [`41-chatops-platforms.md`](13-technology-selection/41-chatops-platforms.md) | ChatOps platforms |
| [`42-compliance-automation.md`](13-technology-selection/42-compliance-automation.md) | Compliance automation |
| [`43-cloud-cost-management.md`](13-technology-selection/43-cloud-cost-management.md) | Cloud cost management |
| [`44-backup-tools.md`](13-technology-selection/44-backup-tools.md) | Backup tools |
| [`45-monorepo-tooling.md`](13-technology-selection/45-monorepo-tooling.md) | Monorepo tooling |

</details>

<br/>

<div align="center"><sub><i>Every service in the diagram above, documented in detail. ↓</i></sub></div>

---

## 🔗 How Services Connect

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

> *Solid arrows = synchronous (gRPC) · Dashed arrows = asynchronous (Kafka events)*

---

## 🔧 Customize This Manifesto

This manifesto is **opinionated about principles, flexible about tooling**. The architectural patterns, operational standards, and engineering practices are universal. The specific technologies named throughout are our **reference implementation** - one proven instantiation of those principles.

| What's universal | What's a reference choice (substitute yours) |
|:-----------------|:---------------------------------------------|
| Structured JSON logging with correlation IDs | Logback, Pino, Serilog, Zap |
| Schema-validated async event streaming | Kafka, Pulsar, NATS, RabbitMQ |
| Managed relational database per service | PostgreSQL/Aurora, Cloud SQL, Azure SQL, PlanetScale |
| Infrastructure as code, no ClickOps | Terraform, Pulumi, CDK, Crossplane |
| Container orchestration with GitOps | EKS, GKE, AKS, self-managed Kubernetes |
| Backend framework with health checks and metrics | Spring Boot, Express/Fastify, ASP.NET, Go stdlib, Django/FastAPI |
| Feature flags for progressive rollout | LaunchDarkly, Unleash, Flagsmith, Split |
| Automated CI/CD with quality gates | GitHub Actions, GitLab CI, CircleCI, Jenkins |

> **Adopting this manifesto?** Search for specific tool names (e.g., "Spring Boot", "Aurora", "EKS") and replace them with your equivalents. The principles, patterns, and operational standards carry over unchanged.

The repo uses two company placeholders: **`{Company}`** (title case) in headings and prose, and **`{company}`** (lowercase) in URLs, package names, GitHub org paths, and other technical identifiers. Replace both with your organization when you fork or adapt this manifesto.

### For agent-native organizations

If you are building an organization where AI agents are first-class engineers - planning sprints, writing code, reviewing PRs, deploying services, and responding to incidents - this manifesto works as your shared operating system with minimal adaptation:

| Step | What to do |
|:-----|:-----------|
| **1. Feed the manifesto to your agents** | Index the manifesto into your agent orchestration layer (RAG, context window, or tool retrieval). Every agent-generated artifact should be checked against manifesto rules |
| **2. Wire AGENTS.md into every repo** | Each repository's `AGENTS.md` tells agents the local rules. Human engineers and AI agents read the same file. See [Context Engineering](./12-ai-engineering/01-context-engineering.md) |
| **3. Use contracts as agent interfaces** | API standards, event schemas, error catalogs, and naming conventions are not just documentation - they are the interface contract between your agents. Agent A publishes an event; Agent B consumes it. The manifesto defines the schema both follow |
| **4. Keep human-in-the-loop gates** | The manifesto marks where human judgment is required: architecture decisions, security trade-offs, production incident response escalations. Preserve these gates even in a fully agentic workflow |
| **5. Measure agent adoption** | Use the same [engineering metrics](./08-program/04-engineering-metrics.md) and [AI adoption metrics](./12-ai-engineering/03-ai-adoption-metrics.md) to track agent effectiveness as you would human effectiveness |

---

## 📏 How This Evolves

This isn't set in stone - it's set in Git. Here's how to shape it.

| | Aspect | Rule |
|:-:|--------|------|
| 👤 | **Owner** | Platform Engineering team |
| 🔀 | **Change it** | PR required - at least one Staff Engineer or Principal approval |
| 📋 | **Deviate from it** | Document your rationale in an ADR. No silent exceptions |
| 🗓️ | **Review cadence** | Quarterly with Tech Leads; full manifesto bi-annually |
| 💬 | **Disagree with it** | `#engineering-discussions` or Architecture Clinic, then open a PR |
| 📝 | **Contribute** | See [CONTRIBUTING.md](./CONTRIBUTING.md) for fork, customization, and adoption guides |

---

## ⚖️ Disclaimer

This manifesto provides engineering best practices and technical guidance. It is **not legal advice**. Sections discussing compliance frameworks (GDPR, PCI-DSS, SOC 2, ISO 27001) describe technical implementation patterns, not legal interpretations. Organizations must consult their own legal counsel to validate compliance posture. All compliance claims are illustrative and require organization-specific assessment.

---

<div align="center">

*Built with conviction by {Company} Platform Engineering.*

*The best engineering cultures don't just build great software.*<br/>
*They write down how - and then keep making it better.*

</div>
