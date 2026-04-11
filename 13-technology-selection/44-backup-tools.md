# 💾 Backup Tools Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Backup tools protect against data loss from accidental deletion, corruption, infrastructure failure, and security incidents. For Kubernetes-native workloads, specialized tools back up cluster state, persistent volumes, and application configuration as a unit.

A strong backup strategy combines application-level backups (database dumps, snapshot APIs) with infrastructure-level backups (volume snapshots, cluster state) and validates recoverability through regular restore testing.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Kubernetes-native | High | CRD-based backup and restore for cluster workloads |
| Persistent volume support | High | CSI snapshot integration and cross-provider storage |
| Schedule and retention | High | Cron-based scheduling with configurable retention |
| Cross-cluster restore | Medium | Migration and disaster recovery across clusters |
| Application consistency | High | Pre and post hooks for database quiescence |
| Encryption | High | Backup encryption at rest and in transit |
| Storage backends | Medium | S3, GCS, Azure Blob, and MinIO compatibility |
| Monitoring and alerting | Medium | Backup success verification and failure alerts |

## 🔄 3. Comparison Matrix

| Feature | Velero | Kasten K10 | Native Snapshots |
|---------|--------|------------|------------------|
| Kubernetes backup | Excellent | Excellent | N/A |
| Persistent volume backup | CSI snapshots | CSI + Kanister | CSI or provider API |
| Database consistency | Pre/post hooks | Blueprint-based | Manual coordination |
| Cross-cluster restore | Yes | Yes | Manual |
| Multi-cloud storage | S3-compatible | Multi-cloud | Provider-specific |
| Scheduling | Cron-based | Policy-based | Cron or provider |
| Encryption | Server-side (storage) | Client-side + server | Provider-managed |
| Disaster recovery | Manual workflow | Automated DR | Manual |
| Compliance reporting | Basic | Built-in dashboard | Cloud console |
| Open source | Yes (Apache 2.0) | Freemium | N/A |
| Pricing | Free | Free (basic) + paid | Cloud provider cost |

## 🧭 4. Decision Framework

Select based on your Kubernetes adoption and recovery requirements:

- **Velero** - Best open-source Kubernetes backup with broad ecosystem support
  - Choose when you want a proven, community-driven backup tool for Kubernetes
  - Strong plugin system for storage providers and volume snapshotters
  - Requires more manual setup for application-consistent backups
- **Kasten K10** - Best for enterprise Kubernetes backup with policy-driven automation
  - Choose when you need application-aware backups with compliance reporting
  - Blueprint system provides database-consistent snapshots out of the box
  - Commercial licensing justified by reduced operational complexity
- **Native Snapshots** - Best for non-Kubernetes workloads and simple volume backups
  - Choose for EC2, RDS, EBS, and other cloud-managed service backups
  - Combine with cloud-native backup services (AWS Backup, GCP Backup)
  - Insufficient alone for Kubernetes cluster state and multi-resource recovery

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Velero + native snapshots | Open-source, simple setup |
| Growth (50-200 eng) | Velero + native snapshots | Proven reliability at moderate scale |
| Enterprise (200-1000 eng) | Kasten K10 + native snapshots | Policy automation and compliance |
| Hyperscale (1000+ eng) | Kasten K10 + centralized DR | Multi-cluster disaster recovery |

Test restore procedures regularly - at minimum quarterly. A backup that has never been restored is a hypothesis, not a safeguard.

Define Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO) for each tier of service. Backup frequency and tooling investment should align with these targets.

## 📚 6. Related Documents

- [Container Orchestration](./14-container-orchestration.md)
- [Cloud Providers](./13-cloud-providers.md)
- [Relational Databases](./08-relational-databases.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
