# 🧭 Engineering Principles

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Overview

These are the engineering principles that guide every technical decision at {Company}. They are not aspirations - they are constraints. Every standard, practice, and tool choice in this manifesto traces back to one or more of these principles.

In the AI era, principles matter more than ever. When AI agents generate code, review pull requests, and operate production services alongside humans, explicit principles become the shared contract that keeps both human and machine output consistent, safe, and aligned with organizational goals.

> **How to use this document:** Before making a technical decision - choosing a tool, designing an API, structuring a service - check it against these principles. If your decision conflicts with a principle, either reconsider the decision or propose a principle amendment via PR.

---

## 📜 2. The Principles

### 🏆 2.1 Own What You Ship

**You build it, you run it, you fix it at 3 AM.** Ownership does not end at the pull request. The team that writes the code owns its deployment, monitoring, incident response, and lifecycle. There are no handoffs to a separate operations team.

**In an agent-native organization:** AI agents that generate or modify code inherit the ownership context of the team they operate within. Agent-authored code follows the same on-call, monitoring, and incident response rules as human-authored code. The team - not the agent - is accountable for production behavior.

### 🤖 2.2 Agent-Native by Default

**Design every artifact for both human and machine consumption.** APIs, documentation, schemas, configuration, and runbooks must be structured so AI agents can parse, reason about, and act on them without special tooling. If an agent cannot read it, it is not well-structured enough for humans either.

**In an agent-native organization:** Every repository includes an `AGENTS.md` file. Every API has a machine-readable schema. Every runbook follows a structured template. Agents are first-class consumers of your engineering knowledge - not afterthoughts bolted on later.

### 📐 2.3 Specification Over Implementation

**Define the contract before you write the code.** API schemas, event contracts, error catalogs, and interface definitions come first. Implementation follows the spec, not the other way around. When the spec and the code disagree, the spec wins until you change it through the proper process.

**In an agent-native organization:** Specs become the primary input for agent-generated code. An agent given a well-defined OpenAPI spec, Avro schema, or proto definition can generate compliant implementations autonomously. Investing in spec quality directly multiplies agent productivity.

### 🔄 2.4 Automate the Boring Stuff

**Pipelines handle repetition. Humans and agents handle architecture and product problems.** If a task is repetitive, error-prone, and requires no judgment, automate it. Code formatting, dependency updates, security scanning, deployment mechanics, and compliance checks are automation targets - not human (or agent) work.

**In an agent-native organization:** Automation is the foundation that frees agents to work on higher-value tasks. Agents should not be formatting code or running linters manually - CI pipelines do that. Agents focus on design, logic, and the creative work that requires reasoning.

### 👁️ 2.5 Observable by Default

**If you cannot see it, you cannot fix it.** Every service ships with structured logging, metrics, distributed tracing, and health checks from day one. Observability is not a retrofit or a nice-to-have - it is a launch requirement. Silent failures are the most expensive kind.

**In an agent-native organization:** Observable systems enable agent-driven operations. AI agents can parse structured logs, query metrics APIs, and trace requests across services to diagnose issues - but only if telemetry is consistent, structured, and machine-readable. Unstructured log messages are opaque to agents.

### 🛤️ 2.6 Pave the Golden Path

**Make the right way the easy way.** Teams should feel pulled toward standards, not pushed. Provide scaffold templates, starter kits, CI/CD pipelines, and sensible defaults so that building a compliant service is faster than building a non-compliant one. Friction is the enemy of adoption.

**In an agent-native organization:** The golden path becomes an agent's default behavior. When an agent scaffolds a new service, it should produce a standards-compliant project on the first attempt - correct structure, proper health checks, right logging format, connected CI pipeline. The golden path templates are an agent's most important context.

### 💥 2.7 Design for Failure

**Every dependency will fail. Plan for it.** Circuit breakers, retries with backoff, timeouts, bulkheads, fallback responses, and graceful degradation are not optional. Services must degrade gracefully under partial failure, not cascade into system-wide outages.

