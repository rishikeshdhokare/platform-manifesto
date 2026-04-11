# ⚡ Cache Stores Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Cache stores reduce latency and database load by serving frequently accessed data from memory. The choice between caching technologies depends on data structure requirements, consistency needs, clustering capabilities, and licensing considerations.

The Redis licensing change to SSPL has made Valkey (the Linux Foundation fork) an increasingly important alternative for organizations with open-source licensing requirements.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Data structure support | High | Support for strings, hashes, lists, sets, sorted sets, streams |
| Clustering and HA | High | Native clustering, replication, and automatic failover |
| Performance | High | Sub-millisecond latency and throughput at peak load |
| Persistence options | Medium | RDB snapshots, AOF logging, or pure ephemeral mode |
| Licensing | Medium | Open-source license compatibility with your organization |
| Managed service | High | Quality of cloud-managed offerings |
| Pub/sub capability | Medium | Built-in messaging for cache invalidation or events |
| Memory efficiency | Medium | Memory overhead per key and compression support |

## 🔄 3. Comparison Matrix

| Feature | Redis | Valkey | Memcached | Hazelcast |
|---------|-------|--------|-----------|-----------|
| License | SSPL (dual) | BSD-3 | BSD | Apache 2.0 |
| Data structures | Rich | Rich (Redis-compat) | Strings only | Rich (Java) |
| Clustering | Redis Cluster | Valkey Cluster | Client-side | Built-in (CP/AP) |
| Persistence | RDB + AOF | RDB + AOF | None | Configurable |
| Pub/sub | Yes | Yes | No | Yes (topics) |
| Lua scripting | Yes | Yes | No | Yes (Java) |
| Streams | Yes | Yes | No | Jet (stream processing) |
| Managed options | ElastiCache, Upstash | ElastiCache (Valkey) | ElastiCache | Viridian |
| Multi-threaded | I/O threads | I/O threads | Yes | Yes |
| Near-cache | No | No | No | Yes (embedded) |

## 🧭 4. Decision Framework

Select based on your data structure needs and licensing requirements:

- **Redis** - Best for feature-rich caching with proven ecosystem
  - Choose when you need the full Redis module ecosystem (RedisJSON, RediSearch)
  - Be aware of SSPL licensing implications for SaaS and redistribution
- **Valkey** - Best open-source alternative with Redis API compatibility
  - Choose when you need Redis semantics with a permissive open-source license
  - AWS ElastiCache now defaults to Valkey for new deployments
- **Memcached** - Best for simple string-based caching at extreme scale
  - Choose when you only need key-value string caching with no persistence
  - Lower memory overhead per key than Redis for simple values
- **Hazelcast** - Best for JVM-native distributed caching with compute
  - Choose when you need near-cache (embedded) for ultra-low latency in Java services
  - Strong for distributed computing alongside caching

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Managed Redis or Valkey | ElastiCache or Upstash for zero-ops |
| Growth (50-200 eng) | Valkey (managed) | Open-source license, Redis compatibility |
| Enterprise (200-1000 eng) | Valkey cluster | Multi-node with automatic failover |
| Hyperscale (1000+ eng) | Valkey + Memcached | Valkey for rich data, Memcached for hot strings |

Benchmark cache hit ratios, tail latencies, and memory efficiency under realistic load patterns during evaluation. A cache that degrades under pressure creates cascading failures worse than having no cache.

Implement cache stampede protection (request coalescing or probabilistic early expiration) regardless of which cache store you select. This is a design pattern, not a product feature.

Document the cache architecture and eviction strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Relational Databases](./08-relational-databases.md)
- [NoSQL Databases](./09-nosql-databases.md)
- [Search Engines](./12-search-engines.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
