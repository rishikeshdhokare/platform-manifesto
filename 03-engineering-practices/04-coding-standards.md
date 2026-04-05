# 📝 Coding Standards

![Status: Mandated](https://img.shields.io/badge/status-mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Philosophy

Code is read far more than it is written. Every decision in this guide optimises for **readability, predictability, and maintainability** - not brevity or cleverness.

When in doubt, ask: *"Will a colleague who has never seen this code understand it in 30 seconds?"* If the answer is no, simplify.

---

## 🔄 2. Automated Enforcement

**Principle:** Use your language's **canonical style guide and formatter**, and enforce it in **pre-commit and CI** so reviews focus on behaviour, not whitespace. Add static analysis for complexity, duplication, and (where available) architectural boundaries.

Most style rules are enforced automatically. You do not need to memorise them - your IDE and CI will tell you when you're wrong.

**Reference implementation (Java):**

| Tool | What It Enforces | When It Runs |
|------|-----------------|-------------|
| **Checkstyle** (Google Java Style) | Formatting, import order, whitespace | Pre-commit + CI |
| **SonarCloud** | Code smells, complexity, duplication | CI on every PR |
| **ArchUnit** | Package dependencies, naming conventions | CI on every PR |

> **Substitution point:** Swap Checkstyle for ESLint/Ruff/gofmt/etc.; swap ArchUnit for an equivalent boundary checker in your ecosystem (see hexagonal architecture doc). Import your IDE's formatter settings for on-save formatting (for example platform IntelliJ settings for Java).

The rules below cover things tools **cannot** enforce - judgment calls that require human standards.

---

## 📏 3. Naming

### 3.1 General Rules

| Element | Convention | Example |
|---------|-----------|---------|
| Class | `UpperCamelCase` | `OrderService`, `PaymentGatewayAdapter` |
| Interface | `UpperCamelCase` (no `I` prefix) | `OrderRepository`, `PaymentGateway` |
| Method | `lowerCamelCase`, verb | `findById`, `calculatePrice`, `publishEvent` |
| Variable | `lowerCamelCase`, noun | `orderId`, `priceAmount`, `activeProviders` |
| Constant | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_ATTEMPTS`, `DEFAULT_TIMEOUT_MS` |
| Package | lowercase, dot-separated | `com.{company}.orders.domain` |
| Test class | `{ClassUnderTest}Test` | `OrderServiceTest` |
| Test method | `methodName_givenCondition_expectedBehaviour` | `calculatePrice_givenDynamicPricing_returnsMultiplied` |

### 3.2 Be Specific - No Generic Names

Generic names hide intent and make code harder to search and understand.

**Reference implementation (Java):**

```java
// ❌ Bad - what is "data"? what is "result"? what is "obj"?
public Object process(Object data) {
    Object result = service.handle(data);
    return result;
}

// ✅ Good - names communicate intent
public PriceEstimate calculatePrice(OrderRequest orderRequest) {
    PriceEstimate priceEstimate = pricingService.estimate(orderRequest);
    return priceEstimate;
}
```

### 3.3 Booleans - Use Positive, Readable Names

**Reference implementation (Java):**

```java
// ❌ Bad
boolean notActive;
boolean flag;
boolean data;

// ✅ Good
boolean isActive;
boolean hasProvider;
boolean isDynamicPricingEnabled;
```

### 3.4 Collections - Use Plural Nouns

**Reference implementation (Java):**

```java
// ❌ Bad
List<Provider> providerList;
Set<String> stringSet;

// ✅ Good
List<Provider> providers;
Set<String> activeZones;
```

---

## 📏 4. Classes

### 4.1 Single Responsibility

A class should have one reason to change. If you find yourself writing "and" when describing what a class does, it probably needs to be split.

**Reference implementation (Java):**

```java
// ❌ Bad - OrderService does too many things
public class OrderService {
    public Order createOrder(...) { ... }
    public void sendConfirmationEmail(...) { ... }   // notification concern
    public PriceEstimate calculatePrice(...) { ... }   // pricing concern
    public void updateProviderLocation(...) { ... }    // location concern
}

// ✅ Good - each class has one job
public class OrderService {
    public Order requestOrder(OrderRequest request) { ... }
    public void startOrder(OrderId orderId) { ... }
    public void completeOrder(OrderId orderId) { ... }
}
```

### 4.2 Class Size

- **Domain classes:** No hard limit, but if a class exceeds ~200 lines, review it
- **Service classes:** Should rarely exceed 150 lines; extract collaborators
- **Controllers:** Should be thin - no business logic; typically < 80 lines

### 4.3 Immutability - Prefer It

Make objects immutable where possible. Mutable shared state is the root of most concurrency bugs.

**Reference implementation (Java):**

```java
// ❌ Bad - mutable, anyone can change anything
public class OrderRequest {
    public String customerId;
    public Location dispatch;
    public Location delivery;
}

// ✅ Good - immutable value object
public final class OrderRequest {
    private final CustomerId customerId;
    private final Location dispatch;
    private final Location delivery;

    public OrderRequest(CustomerId customerId, Location dispatch, Location delivery) {
        this.customerId = Objects.requireNonNull(customerId, "customerId must not be null");
        this.dispatch = Objects.requireNonNull(dispatch, "dispatch must not be null");
        this.delivery = Objects.requireNonNull(delivery, "delivery must not be null");
    }

    // Only getters - no setters
    public CustomerId getCustomerId() { return customerId; }
    public Location getDispatch() { return dispatch; }
    public Location getDelivery() { return delivery; }
}
```

---

## 📏 5. Methods

### 5.1 Method Length

- **Target:** < 20 lines per method
- **Limit:** 40 lines - if exceeded, extract sub-methods
- A method that needs a comment to explain each "section" should be broken into named sub-methods

### 5.2 Number of Parameters

- **Target:** ≤ 3 parameters
- **Limit:** 5 - beyond this, introduce a parameter object

**Reference implementation (Java):**

```java
// ❌ Bad - 6 parameters, easy to pass in wrong order
public PriceEstimate calculate(String city, String vehicleType,
    double distanceKm, int durationMinutes,
    boolean isDynamic, double dynamicMultiplier) { ... }

// ✅ Good - parameter object groups related data
public PriceEstimate calculate(PriceCalculationRequest request) { ... }

public record PriceCalculationRequest(
    String city,
    VehicleType vehicleType,
    double distanceKm,
    int durationMinutes,
    DynamicPricingContext dynamicPricingContext
) {}
```

### 5.3 Return Early - Avoid Deep Nesting

**Reference implementation (Java):**

```java
// ❌ Bad - deeply nested, hard to follow the happy path
public Order processOrder(String orderId, String providerId) {
    Order order = orderRepository.findById(orderId);
    if (order != null) {
        if (order.getStatus() == OrderStatus.REQUESTED) {
            Provider provider = providerRepository.findById(providerId);
            if (provider != null) {
                if (provider.isAvailable()) {
                    order.assign(provider);
                    return orderRepository.save(order);
                } else {
                    throw new ProviderUnavailableException(providerId);
                }
            } else {
                throw new ProviderNotFoundException(providerId);
            }
        } else {
            throw new InvalidOrderStateException(order.getStatus());
        }
    } else {
        throw new OrderNotFoundException(orderId);
    }
}

// ✅ Good - guard clauses, happy path is linear
public Order processOrder(String orderId, String providerId) {
    Order order = orderRepository.findById(orderId)
        .orElseThrow(() -> new OrderNotFoundException(orderId));

    if (order.getStatus() != OrderStatus.REQUESTED) {
        throw new InvalidOrderStateException(order.getStatus());
    }

    Provider provider = providerRepository.findById(providerId)
        .orElseThrow(() -> new ProviderNotFoundException(providerId));

    if (!provider.isAvailable()) {
        throw new ProviderUnavailableException(providerId);
    }

    order.assign(provider);
    return orderRepository.save(order);
}
```

### 5.4 No Magic Numbers or Strings

**Reference implementation (Java):**

```java
// ❌ Bad - what does 3 mean? what is "COMPLETED"?
if (order.getAttempts() > 3) { ... }
if (status.equals("COMPLETED")) { ... }

// ✅ Good
private static final int MAX_DISPATCH_ATTEMPTS = 3;

if (order.getAttempts() > MAX_DISPATCH_ATTEMPTS) { ... }
if (status == OrderStatus.COMPLETED) { ... }
```

---

## ❌ 6. Error Handling

### 6.1 Use Typed Exceptions - Never Generic Ones

**Reference implementation (Java):**

```java
// ❌ Bad
throw new RuntimeException("Order not found");
throw new Exception("Something went wrong");

// ✅ Good - typed, domain-specific exceptions
throw new OrderNotFoundException(orderId);
throw new InvalidOrderStateException(currentStatus);
throw new ProviderUnavailableException(providerId);
```

### 6.2 Exception Hierarchy

All domain exceptions extend from a base:

**Reference implementation (Java):**

```java
// Base exception
public abstract class DomainException extends RuntimeException {
    private final String errorCode;

    protected DomainException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
}

// Specific exceptions
public class OrderNotFoundException extends DomainException {
    public OrderNotFoundException(OrderId orderId) {
        super("ORDERS.ORDER.NOT_FOUND", "Order not found: " + orderId.value());
    }
}

public class InvalidOrderStateException extends DomainException {
    public InvalidOrderStateException(OrderStatus currentStatus) {
        super("ORDERS.ORDER.INVALID_STATE",
            "Operation not permitted in state: " + currentStatus);
    }
}
```

> Error codes follow the format defined in the error catalog (see [`02-architecture-and-api/09-error-catalog.md`](../02-architecture-and-api/09-error-catalog.md)).

### 6.3 Never Swallow Exceptions

**Reference implementation (Java):**

```java
// ❌ Bad - exception silently swallowed; caller has no idea what happened
try {
    orderService.completeOrder(orderId);
} catch (Exception e) {
    // do nothing
}

// ❌ Also bad - logged but still swallowed; flow continues incorrectly
try {
    orderService.completeOrder(orderId);
} catch (Exception e) {
    log.error("Error completing order", e);
}

// ✅ Good - handle it properly or let it propagate
try {
    orderService.completeOrder(orderId);
} catch (OrderNotFoundException e) {
    return ResponseEntity.notFound().build();
} catch (InvalidOrderStateException e) {
    return ResponseEntity.status(HttpStatus.CONFLICT)
        .body(errorResponse(e.getErrorCode(), e.getMessage()));
}
```

### 6.4 Log-and-Throw Is an Anti-Pattern

**Reference implementation (Java):**

```java
// ❌ Bad - this exception will be logged twice: here and by the caller
try {
    providerService.dispatch(providerId, orderId);
} catch (Exception e) {
    log.error("Failed to dispatch provider", e);  // log here
    throw e;                                        // and rethrow - double logging
}

// ✅ Good - let it propagate; the top-level handler logs it once
providerService.dispatch(providerId, orderId);
```

---

## 📏 7. Null Handling

### 7.1 Never Return Null from a Method

**Reference implementation (Java):**

```java
// ❌ Bad - caller must remember to null-check; easy to forget
public Provider findProvider(String providerId) {
    return providerRepository.findById(providerId); // returns null if not found
}

// ✅ Good - use Optional; forces caller to handle the empty case
public Optional<Provider> findProvider(String providerId) {
    return providerRepository.findById(providerId);
}

// ✅ Also good - throw if not finding is exceptional (not a normal case)
public Provider getProvider(String providerId) {
    return providerRepository.findById(providerId)
        .orElseThrow(() -> new ProviderNotFoundException(providerId));
}
```

### 7.2 Validate Constructor Arguments

**Reference implementation (Java):**

```java
// ✅ Good - fail fast; don't let invalid objects exist
public Order(OrderId id, CustomerId customerId, Location dispatch, Location delivery) {
    this.id = Objects.requireNonNull(id, "id must not be null");
    this.customerId = Objects.requireNonNull(customerId, "customerId must not be null");
    this.dispatch = Objects.requireNonNull(dispatch, "dispatch must not be null");
    this.delivery = Objects.requireNonNull(delivery, "delivery must not be null");
}
```

### 7.3 Use `@NonNull` / `@Nullable` Annotations

Annotate method parameters and return types to communicate intent:

> **Substitution point:** Use your language's null-safety or optional types (TypeScript strict nulls, Kotlin nullability, Rust `Option`, etc.) where they replace or complement annotations.

**Reference implementation (Java):**

```java
public PriceEstimate calculate(@NonNull OrderRequest request) { ... }
public @Nullable Provider findNearestAvailableProvider(Location location) { ... }
```

---

## 📏 8. Comments

### 8.1 Code Should Be Self-Documenting

The need for a comment is often a signal that the code should be clearer.

**Reference implementation (Java):**

```java
// ❌ Bad - comment explains what the code does (the code should do that itself)
// Get order by id and check if it's in progress
if (orderRepo.findById(id).getStatus() == OrderStatus.IN_PROGRESS) { ... }

// ✅ Good - method name communicates intent; no comment needed
if (order.isInProgress()) { ... }
```

### 8.2 When Comments Are Good

Comments are valuable when they explain **why**, not **what**:

**Reference implementation (Java):**

```java
// ✅ Good - explains non-obvious business rule
// Providers who cancel more than 3 orders in 24h are temporarily suspended
// per our provider agreement (section 4.2). This is intentional friction,
// not a bug - do not remove without product sign-off.
if (provider.getCancellationsInLast24Hours() >= 3) {
    suspensionService.temporarilySuspend(provider.getId());
}

// ✅ Good - explains a workaround
// Stripe sometimes sends duplicate webhook events within 60 seconds.
// We use the idempotency key to deduplicate before processing.
if (paymentEventRepository.existsByIdempotencyKey(event.getIdempotencyKey())) {
    log.info("Duplicate payment event ignored: {}", event.getIdempotencyKey());
    return;
}
```

### 8.3 TODO Comments

Every `// TODO` comment must reference a Jira ticket:

**Reference implementation (Java):**

```java
// ✅ Good
// TODO(PROJ-1234): Replace polling with WebSocket push when infra is ready

// ❌ Bad - no tracking; will never be fixed
// TODO: fix this later
```

---

## 💡 9. Use of Java Records

**Principle:** Prefer **immutable data carriers** (DTOs, value objects, parameter objects) with validation at construction. Every language has an idiomatic equivalent (structs, data classes, `@dataclass`, records, typed objects).

**Reference implementation (Java):** Use Java Records for immutable data carriers (DTOs, value objects, parameters):

```java
// ✅ Use records for DTOs
public record OrderSummaryDto(
    String orderId,
    String status,
    BigDecimal priceAmount,
    String currency,
    Instant completedAt
) {}

// ✅ Use records for value objects in the domain
public record Location(double lat, double lng) {
    public Location {
        if (lat < -90 || lat > 90) throw new IllegalArgumentException("Invalid latitude: " + lat);
        if (lng < -180 || lng > 180) throw new IllegalArgumentException("Invalid longitude: " + lng);
    }
}

// ✅ Use records for method parameter groups
public record PriceCalculationRequest(
    Location dispatch,
    Location delivery,
    VehicleType vehicleType,
    String city
) {}
```

Do **not** use records for domain entities that have lifecycle and changing state - use classes.

---

## 📏 10. Logging in Code

**Principle:** Use the **structured logger** for your runtime (level-based, parameterised messages, correlation fields). Never rely on standard output for production diagnostics.

**Reference implementation (Java):** Always use SLF4J. Never use `System.out.println`.

```java
// ❌ Bad
System.out.println("Order started: " + orderId);
e.printStackTrace();

// ❌ Bad - string concatenation in log calls (evaluated even if log level disabled)
log.debug("Processing order: " + orderId + " for customer: " + customerId);

// ✅ Good - parameterised logging (lazy evaluation)
log.debug("Processing order: {} for customer: {}", orderId, customerId);

// ✅ Good - structured context for searchability
log.info("Order completed. orderId={}, durationSecs={}, priceAmount={}",
    order.getId(), order.getDurationSeconds(), order.getPriceAmount());

// ✅ Good - always include exception object (not just message) so stack trace is captured
log.error("Failed to dispatch provider. orderId={}, providerId={}", orderId, providerId, e);
```

---

## 📏 11. Dependency Injection

**Principle:** Prefer **explicit constructor injection** (or equivalent) so dependencies are visible and testable. Avoid hidden service location or field assignment by the framework.

**Reference implementation (Java):** Always inject via constructor - never via field injection (`@Autowired` on fields).

```java
// ❌ Bad - field injection; hides dependencies; makes testing harder
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private PricingClient pricingClient;
}

// ✅ Good - constructor injection; dependencies explicit; easy to test
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PricingClient pricingClient;

    public OrderService(OrderRepository orderRepository, PricingClient pricingClient) {
        this.orderRepository = orderRepository;
        this.pricingClient = pricingClient;
    }
}
```

Use Lombok `@RequiredArgsConstructor` to reduce boilerplate if your team has agreed on Lombok:

**Reference implementation (Java):**

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final PricingClient pricingClient;
}
```

---

## 📋 12. Quick Reference - Do / Don't

> **Substitution point:** This table uses Java names (`Optional`, `record`, `@Autowired`); apply the same **ideas** with your language's types and DI style.

| Do | Don't |
|----|-------|
| Return `Optional` when result may be absent | Return `null` |
| Use typed domain exceptions | Throw `RuntimeException` or `Exception` |
| Validate constructor args with `Objects.requireNonNull` | Allow invalid objects to be constructed |
| Use `record` for immutable data containers | Use mutable DTOs with setters |
| Inject via constructor | Use `@Autowired` field injection |
| Use parameterised logging: `log.info("x={}", x)` | Concatenate strings in log calls |
| Name things for what they are | Use `data`, `obj`, `result`, `temp` |
| Return early with guard clauses | Nest 3+ levels of if/else |
| Comment the *why* | Comment the *what* |
| Reference a Jira ticket in every `// TODO` | Write `// TODO: fix later` |

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
