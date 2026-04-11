# 📱 Mobile Development Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Mobile development platform selection involves trade-offs between native performance, code sharing, team structure, and time to market. Cross-platform frameworks have matured significantly, narrowing the gap with native development for most application types.

The decision depends on your app's complexity, performance requirements, team composition, and whether you need to share logic with web or backend.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Code sharing | High | Percentage of code reusable across iOS and Android |
| Native API access | High | Ease of accessing platform-specific capabilities |
| Performance | High | UI rendering smoothness and startup time |
| Developer availability | High | Hiring pool for the chosen technology |
| UI fidelity | Medium | Ability to match native platform design guidelines |
| Hot reload and DX | Medium | Development iteration speed and debugging quality |
| Ecosystem maturity | Medium | Library availability and community support |
| Long-term viability | Medium | Backing organization and adoption trajectory |

## 🔄 3. Comparison Matrix

| Feature | Native (Swift/Kotlin) | React Native | Flutter | KMP |
|---------|----------------------|--------------|---------|-----|
| Code sharing | None | 70-80% UI + logic | 80-90% UI + logic | 60-80% logic only |
| UI rendering | Platform native | Bridge to native | Custom (Skia/Impeller) | Platform native |
| Performance | Best | Good | Strong | Strong |
| Startup time | Best | Good | Good | Best |
| Hot reload | Limited | Yes | Yes | Partial |
| Platform API access | Full | Via bridges | Via channels | Via expect/actual |
| Web target | No | React Native Web | Flutter Web | Kotlin/JS |
| Desktop target | macOS only | Limited | Strong | Via Compose |
| Hiring pool | Large | Large | Growing | Small but growing |
| Backing | Apple/Google | Meta | Google | JetBrains |

## 🧭 4. Decision Framework

Select based on your app category and team structure:

- **Native (Swift/Kotlin)** - Best for platform-intensive apps
  - Choose when your app relies heavily on device hardware, AR, or OS-level features
  - Required when app store performance benchmarks are non-negotiable
- **React Native** - Best for teams with strong JavaScript/TypeScript expertise
  - Choose when you want to share developers between web and mobile
  - Strong for content-driven apps, dashboards, and CRUD applications
- **Flutter** - Best for pixel-perfect custom UIs across platforms
  - Choose when your design system requires identical rendering on all platforms
  - Strong for apps with complex animations and custom widgets
- **KMP (Kotlin Multiplatform)** - Best for sharing business logic with native UIs
  - Choose when you want native UI per platform but shared domain and networking layers
  - Ideal for teams already invested in Kotlin for backend services

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | React Native or Flutter | Ship to both platforms with one team |
| Growth (50-200 eng) | React Native or Flutter | Dedicated mobile team, cross-platform |
| Enterprise (200-1000 eng) | KMP or Native | Separate iOS/Android teams with shared logic |
| Hyperscale (1000+ eng) | Native + KMP shared logic | Maximum platform fidelity at scale |

Build the same feature screen in two candidate frameworks during evaluation. Measure development time, native API integration difficulty, CI build duration, and app binary size before committing.

Factor in the app store review process and platform-specific requirements (permissions, background tasks, push notifications) when evaluating cross-platform options against native development.

Document the platform selection in an Architecture Decision Record (ADR) and publish a mobile starter kit with CI/CD and code signing preconfigured.

## 📚 6. Related Documents

- [Frontend Frameworks](./06-frontend-frameworks.md)
- [Backend Languages](./04-backend-languages.md)
- [CI/CD Platforms](./19-cicd-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
