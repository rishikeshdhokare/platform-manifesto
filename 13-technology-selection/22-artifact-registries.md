# 📦 Artifact Registries Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Artifact registries store and distribute build artifacts - container images, Maven/Gradle packages, npm modules, Python wheels, and Helm charts. The registry is a critical supply chain component where security scanning, access control, and replication directly affect deployment reliability.

Consolidate on a single registry platform when possible. Running separate registries per artifact type increases operational burden and fragments vulnerability scanning.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Format support | High | Container, Maven, npm, PyPI, Helm, and generic artifacts |
| Security scanning | High | Integrated vulnerability scanning and policy enforcement |
| Replication | Medium | Cross-region and cross-site artifact replication |
| Access control | High | Fine-grained permissions per repository and artifact |
| Storage efficiency | Medium | Deduplication, layer sharing, and cleanup policies |
| CI/CD integration | High | Native authentication from your build platform |
| Managed service | Medium | Hosted option with SLA and maintenance included |
| Promotion workflows | Medium | Staging artifacts through dev, staging, and production |

## 🔄 3. Comparison Matrix

| Feature | JFrog Artifactory | Sonatype Nexus | GitHub Packages | Cloud-native (ECR/GAR) |
|---------|-------------------|----------------|-----------------|----------------------|
| Formats supported | 30+ | 20+ | Container, npm, Maven, NuGet | Container + cloud-specific |
| Vulnerability scanning | Xray (integrated) | Lifecycle (IQ) | Dependabot alerts | Cloud scanning (limited) |
| Replication | Multi-site push/pull | Pro (push only) | N/A | Cross-region replication |
| Promotion pipelines | Release bundles | Staging repos | N/A | Tag-based |
| RBAC | Fine-grained | Fine-grained | Org/repo-level | IAM policies |
| Storage backend | S3, GCS, Azure, local | Local, S3 | GitHub-managed | Cloud-native storage |
| Self-hosted | Yes | Yes | No (SaaS only) | No (cloud-managed) |
| HA clustering | Yes (Enterprise) | Yes (Pro+) | N/A | Built-in |
| Cleanup policies | Yes (retention rules) | Yes (cleanup tasks) | Limited | Lifecycle policies |
| Pricing | Per-user + storage | Free OSS + Pro | Free tier + paid | Pay per storage + transfer |

## 🧭 4. Decision Framework

Select based on your artifact diversity and supply chain requirements:

- **JFrog Artifactory** - Best universal registry for polyglot organizations
  - Choose when you need a single platform for all artifact types with advanced security
  - Xray integration provides deep recursive vulnerability scanning
- **Sonatype Nexus** - Best self-hosted option for Java-heavy organizations
  - Choose when you are primarily JVM-based and want a proven on-prem registry
  - Free OSS version covers most basic registry needs
- **GitHub Packages** - Best for GitHub-native workflows with simple needs
  - Choose when your artifacts are primarily container images and npm packages
  - Zero-config authentication from GitHub Actions workflows
- **Cloud-native (ECR/GAR)** - Best for container-only, single-cloud deployments
  - Choose when you only need container images and are fully committed to one cloud
  - Simplest IAM integration with your cloud provider's compute services

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | GitHub Packages or ECR/GAR | Zero ops, free tier covers early needs |
| Growth (50-200 eng) | GitHub Packages + ECR/GAR | Container registry + language packages |
| Enterprise (200-1000 eng) | JFrog Artifactory | Universal registry with security scanning |
| Hyperscale (1000+ eng) | Artifactory (Enterprise) | Multi-site replication with Xray |

Verify vulnerability scanning accuracy and false positive rates against your actual dependency stack before selecting. A noisy scanner that generates constant false alarms erodes developer trust.

Implement artifact promotion workflows (dev to staging to production) with automated quality gates. Never deploy artifacts directly from a development build to production.

Document the registry selection and promotion strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [CI/CD Platforms](./19-cicd-platforms.md)
- [Container Orchestration](./14-container-orchestration.md)
- [Cloud Providers](./13-cloud-providers.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
