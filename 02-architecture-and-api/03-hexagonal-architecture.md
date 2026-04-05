# 🔷 Hexagonal Architecture Guide

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

> **Pattern scope:** Hexagonal (ports and adapters) architecture is **language- and framework-agnostic**. The layers, dependency rule, and testing strategy apply to any stack. This document uses **Java with Spring-style wiring** as the **reference implementation** so teams have a concrete map; map the same roles to your runtime (for example, services and repositories in .NET, modules in Node.js, or use cases in Go).

---

## 🎯 1. What Is Hexagonal Architecture and Why Do We Use It?

Hexagonal architecture (also called "Ports and Adapters") is a way of structuring a service so that **business logic is completely independent of infrastructure** - databases, Kafka, HTTP, AWS SDKs.

**Why it matters for us:**

| Problem Without It | How Hexagonal Solves It |
|-------------------|------------------------|
| Unit tests require a database to run | Domain logic has no DB dependency - tests run in milliseconds |
| Changing from RDS to Aurora requires touching business logic | The DB is an adapter - swap it without touching the domain |
| Kafka details leak into service logic | Kafka is an adapter - domain just calls a port interface |
| Framework annotations or globals spread through core logic | Framework code lives in the infrastructure (adapter) layer only |

The rule is simple: **the domain knows nothing about the outside world.**

---

## 🧩 2. The Three Layers

```
┌──────────────────────────────────────────────────────────┐
│                      API Layer                           │
│   (Controllers, DTOs, OpenAPI, request/response mapping) │
│        HTTP/gRPC handlers live here (e.g. Spring MVC)    │
└────────────────────────┬─────────────────────────────────┘
                         │ calls
┌────────────────────────▼─────────────────────────────────┐
│                    Domain Layer                          │
│   (Entities, Services, Repository interfaces, Events)   │
│         Pure domain - zero framework imports             │
│           This is what you TEST and PROTECT              │
└────────────────────────┬─────────────────────────────────┘
                         │ implements
┌────────────────────────▼─────────────────────────────────┐
│                Infrastructure Layer                      │
│   (ORM adapters, Kafka producers, HTTP clients,          │
│    cloud SDKs, framework config, DB migrations)          │
│              Framework code lives here only              │
└──────────────────────────────────────────────────────────┘
```

> **Substitution point:** In the API layer, "controllers" map to handlers or route modules in other stacks. In infrastructure, JPA-style adapters map to your ORM or data access layer; "Spring config" maps to composition root / DI bootstrap for your framework.

**Visual overview:**

```mermaid
flowchart TB
    subgraph api [API Adapters]
        REST[REST Controller]
        GRPC[gRPC Service]
    end
    subgraph app [Application Layer]
        Ports[Use Case Ports]
    end
    subgraph domain [Domain Core]
        Model[Domain Model]
        Logic[Business Rules]
    end
    subgraph infra [Infrastructure Adapters]
        DB[(PostgreSQL)]
        MQ[Kafka Producer]
        Ext[External API Client]
    end
    api --> Ports
    Ports --> Model
    Model --> Logic
    infra --> Ports
```

**Dependency rule:** Arrows always point inward. Domain never imports from infrastructure or API.

---

## 🧩 3. Full Worked Example - Orders Service

**Reference implementation (Java):** The package layout and code below illustrate the pattern. Use the same **roles** (ports, domain model, adapters, API entry) in your language; names and annotations will differ.

Let's walk through the complete structure for the Orders Service.

### 3.1 Package Structure

