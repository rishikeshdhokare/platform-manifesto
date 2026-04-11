# 🚨 Incident Management Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Incident management platforms orchestrate the full lifecycle of production incidents - from detection and alerting through response coordination and post-mortem analysis. A well-chosen platform reduces mean time to acknowledge (MTTA) and mean time to resolve (MTTR).

The right choice depends on your on-call complexity, integration ecosystem, and how tightly you want incident workflows coupled with your collaboration tools.

Consider incident management as part of your broader observability strategy. The platform should integrate cleanly with your monitoring, logging, and tracing stack.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Alerting reliability | High | Guaranteed delivery across multiple channels |
| On-call scheduling | High | Rotation management, overrides, and escalation |
| Integration depth | High | Connections to monitoring, CI/CD, and chat tools |
| Incident workflow | Medium | Structured response runbooks and status pages |
| Post-mortem support | Medium | Built-in retrospective and timeline tooling |
| Pricing model | Medium | Per-user cost and tier structure |
| API and automation | Medium | Programmable workflows and event routing |
| Mobile experience | Medium | Quality of on-call mobile app |

## 🔄 3. Comparison Matrix

| Feature | PagerDuty | OpsGenie | Incident.io |
|---------|-----------|----------|-------------|
| Alert routing | Advanced | Good | Good |
| On-call scheduling | Excellent | Strong | Good |
| Escalation policies | Advanced | Strong | Basic |
| Slack-native workflow | Plugin | Plugin | Native |
| Status pages | Add-on | Built-in | Add-on |
| Post-mortem tooling | Built-in | Basic | Strong |
| Event intelligence | AIOps | Basic | Growing |
| Atlassian integration | Good | Native | Good |
| API extensibility | Comprehensive | Comprehensive | Good |
| Pricing | High | Medium | Medium |

## 🧭 4. Decision Framework

Select based on your team size and incident response workflow:

- **PagerDuty** - Best for large operations with complex escalation needs
  - Choose when you have multiple on-call teams with layered escalation policies
  - Strong AIOps features for alert noise reduction and event correlation
- **OpsGenie** - Best for Atlassian-centric organizations
  - Choose when Jira and Confluence are your primary collaboration tools
  - Good balance of features and cost for mid-size engineering teams
- **Incident.io** - Best for Slack-first incident response workflows
  - Choose when your team coordinates primarily through Slack channels
  - Modern workflow engine with strong post-mortem and learning capabilities

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | OpsGenie or Incident.io | Cost-effective, fast setup |
| Growth (50-200 eng) | PagerDuty or Incident.io | Structured escalation needed |
| Enterprise (200-1000 eng) | PagerDuty | Advanced routing and AIOps |
| Hyperscale (1000+ eng) | PagerDuty + custom tooling | Event orchestration at scale |

Establish clear on-call expectations and escalation policies before selecting tooling. The platform should reinforce your incident culture, not define it.

Run blameless post-mortems after every significant incident regardless of which platform you choose. Document findings and track action items to completion.

Record the selection in an Architecture Decision Record (ADR). Revisit annually as {Company} grows and incident patterns evolve.

## 📚 6. Related Documents

- [Error Tracking](./25-error-tracking.md)
- [ChatOps Platforms](./41-chatops-platforms.md)
- [CI/CD Platforms](./19-cicd-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
