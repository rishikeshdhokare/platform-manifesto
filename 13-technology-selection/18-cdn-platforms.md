# 🌐 CDN Platforms Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Content delivery networks cache and serve static assets, API responses, and media from edge locations close to end users. Modern CDNs have evolved beyond caching to include edge compute, DDoS protection, WAF, and bot management.

The decision often comes down to whether you want a standalone CDN or a CDN bundled with broader security and edge compute capabilities.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Global PoP coverage | High | Number and distribution of edge locations |
| Cache performance | High | Hit ratio, purge speed, and cache key flexibility |
| Edge compute | Medium | Ability to run logic at the edge (Workers, Functions) |
| Security features | High | DDoS protection, WAF, bot management, and mTLS |
| Origin shielding | Medium | Reduction of origin load via intermediate caching layers |
| Cost model | High | Bandwidth pricing, request fees, and egress costs |
| API and automation | Medium | Terraform provider, API quality, and CI/CD integration |
| Real-time analytics | Medium | Traffic visibility, cache metrics, and error dashboards |

## 🔄 3. Comparison Matrix

| Feature | Cloudflare | CloudFront | Fastly | Akamai |
|---------|------------|------------|--------|--------|
| Global PoPs | 300+ | 400+ | 70+ | 4000+ |
| Edge compute | Workers (V8) | Lambda@Edge, Functions | Compute@Edge (Wasm) | EdgeWorkers |
| DDoS protection | Built-in (all plans) | Shield ($) | Built-in | Built-in |
| WAF | Built-in | AWS WAF ($) | Next-Gen WAF (Signal Sciences) | Kona WAF |
| Purge speed | Instant (global) | Minutes | Instant (150ms) | Seconds |
| Cache configurability | Page Rules, Workers | Behaviors, policies | VCL (powerful) | Property Manager |
| TLS termination | Automatic | ACM integration | Automatic | Automatic |
| Origin shield | Yes (tiered cache) | Yes (origin shield) | Yes (shielding PoPs) | Yes (SureRoute) |
| Terraform support | Strong | Strong | Strong | Good |
| Cost transparency | Simple, predictable | Complex (per-region) | Usage-based | Enterprise contracts |

## 🧭 4. Decision Framework

Select based on your primary requirements and budget model:

- **Cloudflare** - Best all-in-one platform for CDN, security, and edge compute
  - Choose when you want integrated DDoS, WAF, and Workers in a single platform
  - Most transparent pricing with generous free and pro tiers
- **CloudFront** - Best for AWS-native architectures
  - Choose when your origin is in AWS and you want zero-egress-cost to CloudFront
  - Deep integration with S3, ALB, API Gateway, and Lambda@Edge
- **Fastly** - Best for real-time cache control and VCL power users
  - Choose when you need instant purges and highly customizable caching logic
  - Strongest for media streaming and high-traffic API caching
- **Akamai** - Best for the largest enterprise and media delivery at global scale
  - Choose when you need the most extensive PoP network and enterprise SLAs
  - Highest cost but broadest geographic reach

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Cloudflare (Free/Pro) | Best value, integrated security |
| Growth (50-200 eng) | Cloudflare or CloudFront | Based on cloud provider |
| Enterprise (200-1000 eng) | Cloudflare Enterprise or Fastly | Advanced WAF and analytics |
| Hyperscale (1000+ eng) | Multi-CDN strategy | Cloudflare + Fastly or Akamai |

Test cache purge latency and origin shield effectiveness with your actual content types before committing. CDN behavior varies significantly between static assets and dynamic API responses.

Implement cache key design carefully from the start. Poor cache keys lead to low hit rates or stale content serving, both of which undermine the CDN's value to your users.

Document the CDN selection and caching strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Cloud Providers](./13-cloud-providers.md)
- [API Gateways](./16-api-gateways.md)
- [Frontend Frameworks](./06-frontend-frameworks.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