```
com.{company}.orders/
│
├── api/                                    ← API Layer
│   ├── OrderController.java
│   ├── dto/
│   │   ├── OrderRequestDto.java           ← What the caller sends
│   │   ├── OrderResponseDto.java          ← What we return
│   │   └── OrderMapper.java              ← Maps between DTO ↔ Domain
│   └── exception/
│       └── GlobalExceptionHandler.java
│
├── domain/                                 ← Domain Layer (no framework imports)
│   ├── model/
│   │   ├── Order.java                     ← Core entity
│   │   ├── OrderId.java                   ← Value object
│   │   ├── OrderStatus.java              ← Enum
│   │   ├── Location.java                  ← Value object
│   │   └── CustomerId.java               ← Value object
│   ├── service/
│   │   └── OrderService.java             ← Business logic
│   ├── port/
│   │   ├── outbound/
│   │   │   ├── OrderRepository.java       ← Port (interface)
│   │   │   ├── OrderEventPublisher.java   ← Port (interface)
│   │   │   └── PricingPort.java           ← Port (interface)
│   │   └── inbound/
│   │       └── OrderUseCase.java          ← Port (interface)
│   └── event/
│       ├── OrderRequestedEvent.java
│       └── OrderCompletedEvent.java
│
└── infrastructure/                         ← Infrastructure Layer
    ├── persistence/
    │   ├── JpaOrderRepository.java        ← Implements OrderRepository
    │   ├── OrderEntity.java               ← JPA entity
    │   └── OrderEntityMapper.java         ← Maps JPA entity ↔ Domain model
    ├── kafka/
    │   └── KafkaOrderEventPublisher.java  ← Implements OrderEventPublisher
    ├── http/
    │   └── PricingServiceClient.java       ← Implements PricingPort
    └── config/
        ├── KafkaConfig.java
        └── PersistenceConfig.java
```

---

### 3.2 Domain Layer - The Heart

#### The Entity: `Order.java`

This is pure domain code. No framework imports, no persistence mapping, no messaging clients. It **enforces its own invariants**.

> **Other frameworks:** The equivalent is a plain domain type in any language (not an ORM entity, not an HTTP DTO).

```java
package com.{company}.orders.domain.model;

import java.time.Instant;
import java.util.Objects;

public class Order {

    private final OrderId id;
    private final CustomerId customerId;
    private final Location dispatch;
    private final Location delivery;
    private OrderStatus status;
    private ProviderId assignedProviderId;
    private PriceAmount priceAmount;
    private Instant requestedAt;
    private Instant startedAt;
    private Instant completedAt;

    public static Order request(CustomerId customerId, Location dispatch, Location delivery) {
        Order order = new Order();
        order.id = OrderId.generate();
        order.customerId = Objects.requireNonNull(customerId);
        order.dispatch = Objects.requireNonNull(dispatch);
        order.delivery = Objects.requireNonNull(delivery);
        order.status = OrderStatus.REQUESTED;
        order.requestedAt = Instant.now();
        return order;
    }

    public void assignProvider(ProviderId providerId) {
        if (status != OrderStatus.REQUESTED) {
            throw new InvalidOrderStateException(
                "Cannot assign provider to order in state: " + status);
        }
        this.assignedProviderId = Objects.requireNonNull(providerId);
        this.status = OrderStatus.MATCHED;
    }

    public void start() {
        if (status != OrderStatus.MATCHED) {
            throw new InvalidOrderStateException(
                "Cannot start order in state: " + status);
        }
        this.status = OrderStatus.IN_PROGRESS;
        this.startedAt = Instant.now();
    }

    public void complete(PriceAmount priceAmount) {
        if (status != OrderStatus.IN_PROGRESS) {
            throw new InvalidOrderStateException(
                "Cannot complete order in state: " + status);
        }
        this.priceAmount = Objects.requireNonNull(priceAmount);
        this.status = OrderStatus.COMPLETED;
        this.completedAt = Instant.now();
    }

    public void cancel(CancellationReason reason) {
        if (status == OrderStatus.COMPLETED) {
            throw new InvalidOrderStateException("Cannot cancel a completed order");
        }
        this.status = OrderStatus.CANCELLED;
    }

    public OrderId getId() { return id; }
    public OrderStatus getStatus() { return status; }
    public boolean isInProgress() { return status == OrderStatus.IN_PROGRESS; }
    public boolean isCompleted() { return status == OrderStatus.COMPLETED; }
    // ... other getters
}
```

