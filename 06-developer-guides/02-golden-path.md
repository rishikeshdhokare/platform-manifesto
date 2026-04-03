# The Golden Path

> **Status:** Reference  
> **Owner:** Platform Engineering  
> **Last Updated:** 2025

---

## What Is the Golden Path?

The golden path is a **complete, opinionated, end-to-end walkthrough** of building and shipping a service on this platform. It covers everything from creating the repository to running code in production, observable and monitored.

This is not a tutorial — it is **the way things are done here**. Following it means you get all platform capabilities (CI, CD, observability, security, service catalog) for free.

---

## End-to-End Steps

### Step 1: Scaffold the Service

Open Backstage and use the **Java Microservice** template:

```
Backstage → Create → Java Microservice
  Name: orders-service
  Team: team-orders
  Domain: orders
  Port: 8080
```

This creates:
- GitHub repository `{company}/orders-service` with branch protection enabled
- Standard project structure (see below)
- CI pipeline (`.github/workflows/pr.yml`, `main.yml`)
- ArgoCD application manifest added to `platform-config` repo (dev namespace)
- Backstage catalog entry registered
- Slack channel `#service-orders-service` created

**Time: < 5 minutes**

---

### Step 2: Clone & Verify Local Setup

```bash
gh repo clone {company}/orders-service
cd orders-service

# Start dependencies
docker compose up -d

# Verify dependencies healthy
docker compose ps

# Run the service
./gradlew bootRun --args='--spring.profiles.active=local'

# Verify it starts
curl http://localhost:8080/actuator/health
# → {"status":"UP"}
```

**Time: < 10 minutes**

---

### Step 3: Project Structure

The template generates this structure — understand it before adding code:

```
orders-service/
├── src/
│   ├── main/
│   │   ├── java/com/{company}/orders/
│   │   │   ├── api/                    # Controllers, DTOs, OpenAPI annotations
│   │   │   │   ├── OrderController.java
│   │   │   │   └── dto/
│   │   │   ├── domain/                 # Business logic (pure Java, no Spring)
│   │   │   │   ├── Order.java          # Domain entity
│   │   │   │   ├── OrderService.java   # Domain service
│   │   │   │   └── OrderRepository.java # Repository interface (port)
│   │   │   ├── infrastructure/         # Adapters (Spring, JPA, Kafka, AWS)
│   │   │   │   ├── persistence/
│   │   │   │   │   └── JpaOrderRepository.java
│   │   │   │   ├── kafka/
│   │   │   │   │   └── OrderEventProducer.java
│   │   │   │   └── aws/
│   │   │   └── OrdersServiceApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-local.yml
│   │       ├── application-production.yml
│   │       └── db/migration/           # Flyway migrations
│   └── test/
│       ├── java/com/{company}/orders/
│       │   ├── domain/                 # Unit tests
│       │   ├── integration/            # Integration tests (Testcontainers)
│       │   ├── contract/               # Pact consumer/provider tests
│       │   └── architecture/           # ArchUnit tests
│       └── resources/
├── api/
│   └── openapi.yaml                    # OpenAPI 3.1 spec
├── docs/
│   ├── adr/                            # Architecture Decision Records
│   ├── runbook.md                      # Operational runbook
│   ├── performance.md                  # Performance baselines
│   └── dashboards/                     # Grafana dashboard JSON
├── docker-compose.yml
├── Dockerfile
├── build.gradle.kts
├── catalog-info.yaml                   # Backstage registration
└── README.md
```

---

### Step 4: Design the API First

Before writing any implementation code, write the OpenAPI spec at `api/openapi.yaml`.

Example for a `POST /v1/orders` endpoint:

```yaml
openapi: 3.1.0
info:
  title: Orders Service API
  version: 1.0.0

paths:
  /v1/orders:
    post:
      summary: Request a new order
      operationId: requestOrder
      tags: [orders]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderRequest'
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
```

Raise a PR for API review **before** implementing. This is the API-first contract review.

---

### Step 5: Write the Domain Logic

Write the domain layer first — pure Java, no framework dependencies:

```java
// domain/Order.java
public class Order {
    private final OrderId id;
    private OrderStatus status;
    private final CustomerId customerId;
    private final Location dispatch;
    private final Location delivery;
    private Instant startedAt;
    private Instant completedAt;

    public static Order request(CustomerId customerId, Location dispatch, Location delivery) {
        return new Order(OrderId.generate(), OrderStatus.REQUESTED, customerId, dispatch, delivery);
    }

    public void start(ProviderId providerId) {
        if (status != OrderStatus.MATCHED) {
            throw new InvalidOrderStateException("Cannot start order in state: " + status);
        }
        this.status = OrderStatus.IN_PROGRESS;
        this.startedAt = Instant.now();
    }

    public void complete() {
        if (status != OrderStatus.IN_PROGRESS) {
            throw new InvalidOrderStateException("Cannot complete order in state: " + status);
        }
        this.status = OrderStatus.COMPLETED;
        this.completedAt = Instant.now();
    }
}
```

Write unit tests for the domain first:

