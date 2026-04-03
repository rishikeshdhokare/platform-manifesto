# 02 — Architecture & API

How our system is structured and how services communicate with each other and the outside world.

| Document | Description |
|----------|-------------|
| [01-system-architecture.md](./01-system-architecture.md) | Domain map, Kafka backbone, BFF pattern, resilience requirements |
| [02-api-standards.md](./02-api-standards.md) | URL design, versioning, error shapes, pagination, OpenAPI-first process |
| [03-hexagonal-architecture.md](./03-hexagonal-architecture.md) | Ports & adapters — full worked example for the Order service |
| [04-real-time-architecture.md](./04-real-time-architecture.md) | WebSocket, SSE, push, location streaming, scaling, security |
| [05-grpc-standards.md](./05-grpc-standards.md) | When to use gRPC, protos, versioning, Spring Boot, errors, deadlines, health, OTel, testing, security |
| [06-saga-patterns.md](./06-saga-patterns.md) | Distributed transactions — choreography, compensation, order lifecycle saga |
| [07-service-decomposition.md](./07-service-decomposition.md) | When to split or merge services, boundary validation criteria |
| [08-event-schema-evolution.md](./08-event-schema-evolution.md) | Event schema evolution, Avro compatibility, partition keys, breaking change playbook |

← [Back to root](../README.md)
