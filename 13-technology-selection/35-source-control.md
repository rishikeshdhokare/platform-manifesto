# 🗃️ Source Control Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Source control platforms provide Git hosting, code review, and collaboration workflows that form the foundation of your software delivery process. The platform choice affects CI/CD integration, developer experience, and inner-source collaboration patterns.

Beyond basic Git hosting, modern platforms offer integrated CI/CD, security scanning, package registries, and project management - making this one of the most consequential platform decisions.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Code review workflow | High | PR review experience, inline comments, and suggestions |
| CI/CD integration | High | Built-in pipelines or first-class external CI support |
| Branch protection | High | Required reviews, status checks, and merge rules |
| Security features | High | Secret scanning, dependency alerts, and CODEOWNERS |
| Package registry | Medium | Built-in artifact hosting for containers and packages |
| API and automation | Medium | REST and GraphQL APIs for tooling integration |
| Inner-source support | Medium | Cross-team discovery, forking, and contribution |
| Self-hosted option | Medium | On-premises deployment for regulated environments |

## 🔄 3. Comparison Matrix

| Feature | GitHub | GitLab | Bitbucket | Azure DevOps |
|---------|--------|--------|-----------|--------------|
| Code review UX | Excellent | Strong | Good | Good |
| Built-in CI/CD | Actions | CI/CD pipelines | Pipelines | Azure Pipelines |
| Security scanning | Dependabot, CodeQL | SAST, DAST, SCA | Limited | Limited |
| Package registry | Packages | Container + pkg | Limited | Artifacts |
| Copilot integration | Native | Duo (limited) | Limited | GitHub Copilot |
| Inner-source | Excellent | Good | Good | Good |
| Self-hosted | GHES | Self-managed | Data Center | Server |
| SAML SSO | Enterprise | Premium | Premium | Azure AD |
| API quality | Excellent (GraphQL) | Strong (GraphQL) | REST only | REST |
| Marketplace | Extensive | Good | Limited | Moderate |
| Community size | Largest | Large | Medium | Medium |

## 🧭 4. Decision Framework

Select based on your ecosystem and collaboration strategy:

- **GitHub** - Best all-around platform with the largest developer ecosystem
  - Choose when you want the richest marketplace, AI tooling, and community
  - Strongest inner-source support and open-source collaboration model
- **GitLab** - Best single-platform DevOps solution
  - Choose when you want CI/CD, security scanning, and Git in one tool
  - Strong for teams that prefer a single vendor over best-of-breed integration
- **Bitbucket** - Best for Atlassian-centric organizations
  - Choose when deep Jira and Confluence integration is a priority
  - Consider only if your team is already invested in the Atlassian ecosystem
- **Azure DevOps** - Best for Microsoft-centric enterprise environments
  - Choose when Azure AD, Visual Studio, and Azure Cloud are your foundation
  - Strong enterprise governance but weaker developer community ecosystem

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | GitHub | Largest ecosystem, AI tooling, free tier |
| Growth (50-200 eng) | GitHub Team or GitLab Premium | Advanced reviews and security |
| Enterprise (200-1000 eng) | GitHub Enterprise or GitLab Ultimate | SSO, audit, compliance |
| Hyperscale (1000+ eng) | GitHub Enterprise (GHES or Cloud) | Inner-source at scale |

Enforce branch protection rules on all production branches from day one - required reviews, passing CI, and no force pushes.

Standardize on a branching strategy (trunk-based or GitHub Flow) and document it. Inconsistent branching models across teams create integration friction.

## 📚 6. Related Documents

- [CI/CD Platforms](./19-cicd-platforms.md)
- [GitOps Tools](./20-gitops-tools.md)
- [Monorepo Tooling](./45-monorepo-tooling.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
