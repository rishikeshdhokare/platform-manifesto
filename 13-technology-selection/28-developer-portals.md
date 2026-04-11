# 🏗️ Developer Portals Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Developer portals provide a centralized catalog of services, APIs, documentation, and tooling ownership. They reduce cognitive load by giving engineers a single pane of glass to discover infrastructure, understand dependencies, and self-serve common workflows.

The choice between open-source and commercial portals depends on your willingness to invest in customization versus your need for out-of-the-box scorecards and integrations.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Service catalog | High | Automated discovery and metadata management |
| Software templates | High | Scaffolding for new services and infrastructure |
| Scorecards | Medium | Engineering standards tracking and compliance |
| Plugin ecosystem | High | Extensibility through community and custom plugins |
| Integration depth | Medium | Connections to CI/CD, cloud, and observability tools |
| Self-service actions | Medium | Developer workflows triggered from the portal |
| Maintenance burden | High | Operational effort to run and customize |
| Pricing model | Medium | Open-source versus per-developer SaaS pricing |

## 🔄 3. Comparison Matrix

| Feature | Backstage | Port | Cortex | OpsLevel |
|---------|-----------|------|--------|----------|
| Service catalog | Excellent | Excellent | Strong | Strong |
| Software templates | Strong | Good | Limited | Good |
| Scorecards | Community plugin | Built-in | Excellent | Strong |
| Plugin ecosystem | Very large | Growing | Limited | Limited |
| Self-service actions | Scaffolder | Actions | Workflows | Actions |
| API docs integration | Built-in (TechDocs) | Good | Good | Good |
| Setup effort | High | Low | Low | Low |
| Open source | Yes (CNCF) | No | No | No |
| Dependency mapping | Plugin | Built-in | Built-in | Built-in |
| Pricing | Free (infra cost) | Per developer | Per developer | Per developer |

## 🧭 4. Decision Framework

Select based on your platform engineering investment and customization needs:

- **Backstage** - Best for organizations willing to invest in a custom platform
  - Choose when you want maximum flexibility and own the developer experience
  - Large plugin ecosystem but requires dedicated platform team to maintain
- **Port** - Best commercial option with strong data modeling capabilities
  - Choose when you want a flexible catalog without building from scratch
  - Strong self-service action framework with blueprint-based modeling
- **Cortex** - Best for engineering standards and reliability scorecards
  - Choose when driving adoption of engineering best practices is the priority
  - Excellent for tracking service maturity across {Company}
- **OpsLevel** - Best for service ownership and operational readiness
  - Choose when service ownership clarity and operational health are key goals
  - Strong integration with incident management and on-call workflows

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Port or defer adoption | Portal overhead may exceed value |
| Growth (50-200 eng) | Backstage or Port | Service discovery becomes critical |
| Enterprise (200-1000 eng) | Backstage with platform team | Customized developer experience |
| Hyperscale (1000+ eng) | Backstage + custom plugins | Tailored workflows at scale |

Start with the service catalog and software templates before adding scorecards. The catalog provides immediate value by making services discoverable, while templates accelerate new project creation.

Assign clear ownership for every service in the catalog. A portal without accurate ownership data creates more confusion than it resolves.

## 📚 6. Related Documents

- [Documentation Platforms](./30-documentation-platforms.md)
- [CI/CD Platforms](./19-cicd-platforms.md)
- [Cloud Dev Environments](./37-cloud-dev-environments.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
