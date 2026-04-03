# Observability Standards

> **Status:** Mandated  
> **Owner:** Platform Engineering  
> **Last Updated:** 2025

---

## 1. Philosophy

You cannot operate what you cannot observe. Observability is not a feature — it is a prerequisite for production deployment. Every service must be observable from day one, not retrofitted after an incident.

**The three pillars:**
- **Logs** — what happened (events)
- **Metrics** — how the system is behaving (aggregates over time)
- **Traces** — where time was spent (request path)

All three must be in place before a service receives production traffic. The **fourth pillar** — alerting — is only valuable when the first three are right.

---

## 2. Logging

### 2.1 Standard

All services must emit **structured JSON logs** to stdout. The log shipper (Fluent Bit) collects from stdout and forwards to the log backend — applications never write to files.

### 2.2 Required Fields

Every log line must contain:

```json
{
  "timestamp": "2024-11-15T14:30:00.123Z",
  "level": "INFO",
  "service": "orders-service",
  "version": "2.14.3",
  "environment": "production",
  "traceId": "01ARZ3NDEKTSV4RRFFQ69G5FAV",
  "spanId": "1fcb4b0a9c3eb4f8",
  "correlationId": "req_01HXYZ...",
  "message": "Order completed successfully",
  "orderId": "order_abc123",
  "durationMs": 145
}
```

| Field | Source | Required |
|-------|--------|---------|
| `timestamp` | Logback | Yes |
| `level` | Logback | Yes |
| `service` | `spring.application.name` | Yes |
| `version` | `application.version` from build | Yes |
| `environment` | `SPRING_PROFILES_ACTIVE` | Yes |
| `traceId` | OpenTelemetry (auto-injected) | Yes |
| `spanId` | OpenTelemetry (auto-injected) | Yes |
| `correlationId` | MDC (from `X-Request-ID` header) | Yes |
| `message` | Application | Yes |
| Domain fields | Application | As needed |

### 2.3 Logback Configuration

Use the platform-standard `logback-spring.xml`. Include it from the platform BOM:

```xml
<!-- logback-spring.xml -->
<configuration>
  <include resource="logback/platform-json.xml"/>

  <root level="INFO">
    <appender-ref ref="JSON_CONSOLE"/>
  </root>

  <!-- Suppress noisy Spring Boot startup logs -->
  <logger name="org.springframework" level="WARN"/>
  <logger name="org.hibernate" level="WARN"/>
</configuration>
```

### 2.4 Logging Rules

| Rule | Detail |
|------|--------|
| **No sensitive data in logs** | No passwords, tokens, card numbers, PII (names, email, phone) |
| **No stack traces in INFO/WARN** | Stack traces are ERROR level only |
| **No log-and-throw** | Don't log an exception and then throw it — it doubles log noise |
| **Use parameterised logging** | `log.info("Order {} completed", orderId)` not string concat |
| **Log business events** | Order started, payment captured — these are invaluable for debugging |
| **Do not log in a tight loop** | Rate-limit or aggregate high-frequency events |

### 2.5 Log Levels in Production

| Level | When to Use |
|-------|------------|
| `ERROR` | Unhandled exception; service cannot fulfil request |
| `WARN` | Handled error; degraded operation; slow dependency |
| `INFO` | Business events (order started, payment captured); service lifecycle |
| `DEBUG` | Detailed diagnostic info — **disabled in production by default** |

DEBUG can be enabled per-service via a runtime feature flag without redeployment.

### 2.6 Log Backend

- **Amazon OpenSearch Service** — searchable log storage, 30-day hot retention, 90-day cold
- **Fluent Bit** — DaemonSet on EKS, collects from all pod stdout, forwards to OpenSearch
- **Grafana** — log queries via the Loki datasource (for dev/staging; OpenSearch for production)

---

## 3. Metrics

### 3.1 Standard

All services expose a **Prometheus-compatible `/actuator/prometheus` endpoint**. Prometheus scrapes this endpoint via a `ServiceMonitor` CRD (Prometheus Operator).

### 3.2 Spring Boot Actuator Setup

Include in `build.gradle`:
```kotlin
implementation("org.springframework.boot:spring-boot-starter-actuator")
implementation("io.micrometer:micrometer-registry-prometheus")
```

