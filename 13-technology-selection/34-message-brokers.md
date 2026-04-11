# 📬 Message Brokers Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Message brokers enable asynchronous communication between services through publish-subscribe, queue-based, and streaming patterns. The right broker depends on your throughput requirements, ordering guarantees, and operational model.

Understand the distinction between traditional message queues (task distribution) and event streaming platforms (durable log replay). Many architectures benefit from both.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Throughput | High | Messages per second at production scale |
| Ordering guarantees | High | Partition-level or global message ordering |
| Durability | High | Message persistence and replay capability |
| Delivery semantics | High | At-least-once, at-most-once, or exactly-once |
| Operational complexity | Medium | Cluster management and monitoring effort |
| Consumer model | Medium | Push versus pull, consumer groups, and fan-out |
| Managed offering | Medium | Availability of fully managed cloud service |
| Ecosystem integration | Medium | Connector framework and schema registry support |

## 🔄 3. Comparison Matrix

| Feature | Kafka | RabbitMQ | SQS | Pulsar | NATS |
|---------|-------|----------|-----|--------|------|
| Pattern | Event streaming | Message queue | Message queue | Streaming + queue | Pub-sub + queue |
| Throughput | Very high | Medium | High | Very high | Very high |
| Message ordering | Per partition | Per queue | FIFO option | Per partition | Per subject |
| Message replay | Full log | No | No | Full log | JetStream |
| Delivery semantics | At-least-once | At-least-once | At-least-once | Effectively-once | At-least-once |
| Schema registry | Confluent SR | No | No | Built-in | No |
| Managed offering | Confluent, MSK | CloudAMQP | AWS native | StreamNative | Synadia |
| Operational effort | High | Medium | None (serverless) | High | Low |
| Connector ecosystem | Kafka Connect | Plugins | Lambda triggers | Connectors | Limited |
| Protocol support | Kafka protocol | AMQP, MQTT | HTTP, SDK | Kafka-compatible | NATS protocol |

## 🧭 4. Decision Framework

Select based on your messaging pattern and operational capacity:

- **Kafka** - Best for high-throughput event streaming with durable replay
  - Choose when you need event sourcing, CDC, or stream processing at scale
  - Use Confluent Cloud or Amazon MSK to reduce operational burden
- **RabbitMQ** - Best for traditional work queue and routing patterns
  - Choose when you need flexible routing, priority queues, and dead-letter handling
  - Lower complexity than Kafka when you do not need event replay
- **SQS** - Best for serverless and AWS-native task queue workloads
  - Choose when you want zero-ops message queuing with Lambda integration
  - FIFO queues provide ordering when needed at reduced throughput
- **Pulsar** - Best for multi-tenant streaming with tiered storage
  - Choose when you need Kafka-like streaming with built-in geo-replication
- **NATS** - Best lightweight option for low-latency service communication
  - Choose for edge, IoT, or microservice messaging with minimal overhead

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | SQS or NATS | Zero-ops simplicity |
| Growth (50-200 eng) | Kafka (managed) or SQS | Event streaming for growing services |
| Enterprise (200-1000 eng) | Kafka (Confluent) | Full ecosystem with schema registry |
| Hyperscale (1000+ eng) | Kafka + SQS (hybrid) | Streaming for events, queues for tasks |

Define a clear schema evolution strategy before adopting event streaming. Use a schema registry with compatibility checks to prevent breaking consumers.

Establish dead-letter queue (DLQ) patterns and poison message handling from day one. Unprocessable messages must not block consumers or disappear silently.

## 📚 6. Related Documents

- [Data Pipelines](./33-data-pipelines.md)
- [Backend Frameworks](./05-backend-frameworks.md)
- [Container Orchestration](./14-container-orchestration.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
