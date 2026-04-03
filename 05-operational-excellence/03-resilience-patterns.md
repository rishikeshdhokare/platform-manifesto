# 🛡️ Resilience Patterns

![Status: Mandated](https://img.shields.io/badge/status-Mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 🎯 1. Why Resilience Matters

In a distributed system, **everything will fail eventually** — a downstream service, a database connection, a network blip. The question is not whether a dependency will fail, but whether your service fails gracefully when it does.

Without resilience patterns:
- One slow downstream service makes your service slow
- One unavailable downstream service makes your service unavailable
- A cascade of failures takes down the entire platform

With resilience patterns, a failing dependency causes **graceful degradation** — not a cascade.

---

## 🛡️ 2. Our Library: Resilience4j

We use **Resilience4j** for all resilience patterns. It is included in the platform BOM and integrates with Spring Boot and Micrometer automatically.

```kotlin
// build.gradle.kts — already in platform BOM
implementation("io.github.resilience4j:resilience4j-spring-boot3")
implementation("io.github.resilience4j:resilience4j-kotlin")
```

---

## 🛡️ 3. Circuit Breaker

### 3.1 What It Does

A circuit breaker monitors calls to a downstream dependency. When failures exceed a threshold, it "opens" — subsequent calls immediately return an error (or fallback) without hitting the failing service. After a wait period, it "half-opens" to test if the service has recovered.

```
CLOSED → calls go through normally
   │ failure rate > threshold
   ▼
OPEN → calls immediately fail (or use fallback)
   │ after wait duration
   ▼
HALF-OPEN → limited test calls go through
   │ success → CLOSED       │ failure → OPEN
```

### 3.2 Configuration

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      pricingService:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 20              # Evaluate last 20 calls
        failureRateThreshold: 50           # Open if > 50% fail
        waitDurationInOpenState: 30s       # Wait 30s before trying again
        permittedNumberOfCallsInHalfOpenState: 5
        automaticTransitionFromOpenToHalfOpenEnabled: true
        registerHealthIndicator: true      # Shows in /actuator/health

      fulfillmentService:
        slidingWindowSize: 10
        failureRateThreshold: 30           # More sensitive — fulfillment is critical
        waitDurationInOpenState: 10s
        slowCallRateThreshold: 50          # Also open if > 50% of calls are slow
        slowCallDurationThreshold: 500ms   # "Slow" = > 500ms
```

### 3.3 Usage in Code

```java
@Service
public class OrderService {

    private final PricingClient pricingClient;
    private final CircuitBreakerRegistry circuitBreakerRegistry;

    // ✅ Option 1: Annotation (simplest)
    @CircuitBreaker(name = "pricingService", fallbackMethod = "getDefaultPrice")
    public PriceEstimate getPriceEstimate(OrderRequest request) {
        return pricingClient.estimate(request);
    }

    // Fallback — called when circuit is open or call fails
    // Must have same signature + exception parameter
    private PriceEstimate getDefaultPrice(OrderRequest request, Exception ex) {
        log.warn("Pricing service unavailable, using default price. cause={}", ex.getMessage());
        return PriceEstimate.defaultEstimate(request.getRegion());
    }
}
```

```java
// ✅ Option 2: Programmatic (more control)
@Service
public class OrderService {

    private final CircuitBreaker pricingCircuitBreaker;
    private final PricingClient pricingClient;

    public OrderService(CircuitBreakerRegistry registry, PricingClient pricingClient) {
        this.pricingCircuitBreaker = registry.circuitBreaker("pricingService");
        this.pricingClient = pricingClient;
    }

    public PriceEstimate getPriceEstimate(OrderRequest request) {
        return pricingCircuitBreaker.executeSupplier(
            () -> pricingClient.estimate(request)
        );
    }
}
```

### 3.4 When to Use a Fallback vs Let It Fail

| Situation | Use Fallback? | Fallback Strategy |
|-----------|--------------|------------------|
| Pricing service unavailable for price estimate | Yes | Return default estimate with disclaimer |
| Provider profile unavailable during order completion | No — critical data | Let it fail; retry later |
| Notification service unavailable | Yes | Queue for retry; don't fail the order |
| Payment service unavailable | No — cannot proceed | Fail and return error to user |

---

## 🛡️ 4. Retry

### 4.1 What It Does

Automatically retries a failed call. Useful for transient failures — network blips, temporary service unavailability.

**Only retry idempotent operations.** Retrying a `POST /payments` that may have partially succeeded causes double charging.

### 4.2 Configuration

```yaml
resilience4j:
  retry:
    instances:
      pricingService:
        maxAttempts: 3
        waitDuration: 500ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2    # 500ms → 1000ms → 2000ms
        retryExceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
          - org.springframework.web.client.ResourceAccessException
        ignoreExceptions:
          - com.{company}.platform.exception.OrderNotFoundException   # Don't retry 404s
          - com.{company}.platform.exception.InvalidRequestException  # Don't retry 400s
```

### 4.3 Usage

```java
// ✅ Retry with exponential backoff — for idempotent GET calls
@Retry(name = "pricingService", fallbackMethod = "getDefaultPrice")
@CircuitBreaker(name = "pricingService", fallbackMethod = "getDefaultPrice")
public PriceEstimate getPriceEstimate(OrderRequest request) {
    return pricingClient.estimate(request);  // GET — safe to retry
}

// ✅ Do NOT retry POST payment — not idempotent
// Instead, use idempotency keys and let the caller retry
public PaymentResult capturePayment(String idempotencyKey, PriceAmount price) {
    // No @Retry here — payment capture is not idempotent without explicit key handling
    return paymentGateway.capture(idempotencyKey, price);
}
```

### 4.4 Combining Retry + Circuit Breaker

Apply them together — retry first, circuit breaker outside:

```java
// Order matters: retry is inner (executes multiple times), CB is outer (tracks outcomes)
@Retry(name = "fulfillmentService")
@CircuitBreaker(name = "fulfillmentService", fallbackMethod = "fallback")
public AssignmentResult assignProvider(OrderRequest request) {
    return fulfillmentClient.findBestProvider(request);
}
```

---

## 🛡️ 5. Timeout

### 5.1 Why Timeouts Are Non-Negotiable

A service without timeouts can wait indefinitely for a hung dependency. This ties up threads and eventually crashes the service even if the dependency only affects a small percentage of calls.

**Every outbound call must have an explicit timeout.**

### 5.2 Configuration

```yaml
resilience4j:
  timelimiter:
    instances:
      pricingService:
        timeoutDuration: 2s         # Internal service — 2s max
        cancelRunningFuture: true

      geolocationService:
        timeoutDuration: 5s         # External service — more lenient
```

### 5.3 HTTP Client Timeouts (Separate from Resilience4j)

For Spring's `RestClient` / `RestTemplate`, also set timeouts at the HTTP client level:

```java
@Bean
public RestClient pricingServiceRestClient() {
    HttpClient httpClient = HttpClient.create()
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 1000)  // 1s connect timeout
        .responseTimeout(Duration.ofSeconds(2));              // 2s read timeout

    return RestClient.builder()
        .requestFactory(new ReactorClientHttpRequestFactory(httpClient))
        .baseUrl("http://pricing-service.pricing.svc.cluster.local")
        .build();
}
```

---

## 🛡️ 6. Bulkhead

### 6.1 What It Does

A bulkhead isolates failures. If one downstream service is slow and consuming all available threads, it doesn't starve other downstream calls.

Think of it like watertight compartments in a ship — one compartment flooding doesn't sink the whole ship.

### 6.2 Configuration

```yaml
resilience4j:
  bulkhead:
    instances:
      pricingService:
        maxConcurrentCalls: 20       # Max 20 concurrent calls to pricing
        maxWaitDuration: 100ms       # Wait up to 100ms for a slot; then reject

      fulfillmentService:
        maxConcurrentCalls: 50       # Fulfillment handles more concurrency
        maxWaitDuration: 0           # No wait — reject immediately if full
