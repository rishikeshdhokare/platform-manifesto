# 📦 Monorepo Tooling Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Monorepo tools manage build orchestration, task scheduling, dependency tracking, and caching across multiple projects in a single repository. They make monorepos practical at scale by ensuring only affected projects are built and tested on each change.

The choice depends on your language ecosystem, repository size, and whether you need hermetic builds or prioritize developer experience and speed of adoption.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Task orchestration | High | Parallel execution with dependency-aware ordering |
| Remote caching | High | Shared build cache to avoid redundant computation |
| Affected project detection | High | Accurate change impact analysis for CI optimization |
| Language support | High | Polyglot capability across your tech stack |
| Build hermeticity | Medium | Reproducible builds independent of host environment |
| Code generation | Medium | Built-in support for generating code and configs |
| Learning curve | Medium | Time to onboard developers and migrate projects |
| Community and support | Medium | Ecosystem maturity and commercial backing |

## 🔄 3. Comparison Matrix

| Feature | Nx | Turborepo | Bazel | Rush | Pants |
|---------|----|-----------| ------|------|-------|
| Primary ecosystem | JS/TS (polyglot) | JS/TS | Polyglot | JS/TS | Python, Go, JVM |
| Task orchestration | Excellent | Good | Excellent | Good | Excellent |
| Remote caching | Nx Cloud | Vercel cache | Remote execution | Azure, S3 | Pantsd |
| Affected detection | Strong | Good | Query-based | Rush change | Dependency-based |
| Build hermeticity | Optional | No | Strict (sandbox) | No | Strict |
| Code generation | Generators | Limited | Rules | Templates | Codegen backends |
| Dependency graph UI | Built-in | Limited | Query tool | Basic | Built-in |
| Plugin ecosystem | Very large | Growing | Rules ecosystem | Plugins | Backends |
| Migration effort | Low | Very low | High | Medium | Medium |
| CI integration | Strong | Strong | Custom | Good | Good |
| Distributed execution | Nx Agents | No | Remote execution | No | Remote execution |

## 🧭 4. Decision Framework

Select based on your language ecosystem and build requirements:

- **Nx** - Best for JavaScript/TypeScript monorepos with polyglot aspirations
  - Choose when you want a rich developer experience with generators and graph visualization
  - Growing polyglot support for Java, Go, and .NET with plugin architecture
  - Lowest migration barrier for existing JS/TS projects
- **Turborepo** - Best for minimal-setup task caching in JS/TS repositories
  - Choose when you want the simplest possible monorepo tooling with fast results
  - Excellent for teams that need caching and parallelism without complex configuration
- **Bazel** - Best for large polyglot repositories requiring hermetic builds
  - Choose when reproducibility, correctness, and remote execution are non-negotiable
  - Steep learning curve justified only at significant scale or strict compliance needs
- **Rush** - Best for enterprise JS/TS monorepos with strict version policies
  - Choose when you need sophisticated version management and publishing workflows
- **Pants** - Best for Python, Go, and JVM monorepos
  - Choose when your primary languages are Python or Go with build hermeticity needs

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Turborepo or Nx | Fast setup, immediate caching benefit |
| Growth (50-200 eng) | Nx | Generators, affected detection, graph UI |
| Enterprise (200-1000 eng) | Nx or Bazel | Scale-dependent on language diversity |
| Hyperscale (1000+ eng) | Bazel | Remote execution, strict hermeticity |

Enable remote caching from day one. Local-only caching provides limited benefit in CI environments and misses cross-developer cache sharing opportunities.

Measure and track CI pipeline duration as a key metric. Monorepo tooling should reduce average CI time through caching and affected project detection.

## 📚 6. Related Documents

- [Source Control](./35-source-control.md)
- [CI/CD Platforms](./19-cicd-platforms.md)
- [Backend Languages](./04-backend-languages.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
