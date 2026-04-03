# Data Platform Standards

> **Status:** Mandated  
> **Owner:** Data Engineering + Platform Engineering  
> **Last Updated:** 2025

---

## 1. Data Ownership Principle

Each domain service is the **single source of truth** for its own data. No other service reads from a domain's database directly. Data is shared through:
- APIs (synchronous, request-time)
- Domain events on Kafka (asynchronous, eventual consistency)
- Read projections in the data warehouse (analytical, batch)

---

## 2. Event Schema Registry

### 2.1 Standard

All Kafka messages use **Avro** schemas, registered in **AWS Glue Schema Registry**.

### 2.2 Schema Evolution Rules

We follow **backward compatibility** rules — new consumers can read old messages, old consumers can read new messages:

| Change | Allowed? |
|--------|---------|
| Add field with default value | ✅ Yes |
| Remove field with default value | ✅ Yes |
| Add required field (no default) | ❌ No — breaking |
| Remove required field | ❌ No — breaking |
| Change field type | ❌ No — breaking |
| Rename field | ❌ No (add new + deprecate old) |

Breaking schema changes require a new schema version and a migration plan.

### 2.3 Schema Naming

```
{domain}.{entity}.{version}

Examples:
  orders.order.v1
  payments.payment.v2
  providers.provider-location.v1
```

---

## 3. Analytics Data Pipeline

### 3.1 Architecture

```
Operational DBs (Aurora)
        │
        │ CDC via Debezium
        ▼
    Kafka (MSK)
        │
        │ Kafka Connect S3 Sink
        ▼
    S3 (Raw Layer)
        │
        │ AWS Glue ETL
        ▼
  Amazon Redshift (Data Warehouse)
        │
        │ dbt transformations
        ▼
  Redshift (Curated Layer)
        │
        ▼
  Amazon QuickSight / Grafana (Reporting)
```

### 3.2 CDC (Change Data Capture)

- **Debezium** captures row-level changes from Aurora and publishes to Kafka
- Topic naming for CDC: `cdc.{database}.{table}` e.g. `cdc.orders.orders`
- CDC events are never consumed by application services — analytics pipeline only

### 3.3 Data Retention

| Layer | Retention | Storage |
|-------|----------|---------|
| Kafka (hot) | 7-30 days per topic | MSK |
| S3 raw | 7 years | S3 Glacier tiering |
| Redshift curated | 3 years | Redshift |

---

## 4. GDPR & Data Compliance

- **Right to erasure:** Implemented via a `data-deletion-service` that accepts a customer/provider ID and triggers deletion across all operational stores and queues a downstream analytics tombstone event
- **Data residency:** All EU customer data stored in `eu-west-1` or `eu-central-1` only; enforced by SCP
- **Consent:** Consent records stored in a dedicated consent service; downstream systems subscribe to consent-revoked events
- **Data inventory:** Maintained in the data catalog (AWS Glue Data Catalog + Backstage)

---

---

## 5. Data Store Selection Guide

Picking the wrong data store is expensive to fix. Use this decision guide before choosing:

```
Is this data relational / transactional?
  └─ YES → PostgreSQL (RDS or Aurora)

Is this data a cache or session state that can be rebuilt?
  └─ YES → Redis (ElastiCache)

Is this a stream of events that multiple consumers need?
  └─ YES → Kafka (MSK)

Is this binary data (images, documents, files)?
  └─ YES → S3

Is this for full-text or geospatial search?
  └─ YES → OpenSearch

Is this for analytics / reporting (read-heavy, large scans)?
  └─ YES → Redshift

Is this for real-time geospatial queries (provider locations)?
  └─ YES → Redis with GEO commands
```

---

## 6. Provider Location — Geospatial Pattern

Provider location updates are the highest-frequency writes on the platform (~5 updates/second/active provider). They use Redis geospatial indexes:

```java
// Writing provider location (from location update consumer)
@Component
public class ProviderLocationRepository {

    private final RedisTemplate<String, String> redisTemplate;
    private static final String GEO_KEY = "provider:locations";

    public void updateLocation(ProviderId providerId, double lat, double lng) {
        redisTemplate.opsForGeo().add(
            GEO_KEY,
            new Point(lng, lat),     // Note: Redis uses lng, lat order
            providerId.value()
        );
        // Set expiry: if a provider hasn't updated in 5 minutes, remove them
        redisTemplate.expire(GEO_KEY + ":" + providerId.value(), Duration.ofMinutes(5));
    }

    // Find providers within radius (for fulfillment)
    public List<ProviderId> findProvidersWithinRadius(Location center, double radiusKm) {
        Circle circle = new Circle(
            new Point(center.lng(), center.lat()),
            new Distance(radiusKm, Metrics.KILOMETERS)
        );

        GeoResults<RedisGeoCommands.GeoLocation<String>> results =
            redisTemplate.opsForGeo().radius(GEO_KEY, circle);

        return results.getContent().stream()
            .map(r -> ProviderId.of(r.getContent().getName()))
            .collect(Collectors.toList());
    }
}
```

---

## 7. Analytics Pipeline — Step by Step

### 7.1 Change Data Capture (CDC) with Debezium

Debezium captures every INSERT, UPDATE, DELETE from Aurora and publishes to Kafka without any application code changes.

```yaml
# Debezium connector config (deployed as Kafka Connect connector)
{
  "name": "orders-cdc-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "orders-aurora-cluster.xxx.eu-west-1.rds.amazonaws.com",
    "database.port": "5432",
    "database.user": "debezium_user",
    "database.password": "${file:/opt/kafka/config/secrets.properties:db.password}",
    "database.dbname": "orders",
    "database.server.name": "orders-db",
    "table.include.list": "public.orders,public.order_events",
    "topic.prefix": "cdc.orders",
    "plugin.name": "pgoutput",
    "publication.name": "debezium_publication",
    "slot.name": "debezium_slot"
  }
}
```

This produces events on Kafka topics:
- `cdc.orders.public.orders` — all changes to the orders table
- `cdc.orders.public.order_events` — all changes to order_events

### 7.2 CDC Event Shape

```json
{
  "before": null,
  "after": {
    "id": "order-abc123",
    "customer_id": "customer-xyz",
    "status": "COMPLETED",
    "price_amount": 1250,
    "completed_at": "2024-11-15T14:30:00Z"
  },
  "op": "c",           // "c" = create, "u" = update, "d" = delete
  "ts_ms": 1700055000000
}
```

### 7.3 Data Warehouse Loading

Kafka Connect S3 Sink writes CDC events to S3 in Parquet format. AWS Glue crawls the S3 bucket and populates the Glue Data Catalog. Redshift Spectrum or a Glue ETL job loads the data into Redshift.

---

## 8. Data Ownership Anti-Patterns

These are the most common mistakes. Avoid them:

| Anti-Pattern | Why It's Wrong | Fix |
|-------------|---------------|-----|
| Service A reads from Service B's database | Creates hidden coupling; B can't change schema without breaking A | A calls B's API or subscribes to B's events |
| Shared "common" database across services | Schema changes affect all services; can't deploy independently | Each service owns its own schema |
| Putting business logic in the data warehouse | Analytics DB is for reading, not processing | Process in the owning service; project results to analytics |
| Using Kafka as a database (relying on retention for state) | Topic data expires; compacted topics are not a database | Use a real DB as source of truth; Kafka for events only |
| Storing PII in Kafka topics | Hard to delete on GDPR erasure request | Store a reference ID in Kafka; PII stays in the DB |

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