`application.yml`:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics
  metrics:
    tags:
      service: ${spring.application.name}
      environment: ${spring.profiles.active}
      version: ${application.version}
```

### 3.3 RED Method — Required Metrics Per Service

Every HTTP service must expose these three metric families (Micrometer provides them automatically for Spring Boot):

| Metric Type | Metric Name | Description |
|-------------|-------------|-------------|
| **R**ate | `http_server_requests_seconds_count` | Requests per second |
| **E**rrors | `http_server_requests_seconds_count{status=~"5.."}` | Error rate |
| **D**uration | `http_server_requests_seconds` (histogram) | Latency distribution |

### 3.4 Business Metrics (Mandatory for Core Services)

Beyond RED, core services must instrument key business events:

```java
// In OrderService.java
@Autowired MeterRegistry meterRegistry;

public Order completeOrder(String orderId) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    order.complete();
    orderRepository.save(order);

    // Business metric
    meterRegistry.counter("orders.completed",
        "region", order.getRegion(),
        "service_type", order.getServiceType().name()
    ).increment();

    meterRegistry.timer("orders.duration",
        "service_type", order.getServiceType().name()
    ).record(order.getDurationSeconds(), TimeUnit.SECONDS);

    return order;
}
```

### 3.5 Kafka Consumer Metrics

All Kafka consumers must expose consumer lag metrics. The platform Kafka configuration exports these automatically via JMX → Prometheus.

Key metric: `kafka_consumer_group_lag` — alerts when lag exceeds threshold.

### 3.6 Grafana Dashboards

Every service **must** have a Grafana dashboard before production deployment. The platform team provides a **standard dashboard template** — teams import it and customise.

Minimum dashboard panels:
1. Request rate (req/s)
2. Error rate (%)
3. P50 / P95 / P99 latency
4. JVM heap usage
5. Active threads / virtual threads
6. Database connection pool utilisation
7. Kafka consumer lag (if applicable)
8. Downstream dependency health (circuit breaker state)

Dashboards are stored as JSON in the service repository at `docs/dashboards/` and provisioned via Grafana's Terraform provider.

---

## 4. Distributed Tracing

### 4.1 Standard

**OpenTelemetry** is the standard instrumentation library. Traces are exported to **AWS X-Ray**.

### 4.2 Setup

Add the OpenTelemetry Java agent — no code changes required for basic tracing:

```dockerfile
# Dockerfile
COPY --from=ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:latest \
     /javaagent.jar /app/javaagent.jar

ENTRYPOINT ["java", \
  "-javaagent:/app/javaagent.jar", \
  "-jar", "/app/service.jar"]
```

Environment variables (set in Helm values):
```yaml
env:
  OTEL_SERVICE_NAME: orders-service
  OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector.platform.svc.cluster.local:4318
  OTEL_PROPAGATORS: tracecontext,b3multi
  OTEL_RESOURCE_ATTRIBUTES: deployment.environment=production
```

### 4.3 Trace Propagation

- All inbound HTTP requests: Extract `traceparent` header (W3C TraceContext)
- All outbound HTTP calls: Inject `traceparent` header automatically (via OTel agent)
- All Kafka messages: Propagate trace context in message headers

The correlation ID (`X-Request-ID` / `X-Correlation-ID`) is bridged into the trace context automatically by the platform logging configuration.

### 4.4 Custom Spans

For important business operations, add custom spans:

```java
@Autowired Tracer tracer;