**In an agent-native organization:** Agents must be taught failure modes as first-class design considerations. When an agent designs or implements a service integration, it must include resilience patterns by default - not as an afterthought suggested during code review. The manifesto's resilience patterns are part of every agent's context.

### 🔒 2.8 Security Is Everyone's Job

**Security runs in every pipeline, in every environment, from the first commit.** It is not a phase at the end or a team you hand things off to. Dependency scanning, secret detection, SAST, DAST, and least-privilege access are embedded in the development workflow. Every engineer - and every agent - is a security practitioner.

**In an agent-native organization:** AI agents must be constrained by the same security boundaries as human engineers. Agents never have broader access than the team they serve. Agent-generated code is scanned with the same SAST/DAST tooling. Agents do not store, log, or expose secrets - and the pipeline enforces this automatically.

### 📦 2.9 Ship Small, Learn Fast

**Small increments, tight feedback loops.** Ship the smallest meaningful change, get it in front of users (or production traffic), and learn from the result. Large, long-lived branches and big-bang releases are symptoms of a broken feedback loop. Feature flags, canary deployments, and A/B tests make small-batch shipping safe.

**In an agent-native organization:** Agents excel at small, well-scoped changes. Structure work so agents handle focused tasks - a single API endpoint, one migration script, a specific bug fix - rather than sprawling multi-service features. Small scope plus tight feedback is where agent-generated code is most reliable.

### 📖 2.10 Everything in Git

**If it is not in Git, it does not exist.** Infrastructure, configuration, runbooks, ADRs, diagrams, and deployment manifests live in version-controlled repositories. No ClickOps. No tribal knowledge. No "ask Sarah, she knows how that works." Git is the single source of truth.

**In an agent-native organization:** Git-as-source-of-truth is what makes agents viable at all. Agents cannot access tribal knowledge locked in someone's head or a Confluence page buried three clicks deep. When everything is in Git - structured, versioned, and discoverable - agents can find, read, and apply organizational knowledge autonomously.

---

## 🗺️ 3. Principles to Manifesto Map

Every principle is implemented by specific documents across the manifesto. Use this table to find the standards that put each principle into practice.

| Principle | Implementing Documents |
|-----------|----------------------|
| **Own What You Ship** | [Team Topology](../07-ways-of-working/01-team-topology.md), [Incident Management](../05-operational-excellence/04-incident-management.md), [Service Catalog](../01-platform-standards/04-service-catalog.md) |
| **Agent-Native by Default** | [Context Engineering](../12-ai-engineering/01-context-engineering.md), [AI-Assisted SDLC](../12-ai-engineering/02-ai-assisted-sdlc.md), [Repository Standards](../01-platform-standards/03-repository-standards.md) |
| **Specification Over Implementation** | [API Standards](../02-architecture-and-api/02-api-standards.md), [gRPC Standards](../02-architecture-and-api/05-grpc-standards.md), [Event Schema Evolution](../02-architecture-and-api/08-event-schema-evolution.md), [Error Catalog](../02-architecture-and-api/09-error-catalog.md) |
| **Automate the Boring Stuff** | [CI Practices](./02-ci-practices.md), [CD Practices](./03-cd-practices.md), [Frontend CI/CD](./10-frontend-ci-cd.md) |
| **Observable by Default** | [Observability Standards](../05-operational-excellence/01-observability-standards.md), [Observability in Practice](../05-operational-excellence/02-observability-in-practice.md), [Backend Framework Standards](./09-backend-framework-standards.md) |
| **Pave the Golden Path** | [Golden Path](../06-developer-guides/02-golden-path.md), [Developer Experience](../06-developer-guides/01-developer-experience.md), [Local Development](../06-developer-guides/10-local-development.md) |
| **Design for Failure** | [Resilience Patterns](../05-operational-excellence/03-resilience-patterns.md), [Load Shedding](../05-operational-excellence/05-load-shedding.md), [Chaos Engineering](../05-operational-excellence/06-chaos-engineering.md), [Disaster Recovery](../05-operational-excellence/07-disaster-recovery-playbook.md) |
| **Security Is Everyone's Job** | [Security](../04-infrastructure-and-cloud/03-security.md), [Security Operations](../04-infrastructure-and-cloud/10-security-operations.md), [Privacy Engineering](../04-infrastructure-and-cloud/08-privacy-engineering.md) |
| **Ship Small, Learn Fast** | [Git Workflow](./05-git-workflow.md), [A/B Testing](./07-ab-testing.md), [Testing in Production](../05-operational-excellence/08-testing-in-production.md), [CD Practices](./03-cd-practices.md) |
| **Everything in Git** | [Repository Standards](../01-platform-standards/03-repository-standards.md), [Configuration Management](../04-infrastructure-and-cloud/04-configuration-management.md), [Cloud Architecture](../04-infrastructure-and-cloud/01-cloud-architecture.md) |

