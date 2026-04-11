# 🧪 Testing Frameworks Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

A comprehensive testing strategy spans multiple layers - unit, integration, end-to-end, load, contract, and mobile testing. Each layer requires specialized frameworks optimized for speed, reliability, and developer experience.

Standardize on one framework per layer across {Company} to maximize knowledge sharing, reduce CI/CD complexity, and simplify onboarding.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Execution speed | High | Test suite runtime and parallelization support |
| Developer experience | High | API clarity, debugging, and watch mode |
| CI/CD integration | High | Pipeline compatibility and reporting output |
| Ecosystem maturity | Medium | Plugin availability and community support |
| Cross-platform support | Medium | Browser, mobile, and API testing in one tool |
| Flake resistance | High | Built-in retry, auto-wait, and isolation features |
| Reporting and analytics | Medium | Test result dashboards and trend tracking |
| Language compatibility | Medium | Alignment with your primary language stack |

## 🔄 3. Comparison Matrix

| Layer | Recommended | Alternatives | Notes |
|-------|-------------|-------------|-------|
| Unit (JVM) | JUnit 5 | TestNG | Standard for Java and Kotlin |
| Unit (JS/TS) | Vitest | Jest | Faster, ESM-native, Jest-compatible API |
| Unit (Python) | pytest | unittest | Fixture system and plugin ecosystem |
| E2E (Web) | Playwright | Cypress, Selenium | Multi-browser, auto-wait, fast |
| E2E (API) | Playwright API + REST Assured | Postman, Karate | Typed assertions, CI-friendly |
| Load testing | k6 | Gatling, Locust | JavaScript scripting, low resource use |
| Contract testing | Pact | Spring Cloud Contract | Consumer-driven, language agnostic |
| Mobile (iOS) | XCTest | EarlGrey | Apple native, XCode integrated |
| Mobile (Android) | Espresso | UI Automator | Google native, fast on-device |
| Mobile (Cross) | Maestro | Appium, Detox | YAML-based, low flake rate |

## 🧭 4. Decision Framework

Select by test layer and language ecosystem:

- **Playwright** - Best default for all end-to-end web and API testing
  - Choose for multi-browser testing with built-in auto-wait and tracing
  - Single tool for UI, API, and component testing reduces tool sprawl
- **Vitest** - Best for JavaScript and TypeScript unit testing
  - Choose for ESM-native speed with Jest-compatible migration path
  - Excellent integration with Vite-based frontend projects
- **k6** - Best for developer-friendly load and performance testing
  - Choose when you want load tests written in JavaScript alongside application code
  - Low resource overhead compared to JMeter or Gatling
- **Pact** - Best for contract testing in microservice architectures
  - Choose when service-to-service API compatibility must be verified independently

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Vitest + Playwright | Minimal tooling, maximum coverage |
| Growth (50-200 eng) | Add k6 + Pact | Load and contract testing for reliability |
| Enterprise (200-1000 eng) | Full stack per layer | Dedicated framework per testing layer |
| Hyperscale (1000+ eng) | Centralized test platform | Shared infra for execution and reporting |

Aim for a testing pyramid - many fast unit tests, fewer integration tests, and minimal E2E tests. Invert this pyramid only when the cost of production failure justifies heavy E2E investment.

Track test suite execution time as a first-class CI/CD metric. Slow tests erode developer productivity and encourage skipping.

Document your testing strategy in an Architecture Decision Record (ADR) and standardize frameworks per layer across {Company}.

## 📚 6. Related Documents

- [CI/CD Platforms](./19-cicd-platforms.md)
- [Backend Frameworks](./05-backend-frameworks.md)
- [Frontend Frameworks](./06-frontend-frameworks.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
