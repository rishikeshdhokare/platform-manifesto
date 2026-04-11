# 🐛 Error Tracking Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Error tracking platforms capture, aggregate, and prioritize application errors and crashes in real time. They provide stack traces, contextual metadata, and trend analysis so teams can triage issues quickly and reduce user impact.

The right choice depends on your language ecosystem, mobile versus backend split, and whether you need performance monitoring bundled with error tracking.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| SDK coverage | High | Language and framework support breadth |
| Grouping accuracy | High | Intelligent deduplication and issue grouping |
| Real-time alerting | High | Latency from error occurrence to notification |
| Performance monitoring | Medium | Bundled APM and transaction tracing |
| Release tracking | Medium | Correlation of errors with deployment versions |
| Source map support | Medium | Unminified stack traces for frontend apps |
| Data retention | Medium | How long raw event data is retained |
| Pricing model | Medium | Event-based versus seat-based pricing |

## 🔄 3. Comparison Matrix

| Feature | Sentry | Bugsnag | Rollbar | Crashlytics |
|---------|--------|---------|---------|-------------|
| Backend SDKs | Excellent | Good | Good | Limited |
| Frontend SDKs | Excellent | Good | Good | N/A |
| Mobile crash reporting | Good | Strong | Limited | Excellent |
| Issue grouping | Fingerprinting | Smart | Fingerprinting | Automatic |
| Performance monitoring | Built-in | Limited | Limited | Firebase APM |
| Release tracking | Strong | Strong | Good | Firebase |
| Source map support | Excellent | Good | Good | N/A |
| Self-hosted option | Yes | No | No | No |
| Open source | Yes (relay) | No | No | No |
| Pricing | Event-based | Event-based | Event-based | Free (Firebase) |

## 🧭 4. Decision Framework

Select based on your platform targets and monitoring needs:

- **Sentry** - Best all-around choice for full-stack applications
  - Choose when you need a single platform for backend, frontend, and mobile
  - Strong open-source community and self-hosted deployment option
- **Bugsnag** - Best for mobile-first teams needing stability scoring
  - Choose when mobile crash analysis and stability metrics are your priority
  - Strong integration with mobile release management workflows
- **Rollbar** - Best for backend-focused teams wanting fast onboarding
  - Choose when you need lightweight error tracking without APM overhead
  - Good for teams that prefer a focused tool over an all-in-one platform
- **Crashlytics** - Best for Firebase-native mobile applications
  - Choose when you are already invested in the Firebase ecosystem
  - Free tier makes it attractive but limited to mobile crash reporting

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Sentry | Best value with broadest coverage |
| Growth (50-200 eng) | Sentry | Performance monitoring and release tracking |
| Enterprise (200-1000 eng) | Sentry (self-hosted or business) | Data sovereignty and volume pricing |
| Hyperscale (1000+ eng) | Sentry + custom pipeline | Route high-volume events to data warehouse |

Integrate error tracking into your CI/CD pipeline so new deployments are tagged with release versions. This enables instant correlation between deploys and error spikes.

Set up alert rules that route critical errors to on-call channels and suppress known or low-impact issues to reduce noise.

Document the selection in an Architecture Decision Record (ADR) and ensure all services emit structured error context from day one.

## 📚 6. Related Documents

- [Incident Management](./24-incident-management.md)
- [Security Scanning](./26-security-scanning.md)
- [Testing Frameworks](./29-testing-frameworks.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