#### Value Object: `OrderId.java`

```java
package com.{company}.orders.domain.model;

import java.util.UUID;

public record OrderId(String value) {

    public OrderId {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("OrderId must not be blank");
        }
    }

    public static OrderId generate() {
        return new OrderId(UUID.randomUUID().toString());
    }

    public static OrderId of(String value) {
        return new OrderId(value);
    }

    @Override
    public String toString() { return value; }
}
```

#### Outbound Port: `OrderRepository.java`

This is an **interface in the domain layer**. The domain defines what it needs; infrastructure provides it.

```java
package com.{company}.orders.domain.port.outbound;

import com.{company}.orders.domain.model.Order;
import com.{company}.orders.domain.model.OrderId;
import java.util.Optional;

public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId orderId);
    boolean existsById(OrderId orderId);
}
```

#### Outbound Port: `OrderEventPublisher.java`

```java
package com.{company}.orders.domain.port.outbound;

import com.{company}.orders.domain.event.OrderCompletedEvent;
import com.{company}.orders.domain.event.OrderRequestedEvent;

public interface OrderEventPublisher {
    void publish(OrderRequestedEvent event);
    void publish(OrderCompletedEvent event);
}
```

#### Domain Service: `OrderService.java`

```java
package com.{company}.orders.domain.service;

import com.{company}.orders.domain.model.*;
import com.{company}.orders.domain.port.outbound.*;
import com.{company}.orders.domain.event.*;

public class OrderService {

    private final OrderRepository orderRepository;
    private final OrderEventPublisher eventPublisher;
    private final PricingPort pricingPort;

    public OrderService(OrderRepository orderRepository,
                        OrderEventPublisher eventPublisher,
                        PricingPort pricingPort) {
        this.orderRepository = orderRepository;
        this.eventPublisher = eventPublisher;
        this.pricingPort = pricingPort;
    }

    public Order requestOrder(CustomerId customerId, Location dispatch, Location delivery) {
        Order order = Order.request(customerId, dispatch, delivery);
        Order saved = orderRepository.save(order);
        eventPublisher.publish(new OrderRequestedEvent(saved.getId(), customerId, dispatch, delivery));
        return saved;
    }

    public Order completeOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));

        PriceAmount price = pricingPort.calculateFinalPrice(orderId);
        order.complete(price);

        Order saved = orderRepository.save(order);
        eventPublisher.publish(new OrderCompletedEvent(saved.getId(), price));
        return saved;
    }
}
```

---

### 3.3 Infrastructure Layer - Adapters

#### JPA Adapter: `JpaOrderRepository.java`

This **implements** the domain's `OrderRepository` port. Persistence and framework wiring live here only.

> **Substitution point:** `@Repository` and Spring Data are Java/Spring specifics. In another stack, the adapter might be a class that implements the port using your ORM, SQL client, or repository pattern, registered in your composition root (Spring `@Bean`, ASP.NET DI, manual wiring, etc.).

```java
package com.{company}.orders.infrastructure.persistence;

import com.{company}.orders.domain.model.Order;
import com.{company}.orders.domain.model.OrderId;
import com.{company}.orders.domain.port.outbound.OrderRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public class JpaOrderRepository implements OrderRepository {

    private final SpringDataOrderRepository springRepo;
    private final OrderEntityMapper mapper;

    public JpaOrderRepository(SpringDataOrderRepository springRepo,
                               OrderEntityMapper mapper) {
        this.springRepo = springRepo;
        this.mapper = mapper;
    }

    @Override
    public Order save(Order order) {
        OrderEntity entity = mapper.toEntity(order);
        OrderEntity saved = springRepo.save(entity);
        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Order> findById(OrderId orderId) {
        return springRepo.findById(orderId.value())
            .map(mapper::toDomain);
    }

    @Override
    public boolean existsById(OrderId orderId) {
        return springRepo.existsById(orderId.value());
    }
}

interface SpringDataOrderRepository extends JpaRepository<OrderEntity, String> {}
```

