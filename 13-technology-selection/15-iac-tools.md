# 📐 Infrastructure as Code Tools Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Infrastructure as Code (IaC) tools define cloud resources in version-controlled files, enabling repeatable provisioning, drift detection, and infrastructure review via pull requests. The choice between declarative DSL, imperative SDK, and Kubernetes-native approaches depends on your team's skills and operational model.

Standardize on one primary IaC tool. Running Terraform and Pulumi and CDK in the same organization creates fragmented state management and inconsistent patterns.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Multi-cloud support | Medium | Ability to provision across AWS, Azure, GCP, and SaaS |
| Language and DX | High | Authoring experience - DSL vs. general-purpose language |
| State management | High | State backend reliability, locking, and collaboration |
| Module ecosystem | High | Reusable module registry and community contributions |
| Plan and preview | High | Quality of change preview before applying |
| CI/CD integration | High | Pipeline support for plan, review, and apply workflows |
| Drift detection | Medium | Ability to detect and reconcile out-of-band changes |
| Learning curve | Medium | Time for a new engineer to become productive |

## 🔄 3. Comparison Matrix

| Feature | Terraform/OpenTofu | Pulumi | AWS CDK | Crossplane |
|---------|-------------------|--------|---------|------------|
| Language | HCL | TypeScript, Python, Go | TypeScript, Python, Go | YAML (K8s CRDs) |
| Multi-cloud | Strong | Strong | AWS only | Strong |
| State management | S3/TF Cloud/Spacelift | Pulumi Cloud/self-managed | CloudFormation | Kubernetes etcd |
| Module registry | Largest | Growing | Constructs Hub | Compositions |
| Plan preview | terraform plan | pulumi preview | cdk diff | Dry-run |
| Drift detection | terraform plan | pulumi refresh | CloudFormation drift | Continuous reconciliation |
| GitOps-native | Via CI/CD | Via CI/CD | Via CI/CD | Native (controller) |
| Testing | Terratest | Unit tests (native) | CDK assertions | K8s testing |
| Secret handling | External + sensitive vars | Native encryption | SSM/Secrets Manager | External Secrets |
| License | BSL (TF) / MPL (OpenTofu) | Apache 2.0 | Apache 2.0 | Apache 2.0 |

## 🧭 4. Decision Framework

Select based on your team's skills and cloud strategy:

- **Terraform/OpenTofu** - Best default for multi-cloud and broadest ecosystem
  - Choose when you need the largest provider and module ecosystem
  - Use OpenTofu for BSL licensing concerns; use Spacelift or Atlantis for CI/CD
- **Pulumi** - Best for teams that prefer general-purpose programming languages
  - Choose when your infrastructure engineers are stronger in TypeScript/Python than HCL
  - Native testing with standard language test frameworks
- **AWS CDK** - Best for AWS-only organizations wanting type-safe constructs
  - Choose when you are fully committed to AWS and want IDE-assisted authoring
  - Generates CloudFormation under the hood with all its limits
- **Crossplane** - Best for Kubernetes-native infrastructure management
  - Choose when you want to manage cloud resources via the Kubernetes API
  - Continuous reconciliation model aligns with GitOps principles

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Terraform/OpenTofu | Simplest mental model, most tutorials |
| Growth (50-200 eng) | Terraform + Atlantis/Spacelift | PR-based apply workflow |
| Enterprise (200-1000 eng) | Terraform or Pulumi | Module library with governance |
| Hyperscale (1000+ eng) | Crossplane + Terraform | K8s-native for platforms, TF for foundations |

Validate your IaC tool by provisioning a complete environment end-to-end (networking, compute, database, DNS, monitoring) rather than testing individual resources in isolation.

Establish module versioning and a shared module registry early. Unversioned shared modules become a source of breaking changes across teams and erode trust in the platform.

Document the IaC tool selection and module strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Cloud Providers](./13-cloud-providers.md)
- [Container Orchestration](./14-container-orchestration.md)
- [GitOps Tools](./20-gitops-tools.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