---

## 🔗 4. Relationship to the Non-Negotiables

The root [README](../README.md) lists eight non-negotiables - the compressed, single-sentence version of the platform's beliefs. This document expands on them and adds two principles critical to the AI era:

| Non-Negotiable (Root README) | Corresponding Principle Here |
|------------------------------|------------------------------|
| Pave the golden path | 2.6 Pave the Golden Path |
| Ship artifacts, not code | 2.9 Ship Small, Learn Fast (artifact immutability is a practice of this principle) |
| Observable by default | 2.5 Observable by Default |
| Own your data, respect boundaries | 2.1 Own What You Ship (data ownership follows service ownership) |
| Everything in Git | 2.10 Everything in Git |
| Automate the boring | 2.4 Automate the Boring Stuff |
| Design for failure | 2.7 Design for Failure |
| Security is not a phase | 2.8 Security Is Everyone's Job |
| *New for AI era* | 2.2 Agent-Native by Default |
| *New for AI era* | 2.3 Specification Over Implementation |

---

## 🧪 5. Applying Principles in Practice

### Decision framework

When two principles appear to conflict, use this priority order:

1. **Security Is Everyone's Job** - security constraints always win
2. **Design for Failure** - resilience is non-negotiable
3. **Own What You Ship** - clear ownership resolves ambiguity
4. **Observable by Default** - you cannot make good decisions without visibility
5. **Specification Over Implementation** - contracts prevent accidental coupling
6. **Everything in Git** - traceability enables everything else
7. **Ship Small, Learn Fast** - smaller scope reduces risk
8. **Pave the Golden Path** - consistency enables velocity
9. **Automate the Boring Stuff** - automation frees capacity
10. **Agent-Native by Default** - AI-readiness amplifies all other principles

### Principle review cadence

These principles evolve with the organization. Proposed changes follow the same process as any manifesto update: open a PR, get Staff Engineer approval, and document the rationale. Principles are reviewed during the bi-annual manifesto review cycle.

---

## 📋 6. Quick Reference Card

| # | Principle | One-Liner |
|---|-----------|-----------|
| 2.1 | Own What You Ship | You build it, you run it, you fix it |
| 2.2 | Agent-Native by Default | Design for humans and machines alike |
| 2.3 | Specification Over Implementation | Contract first, code second |
| 2.4 | Automate the Boring Stuff | Pipelines do repetition, people do thinking |
| 2.5 | Observable by Default | If you cannot see it, you cannot fix it |
| 2.6 | Pave the Golden Path | Make the right way the easy way |
| 2.7 | Design for Failure | Every dependency will fail - plan for it |
| 2.8 | Security Is Everyone's Job | Embedded in the workflow, not bolted on |
| 2.9 | Ship Small, Learn Fast | Small increments, tight feedback loops |
| 2.10 | Everything in Git | If it is not in Git, it does not exist |

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
