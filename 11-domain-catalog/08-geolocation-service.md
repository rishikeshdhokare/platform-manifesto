# Geolocation Service — Domain Documentation

| Field | Value |
|-------|-------|
| **Status** | Active |
| **Owner** | Team Orders |
| **Last Updated** | 2025 |

**Service identifier:** `com.{company}.geolocation`

---

## 1. Overview

The **Geolocation Service** domain powers **ETA calculation**, **route planning**, **geocoding**, and **reverse geocoding** for the platform. It is intentionally a **proxy service**: it wraps external mapping providers (**Google Maps**, **HERE**) behind a **stable internal API** so order flows do not depend directly on vendor SDKs or URL shapes.

### 1.1 Ownership

| Owns | Does not own |
|------|----------------|
| **Route caching** and cache key design | Raw map tiles, proprietary map data, or POI databases at rest |
| **Provider abstraction** (ports/adapters) | Ground truth of road networks (owned by providers) |
| Normalized **Route**, **ETA**, **Geocoding** contracts for the platform | Business decisions built on top of ETAs (Fulfillment, Pricing, etc.) |

All persistent geographic truth remains with the external providers; the platform holds only **ephemeral** computed results in cache.

---

## 2. Architecture

Internal platform consumers call **Geolocation Service**, which uses **Redis** before hitting external APIs. **Cache hit** avoids provider cost and latency; **cache miss** fetches from the configured provider.

```mermaid
flowchart TB
    subgraph internal [Internal platform services]
        OE[Orders / ETA consumers]
        FE[Fulfillment Engine]
        OTH[Other callers]
    end

    GS[Geolocation Service — com.{company}.geolocation]
    R[(Redis cache)]

    subgraph external [External providers]
        G[Google Maps APIs]
        H[HERE APIs]
    end

    OE --> GS
    FE --> GS
    OTH --> GS

    GS --> R
    R -->|cache HIT| GS
    R -->|cache MISS| GS
    GS -->|fetch| G
    GS -->|fallback / secondary| H
    GS -->|populate| R
```

---

## 3. Domain model

Types exposed by the Geolocation bounded context (language-agnostic view).

```mermaid
classDiagram
    class Location {
        +double lat
        +double lng
        +Optional accuracyMeters
    }

    class RouteRequest {
        +Location origin
        +Location destination
        +String mode
        +Instant requestedAt
    }

    class Route {
        +String polyline
        +int distanceMeters
        +int durationSeconds
        +String providerRef
    }

    class ETAResponse {
        +int durationSeconds
        +int distanceMeters
        +Instant computedAt
    }

    class GeocodingResult {
        +Location location
        +String formattedAddress
        +String providerRef
    }

    class DistanceMatrix {
        +MatrixCell[][] cells
    }

    class MatrixCell {
        +int durationSeconds
        +int distanceMeters
    }

    RouteRequest --> Location
    Route --> Location
    ETAResponse --> Route : may derive from
    GeocodingResult --> Location
    DistanceMatrix --> MatrixCell
```

---

## 4. Provider abstraction (hexagonal)

The **domain port** defines operations (calculate route, ETA, geocode, matrix). **Adapters** implement that port per vendor (**Google**, **HERE**), making provider switches a configuration and adapter concern—not a rewrite of platform logic.

```mermaid
flowchart LR
    subgraph domain [Domain core — com.{company}.geolocation]
        P[MappingPort interface]
    end

    subgraph adapters [Adapters]
        GA[GoogleMapsAdapter]
        HA[HereAdapter]
    end

    P --> GA
    P --> HA
    GA --> GAPI[Google APIs]
    HA --> HAPI[HERE APIs]
```

---

## 5. API surface

**gRPC** is the primary interface (names are logical service methods).

| RPC | Purpose |
|-----|---------|
| `CalculateRoute` | Turn **RouteRequest** into a **Route** (polyline, distance, duration) |
| `GetETA` | Duration (and optionally distance) for origin → destination |
| `Geocode` | Address or place text → **GeocodingResult** |
| `ReverseGeocode` | **Location** → formatted address / **GeocodingResult** |
| `GetDistanceMatrix` | Many-to-many durations/distances → **DistanceMatrix** |

---

## 6. Caching strategy

**Redis**, TTL-based, no relational database.

| Cache | TTL | Key idea |
|-------|-----|----------|
| **Route** | **5 minutes** | Same origin/destination pair (and mode) within TTL returns cached **Route** / ETA slice |
| **Geocoding** | **24 hours** | Stable address ↔ coordinate mappings change slowly |

All cached data is **ephemeral** and **reconstructible** by re-calling providers.

---

## 7. Data store

| Store | Usage |
|-------|--------|
| **Redis** | Route and geocoding cache only |
| **Relational DB** | **None** for this domain |

---

## 8. Resilience

- **Circuit breaker** on outbound calls to the primary mapping provider to avoid cascading failures and thundering herds.
- **Fallback** to a **secondary** provider (e.g., HERE when Google is unhealthy, or vice versa per platform config) when the primary is open or errors exceed thresholds.

```mermaid
flowchart TD
    Req[Incoming RPC] --> CB{Circuit closed?}
    CB -->|Yes| Primary[Call primary provider]
    CB -->|Open| Fallback[Call secondary provider]
    Primary --> OK{Success?}
    OK -->|Yes| Cache[Write Redis if applicable]
    OK -->|No| Trip[Trip breaker / try secondary]
    Trip --> Fallback
    Fallback --> Cache
    Cache --> Resp[Return response]
```

---

## 9. Key metrics

| Metric | Target / note |
|--------|----------------|
| **Cache hit rate** | Target **> 60%** (tune keys, TTL, and traffic patterns) |
| **Provider latency P99** | Track per adapter for SLOs |
| **Provider error rate** | Drives circuit breaker and fallback tuning |
| **Cost per API call** | Finance and capacity; cache reduces spend |

---

## 10. Team

**Team Orders** — owns Geolocation Service, provider contracts, and cache/resilience behavior for the platform experience.

---

← [Back to Domain Catalog](./README.md)