public AssignmentResult assignProvider(OrderRequest request) {
    Span span = tracer.spanBuilder("fulfillment.assignProvider")
        .setAttribute("region", request.getRegion())
        .setAttribute("service_type", request.getServiceType())
        .startSpan();
    try (var scope = span.makeCurrent()) {
        return doAssignment(request);
    } catch (Exception e) {
        span.recordException(e);
        span.setStatus(StatusCode.ERROR);
        throw e;
    } finally {
        span.end();
    }
}
```

---

## 5. Alerting

### 5.1 Philosophy

**Alert on symptoms, not causes.** High error rate is a symptom (alert on it). High CPU is a cause (usually don't alert on it — alert on the downstream effect like slow response times).

An alert should always answer: **"What is the user impact?"**

### 5.2 Alerting Stack

- **Prometheus** — evaluates alert rules
- **Grafana Alerting** — alert routing and deduplication
- **PagerDuty** — on-call escalation for P1/P2 alerts
- **Slack** — notification for P3/P4 alerts

### 5.3 Severity Levels

| Severity | Definition | Response SLA | Channel |
|----------|-----------|-------------|---------|
| **P1** | Production down or severely degraded; significant user impact | Acknowledge in 5 min, resolve in 30 min | PagerDuty + Slack |
| **P2** | Degraded performance; partial user impact | Acknowledge in 15 min, resolve in 2 hours | PagerDuty + Slack |
| **P3** | Non-critical issue; no immediate user impact | Next business day | Slack only |
| **P4** | Warning threshold; monitor | Weekly review | Dashboard only |

### 5.4 Standard Alert Rules (Platform-Provided)

The platform ships a `standard-alerts` Prometheus rule template. Every service automatically gets:

```yaml
# Automatically applied to all services
groups:
- name: service-standard-alerts
  rules:
  - alert: HighErrorRate
    expr: |
      rate(http_server_requests_seconds_count{status=~"5..",service="{{ .service }}"}[5m])
      / rate(http_server_requests_seconds_count{service="{{ .service }}"}[5m]) > 0.01
    for: 2m
    severity: P1

  - alert: HighLatency
    expr: |
      histogram_quantile(0.99, rate(http_server_requests_seconds_bucket{service="{{ .service }}"}[5m])) > 1.0
    for: 5m
    severity: P2

  - alert: PodCrashLooping
    expr: |
      rate(kube_pod_container_status_restarts_total{namespace="{{ .namespace }}"}[15m]) > 0
    severity: P1

  - alert: KafkaConsumerLagHigh
    expr: |
      kafka_consumer_group_lag{service="{{ .service }}"} > 10000
    for: 5m
    severity: P2
```

### 5.5 Runbook Requirement

**Every P1 and P2 alert must have a runbook.** The alert annotation must link to it:

```yaml
annotations:
  summary: "High error rate on {{ $labels.service }}"
  runbook: "https://wiki.{company}.internal/runbooks/{{ $labels.service }}#high-error-rate"
```

A P1 alert without a runbook is treated as an observability bug.

---

## 6. SLOs and Error Budgets

### 6.1 Every Core Service Must Define SLOs

| Service | Availability SLO | Latency SLO (P99) |
|---------|-----------------|-------------------|
| Orders Service | 99.9% | < 500ms |
| Fulfillment Engine | 99.95% | < 200ms |
| Payment Service | 99.99% | < 1000ms |
| Pricing Service | 99.9% | < 100ms |

### 6.2 Error Budget Tracking

- SLO compliance and error budget burn rate are tracked in Grafana (SLO dashboard)
- When error budget is 50% consumed in a 30-day window — team is alerted
- When error budget is exhausted — new feature work pauses until reliability is restored
- Error budget policy is reviewed quarterly by the CTO

---

## 7. Business Metrics & Product Analytics

System observability (RED metrics, SLOs) tells you if the platform is healthy. Business observability tells you if the **product** is healthy. Both are required.

### 7.1 Business Metrics Are Not System Metrics

| Concern | System Metric | Business Metric |
|---------|--------------|-----------------|
| Orders | `http_server_requests_seconds_count` (request rate) | `orders.requested` / `orders.completed` (conversion rate) |
| Fulfillment | `fulfillment.assignProvider` span duration | Assignment success rate, average wait time |
| Payments | Payment service error rate | Payment success rate, average price, revenue per hour |
| Providers | Provider service pod count | Active providers per region per hour, provider utilization % |

System metrics are owned by the service team. Business metrics are owned jointly by engineering and product.

### 7.2 Key Business Events to Instrument

Every core domain must emit business events as Micrometer metrics:

```java
// In OrderService — business metric
meterRegistry.counter("business.orders.requested",
    "region", order.getRegion(),
    "service_type", order.getServiceType().name()
).increment();

meterRegistry.counter("business.orders.completed",
    "region", order.getRegion()
).increment();