#### Kafka Adapter: `KafkaOrderEventPublisher.java`

> **Substitution point:** `@Component` marks a Spring-managed adapter. Other runtimes use equivalent registration (DI container, factory, or module exports).

```java
package com.{company}.orders.infrastructure.kafka;

import com.{company}.orders.domain.event.OrderCompletedEvent;
import com.{company}.orders.domain.event.OrderRequestedEvent;
import com.{company}.orders.domain.port.outbound.OrderEventPublisher;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class KafkaOrderEventPublisher implements OrderEventPublisher {

    private static final String ORDER_REQUESTED_TOPIC = "orders.order.requested";
    private static final String ORDER_COMPLETED_TOPIC = "orders.order.completed";

    private final KafkaTemplate<String, Object> kafkaTemplate;

    public KafkaOrderEventPublisher(KafkaTemplate<String, Object> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @Override
    public void publish(OrderRequestedEvent event) {
        kafkaTemplate.send(ORDER_REQUESTED_TOPIC, event.orderId().value(), event);
    }

    @Override
    public void publish(OrderCompletedEvent event) {
        kafkaTemplate.send(ORDER_COMPLETED_TOPIC, event.orderId().value(), event);
    }
}
```

#### Spring Config: `DomainConfig.java`

This wires together domain services (no framework annotations on the domain) with their infrastructure adapters.

> **Substitution point:** `@Configuration` / `@Bean` are the Spring composition root. Any stack needs **one place** that constructs domain services and injects real adapters (constructor injection, service provider, or functional composition).

```java
package com.{company}.orders.infrastructure.config;

import com.{company}.orders.domain.service.OrderService;
import com.{company}.orders.domain.port.outbound.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DomainConfig {

    @Bean
    public OrderService orderService(OrderRepository orderRepository,
                                     OrderEventPublisher eventPublisher,
                                     PricingPort pricingPort) {
        return new OrderService(orderRepository, eventPublisher, pricingPort);
    }
}
```

---

### 3.4 API Layer - The Entry Point

> **Substitution point:** `@RestController`, `ResponseEntity`, and JWT parameter types are Spring Web / Jakarta idioms. Map to your HTTP framework's handlers, response types, and auth context.

```java
package com.{company}.orders.api;

import com.{company}.orders.domain.model.*;
import com.{company}.orders.domain.service.OrderService;
import com.{company}.orders.api.dto.*;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import jakarta.validation.Valid;

@RestController
@RequestMapping("/v1/orders")
public class OrderController {

    private final OrderService orderService;
    private final OrderMapper orderMapper;

    public OrderController(OrderService orderService, OrderMapper orderMapper) {
        this.orderService = orderService;
        this.orderMapper = orderMapper;
    }

    @PostMapping
    public ResponseEntity<OrderResponseDto> requestOrder(
            @Valid @RequestBody OrderRequestDto request,
            @AuthenticationPrincipal JwtPrincipal caller) {

        CustomerId customerId = CustomerId.of(caller.getUserId());
        Location dispatch = new Location(request.dispatch().lat(), request.dispatch().lng());
        Location delivery = new Location(request.delivery().lat(), request.delivery().lng());

        Order order = orderService.requestOrder(customerId, dispatch, delivery);

        return ResponseEntity.status(201).body(orderMapper.toResponse(order));
    }

    @PostMapping("/{orderId}/complete")
    public ResponseEntity<OrderResponseDto> completeOrder(@PathVariable String orderId) {
        Order order = orderService.completeOrder(OrderId.of(orderId));
        return ResponseEntity.ok(orderMapper.toResponse(order));
    }
}
```

---

## 🧪 4. How to Test Each Layer

