# 🔐 Secret Management Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Secret management tools securely store, rotate, and distribute sensitive credentials - API keys, database passwords, TLS certificates, and encryption keys. Hardcoded secrets in source code or environment files are a critical security anti-pattern.

The choice depends on your cloud strategy, compliance requirements, and whether you need dynamic secret generation or static secret storage.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Dynamic secrets | High | On-demand credential generation with automatic expiry |
| Rotation support | High | Automated rotation without application downtime |
| Access control | High | Fine-grained policies for secret access |
| Audit logging | High | Complete trail of who accessed what and when |
| Kubernetes integration | Medium | Native sidecar injection or CSI driver support |
| Multi-cloud support | Medium | Consistent secret access across cloud providers |
| Developer experience | Medium | CLI, SDK, and local development workflow |
| Encryption model | High | Encryption at rest and in transit with key management |

## 🔄 3. Comparison Matrix

| Feature | HashiCorp Vault | AWS Secrets Manager | Doppler |
|---------|-----------------|---------------------|---------|
| Dynamic secrets | Excellent | Limited | No |
| Secret rotation | Configurable | Built-in (RDS, etc.) | Manual + sync |
| Access control | Policy-based (HCL) | IAM policies | Role-based |
| Audit logging | Comprehensive | CloudTrail | Built-in |
| PKI and certificates | Built-in CA | ACM (separate) | No |
| Kubernetes integration | Agent, CSI | ASCP | Operator |
| Multi-cloud | Yes | AWS only | Yes (sync) |
| Encryption as a service | Transit engine | KMS (separate) | No |
| Managed offering | HCP Vault | AWS native | SaaS |
| Setup complexity | High | Low | Very low |
| Pricing | Per instance/HCP | Per secret + API calls | Per seat |

## 🧭 4. Decision Framework

Select based on your security requirements and operational model:

- **HashiCorp Vault** - Best for dynamic secrets and advanced PKI needs
  - Choose when you need on-demand database credentials and certificate management
  - Most powerful option but highest operational complexity
  - Use HCP Vault to reduce management overhead while keeping full capabilities
- **AWS Secrets Manager** - Best for AWS-native applications with simple needs
  - Choose when your workloads run entirely on AWS and need integrated rotation
  - Lowest friction for RDS, Redshift, and other AWS service credential rotation
  - Limited to AWS ecosystem with no multi-cloud story
- **Doppler** - Best developer experience for teams wanting zero-ops secrets
  - Choose when you want the fastest onboarding and multi-environment sync
  - Excellent for syncing secrets across CI/CD, cloud platforms, and local dev
  - Does not support dynamic secret generation or PKI

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Doppler or AWS Secrets Manager | Fast setup, minimal ops |
| Growth (50-200 eng) | Doppler or HCP Vault | Multi-cloud or dynamic secrets |
| Enterprise (200-1000 eng) | Vault (self-hosted or HCP) | Dynamic secrets, PKI, full audit |
| Hyperscale (1000+ eng) | Vault + cloud-native hybrid | Layered strategy per workload |

Never store secrets in source control, environment files, or CI/CD configuration in plain text. Use the secret manager's SDK or sidecar injection in all environments.

Implement automatic rotation for all database and API credentials. Target a maximum secret lifetime of 90 days, with shorter lifetimes for high-sensitivity credentials.

## 📚 6. Related Documents

- [Identity Providers](./27-identity-providers.md)
- [Security Scanning](./26-security-scanning.md)
- [Container Orchestration](./14-container-orchestration.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