```

### 6.3 Usage

```java
@Bulkhead(name = "pricingService", type = Bulkhead.Type.SEMAPHORE)
@CircuitBreaker(name = "pricingService", fallbackMethod = "getDefaultPrice")
public PriceEstimate getPriceEstimate(OrderRequest request) {
    return pricingClient.estimate(request);
}
```

---

## 🧩 7. Complete Example: Calling the Fulfillment Service

Here's a full, production-ready example combining all patterns:

```java
@Service
public class OrderDispatchService {

    private static final Logger log = LoggerFactory.getLogger(OrderDispatchService.class);

    private final FulfillmentServiceClient fulfillmentClient;
    private final OrderRepository orderRepository;

    public OrderDispatchService(FulfillmentServiceClient fulfillmentClient,
                                OrderRepository orderRepository) {
        this.fulfillmentClient = fulfillmentClient;
        this.orderRepository = orderRepository;
    }

    /**
     * Finds the best available provider for an order request.
     *
     * Resilience stack (inner to outer):
     *   1. Timeout: cancel if takes > 2s
     *   2. Retry: retry up to 3x with backoff on transient errors
     *   3. Bulkhead: max 50 concurrent calls to fulfillment service
     *   4. Circuit breaker: open after 30% failure rate
     */
    @TimeLimiter(name = "fulfillmentService")
    @Retry(name = "fulfillmentService")
    @Bulkhead(name = "fulfillmentService", type = Bulkhead.Type.SEMAPHORE)
    @CircuitBreaker(name = "fulfillmentService", fallbackMethod = "handleFulfillmentUnavailable")
    public AssignmentResult assignProvider(Order order) {
        log.info("Finding provider for order. orderId={}, region={}", order.getId(), order.getRegion());
        return fulfillmentClient.findBestProvider(new AssignmentRequest(
            order.getId().value(),
            order.getDispatch(),
            order.getServiceType()
        ));
    }

