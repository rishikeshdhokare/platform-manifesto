# 🗄️ Relational Databases Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Relational databases remain the backbone of transactional systems. The choice between self-managed, cloud-managed, and distributed SQL databases affects operational burden, cost, scalability, and data sovereignty.

For most teams, a managed PostgreSQL-compatible service provides the best balance of capability, ecosystem, and operational simplicity.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| SQL compatibility | High | Standards compliance and feature richness |
| Managed service quality | High | Automated backups, failover, and patching |
| Horizontal scalability | Medium | Ability to shard or distribute across nodes |
| Performance at scale | High | Query throughput and latency under production load |
| Extension ecosystem | Medium | Support for extensions (PostGIS, pg_cron, etc.) |
| Multi-region support | Medium | Active-active or read replica across regions |
| Cost efficiency | High | Pricing model relative to workload characteristics |
| Migration tooling | Medium | Schema migration and data transfer capabilities |

## 🔄 3. Comparison Matrix

| Feature | PostgreSQL | MySQL | Aurora (PostgreSQL) | CockroachDB |
|---------|------------|-------|-------------------|-------------|
| SQL standards | Excellent | Good | Excellent | Good |
| JSON support | Strong (JSONB) | Good (JSON) | Strong (JSONB) | Good |
| Extensions | Very rich | Limited | Subset of PG | Limited |
| Max connections | Needs pooling | High | High | Very high |
| Horizontal scale | Read replicas | Read replicas | Read replicas (15) | Native sharding |
| Multi-region writes | Manual | Manual | Limited | Built-in |
| Managed options | RDS, Cloud SQL | RDS, Cloud SQL | AWS only | Cloud + self-host |
| Storage auto-scaling | Manual | Manual | Automatic | Automatic |
| Logical replication | Yes | Yes | Yes | Yes (CDC) |
| Cost at 1TB | Low | Low | Medium | High |

## 🧭 4. Decision Framework

Select based on your scale requirements and operational model:

- **PostgreSQL (managed)** - Best default choice for most workloads
  - Choose when you want maximum flexibility, extensions, and community support
  - Use RDS, Cloud SQL, or Neon for managed operation
- **MySQL** - Best for teams with existing MySQL expertise
  - Choose when your ORM and tooling are already MySQL-optimized
  - Consider only if migrating from MySQL would provide insufficient benefit
- **Aurora PostgreSQL** - Best for AWS-native applications needing managed scale
  - Choose when you want PostgreSQL compatibility with automatic storage scaling
  - Strong for read-heavy workloads needing up to 15 read replicas
- **CockroachDB** - Best for globally distributed transactional workloads
  - Choose when you need multi-region writes with serializable isolation
  - Justified only when global consistency is a hard requirement

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Managed PostgreSQL (RDS/Neon) | Lowest operational overhead |
| Growth (50-200 eng) | Aurora PostgreSQL or Cloud SQL | Managed scaling, read replicas |
| Enterprise (200-1000 eng) | Aurora PostgreSQL | Multi-AZ with automated failover |
| Hyperscale (1000+ eng) | CockroachDB or Aurora + sharding | Global distribution requirements |

Test failover, backup restoration, and point-in-time recovery procedures during evaluation. Production reliability and disaster recovery capability matter more than raw benchmark throughput.

Establish connection pooling (PgBouncer or built-in) and query performance monitoring from day one. Database performance issues compound as data grows and are expensive to fix retroactively.

Document the database selection in an Architecture Decision Record (ADR) with capacity projections for the next 12-18 months.

## 📚 6. Related Documents

- [NoSQL Databases](./09-nosql-databases.md)
- [Cache Stores](./10-cache-stores.md)
- [Cloud Providers](./13-cloud-providers.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
