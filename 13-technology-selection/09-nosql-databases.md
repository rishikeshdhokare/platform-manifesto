# 📦 NoSQL Databases Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

NoSQL databases serve workloads where relational models add unnecessary friction - high-velocity writes, flexible schemas, graph traversals, or time-series aggregations. Each NoSQL category excels at a specific access pattern.

Use NoSQL to complement your relational database, not replace it. Default to PostgreSQL unless you have a clear access pattern that demands a specialized store.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Data model fit | High | Alignment between your access patterns and the database model |
| Write throughput | High | Sustained ingestion rate for your workload |
| Query flexibility | Medium | Ad-hoc query support beyond primary key lookups |
| Operational maturity | High | Managed service quality, backup, and failover |
| Consistency model | High | Tunable consistency vs. strong consistency guarantees |
| Cost at scale | High | Storage and throughput pricing at production volumes |
| Multi-region support | Medium | Global replication and conflict resolution |
| Ecosystem integration | Medium | Driver quality, ORM support, and monitoring tools |

## 🔄 3. Comparison Matrix

| Feature | MongoDB | DynamoDB | Cassandra | Neo4j | TimescaleDB |
|---------|---------|----------|-----------|-------|-------------|
| Data model | Document | Key-value/document | Wide column | Graph | Time-series (SQL) |
| Query language | MQL | PartiQL | CQL | Cypher | SQL |
| Consistency | Tunable | Strong (per-item) | Tunable | ACID | ACID |
| Write throughput | High | Very high | Very high | Medium | High |
| Ad-hoc queries | Strong | Limited | Limited | Strong (graph) | Strong (SQL) |
| Managed service | Atlas | AWS native | Astra | AuraDB | Timescale Cloud |
| Horizontal scale | Sharding | Automatic | Linear | Limited | Hypertables |
| Multi-region | Atlas global | Global tables | Multi-DC | Fabric | Replicas |
| Cost model | Storage + ops | RCU/WCU | Per-node | Per-node | Storage-based |
| Schema flexibility | High | High | Medium | Medium | Structured |

## 🧭 4. Decision Framework

Select based on your dominant access pattern:

- **MongoDB** - Best for document-oriented workloads with flexible schemas
  - Choose when your data is naturally hierarchical and query patterns vary
  - Atlas provides strong managed experience with search and vector capabilities
- **DynamoDB** - Best for high-throughput key-value access on AWS
  - Choose when access patterns are known and you need single-digit-ms latency at any scale
  - Design tables around access patterns first; avoid if queries are unpredictable
- **Cassandra** - Best for massive write-heavy, append-only workloads
  - Choose when you need linear horizontal scaling for event logs or IoT data
  - Use DataStax Astra for managed operation
- **Neo4j** - Best for relationship-heavy queries (social graphs, fraud detection)
  - Choose when your queries involve multi-hop traversals across connected data
- **TimescaleDB** - Best for time-series metrics, events, and IoT telemetry
  - Choose when you want SQL queries over time-partitioned data

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | MongoDB Atlas or DynamoDB | Fully managed, zero-ops |
| Growth (50-200 eng) | DynamoDB + MongoDB Atlas | Access-pattern-specific selection |
| Enterprise (200-1000 eng) | Multi-store strategy | Document, key-value, and time-series |
| Hyperscale (1000+ eng) | Purpose-built per workload | Cassandra for scale, Neo4j for graphs |

Prototype your top three access patterns against candidate databases with production-representative data volumes before committing. NoSQL performance is highly dependent on data modeling decisions.

Avoid using NoSQL as a general-purpose database. Each NoSQL engine excels at specific access patterns and performs poorly when used outside its design sweet spot.

Document the database selection and data modeling rationale in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Relational Databases](./08-relational-databases.md)
- [Vector Databases](./11-vector-databases.md)
- [Search Engines](./12-search-engines.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