    // Called when circuit is open OR all retries exhausted
    private AssignmentResult handleFulfillmentUnavailable(Order order, Exception ex) {
        log.error("Fulfillment service unavailable. orderId={}", order.getId(), ex);

        // Update order status so the customer knows and can retry
        order.markAsFulfillmentFailed();
        orderRepository.save(order);

        // Don't silently fail — throw so the caller knows
        throw new FulfillmentServiceUnavailableException(
            "Unable to find provider — fulfillment service is currently unavailable");
    }
}
```

---

## 👁️ 8. Observability for Resilience

Resilience4j automatically exports metrics to Prometheus. Monitor these:

| Metric | What It Tells You |
|--------|--------------------|
| `resilience4j_circuitbreaker_state{name,state}` | Current state of each circuit breaker |
| `resilience4j_circuitbreaker_failure_rate` | % of calls failing — rising trend is a warning |
| `resilience4j_retry_calls_total{kind="failed_with_retry"}` | How often retries are needed |
| `resilience4j_bulkhead_available_concurrent_calls` | How close you are to bulkhead limit |

Add these to your Grafana dashboard. A circuit breaker transitioning to OPEN is a P2 alert.

```yaml
# Alert: circuit breaker opened
- alert: CircuitBreakerOpen
  expr: resilience4j_circuitbreaker_state{state="open"} == 1
  for: 1m
  severity: P2
  annotations:
    summary: "Circuit breaker {{ $labels.name }} is OPEN on {{ $labels.service }}"
```

---

## 📋 9. Quick Reference

| Pattern | Problem It Solves | When to Use |
|---------|------------------|-------------|
| **Circuit Breaker** | Slow/failing downstream causes cascade failure | All synchronous outbound calls |
| **Retry** | Transient network / service blips | Idempotent GET calls; be careful with POSTs |
| **Timeout** | Hung downstream ties up threads indefinitely | All outbound calls — no exceptions |
| **Bulkhead** | One slow downstream starves other calls | When you call multiple different downstreams |
| **Fallback** | Return something useful when downstream fails | When degraded response is better than error |

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
