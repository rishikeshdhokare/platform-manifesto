# ☁️ Cloud Development Environments Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Cloud development environments (CDEs) provide on-demand, pre-configured workspaces that eliminate "works on my machine" problems. Engineers connect to remote compute via their preferred IDE, getting consistent tooling, dependencies, and access controls.

CDEs reduce onboarding time from days to minutes and ensure every developer works with identical, reproducible environments.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Startup speed | High | Time from workspace request to coding readiness |
| IDE support | High | VS Code, JetBrains, and terminal access options |
| Resource flexibility | Medium | CPU, memory, and GPU configuration options |
| Prebuilds | High | Pre-warmed images to eliminate cold start delays |
| Git integration | High | Automatic branch-based workspace creation |
| Cost controls | Medium | Idle shutdown, spending limits, and usage reporting |
| Security model | High | Network isolation, secret injection, and SSO |
| Self-hosted option | Medium | Running on your own cloud infrastructure |

## 🔄 3. Comparison Matrix

| Feature | GitHub Codespaces | Gitpod | Coder | DevPod |
|---------|-------------------|--------|-------|--------|
| Hosting model | GitHub-managed | SaaS + self-hosted | Self-hosted | Client-side |
| IDE support | VS Code (web + desktop) | VS Code, JetBrains | Any IDE | Any IDE |
| Prebuilds | Yes | Yes | Templates | Devcontainer spec |
| Configuration | devcontainer.json | .gitpod.yml | Terraform templates | devcontainer.json |
| GPU support | Yes | Yes | Yes | Provider-dependent |
| Idle auto-stop | Configurable | Configurable | Configurable | Manual |
| SSO integration | GitHub Enterprise | OIDC, SAML | OIDC, SAML | N/A (local) |
| Infrastructure control | None (managed) | Self-hosted option | Full control | Full control |
| Offline capability | No | No | No | Yes (local provider) |
| Pricing | Per core-hour | Per core-hour | Self-hosted (free OSS) | Free (OSS) |

## 🧭 4. Decision Framework

Select based on your infrastructure strategy and developer autonomy model:

- **GitHub Codespaces** - Best for GitHub-native teams wanting zero-ops CDEs
  - Choose when you want instant workspaces without managing infrastructure
  - Tightest integration with GitHub repos, PRs, and Actions
- **Gitpod** - Best for teams wanting managed CDEs with self-hosted flexibility
  - Choose when you need fast ephemeral workspaces with open-source foundations
  - Strong prebuild system and multi-IDE support across VS Code and JetBrains
- **Coder** - Best for platform teams wanting full infrastructure control
  - Choose when you must run workspaces on your own cloud with custom templates
  - Terraform-based provisioning enables any compute backend
- **DevPod** - Best for teams wanting CDE ergonomics without remote infrastructure
  - Choose when you want devcontainer-compatible workspaces on any backend
  - Open-source client with provider plugins for local, cloud, or Kubernetes

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | GitHub Codespaces or DevPod | Zero-ops, fast onboarding |
| Growth (50-200 eng) | Codespaces or Gitpod | Prebuilds for large repos |
| Enterprise (200-1000 eng) | Coder or Gitpod (self-hosted) | Infrastructure control and compliance |
| Hyperscale (1000+ eng) | Coder on internal cloud | Custom templates per team |

Start with devcontainer.json as your workspace specification format. It is an open standard supported by most CDE platforms and ensures portability.

Measure onboarding time before and after CDE adoption. A reduction from days to minutes is the primary value proposition and justifies the investment.

## 📚 6. Related Documents

- [Developer Portals](./28-developer-portals.md)
- [Container Orchestration](./14-container-orchestration.md)
- [Source Control](./35-source-control.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
