# 🧭 Vector Databases Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Vector databases enable similarity search over high-dimensional embeddings, powering RAG pipelines, semantic search, recommendation engines, and anomaly detection. The market spans purpose-built vector stores and vector extensions for existing databases.

Start with pgvector if you already use PostgreSQL. Move to a purpose-built solution only when query latency, index size, or advanced filtering outgrows what pgvector can handle.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Query latency | High | p99 latency for nearest-neighbor search at target scale |
| Index scalability | High | Maximum vector count and dimension support |
| Hybrid search | High | Combined vector + metadata filtering in a single query |
| Managed service | Medium | Hosted offering quality and SLA |
| Integration simplicity | High | SDK quality, LangChain/LlamaIndex compatibility |
| Cost model | Medium | Pricing per vector stored and per query |
| Operational overhead | Medium | Index management, backups, and cluster operations |
| Multi-tenancy | Medium | Namespace or collection-level isolation |

## 🔄 3. Comparison Matrix

| Feature | pgvector | Pinecone | Weaviate | Qdrant |
|---------|----------|----------|----------|--------|
| Deployment | PG extension | SaaS only | Self-host + cloud | Self-host + cloud |
| Max vectors | Millions | Billions | Billions | Billions |
| Hybrid search | SQL WHERE + ANN | Metadata filters | BM25 + vector | Payload filters |
| HNSW support | Yes | Proprietary | Yes | Yes |
| Quantization | Limited | Automatic | Yes (PQ, BQ) | Yes (scalar, PQ) |
| Multi-tenancy | Schema-based | Namespaces | Tenants | Collections |
| Backup/restore | pg_dump | Managed | Snapshots | Snapshots |
| LangChain support | Yes | Yes | Yes | Yes |
| ACID transactions | Yes (PostgreSQL) | No | No | No |
| Cost at 10M vectors | Low (DB add-on) | High | Medium | Medium |

## 🧭 4. Decision Framework

Select based on your scale and operational preferences:

- **pgvector** - Best starting point for teams already on PostgreSQL
  - Choose when your vector count is under 5-10 million and you value operational simplicity
  - Leverage existing PostgreSQL backups, monitoring, and access control
- **Pinecone** - Best fully managed experience with zero infrastructure
  - Choose when you want pure SaaS with no cluster management
  - Strong for teams without dedicated infrastructure engineers
- **Weaviate** - Best for hybrid search combining BM25 and vector retrieval
  - Choose when your RAG pipeline needs keyword + semantic search fusion
  - Strong multi-modal support for text, image, and video embeddings
- **Qdrant** - Best for high-performance self-hosted deployments
  - Choose when you need fine-grained control over indexing and quantization
  - Rust-based engine with strong filtering performance

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | pgvector | Zero additional infrastructure |
| Growth (50-200 eng) | pgvector or Pinecone | Evaluate based on vector count growth |
| Enterprise (200-1000 eng) | Weaviate or Qdrant | Self-hosted for data sovereignty |
| Hyperscale (1000+ eng) | Purpose-built (Qdrant/Weaviate) | Dedicated vector infrastructure |

Start with a small embedding collection and measure retrieval quality using precision@k and recall@k metrics before committing to dedicated vector infrastructure.

Evaluate whether your query patterns require metadata filtering alongside vector similarity. Filtering performance varies significantly across vector database implementations and can dominate query latency.

Document the vector database selection and embedding strategy in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [LLM Providers](./03-llm-providers.md)
- [Search Engines](./12-search-engines.md)
- [Relational Databases](./08-relational-databases.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
