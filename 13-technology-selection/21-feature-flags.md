# 🚩 Feature Flags Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Feature flag platforms enable trunk-based development by decoupling deployment from release. They support progressive rollouts, A/B testing, kill switches, and entitlement management. The choice between managed SaaS and self-hosted solutions depends on budget, data residency requirements, and operational capacity.

Every production service should use feature flags for new features. The platform choice determines how flags are evaluated (server-side vs. client-side), how targeting rules are managed, and the analytics available for rollout decisions.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| SDK coverage | High | Availability of SDKs for your language and platform stack |
| Evaluation performance | High | Local evaluation latency and fallback on network failure |
| Targeting rules | High | User segments, percentages, and custom attribute targeting |
| Audit trail | High | History of flag changes with who, what, and when |
| Experimentation | Medium | Built-in A/B testing, metrics tracking, and statistical analysis |
| Self-hosted option | Medium | Ability to run entirely within your infrastructure |
| Cost model | High | Per-seat, per-flag, or per-evaluation pricing |
| Integration depth | Medium | CI/CD, observability, and incident management integrations |

## 🔄 3. Comparison Matrix

| Feature | LaunchDarkly | Unleash | Flagsmith | GrowthBook |
|---------|-------------|---------|-----------|------------|
| Deployment | SaaS | Self-host + managed | Self-host + managed | Self-host + managed |
| Local evaluation | Yes (SDK streaming) | Yes (SDK polling) | Yes (SDK polling) | Yes (SDK) |
| Targeting rules | Very rich | Good | Good | Good |
| A/B testing | Yes (Experimentation) | Limited | Limited | Strong (core feature) |
| Audit log | Comprehensive | Yes | Yes | Yes |
| SDK languages | 25+ | 15+ | 15+ | 10+ |
| API-first | Yes | Yes | Yes | Yes |
| RBAC | Strong | Good | Good | Good |
| Analytics | Built-in + integrations | Metrics API | Analytics | Built-in Bayesian |
| Cost | High (per-seat) | Free OSS + enterprise | Free OSS + enterprise | Free OSS + cloud |

## 🧭 4. Decision Framework

Select based on your budget and experimentation needs:

- **LaunchDarkly** - Best enterprise feature management platform
  - Choose when you need the richest targeting, analytics, and SDK coverage
  - Highest cost but lowest integration friction and best support
- **Unleash** - Best self-hosted option for flag management
  - Choose when you need data residency control and open-source flexibility
  - Solid SDK coverage with a proven track record at scale
- **Flagsmith** - Best for teams wanting a balanced open-source platform
  - Choose when you want self-hosted or managed with reasonable pricing
  - Good API-first design for infrastructure automation
- **GrowthBook** - Best for experiment-driven organizations
  - Choose when A/B testing and statistical analysis are as important as flag management
  - Built-in Bayesian statistics engine for experiment analysis

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | GrowthBook or Unleash OSS | Free, self-hosted, experiment-ready |
| Growth (50-200 eng) | Unleash or Flagsmith | Self-hosted with growing feature needs |
| Enterprise (200-1000 eng) | LaunchDarkly or Unleash Enterprise | Governance and audit compliance |
| Hyperscale (1000+ eng) | LaunchDarkly | Scale-proven with global edge evaluation |

Define a flag hygiene process from day one. Stale flags accumulate as technical debt rapidly. Set expiration dates and assign owners to every flag at creation time.

Limit the number of active flags per service. High flag counts increase testing complexity exponentially and make production behavior harder to reason about during incidents.

Document the feature flag platform selection and lifecycle policy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [CI/CD Platforms](./19-cicd-platforms.md)
- [GitOps Tools](./20-gitops-tools.md)
- [Observability Platforms](./23-observability-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
