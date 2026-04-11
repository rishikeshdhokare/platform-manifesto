# 🔄 CI/CD Platforms Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

CI/CD platforms automate building, testing, and deploying code changes. The choice affects developer velocity, pipeline reliability, cost, and how tightly your delivery process integrates with your source control and deployment targets.

Prefer a platform that is native to your source control provider. Pipeline-as-code with version-controlled workflow definitions is a non-negotiable requirement.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Source control integration | High | Native triggers, status checks, and PR feedback |
| Pipeline-as-code | High | Version-controlled workflow definitions in YAML or DSL |
| Runner flexibility | High | Self-hosted, cloud-hosted, and GPU runner options |
| Build speed | High | Caching, parallelism, and incremental build support |
| Secret management | High | Encrypted secrets, OIDC, and vault integration |
| Marketplace/reusability | Medium | Shared actions, orbs, or templates across teams |
| Cost model | Medium | Free tier, per-minute pricing, and concurrency limits |
| Security scanning | Medium | Built-in SAST, DAST, and dependency scanning |

## 🔄 3. Comparison Matrix

| Feature | GitHub Actions | GitLab CI | Jenkins | CircleCI |
|---------|---------------|-----------|---------|----------|
| Config format | YAML (workflows) | YAML (.gitlab-ci.yml) | Groovy (Jenkinsfile) | YAML (config.yml) |
| Hosted runners | Yes (Linux, macOS, Windows) | Yes (shared, SaaS) | No (self-host only) | Yes (Docker, machine) |
| Self-hosted runners | Yes | Yes | Yes (primary model) | Yes (runner agent) |
| Caching | Actions cache | Built-in cache | Plugin-based | Built-in cache |
| Matrix builds | Yes | Yes (parallel) | Yes (matrix) | Yes (matrix) |
| OIDC for cloud auth | Yes | Yes | Plugin-based | Yes |
| Marketplace | GitHub Marketplace (large) | CI/CD catalog | Plugin ecosystem (huge) | Orbs registry |
| Monorepo support | Path filters | Rules (changes) | Plugin-based | Path filtering |
| Security scanning | Dependabot, CodeQL | SAST/DAST built-in | Plugins | Limited |
| Container registry | GHCR | GitLab Registry | External | None (external) |

## 🧭 4. Decision Framework

Select based on your source control platform and operational preferences:

- **GitHub Actions** - Best for GitHub-hosted repositories
  - Choose when your code lives in GitHub and you want native PR integration
  - Large marketplace of reusable actions for common CI/CD tasks
- **GitLab CI** - Best for GitLab-hosted repositories with integrated DevSecOps
  - Choose when you want CI/CD, security scanning, and registry in one platform
  - Built-in SAST, DAST, and dependency scanning at no additional cost
- **Jenkins** - Best for fully self-hosted, highly customizable pipelines
  - Choose only when regulatory requirements mandate self-hosted CI infrastructure
  - High operational burden; prefer managed alternatives for new projects
- **CircleCI** - Best for teams needing advanced caching and build performance
  - Choose when build speed and Docker layer caching are critical
  - Strong for monorepo workflows with granular path-based triggers

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | GitHub Actions | Free tier, lowest setup friction |
| Growth (50-200 eng) | GitHub Actions + larger runners | Self-hosted runners for cost control |
| Enterprise (200-1000 eng) | GitHub Actions or GitLab CI | Based on source control platform |
| Hyperscale (1000+ eng) | Platform-managed runners | Custom runner fleet with auto-scaling |

Evaluate CI/CD platforms by migrating one representative pipeline end-to-end, including secret management, build caching, artifact publishing, and deployment steps.

Standardize reusable pipeline templates early. Duplicated pipeline definitions across repositories become a maintenance burden that grows linearly with your service count.

Document the CI/CD platform selection in an Architecture Decision Record (ADR) and publish shared workflow templates.

## 📚 6. Related Documents

- [GitOps Tools](./20-gitops-tools.md)
- [Artifact Registries](./22-artifact-registries.md)
- [Container Orchestration](./14-container-orchestration.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
