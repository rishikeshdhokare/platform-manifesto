# 🚪 API Gateways Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

API gateways handle cross-cutting concerns for your service fleet - routing, authentication, rate limiting, transformation, and observability. The choice ranges from cloud-native managed gateways to self-hosted, highly configurable proxies.

Decide first whether the gateway is for north-south traffic (external clients to services) or east-west traffic (service-to-service). Most organizations use an API gateway for north-south and a service mesh for east-west.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Routing flexibility | High | Path, header, and weight-based routing capabilities |
| Auth integration | High | OAuth2, JWT validation, API key, and mTLS support |
| Rate limiting | High | Per-client, per-route, and global rate limiting |
| Plugin ecosystem | Medium | Extensibility via plugins, filters, or middleware |
| Performance overhead | High | Added latency per request at the gateway layer |
| Observability | High | Metrics, distributed tracing, and access logging |
| Developer portal | Medium | API documentation, key management, and onboarding |
| Managed vs. self-hosted | Medium | Operational model and control trade-offs |

## 🔄 3. Comparison Matrix

| Feature | Kong | AWS API Gateway | Envoy Gateway | Traefik |
|---------|------|-----------------|---------------|---------|
| Deployment model | Self-host + Konnect | Fully managed | Self-host (K8s) | Self-host |
| Protocol support | HTTP, gRPC, WebSocket | HTTP, WebSocket | HTTP, gRPC, TCP | HTTP, gRPC, TCP |
| Auth plugins | Strong (OIDC, JWT, key) | Cognito, Lambda auth | Ext auth filter | ForwardAuth |
| Rate limiting | Built-in | Built-in | Rate limit filter | Built-in |
| K8s Gateway API | Yes | N/A | Native | Yes |
| Latency overhead | Low | Low | Very low | Low |
| Plugin system | Lua, Go, Wasm | Lambda extensions | Wasm, ext proc | Middleware (Go) |
| Developer portal | Kong Developer Portal | API Gateway console | None (external) | None (external) |
| Config format | Declarative YAML/API | CloudFormation/SAM | K8s CRDs | YAML, labels |
| Cost | Free OSS + enterprise | Per-request pricing | Free OSS | Free OSS + enterprise |

## 🧭 4. Decision Framework

Select based on your infrastructure model and feature needs:

- **Kong** - Best full-featured gateway with developer portal
  - Choose when you need a comprehensive API management platform
  - Strong plugin ecosystem for auth, transformations, and analytics
- **AWS API Gateway** - Best for serverless and AWS-native architectures
  - Choose when your services are Lambda-based or you want zero gateway ops
  - Pay-per-request pricing aligns well with low-traffic and spiky workloads
- **Envoy Gateway** - Best for Kubernetes-native, high-performance routing
  - Choose when you want Kubernetes Gateway API compliance and Envoy's performance
  - Natural pairing with Envoy-based service meshes (Istio, Cilium)
- **Traefik** - Best for auto-discovery and container-native environments
  - Choose when you want automatic service registration from Docker or Kubernetes labels
  - Simple configuration with Let's Encrypt auto-TLS

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | AWS API Gateway or Traefik | Zero ops or simple self-host |
| Growth (50-200 eng) | Kong or Envoy Gateway | Feature-rich with team growth |
| Enterprise (200-1000 eng) | Kong Enterprise or Envoy Gateway | Portal, governance, and RBAC |
| Hyperscale (1000+ eng) | Envoy Gateway + Kong | Per-domain gateway federation |

Load test the gateway at 2x your expected peak traffic before production deployment. Gateway failures cascade to every service behind them and represent a single point of failure.

Implement gateway configuration as code with CI/CD validation. Manual gateway changes are a leading cause of production incidents in API-heavy architectures and must be avoided.

Document the gateway selection and routing strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Service Meshes](./17-service-meshes.md)
- [Container Orchestration](./14-container-orchestration.md)
- [Backend Frameworks](./05-backend-frameworks.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
