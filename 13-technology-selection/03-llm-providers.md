# 🧠 LLM Provider Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Large language model providers offer foundation models via API for code generation, summarization, classification, and agentic workflows. Provider selection affects cost, latency, quality, and vendor lock-in.

A multi-provider strategy with a unified gateway is recommended to avoid single-vendor dependency and to leverage model strengths for different tasks.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Model quality | High | Benchmark performance on coding and reasoning tasks |
| API reliability | High | Uptime SLAs and rate limit headroom |
| Latency | High | Time to first token and tokens per second |
| Pricing | High | Cost per million tokens (input and output) |
| Context window | Medium | Maximum token capacity per request |
| Fine-tuning support | Medium | Ability to train custom models on your data |
| Data privacy | High | Data retention, processing location, compliance certs |
| Multi-modal support | Low | Image, audio, and video understanding capability |

## 🔄 3. Comparison Matrix

| Feature | Anthropic (Claude) | OpenAI (GPT) | Google (Gemini) | AWS Bedrock | Azure OpenAI |
|---------|-------------------|--------------|-----------------|-------------|--------------|
| Coding quality | Strong | Strong | Good | Varies | Strong |
| Reasoning depth | Strong | Strong | Good | Varies | Strong |
| Context window | 200K | 128K | 1M+ | Varies | 128K |
| Latency | Good | Good | Good | Good | Good |
| Enterprise SLA | Yes | Yes | Yes | Yes | Yes |
| Data isolation | Strong | Good | Good | Strong | Strong |
| Fine-tuning | Limited | Yes | Yes | Yes | Yes |
| On-prem option | No | No | No | No | No |
| Multi-model gateway | No | No | No | Yes | No |

## 🧭 4. Decision Framework

Select based on your primary use case and compliance requirements:

- **Anthropic Claude** - Best for complex reasoning, coding, and agentic tasks
  - Choose when code quality and instruction following are critical
  - Strong safety profile and data handling commitments
- **OpenAI GPT** - Best ecosystem breadth and developer adoption
  - Choose when you need the widest third-party integration support
  - Strong fine-tuning platform for custom model training
- **Google Gemini** - Best for ultra-long context and multi-modal workloads
  - Choose when you process very large documents or need vision capabilities
  - Deep integration with Google Cloud services
- **AWS Bedrock** - Best for multi-model strategy on AWS
  - Choose when you want a unified API across multiple providers
  - Native IAM, VPC, and CloudWatch integration

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Single provider (Anthropic or OpenAI) | Simplicity over flexibility |
| Growth (50-200 eng) | Primary + fallback provider | Add a gateway abstraction layer |
| Enterprise (200-1000 eng) | Multi-provider via gateway | Route by task type, cost, and latency |
| Hyperscale (1000+ eng) | Bedrock or custom gateway | Centralized governance and cost control |

Build a model router or gateway abstraction from the start so you can switch providers or add fallbacks without changing application code. Track cost per task type to optimize model routing over time.

Negotiate enterprise agreements early - volume-based pricing can reduce costs by 30-50% compared to on-demand rates. Ensure your contract includes data processing terms that meet {Company}'s compliance requirements.

Document provider selection and routing strategy in an Architecture Decision Record (ADR) and revisit quarterly as model capabilities shift.

## 📚 6. Related Documents

- [AI Coding Assistants](./01-ai-coding-assistants.md)
- [AI Code Review Tools](./02-ai-code-review.md)
- [Vector Databases](./11-vector-databases.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
