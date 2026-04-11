# 🏗️ 02 - Architecture & API

> How our system is structured and how services communicate with each other and the outside world.

| Document | Description |
|----------|-------------|
| [01-system-architecture.md](./01-system-architecture.md) | Domain map, Kafka backbone, BFF pattern, resilience requirements |
| [02-api-standards.md](./02-api-standards.md) | URL design, versioning, error shapes, pagination, OpenAPI-first process |
| [03-hexagonal-architecture.md](./03-hexagonal-architecture.md) | Ports & adapters - full worked example for the Order service |
| [04-real-time-architecture.md](./04-real-time-architecture.md) | WebSocket, SSE, push, location streaming, scaling, security |
| [05-grpc-standards.md](./05-grpc-standards.md) | When to use gRPC, proto conventions, versioning, errors, deadlines, health, observability, testing, security |
| [06-saga-patterns.md](./06-saga-patterns.md) | Distributed transactions - choreography, compensation, order lifecycle saga |
| [07-service-decomposition.md](./07-service-decomposition.md) | When to split or merge services, boundary validation criteria |
| [08-event-schema-evolution.md](./08-event-schema-evolution.md) | Event schema evolution, Avro compatibility, partition keys, breaking change playbook |
| [09-error-catalog.md](./09-error-catalog.md) | Error catalog, exception handling, error code registry, frontend error boundaries |
| [10-monolith-first-guide.md](./10-monolith-first-guide.md) | When to start with a monolith, modular monolith pattern, migration path to microservices |
| [11-api-versioning.md](./11-api-versioning.md) | API versioning strategy, breaking change policy, deprecation |
| [12-rate-limiting.md](./12-rate-limiting.md) | Rate limiting and throttling patterns, per-tenant limits |
| [13-webhook-standards.md](./13-webhook-standards.md) | Webhook design, payload signing, delivery guarantees |
| [14-batch-processing.md](./14-batch-processing.md) | Batch processing, CRON jobs, workflow orchestration |
| [15-contract-testing.md](./15-contract-testing.md) | Consumer-driven contracts, Pact, CI integration |
| [16-messaging-patterns.md](./16-messaging-patterns.md) | Messaging patterns beyond Kafka, queues, outbox pattern |
| [17-graphql-standards.md](./17-graphql-standards.md) | GraphQL standards, federation, BFF integration |
| [18-event-sourcing-cqrs.md](./18-event-sourcing-cqrs.md) | Event sourcing and CQRS patterns |
| [19-api-lifecycle.md](./19-api-lifecycle.md) | API lifecycle management, design through sunset |

---
<div align="center">

🏠 [Back to root](../README.md)

</div>
