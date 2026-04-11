# 📋 Project Management Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Project management tools coordinate engineering work across teams - tracking issues, sprints, roadmaps, and cross-team dependencies. The right tool balances structure for leadership visibility with speed for individual developers.

The choice between heavyweight project management and lightweight issue tracking significantly impacts engineering velocity and process overhead.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Issue tracking speed | High | Time to create, update, and close issues |
| Workflow customization | Medium | Custom statuses, fields, and automations |
| Roadmap and planning | High | Sprint planning, epics, and cross-team views |
| Developer integration | High | Git, CI/CD, and IDE integration depth |
| Search and filtering | Medium | JQL-style queries and saved views |
| Reporting and metrics | Medium | Velocity, cycle time, and burndown charts |
| API and automation | Medium | Programmatic access and webhook support |
| Pricing model | Medium | Per-seat cost relative to feature tier |

## 🔄 3. Comparison Matrix

| Feature | Jira | Linear | Shortcut |
|---------|------|--------|----------|
| Issue creation speed | Slow (forms) | Fast (keyboard) | Fast (keyboard) |
| Workflow customization | Highly flexible | Opinionated | Moderate |
| Sprint planning | Comprehensive | Cycles | Iterations |
| Roadmaps | Built-in | Built-in | Built-in |
| Cross-team dependencies | Advanced | Projects | Epics |
| Git integration | Strong | Strong | Strong |
| Slack integration | Good | Excellent | Good |
| Custom fields | Extensive | Limited | Moderate |
| Automation rules | Powerful | Built-in | Built-in |
| Reporting depth | Very deep | Clean metrics | Good |
| Mobile app | Good | Excellent | Good |
| Pricing | Complex tiers | Per seat (flat) | Per seat (flat) |

## 🧭 4. Decision Framework

Select based on your process maturity and team preferences:

- **Jira** - Best for organizations needing deep customization and governance
  - Choose when you require complex workflows, custom fields, and advanced reporting
  - Strong for regulated industries and large program management
  - Highest configuration overhead but most flexible for diverse team processes
- **Linear** - Best for engineering teams prioritizing speed and developer experience
  - Choose when keyboard-driven workflows and clean UX are priorities
  - Opinionated defaults reduce setup time and enforce good practices
  - Excellent for teams that value velocity over configuration options
- **Shortcut** - Best mid-ground between Jira's power and Linear's simplicity
  - Choose when you need more structure than Linear without Jira's complexity
  - Good balance for cross-functional teams with mixed technical backgrounds

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Linear | Speed over ceremony, fast onboarding |
| Growth (50-200 eng) | Linear or Shortcut | Clean workflows with roadmap visibility |
| Enterprise (200-1000 eng) | Jira or Linear | Cross-team coordination at scale |
| Hyperscale (1000+ eng) | Jira | Advanced portfolio and program management |

Resist the urge to over-customize your project management tool. Complex workflows and dozens of custom fields create process debt that slows everyone down.

Measure cycle time (start to deploy) rather than story points. Cycle time directly reflects delivery capability and is harder to game.

## 📚 6. Related Documents

- [Documentation Platforms](./30-documentation-platforms.md)
- [ChatOps Platforms](./41-chatops-platforms.md)
- [Source Control](./35-source-control.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