// Conversion rate = orders.completed / orders.requested (computed in Grafana)
```

### 7.3 Standard Business Dashboard Panels

Every core service Grafana dashboard must include a **Business** section with:

| Panel | Metric | Purpose |
|-------|--------|---------|
| Order conversion rate | completed / requested | Are customers successfully completing orders? |
| Average wait time | Time from requested to assigned | Is fulfillment fast enough? |
| Provider utilization | Active orders / online providers | Is supply meeting demand? |
| Revenue per hour | Sum of prices per hour by region | Business health signal |
| Cancellation rate | cancelled / requested by reason | UX or supply problem? |
| Dynamic pricing frequency | % of orders with dynamic pricing > 1.0 | Pricing health |

### 7.4 Data Freshness SLA

Business metrics must be available in dashboards within the following windows:

| Metric Type | Freshness Target | Pipeline |
|-------------|-----------------|----------|
| Real-time counters (Prometheus) | < 30 seconds | Direct scrape |
| Kafka-derived analytics | < 5 minutes | Kafka → analytics consumer |
| CDC-derived warehouse metrics | < 30 minutes | Debezium → S3 → Redshift |
| Daily aggregates | < 2 hours after midnight UTC | Scheduled Glue ETL |

### 7.5 Product Analytics vs Engineering Dashboards

Product analytics dashboards live in **Amazon QuickSight** (connected to Redshift), not Grafana. Grafana is for operational visibility; QuickSight is for product decision-making.

The boundary is clear:
- **Grafana**: "Is the system healthy right now?" (engineering, on-call)
- **QuickSight**: "How did the product perform this week/month?" (product, leadership)

Both use the same underlying data — CDC events flow to Redshift, Prometheus metrics flow to Grafana. No duplicate instrumentation.

---

## 8. Event Catalog

### 8.1 Why an Event Catalog

As the number of Kafka topics, event schemas, and consuming services grows, it becomes increasingly difficult to answer basic questions: "Who publishes this event?", "Who consumes it?", "What does the payload look like?", "Is this event deprecated?"

An **event catalog** is a browsable, searchable, always-up-to-date registry of all domain events in the platform.

### 8.2 Tooling: EventCatalog

We use [EventCatalog](https://eventcatalog.dev) as our event documentation platform.

| Concern | Implementation |
|---------|---------------|
| **Source of truth** | EventCatalog site, generated from event definitions in Git |
| **Schema registry** | AWS Glue Schema Registry (Avro) — EventCatalog links to schema versions |
| **Hosting** | Deployed as a static site to internal CDN (e.g., `events.{company}.internal`) |
| **CI integration** | EventCatalog is rebuilt on every merge to the events repo |

### 8.3 What Gets Cataloged

Every domain event must have an entry containing:

| Field | Description |
|-------|-------------|
| **Event name** | e.g., `orders.order.completed` |
| **Domain** | The bounded context that publishes the event |
| **Producer** | The service that publishes it |
| **Consumers** | All known downstream consumers |
| **Schema** | Link to the Avro schema in Glue Schema Registry (with version) |
| **Payload example** | A realistic JSON example |
| **Retention** | Kafka topic retention period |
| **SLA** | Expected publishing latency (e.g., "within 500ms of state change") |
| **Deprecation status** | Active / deprecated / sunset date |

### 8.4 Event Documentation as Code

Event definitions live alongside the service code. Each service's repository contains an `events/` directory:

```
orders-service/
├── src/
├── events/
│   ├── orders.order.requested.yaml
│   ├── orders.order.completed.yaml
│   └── orders.order.cancelled.yaml
└── ...
```

Example event definition:

```yaml
name: orders.order.completed
version: 2.1.0
summary: Published when an order reaches terminal completed state
domain: Orders
producers:
  - orders-service
consumers:
  - payment-service
  - notifications-service
  - analytics-pipeline
schema:
  type: avro
  registry: glue
  subject: orders.order.completed-value
retention: 30d
sla: "< 500ms from state transition"
```

### 8.5 Governance

| Rule | Enforcement |
|------|-------------|
| Every new Kafka topic must have an EventCatalog entry | CI check blocks PR if topic exists without matching `.yaml` |
| Schema changes must update the catalog entry | PR template includes checklist item |
| Deprecated events must have a sunset date | Quarterly audit of catalog for stale entries |
| Consumer list must be kept current | Consuming teams are responsible for adding themselves |

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