```java
// domain/OrderTest.java
class OrderTest {

    @Test
    void start_givenOrderIsMatched_transitionsToInProgress() {
        Order order = Order.request(CustomerId.of("customer-1"), DISPATCH, DELIVERY);
        order.match(ProviderId.of("provider-1"));

        order.start(ProviderId.of("provider-1"));

        assertThat(order.getStatus()).isEqualTo(OrderStatus.IN_PROGRESS);
    }

    @Test
    void start_givenOrderIsRequested_throwsInvalidState() {
        Order order = Order.request(CustomerId.of("customer-1"), DISPATCH, DELIVERY);

        assertThatThrownBy(() -> order.start(ProviderId.of("provider-1")))
            .isInstanceOf(InvalidOrderStateException.class);
    }
}
```

---

### Step 6: Add Integration Tests

Test the persistence and Kafka layers with Testcontainers:

```java
// integration/OrderRepositoryIntegrationTest.java
@Tag("integration")
class OrderRepositoryIntegrationTest extends BaseIntegrationTest {

    @Autowired OrderRepository orderRepository;

    @Test
    @Transactional
    void save_andFind_persistsOrder() {
        Order order = Order.request(CustomerId.of("customer-1"), DISPATCH, DELIVERY);

        orderRepository.save(order);

        Order found = orderRepository.findById(order.getId()).orElseThrow();
        assertThat(found.getStatus()).isEqualTo(OrderStatus.REQUESTED);
        assertThat(found.getCustomerId()).isEqualTo(CustomerId.of("customer-1"));
    }
}
```

---

### Step 7: Raise a Pull Request

```bash
git checkout -b feat/ORD-1001-request-order-endpoint
git add .
git commit -m "feat(orders): add order request endpoint"
git push origin feat/ORD-1001-request-order-endpoint
gh pr create --fill
```

The PR pipeline automatically runs:
- Checkstyle
- Unit tests
- Integration tests
- Coverage check (≥ 80%)
- Snyk scan
- SonarCloud analysis
- Docker build

**Estimated pipeline time: 8-10 minutes**

Get one review approval → merge.

---

### Step 8: Merge & Watch the Pipeline

On merge to `main`:

```
GitHub Actions → build + test → push image to ECR
                            → update image tag in platform-config
                            → ArgoCD detects change
                            → deploys to dev namespace
                            → smoke tests run
                            → promotes to staging
                            → E2E tests run
                            → deploys to production (canary)
                            → canary analysis passes
                            → full production rollout
```

Watch in real time:
- **GitHub Actions** — CI progress
- **ArgoCD UI** — deployment progress
- **Grafana** — service metrics appearing live

**Total time, merge to production: ~15 minutes**

---

### Step 9: Verify in Production

```bash
# Check the service is healthy in production
kubectl -n orders-production get pods
kubectl -n orders-production logs -l app=orders-service --tail=50

# Call the API (staging first)
curl -H "Authorization: Bearer $TOKEN" \
     https://api-staging.{company}.com/v1/orders

# Check Grafana dashboard
open https://grafana.{company}.internal/d/orders-service
```

---

### Step 10: Add a Runbook

Before you declare the service production-ready, write the runbook at `docs/runbook.md`:

```markdown
# Orders Service — Runbook

## Service Overview
[One paragraph]

## Common Alerts and Responses

### HighErrorRate
**Impact:** Customers cannot create orders
**First check:** Grafana → Orders dashboard → Error rate panel
**Common causes:**
  1. Database connection exhausted → check connection pool metrics
  2. Downstream service (fulfillment) is down → check fulfillment service health
**Resolution steps:**
  1. ...

### KafkaConsumerLagHigh
**Impact:** Order events delayed; downstream services (payments, notifications) lagging
**First check:** Kafka consumer group lag dashboard
...

## Escalation
On-call: PagerDuty → team-orders
Tech Lead: @lead-engineer
```

Link the runbook from `catalog-info.yaml`.

---

## Checklist: Is My Service Golden Path Compliant?

```
Repository & Code
[ ] Created via Backstage template (not manually)
[ ] catalog-info.yaml registered and accurate
[ ] CODEOWNERS file present
[ ] README complete (local dev instructions work)
[ ] OpenAPI spec at api/openapi.yaml
[ ] ADR raised for any non-standard decisions

Testing
[ ] Unit test coverage ≥ 80%
[ ] Integration tests use Testcontainers
[ ] ArchUnit tests present
[ ] Pact contracts defined for all consumer relationships

CI/CD
[ ] PR pipeline passes in < 10 minutes
[ ] All quality gates green (coverage, Snyk, Sonar)
[ ] No secrets in repository (Gitleaks clean)
[ ] Main pipeline deploys to dev automatically

Observability
[ ] Structured JSON logging with correlation IDs
[ ] Prometheus metrics endpoint active
[ ] Grafana dashboard created
[ ] At least one P1/P2 alert rule configured
[ ] All P1/P2 alerts have runbook links
[ ] Runbook complete at docs/runbook.md
[ ] Distributed tracing configured (OTel agent)

Production Readiness
[ ] Readiness and liveness probes configured
[ ] Graceful shutdown (30s drain) configured
[ ] HPA configured (min 3 replicas in production)
[ ] Resource requests and limits set
[ ] Pod anti-affinity across AZs
[ ] Feature flags for unreleased features
[ ] PagerDuty service configured
[ ] SLO defined and tracked
```

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
