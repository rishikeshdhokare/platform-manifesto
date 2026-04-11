# 📊 Observability Platforms Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Observability platforms provide the metrics, logs, traces, and profiling data needed to understand system behavior in production. The choice between SaaS vendors and self-hosted open-source stacks determines your cost structure, data retention flexibility, and operational overhead.

Regardless of platform, standardize on OpenTelemetry (OTel) for instrumentation. OTel decouples your application code from the observability backend, enabling future migrations without code changes.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Three pillars coverage | High | Unified metrics, logs, and traces in a single platform |
| OpenTelemetry support | High | Native OTel collector and SDK compatibility |
| Query and exploration | High | Speed and flexibility of data querying and dashboarding |
| Alerting | High | Alert rules, routing, escalation, and on-call integration |
| Cost model | High | Pricing per host, per-GB ingested, or per-metric series |
| Data retention | Medium | Configurable retention periods and tiered storage |
| Infrastructure monitoring | High | Auto-discovery of hosts, containers, and cloud resources |
| Custom dashboards | Medium | Ease of building and sharing team-specific dashboards |

## 🔄 3. Comparison Matrix

| Feature | Datadog | Grafana Stack | New Relic |
|---------|---------|---------------|-----------|
| Deployment | SaaS | Self-host + Grafana Cloud | SaaS |
| Metrics | Custom metrics + infra | Prometheus/Mimir | Custom metrics + infra |
| Logs | Log Management | Loki | Log Management |
| Traces | APM | Tempo | Distributed Tracing |
| Profiling | Continuous Profiler | Pyroscope | Limited |
| OTel support | Yes (collector + SDK) | Native (preferred) | Yes (collector + SDK) |
| Dashboarding | Strong (built-in) | Grafana (excellent) | Strong (built-in) |
| Alerting | Monitors + PagerDuty | Grafana Alerting | Alert Policies |
| Pricing model | Per-host + per-GB | Self-host (infra) or per-metric | Per-GB ingested (all data) |
| Cost at scale | High (can spike) | Low-medium (self-host) | Moderate (predictable) |
| AI/ML features | Watchdog (anomaly detection) | ML-based alerting (Cloud) | Applied Intelligence |

## 🧭 4. Decision Framework

Select based on your team's operational capacity and budget:

- **Datadog** - Best all-in-one SaaS platform for fast time-to-value
  - Choose when you want the broadest integration catalog and lowest setup friction
  - Be aware of cost growth; implement governance for custom metrics and log volume
- **Grafana Stack** - Best for cost control and data ownership
  - Choose when you have platform engineers to operate the stack (or use Grafana Cloud)
  - Grafana Cloud offers a managed experience with generous free tiers
  - Self-hosted Mimir + Loki + Tempo provides full control over retention and cost
- **New Relic** - Best predictable pricing for growing organizations
  - Choose when you want per-GB-ingested pricing across all telemetry types
  - All-in-one platform with no per-host fees and generous free tier

Regardless of choice, instrument all services with OpenTelemetry SDKs and use the OTel Collector as an intermediary. This protects against vendor lock-in and enables dual-shipping during migrations.

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Grafana Cloud or New Relic | Generous free tiers |
| Growth (50-200 eng) | Datadog or Grafana Cloud | SaaS for velocity or Cloud for cost |
| Enterprise (200-1000 eng) | Datadog or Grafana Stack | Negotiate EDP or self-host for control |
| Hyperscale (1000+ eng) | Grafana Stack (self-hosted) | Cost control at massive ingest volumes |

Run a cost projection with your actual metric cardinality, log volume, and trace sampling rate before selecting. Observability costs can grow faster than infrastructure costs if left ungoverned.

Implement OTel Collector pipelines for sampling, filtering, and routing telemetry before it reaches the backend. Client-side filtering reduces both cost and noise simultaneously.

Document the observability platform selection and data governance policy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Service Meshes](./17-service-meshes.md)
- [Cloud Providers](./13-cloud-providers.md)
- [CI/CD Platforms](./19-cicd-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
