# 📊 Executive Summary

![Status: Guidance](https://img.shields.io/badge/status-Guidance-orange?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 1. 📋 What This Is

The {Company} Platform Engineering Manifesto is a **single, opinionated source of truth** for how software is designed, built, deployed, and operated. It is **company-agnostic in spirit**: the principles apply to any organization; named tools are a reference implementation you can swap for your own stack. It is not a wiki of suggestions - it records **decisions already made** so teams spend less time re-debating and more time shipping. Changes flow through the same rigor as code: pull requests, review, and version history in Git.

---

## 2. 💼 Why This Matters

Treating engineering standards as a **managed asset** produces measurable business outcomes:

- **Shorter onboarding** - New engineers and partners ramp on one playbook instead of tribal knowledge scattered across teams.
- **Predictable quality** - Shared patterns for testing, security, and operations reduce defects and production incidents.
- **Faster time-to-market** - Golden paths and reusable platform capabilities cut duplicate work and approval cycles.
- **Lower risk** - Security, compliance, and resilience are embedded by default, not bolted on under audit pressure.

---

## 3. 🎯 Key Principles

The manifesto rests on **eight non-negotiables**. In business terms:

1. **Pave the golden path** - Make the approved approach the easiest path so adoption is pull, not push.
2. **Ship artifacts, not code** - The same tested build that leaves CI is what runs in production; no manual rebuilds between stages.
3. **Observable by default** - Every service must be diagnosable with logs, metrics, and traces from day one.
4. **Own your data, respect boundaries** - Each service owns its data; cross-cutting access without contracts is ruled out to contain blast radius and coupling.
5. **Everything in Git** - Infrastructure, configuration, runbooks, and decisions live in version control for auditability and repeatability.
6. **Automate the boring** - Pipelines and platforms handle repetition; people focus on product and architecture.
7. **Design for failure** - Dependencies will fail; systems must degrade gracefully with retries, limits, and clear recovery paths.
8. **Security is not a phase** - Security checks run in every pipeline and environment from the first commit, not only before release.

For the full technical framing, see [The Non-Negotiables](./README.md#-the-non-negotiables) in the root README.

---

## 4. 🗺️ Adoption Phases

Adoption is intentionally **phased** so reliability and delivery keep pace. At a high level:

| Phase | Focus | Outcome |
|-------|--------|---------|
| **Foundation** | Platform baseline, first golden-path service, core CI/CD and observability | One service can run end-to-end on the target platform with governed promotion to production |
| **Scaling** | Broad migration of services and teams onto shared patterns | Most workloads on standard build, deploy, and operate paths; fewer one-off stacks |
| **Optimization** | Cost, performance, maturity, and continuous improvement | Measured efficiency gains and a sustained internal product mindset for the platform |

Timelines, exit criteria, and risk management are spelled out in the [Migration Roadmap](./08-program/02-migration-roadmap.md). Treat that document as the operational program plan; this page is the executive orientation.

---

## 5. 🏢 Required Organizational Capabilities

Getting value from the manifesto is not only a documentation exercise. You typically need:

- **A platform or enabling team** - People accountable for the golden path, shared tooling, and developer experience.
- **Engineering leadership buy-in** - VPs and directors who treat standards as policy, fund migration work, and back enforcement in CI and architecture review.
- **CI/CD and infrastructure foundations** - Automated build, test, security scanning, and promotion; infrastructure as code; secrets and config management fit for regulated change.
- **Governance that rewards compliance** - Clear exceptions (for example ADRs), not silent drift; product teams should see standards as acceleration, not bureaucracy.

### For agent-native organizations

If your organization uses AI agents as first-class engineering participants - planning, coding, reviewing, deploying, and operating services alongside humans - you additionally need:

- **Agent identity and access** - Service accounts or OIDC identities for agents with the same RBAC and audit trail as human engineers.
- **Human-in-the-loop gates** - Explicit policy for which actions agents may perform autonomously (e.g. opening PRs, running CI) and which require human approval (e.g. merging to main, production deploys, architecture decisions).
- **Manifesto-as-context** - The manifesto indexed into your agent orchestration layer (RAG, context window, or tool retrieval) so every agent-generated artifact is checked against the same standards humans follow.
- **Agent effectiveness metrics** - The same DORA and engineering metrics applied to agent-driven work, supplemented by [AI adoption metrics](./12-ai-engineering/03-ai-adoption-metrics.md).

The manifesto's opinionated, machine-parseable structure (numbered files, explicit contracts, deterministic rules) makes it consumable by agents without adaptation. See [Agent-Native by Design](./README.md#-agent-native-by-design) in the root README.

---

## 6. ⚠️ Risks of Not Adopting

If teams continue without a shared playbook, common failure modes include:

- **Slower hiring and onboarding** - Every team reinvents conventions; new joiners face conflicting instructions.
- **Operational fragility** - Inconsistent observability, deployment, and failure handling make incidents harder to detect and resolve.
- **Security and compliance gaps** - Ad-hoc practices increase audit findings, breach risk, and emergency remediation cost.
- **Vendor and architectural lock-in** - Undocumented choices become expensive to change; duplicate platforms inflate run cost.
- **Reduced delivery velocity** - Integration pain, rework, and manual release steps compound as the product surface grows.

---

## 7. 🚀 Getting Started

- **For new engineers and day-one setup**, start with the [New Joiner Onboarding Guide](./ONBOARDING.md).
- **For program planning, phases, and migration mechanics**, use the [Migration Roadmap](./08-program/02-migration-roadmap.md).
- **For the full library of standards**, open the [Platform Manifesto README](./README.md).

Procurement and vendor conversations should align intake with [Vendor Intake](./08-program/05-vendor-intake.md) and related program documents once engineering has pointed you to the right sections.

---

## 8. 🤖 The Manifesto as Agent Context

This manifesto is not only a human reference - it is the shared operating system for AI agents across your engineering organization. When indexed into your agent orchestration layer (RAG, context window, or tool retrieval), it gives every agent the same standards, contracts, and guardrails that human engineers follow. Agent-generated code, architecture proposals, and operational decisions are checked against manifesto rules automatically.

The structured, opinionated format - numbered files, explicit contracts, deterministic rules - makes the manifesto machine-parseable without adaptation. For details on wiring the manifesto into your agent stack, see [Context Engineering](./12-ai-engineering/01-context-engineering.md). For the autonomy model governing what agents may do without human approval, see [Trust-Tiered Autonomy](./12-ai-engineering/04-trust-tiered-autonomy.md).

---
<div align="center">

🏠 [Back to root](./README.md)

</div>