| Layer | Test Type | What You Mock |
|-------|-----------|--------------|
| Domain (entities, services) | Unit tests | Nothing - pure domain, no mocks needed for entities; mock ports for services |
| Infrastructure (DB, messaging) | Integration tests | Nothing - use real dependencies in containers or test doubles as your stack supports |
| API (HTTP entry) | Narrow slice or contract tests | Mock or substitute the application / domain service |

> **Substitution point:** `@WebMvcTest` is Spring's controller slice. Other frameworks use equivalent HTTP-layer tests without starting the full stack.

**Reference implementation (Java):**

```java
class OrderServiceTest {

    private final OrderRepository orderRepository = mock(OrderRepository.class);
    private final OrderEventPublisher eventPublisher = mock(OrderEventPublisher.class);
    private final PricingPort pricingPort = mock(PricingPort.class);

    private final OrderService orderService =
        new OrderService(orderRepository, eventPublisher, pricingPort);

    @Test
    void requestOrder_savesAndPublishesEvent() {
        CustomerId customerId = CustomerId.of("customer-1");
        Location dispatch = new Location(25.2, 55.3);
        Location delivery = new Location(25.3, 55.4);
        when(orderRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

        Order order = orderService.requestOrder(customerId, dispatch, delivery);

        assertThat(order.getStatus()).isEqualTo(OrderStatus.REQUESTED);
        verify(eventPublisher).publish(any(OrderRequestedEvent.class));
    }
}
```

---

## ❌ 5. Common Mistakes to Avoid

> **Substitution point:** The examples below use Spring and JPA names. The underlying mistakes are universal: persistence or messaging types in the domain, framework injection in core logic, skipping ports, and fat controllers or anemic domain models.

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `@Entity` on domain model | JPA annotation in domain - breaks separation | Separate `OrderEntity` (infra) from `Order` (domain) |
| `@Autowired` in domain service | Spring in domain | Constructor inject via `DomainConfig` |
| Calling `KafkaTemplate` from domain service | Kafka in domain | Define `OrderEventPublisher` port; inject it |
| Calling another service's repository directly | Cross-domain DB access | Go via API or event |
| Putting business logic in controller | Controller becomes god class | Move to domain service |
| Putting business logic in JPA entity | Anemic domain model | Move to domain model class |

---

## 📏 6. The ArchUnit Rules That Enforce This

**Principle:** Architectural rules should fail the build when layers leak (domain depending on frameworks, API calling infrastructure directly, etc.).

**Reference implementation (Java):** [ArchUnit](https://www.archunit.org/) expresses those rules against the bytecode. Comparable tools exist for other ecosystems (for example dependency-cruiser or custom lint rules in TypeScript, import-linter in Python, NetArchTest for .NET, or module boundaries in Go).

These are pre-configured in the platform template:

```java
@AnalyzeClasses(packages = "com.{company}.orders")
class HexagonalArchitectureTest {

    @ArchTest
    static final ArchRule domainMustNotDependOnSpring =
        noClasses().that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
            .resideInAnyPackage("org.springframework..");

    @ArchTest
    static final ArchRule domainMustNotDependOnJpa =
        noClasses().that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
            .resideInAnyPackage("jakarta.persistence..", "org.hibernate..");

    @ArchTest
    static final ArchRule infraMustNotBeCalledByApi =
        noClasses().that().resideInAPackage("..api..")
            .should().dependOnClassesThat()
            .resideInAPackage("..infrastructure..");

    @ArchTest
    static final ArchRule controllersMustNotCallRepositories =
        noClasses().that().haveNameMatching(".*Controller")
            .should().dependOnClassesThat()
            .haveNameMatching(".*Repository");
}
```

If you violate the architecture, the build fails. This is intentional.

### Related Documents

- [API Standards](./02-api-standards.md) - URL design, versioning, error shapes
- [gRPC Standards](./05-grpc-standards.md) - internal service communication

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
