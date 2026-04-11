# 🔧 Data Pipelines Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Data pipeline tools orchestrate the movement, transformation, and validation of data across systems. The landscape includes workflow orchestrators (Airflow, Dagster, Prefect), durable execution engines (Temporal), and transformation frameworks (dbt).

Most teams need at least two layers: an orchestrator for scheduling and dependency management, and a transformation tool for SQL-based analytics engineering.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| DAG authoring | High | Ease of defining, testing, and versioning pipelines |
| Observability | High | Run history, lineage tracking, and failure diagnostics |
| Scalability | High | Handling thousands of concurrent tasks |
| Error recovery | High | Retry logic, backfill support, and partial re-runs |
| Testing support | Medium | Unit and integration testing of pipeline logic |
| Managed offering | Medium | Availability of hosted or managed service |
| Community ecosystem | Medium | Pre-built integrations, operators, and plugins |
| Developer experience | Medium | Local development, debugging, and iteration speed |

## 🔄 3. Comparison Matrix

| Feature | Airflow | Dagster | Prefect | Temporal | dbt |
|---------|---------|---------|---------|----------|-----|
| Primary use case | Orchestration | Orchestration | Orchestration | Durable workflows | SQL transforms |
| DAG definition | Python | Python (assets) | Python (flows) | Code (workflows) | SQL + Jinja |
| Asset-centric | No | Yes | Limited | No | Yes (models) |
| Data lineage | Plugin | Built-in | Limited | N/A | Built-in |
| Testing framework | Limited | Strong | Good | Strong | Built-in |
| Managed offering | MWAA, Astronomer | Dagster Cloud | Prefect Cloud | Temporal Cloud | dbt Cloud |
| Kubernetes-native | Yes (KPO) | Yes | Yes | Yes | N/A |
| Community size | Very large | Growing | Medium | Growing | Very large |
| Learning curve | Moderate | Moderate | Low | High | Low |
| Backfill support | Manual | Asset-based | Flow reruns | Workflow reset | CLI-based |

## 🧭 4. Decision Framework

Select based on your pipeline complexity and team composition:

- **Airflow** - Best for teams with existing Airflow expertise and large DAG portfolios
  - Choose when you have extensive operator libraries and established patterns
  - Largest community but showing its age compared to newer alternatives
- **Dagster** - Best modern orchestrator with asset-centric data modeling
  - Choose when data lineage, testability, and software engineering practices matter
  - Strong local development experience and built-in observability
- **Prefect** - Best for Python-first teams wanting minimal orchestration overhead
  - Choose when you want the fastest path from Python scripts to production pipelines
- **Temporal** - Best for durable execution of complex, long-running workflows
  - Choose when you need guaranteed completion of multi-step business processes
- **dbt** - Best for SQL-based analytics transformations (pair with an orchestrator)
  - Choose as your transformation layer on top of any data warehouse

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Dagster + dbt | Modern defaults, strong DX |
| Growth (50-200 eng) | Dagster + dbt Cloud | Managed orchestration and transforms |
| Enterprise (200-1000 eng) | Airflow or Dagster + dbt | Proven scale with managed options |
| Hyperscale (1000+ eng) | Airflow (Astronomer) + Temporal + dbt | Layered orchestration strategy |

Treat pipelines as production software with CI/CD, testing, and code review. Pipeline failures at 3 AM should not be the norm.

Implement data quality checks (Great Expectations or dbt tests) at pipeline boundaries to catch issues before they propagate downstream.

## 📚 6. Related Documents

- [Data Warehouses](./32-data-warehouses.md)
- [Message Brokers](./34-message-brokers.md)
- [CI/CD Platforms](./19-cicd-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
