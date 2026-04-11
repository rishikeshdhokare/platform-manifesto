# 🔒 Security Scanning Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Security scanning encompasses multiple disciplines - static application security testing (SAST), dynamic application security testing (DAST), software composition analysis (SCA), secrets detection, and container image scanning. A mature security posture requires coverage across all five.

Shift-left by integrating scanning into CI/CD pipelines and developer workflows rather than relying solely on periodic audits.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Detection accuracy | High | True positive rate with low false positives |
| Language coverage | High | Support for your primary language ecosystem |
| CI/CD integration | High | Native pipeline plugins and exit code gating |
| Fix guidance | Medium | Actionable remediation advice and auto-fix |
| Scan speed | Medium | Time to complete without slowing CI pipelines |
| Vulnerability database | High | Freshness and breadth of CVE and advisory data |
| License compliance | Medium | OSS license detection and policy enforcement |
| Developer experience | Medium | IDE plugins and PR-level feedback |

## 🔄 3. Comparison Matrix

| Capability | Snyk | SonarQube | Checkmarx | Trivy | GitLeaks |
|------------|------|-----------|-----------|-------|----------|
| SAST | Good | Excellent | Excellent | No | No |
| DAST | Limited | No | Yes | No | No |
| SCA | Excellent | Limited | Good | Good | No |
| Container scanning | Strong | No | Good | Excellent | No |
| Secrets detection | Good | Limited | Good | Good | Excellent |
| IDE integration | Strong | Strong | Good | Limited | Limited |
| CI/CD plugins | Excellent | Excellent | Good | Excellent | Excellent |
| Auto-fix PRs | Yes | No | Limited | No | No |
| Open source | Partial | Community ed. | No | Yes | Yes |
| Pricing | Usage-based | Free/paid | Enterprise | Free | Free |

## 🧭 4. Decision Framework

Build a layered scanning strategy rather than selecting a single tool:

- **Snyk** - Best all-in-one commercial platform for SCA and container scanning
  - Choose as your primary SCA tool with strong developer experience
  - Auto-fix pull requests reduce remediation friction significantly
- **SonarQube** - Best for deep SAST analysis with quality gate enforcement
  - Choose for code quality and security analysis in a single platform
  - Self-hosted Community Edition provides strong baseline coverage
- **Trivy** - Best open-source option for container and infrastructure scanning
  - Choose for container image, filesystem, and IaC scanning in CI pipelines
  - Zero cost with comprehensive vulnerability database coverage
- **GitLeaks** - Best focused tool for secrets detection in Git history
  - Choose as a pre-commit hook to prevent secret leakage at the source

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Snyk free + Trivy + GitLeaks | Maximum coverage at zero cost |
| Growth (50-200 eng) | Snyk paid + SonarQube | SCA and SAST with quality gates |
| Enterprise (200-1000 eng) | Snyk + SonarQube + Checkmarx | Full SAST, DAST, and SCA coverage |
| Hyperscale (1000+ eng) | Multi-vendor + central dashboard | Unified vulnerability management |

Enforce scanning in CI pipelines with blocking gates for critical and high severity vulnerabilities. Allow medium and low findings to be tracked as backlog items.

Establish a vulnerability SLA - for example, critical within 48 hours, high within one week, medium within 30 days.

Document your scanning strategy in an Architecture Decision Record (ADR) and review tool coverage as your language stack evolves.

## 📚 6. Related Documents

- [CI/CD Platforms](./19-cicd-platforms.md)
- [Container Orchestration](./14-container-orchestration.md)
- [Compliance Automation](./42-compliance-automation.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
