# 🏢 Data Warehouses Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Data warehouses provide optimized storage and query engines for analytical workloads - reporting, dashboards, machine learning feature stores, and ad hoc analysis. Modern cloud warehouses separate compute from storage, enabling elastic scaling and cost control.

The choice depends on your cloud provider strategy, data volume, query patterns, and whether you need a unified lakehouse architecture.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Query performance | High | Speed on complex analytical queries at scale |
| Compute-storage separation | High | Independent scaling of compute and storage |
| Ecosystem integration | High | Connectors to BI tools, ETL, and ML platforms |
| Data sharing | Medium | Cross-organization and cross-cloud data sharing |
| Semi-structured data | Medium | Native JSON, Parquet, and Avro support |
| Cost model | High | Pay-per-query versus provisioned compute pricing |
| Governance and security | High | Row-level security, masking, and audit logging |
| Multi-cloud support | Medium | Ability to run across cloud providers |

## 🔄 3. Comparison Matrix

| Feature | Snowflake | BigQuery | Redshift | Databricks |
|---------|-----------|----------|----------|------------|
| Architecture | Cloud-native | Serverless | Cluster-based | Lakehouse |
| Compute-storage separation | Full | Full | RA3 nodes | Full |
| Serverless option | Yes | Default | Serverless v2 | SQL Warehouses |
| Semi-structured data | VARIANT type | Nested/repeated | SUPER type | Delta Lake |
| Data sharing | Marketplace | Analytics Hub | Data sharing | Delta Sharing |
| Streaming ingestion | Snowpipe | Streaming API | Kinesis | Structured Streaming |
| ML integration | Snowpark | BigQuery ML | SageMaker | MLflow native |
| Multi-cloud | Yes | GCP-primary | AWS-only | Yes |
| Concurrency handling | Virtual warehouses | Slots | WLM queues | SQL Warehouses |
| Pricing | Credit-based | Per TB scanned | Per node-hour | DBU-based |

## 🧭 4. Decision Framework

Select based on your cloud strategy and data architecture vision:

- **Snowflake** - Best multi-cloud data warehouse with strong data sharing
  - Choose when you need cloud-agnostic deployment and cross-org data sharing
  - Virtual warehouse model provides predictable workload isolation
- **BigQuery** - Best serverless option for GCP-native organizations
  - Choose when you want zero-ops analytics with pay-per-query pricing
  - Strong for teams that value simplicity over configuration control
- **Redshift** - Best for AWS-native organizations with existing investments
  - Choose when your data ecosystem is fully on AWS and S3
  - Redshift Serverless reduces operational overhead significantly
- **Databricks** - Best for unified analytics and machine learning workloads
  - Choose when you need a lakehouse combining warehousing and data science
  - Delta Lake format provides ACID transactions on data lake storage

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | BigQuery or Snowflake | Serverless, pay per use |
| Growth (50-200 eng) | Snowflake or BigQuery | Data sharing and governance |
| Enterprise (200-1000 eng) | Snowflake or Databricks | Multi-team workload isolation |
| Hyperscale (1000+ eng) | Databricks lakehouse | Unified analytics and ML at scale |

Implement column-level cost attribution from day one. Without cost visibility, warehouse spending grows unchecked as teams add queries and dashboards.

Establish data modeling standards (dimensional modeling or OBT) and use dbt or similar transformation tools to keep warehouse logic version-controlled and testable.

## 📚 6. Related Documents

- [Data Pipelines](./33-data-pipelines.md)
- [Cloud Providers](./13-cloud-providers.md)
- [Cloud Cost Management](./43-cloud-cost-management.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
