# 🏗️ Backend Frameworks Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Backend framework selection determines your development velocity, operational patterns, and the shape of your service architecture. The framework should align with your chosen language, team expertise, and the complexity of your domain logic.

Standardizing on one primary framework reduces onboarding time, simplifies shared libraries, and enables consistent observability instrumentation.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Developer productivity | High | Speed of building and iterating on features |
| Ecosystem and plugins | High | Middleware, auth, ORM, and validation libraries |
| Performance overhead | Medium | Framework-added latency and memory consumption |
| Observability support | High | Built-in metrics, tracing, and structured logging |
| Testing ergonomics | High | Ease of writing unit, integration, and contract tests |
| Community and docs | Medium | Quality of documentation and community support |
| Enterprise adoption | Medium | Production track record at scale |
| gRPC and async support | Medium | Native support for gRPC, WebSockets, and streaming |

## 🔄 3. Comparison Matrix

| Feature | Spring Boot | NestJS | FastAPI | Go (stdlib + chi) |
|---------|-------------|--------|---------|-------------------|
| Language | Java/Kotlin | TypeScript | Python | Go |
| Dependency injection | Built-in | Built-in | Manual/depends | Manual |
| ORM integration | Spring Data/JPA | TypeORM/Prisma | SQLAlchemy | sqlc/GORM |
| API documentation | SpringDoc/OpenAPI | Swagger module | Auto-generated | swaggo |
| gRPC support | Strong | Good | Good | Strong |
| Startup time | Slow (mitigated by GraalVM) | Fast | Fast | Very fast |
| Observability | Micrometer/OTel | OTel | OTel | OTel |
| Auth patterns | Spring Security | Passport/Guards | Depends | Middleware |
| Testing | JUnit/Mockito | Jest | pytest | go test |
| Learning curve | Steep | Moderate | Low | Low |

## 🧭 4. Decision Framework

Select based on your language choice and service complexity:

- **Spring Boot** - Best for complex enterprise microservices on JVM
  - Choose when you need rich DI, mature ORM, and battle-tested patterns
  - Use Spring Boot 3+ with virtual threads for modern concurrency
- **NestJS** - Best for TypeScript-first teams building structured APIs
  - Choose when you want Angular-style architecture on the backend
  - Strong for teams sharing code between frontend and backend
- **FastAPI** - Best for Python ML/AI serving and lightweight APIs
  - Choose when your service wraps ML models or data processing pipelines
  - Automatic OpenAPI docs and async support out of the box
- **Go stdlib + router** - Best for high-performance, minimal-overhead services
  - Choose when you want full control with minimal abstraction
  - Ideal for platform services, proxies, and infrastructure tooling

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | NestJS or FastAPI | Fastest time to first endpoint |
| Growth (50-200 eng) | Spring Boot or NestJS | Structure pays off as codebase grows |
| Enterprise (200-1000 eng) | Spring Boot primary | Mature ecosystem for complex domains |
| Hyperscale (1000+ eng) | Spring Boot + Go | Domain services + platform tooling |

Run a framework proof-of-concept that includes observability setup, database integration, authentication, and a CI/CD pipeline - not just a hello-world endpoint. This reveals integration friction that benchmarks miss.

Standardize shared libraries for logging, tracing, error handling, and configuration management on your chosen framework to ensure consistency across all services.

Document the framework selection in an Architecture Decision Record (ADR) and publish a service template for teams to bootstrap from.

## 📚 6. Related Documents

- [Backend Languages](./04-backend-languages.md)
- [API Gateways](./16-api-gateways.md)
- [Service Meshes](./17-service-meshes.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
