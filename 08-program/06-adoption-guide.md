# 🚀 Adoption Guide

![Status: Guidance](https://img.shields.io/badge/status-Guidance-orange?style=flat-square) ![Owner: CTO](https://img.shields.io/badge/owner-CTO-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Overview

Not every organization needs the full manifesto from day one. Adopting platform engineering is a function of scale - what makes sense for a 5-person startup will crush a 10-person team under process weight, and what works for 10 engineers will collapse into chaos at 200.

This guide maps manifesto adoption to three scale phases. Each phase identifies the minimum viable platform for that stage, what to adopt first, and the anti-patterns that signal you have outgrown your current phase.

> **Rule:** Adopt the platform that matches your current scale, not the scale you aspire to. Premature platform engineering is as dangerous as having no platform at all.

Cross-references: [Maturity Model](./01-maturity-model.md) for self-assessment, [Migration Roadmap](./02-migration-roadmap.md) for phased execution.

---

## 🌱 2. Phase 1 - Startup (1-10 Engineers)

At this scale, every engineer knows the full codebase. The goal is to establish habits that will survive growth, without slowing down the speed advantage of a small team.

### What to adopt first

| Priority | Manifesto Area | Why Now |
|:---------|:---------------|:--------|
| **1** | [Git Workflow](../03-engineering-practices/05-git-workflow.md) | Trunk-based development prevents merge hell before it starts |
| **2** | [Coding Standards](../03-engineering-practices/04-coding-standards.md) | Naming and error handling conventions are cheap to adopt early, expensive to retrofit |
| **3** | [Repository Standards](../01-platform-standards/03-repository-standards.md) | README templates, PR templates, and branch protection set the baseline |
| **4** | [Testing Pyramid](../03-engineering-practices/01-testing-pyramid.md) | Unit and integration tests only - skip contract and E2E until you have multiple services |
| **5** | [CI Practices](../03-engineering-practices/02-ci-practices.md) | One pipeline per repo with lint, test, and build gates |

### Adoption checklist

- [ ] Single CI pipeline with lint, test, and build gates
- [ ] Trunk-based development with short-lived branches
- [ ] Consistent naming conventions across all code
- [ ] README template in every repository
- [ ] Basic structured logging (JSON format, correlation IDs)
- [ ] One shared on-call rotation (everyone rotates)

### What to skip

Do not adopt service mesh, multi-region, capacity planning, or formal architecture reviews at this scale. You likely have one or two services. Keep it simple.

---

## 📈 3. Phase 2 - Growth (10-100 Engineers)

Multiple teams now own different services. Tribal knowledge no longer scales. This is the phase where the absence of a platform becomes visibly painful - inconsistent deployments, duplicated infrastructure work, and onboarding that takes weeks instead of days.

### What to adopt next

| Priority | Manifesto Area | Why Now |
|:---------|:---------------|:--------|
| **1** | [Service Catalog](../01-platform-standards/04-service-catalog.md) | You need to know what exists before you can standardize it |
| **2** | [Golden Path](../06-developer-guides/02-golden-path.md) | New services must start from a template, not from scratch |
| **3** | [Observability Standards](../05-operational-excellence/01-observability-standards.md) | SLOs, dashboards, and alerting become mandatory when you cannot hold the full system in your head |
| **4** | [CD Practices](../03-engineering-practices/03-cd-practices.md) | GitOps and canary deployments replace ad-hoc deploy scripts |
| **5** | [API Standards](../02-architecture-and-api/02-api-standards.md) | Consistent error formats, versioning, and pagination across all service boundaries |
| **6** | [Incident Management](../05-operational-excellence/04-incident-management.md) | Formal on-call rotations, severity levels, and postmortems |
| **7** | [Team Topology](../07-ways-of-working/01-team-topology.md) | Stream-aligned teams with clear ownership boundaries |

### Adoption checklist

- [ ] Service catalog listing all services with owners
- [ ] Golden-path template producing compliant services in under 30 minutes
- [ ] SLOs defined for all customer-facing services
- [ ] Structured incident management with postmortems
- [ ] API standards enforced in CI (linting, contract tests)
- [ ] Formal on-call rotations per team
- [ ] Onboarding guide with week 1-4 milestones
- [ ] At least one platform engineer or enabling team member
- [ ] Feature flags for progressive rollout

### What to skip

Formal FinOps, multi-tenancy, chaos engineering, and a full technology radar are premature at this stage. Invest in them when operational costs or reliability requirements justify the overhead.

---

## 🏢 4. Phase 3 - Scale (100-1000+ Engineers)

At this scale, the platform is a product. Multiple teams depend on shared infrastructure, and inconsistency in any dimension - security, observability, deployment - creates compounding risk. This is where you adopt the full manifesto.

### What to adopt

Everything not yet adopted, with emphasis on:

| Priority | Manifesto Area | Why Now |
|:---------|:---------------|:--------|
| **1** | [Maturity Model](./01-maturity-model.md) | Systematic self-assessment across all 11 dimensions |
| **2** | [FinOps](../04-infrastructure-and-cloud/05-finops.md) | Cloud spend is now a material line item requiring allocation and optimization |
| **3** | [Security Operations](../04-infrastructure-and-cloud/10-security-operations.md) | Threat modeling, SIEM, pen testing, and SBOM management |
| **4** | [Chaos Engineering](../05-operational-excellence/06-chaos-engineering.md) | Game days and fault injection to validate resilience at scale |
| **5** | [Engineering Metrics](./04-engineering-metrics.md) | DORA metrics, developer satisfaction, and platform adoption tracking |
| **6** | [Multi-Region Patterns](../06-developer-guides/06-multi-region-patterns.md) | Availability requirements now justify multi-region investment |
| **7** | [AI Engineering](../12-ai-engineering/01-context-engineering.md) | Context engineering and AI-assisted workflows across the organization |

### Adoption checklist

- [ ] All teams at maturity Level 3 or above across core dimensions
- [ ] Dedicated platform team operating as an internal product team
- [ ] FinOps practice with per-team cost allocation and optimization targets
- [ ] Formal technology radar with adopt, trial, assess, and hold rings
- [ ] Architecture review board or async RFC process for cross-cutting changes
- [ ] Chaos engineering game days at least quarterly
- [ ] DORA metrics tracked and published organization-wide
- [ ] AI context engineering wired into all repositories
- [ ] Vendor intake process for all new SaaS and tooling decisions
- [ ] Disaster recovery playbook tested at least annually

---

## ⚠️ 5. Common Anti-Patterns

| Anti-Pattern | What It Looks Like | Fix |
|:-------------|:-------------------|:----|
| **Premature platform** | 5-person team building a service mesh and internal developer portal | Strip back to Phase 1 essentials. Build platform when the pain is real, not anticipated |
| **Standards without enforcement** | Manifesto exists but CI does not enforce it. Teams drift silently | Add automated gates: lint rules, contract tests, mandatory PR templates |
| **Platform without product thinking** | Platform team ships tools nobody asked for. Adoption is mandated, not pulled | Run developer surveys, track adoption metrics, treat platform as an internal product |
| **Big-bang adoption** | Attempt to adopt all 12 sections simultaneously | Follow the phased approach. Adopt incrementally, prove value, then expand |
| **Copy-paste without adaptation** | Fork this manifesto and adopt it verbatim without adjusting to your context | Customize tool choices, skip irrelevant sections, and adapt examples to your domain |
| **Skipping foundations** | Jump to chaos engineering and multi-region without basic CI/CD and observability | Audit against Phase 1 and Phase 2 checklists before investing in Phase 3 capabilities |

---

## 📊 6. How to Measure Adoption Progress

Use the [Maturity Model](./01-maturity-model.md) for detailed self-assessment. For a quick adoption health check:

| Signal | Healthy | Unhealthy |
|:-------|:--------|:----------|
| **New service time** | < 30 minutes from template to running in staging | Days of manual setup and configuration |
| **Onboarding time** | First PR merged within week 1 | Takes 2+ weeks to get a dev environment running |
| **Incident response** | Structured postmortems, SLOs tracked, alerts actionable | Ad-hoc debugging, no SLOs, alert fatigue |
| **Cross-team consistency** | Services use shared patterns, errors, and observability | Every team has its own conventions and tooling |
| **Platform adoption** | Teams pull from golden path voluntarily | Standards are mandated but ignored in practice |

See also: [Engineering Metrics](./04-engineering-metrics.md) for quantitative tracking and [Migration Roadmap](./02-migration-roadmap.md) for phased execution plans.

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
