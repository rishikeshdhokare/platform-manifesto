# 01 — Mobile & Frontend Standards

> **Status:** Mandated · **Owner:** Platform Engineering · **Last Updated:** 2025

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Mobile Tech Stack](#2-mobile-tech-stack)
3. [BFF Pattern for Mobile](#3-bff-pattern-for-mobile)
4. [API Contract Patterns](#4-api-contract-patterns)
5. [Offline-First Design](#5-offline-first-design)
6. [Push Notification Standards](#6-push-notification-standards)
7. [Performance Budgets](#7-performance-budgets)
8. [App Versioning](#8-app-versioning)
9. [Deep Linking](#9-deep-linking)
10. [Accessibility](#10-accessibility)
11. [Mobile CI/CD](#11-mobile-cicd)
12. [Error Handling UX](#12-error-handling-ux)

---

## 1. Philosophy

Mobile apps **are** the product for {Company} customers and providers. Every interaction — placing an order, tracking a delivery, completing a transaction — happens on a phone in variable network conditions, in bright sunlight, on the move. There is no second chance to make a good impression.

**Guiding principles:**

- **Performance is a feature.** A laggy app is a broken app. Every millisecond matters when a customer is waiting or a provider is fulfilling an order.
- **Reliability is non-negotiable.** The app must work on slow 3G connections, in underground parking, and during network transitions. Offline-first is the default posture.
- **Accessibility is mandatory.** {Company} serves diverse populations. RTL, screen readers, and WCAG compliance are first-class concerns.
- **Battery and data are sacred resources.** Background work, polling, and large payloads directly impact our users' daily lives.

---

## 2. Mobile Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Framework** | React Native | Cross-platform delivery with native escape hatches |
| **Language** | TypeScript (strict mode) | Type safety across the entire client codebase |
| **State Management** | Zustand | Lightweight, predictable, minimal boilerplate |
| **Navigation** | React Navigation | De facto standard, deep linking support |
| **Networking** | Axios + React Query | Caching, retries, and background refetch |
| **Local Storage** | SQLite (via `react-native-quick-sqlite`) | Offline data persistence |
| **Maps** | Mapbox GL | Custom styling, regional coverage, offline tiles |
| **Unit Testing** | Jest | Fast, well-integrated with React Native |
| **E2E Testing** | Detox | Grey-box testing on real simulators/emulators |
| **Crash Reporting** | Firebase Crashlytics | Real-time crash analytics |

**Native modules** are permitted when React Native bridges are insufficient (e.g., background location tracking, Bluetooth beacons). All native modules must be wrapped in a TypeScript interface so they can be mocked in tests.

---

## 3. BFF Pattern for Mobile

Each mobile client has a dedicated **Backend-for-Frontend (BFF)** that acts as its API gateway. The BFF is owned by the mobile team and exists to serve the mobile client's exact needs.

### Architecture

```mermaid
flowchart LR
    subgraph "Client Layer"
        RA["Customer App\n(React Native)"]
        DA["Provider App\n(React Native)"]
    end

    subgraph "BFF Layer"
        RBFF["Customer BFF\n(Node.js / TypeScript)"]
        DBFF["Provider BFF\n(Node.js / TypeScript)"]
    end

    subgraph "Internal Services"
        MS["Fulfillment Service"]
        PS["Pricing Service"]
        TS["Order Service"]
        US["User Service"]
        NS["Notification Service"]
    end

    RA --> RBFF
    DA --> DBFF
    RBFF --> MS
    RBFF --> PS
    RBFF --> TS
    RBFF --> US
    DBFF --> MS
    DBFF --> TS
    DBFF --> US
    DBFF --> NS
```

### BFF Responsibilities

| Responsibility | Example |
|---------------|---------|
| **Response shaping** | Combine order + provider + vehicle into a single `OrderDetailView` payload |
| **Aggregation** | Fetch price estimate + dynamic pricing + ETA in a single request |
| **Caching** | Cache service-type lists (TTL 5 min) so the app doesn't re-fetch on every launch |
| **Authentication** | Validate JWT, attach user context to downstream calls |
| **Platform adaptation** | Return different image resolutions for iOS vs Android |

### BFF Anti-Patterns

- **Business logic in the BFF.** The BFF shapes data; it does not compute prices, match customers, or manage orders.
- **Shared BFF across clients.** Customer and Provider have different needs. A shared BFF becomes a monolith.
- **Direct database access.** BFFs call services, never databases.
- **Excessive orchestration.** If a BFF call fans out to more than 4 internal services, the downstream services may need to provide a composite API.

### Splitting an Overgrown BFF

When a BFF exceeds ~50 endpoints or is maintained by more than two teams, split by domain:

| Before | After |
|--------|-------|
| Customer BFF (monolith) | Customer Order BFF, Customer Payment BFF, Customer Profile BFF |

Each sub-BFF owns a URL prefix (`/orders/*`, `/payments/*`, `/profile/*`) and is independently deployable. An API gateway (Kong) routes traffic to the correct sub-BFF.

---

## 4. API Contract Patterns

### Sparse Fieldsets

Mobile screens rarely need every field. BFF endpoints support a `fields` query parameter:

```
GET /v1/orders/current?fields=id,status,provider.name,provider.vehicle,eta
```

The BFF only fetches and returns the requested fields, reducing payload size and downstream load.

### Response Compression

All BFF responses use **gzip** compression (`Content-Encoding: gzip`). Typical payload reduction: 60–80%.

### Minimal Payloads

- Timestamps: ISO 8601 strings, not epoch + timezone objects.
- Coordinates: `[lng, lat]` arrays, not `{ longitude, latitude }` objects.
- Enums: Short string codes (`"ECO"`, `"CMF"`, `"PRM"`), not full words.
- Images: Return CDN URLs with width/height hints; never inline base64.

### Cursor-Based Pagination

All list endpoints use cursor-based pagination — never offset-based:

```json
{
  "data": [ ... ],
  "cursor": {
    "next": "eyJpZCI6MTAwfQ==",
    "hasMore": true
  }
}
```

Cursors are opaque, base64-encoded tokens. The client passes `?cursor=<token>` to fetch the next page. This ensures stable pagination even when new data is inserted.

### Cache-Control Headers

| Resource | Cache-Control | Rationale |
|----------|--------------|-----------|
| Service types | `max-age=300` | Rarely changes |
| Order history | `max-age=60` | Recent orders may update |
| Current order | `no-cache` | Always fresh |
| User profile | `max-age=120, stale-while-revalidate=60` | Moderate staleness OK |
| Price estimate | `no-store` | Time-sensitive, never cache |

---

## 5. Offline-First Design

{Company} apps must remain functional when the network drops. Customers should see their current order, and providers should see their active tasks, even in tunnels or dead zones.

### Online/Offline Logic Flow

```mermaid
flowchart TD
    A["User Action\n(e.g., view order)"] --> B{"Network\navailable?"}
    B -- Yes --> C["Fetch from BFF"]
    C --> D["Update local SQLite"]
    D --> E["Render from\nlocal store"]
    B -- No --> F["Read from\nlocal SQLite"]
    F --> G{"Data\nexists?"}
    G -- Yes --> H["Render cached data\n+ offline indicator"]
    G -- No --> I["Show offline\nempty state"]
    C --> J{"Request\nfailed?"}
    J -- Yes --> F
    J -- No --> E

    style H fill:#fff3cd,stroke:#856404
    style I fill:#f8d7da,stroke:#721c24
```

### Optimistic Updates

For write operations (e.g., rating an order, updating profile), the app applies changes locally **before** the server confirms:

1. Apply change to local state and render immediately.
2. Send request to BFF in the background.
3. If successful: no further action (local state is already correct).
4. If failed: roll back local state, show inline error, offer retry.

### Local Storage (SQLite)

| Table | Purpose | Retention |
|-------|---------|-----------|
| `orders` | Active and recent orders | 30 days |
| `provider_profile` | Cached provider info for current order | Session |
| `price_estimates` | Last price estimate per route | 10 minutes |
| `notifications` | Push notification history | 90 days |
| `user_preferences` | Saved addresses, payment defaults | Permanent |

### Conflict Resolution

{Company} uses **last-write-wins (LWW)** with server timestamps:

- Every mutable record includes an `updatedAt` (server-generated UTC timestamp).
- On sync, the client sends its local version; the server compares `updatedAt`.
- If the server version is newer, the server version wins. The client is updated.
- If the client version is newer (server was behind), the client version is accepted.
- For critical data (payments, order state), the **server is always authoritative** — no client-side conflict resolution.

### Sync on Reconnect

When network connectivity is restored:

1. The app detects reconnection via `NetInfo` listener.
2. A sync queue of pending mutations is replayed in order (FIFO).
3. Each mutation is retried with exponential backoff (max 3 attempts).
4. Stale cached data is re-fetched for the active screen.
5. A "Back online" toast confirms reconnection to the user.

---

## 6. Push Notification Standards

### Notification Flow

```mermaid
sequenceDiagram
    participant Svc as Internal Service
    participant NS as Notification Service
    participant FCM as FCM / APNs
    participant App as {Company} App

    Svc->>NS: Send notification event\n(Kafka: notification.requested)
    NS->>NS: Resolve user → device tokens\nApply preferences & quiet hours
    NS->>FCM: Dispatch push payload
    FCM->>App: Deliver notification
    App->>App: Handle foreground:\nshow in-app banner
    App-->>NS: Delivery receipt\n(analytics callback)
```

### FCM / APNs Integration

| Platform | Provider | Configuration |
|----------|----------|--------------|
| Android | Firebase Cloud Messaging (FCM) | Server key stored in AWS Secrets Manager |
| iOS | Apple Push Notification service (APNs) | `.p8` auth key, rotated annually |

Both providers are abstracted behind the **Notification Service**. Mobile teams never call FCM/APNs directly.

### Notification Channels

| Channel | Priority | Examples |
|---------|----------|----------|
| **Order Updates** | High | Provider assigned, provider arriving, order started, order completed |
| **Account** | Default | Payment method expiring, profile verification required |
| **Promotions** | Low | Discount codes, referral bonuses, new features |
| **Safety** | Critical | SOS confirmation, safety check-in reminder |

Android notification channels are registered at app startup and map directly to these categories, allowing users to control each independently.

### Silent Push for Data Sync

Silent pushes (content-available on iOS, data-only on Android) trigger background data sync without user-visible notifications. Use cases:

- Refresh order state when a status change occurs.
- Update price rules or service type configuration.
- Invalidate cached data on the device.

Silent pushes must not be sent more than **3 times per hour** to avoid OS throttling.

### Token Lifecycle Management

| Event | Action |
|-------|--------|
| App install / first launch | Register device token with Notification Service |
| Token refresh (FCM callback) | Update token via `PUT /v1/devices/{deviceId}` |
| User logout | Disassociate token from user (keep token for re-login) |
| App uninstall detected | Mark token as inactive after delivery failures |
| Token inactive > 30 days | Purge from token store |

---

## 7. Performance Budgets

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Cold app startup** | < 3 seconds | Time from tap to interactive home screen |
| **API response (p95)** | < 2 seconds | BFF round-trip measured at client |
| **Time to Interactive (TTI)** | < 5 seconds | First screen fully interactive after cold start |
| **JS bundle size** | < 5 MB (compressed) | Hermes bytecode bundle |
| **Scrolling frame rate** | 60 fps | No dropped frames on order list, map |
| **Memory usage** | < 300 MB | Sustained usage during active order |
| **Battery drain** | < 5% / hour active use | Measured on reference devices |

### Monitoring

- **React Native Performance** (`react-native-performance`) tracks TTI, component render times, and JS thread usage.
- **Flipper** for development-time profiling (network, layout, databases).
- Custom **Datadog RUM** integration reports real-user metrics segmented by device tier, OS version, and city.
- **Performance regression CI gate:** startup time and bundle size are measured on every PR. Regressions beyond 10% block the merge.

### Reference Devices

Performance budgets are validated against these baseline devices:

| Tier | Device | OS |
|------|--------|----|
| Low-end | Samsung Galaxy A13 | Android 12 |
| Mid-range | Samsung Galaxy A54 | Android 14 |
| High-end | iPhone 13 | iOS 17 |

If a feature does not meet budgets on the low-end device, it does not ship.

---

## 8. App Versioning

### Minimum Supported Version

The {Company} backend enforces a **minimum supported app version**. On every API call, the BFF checks the `X-App-Version` header:

| Response | Meaning | Client Behavior |
|----------|---------|-----------------|
| `200 OK` | Version is supported | Continue normally |
| `426 Upgrade Required` | Version is deprecated | Show soft upgrade prompt |
| `403 Forbidden` (+ `FORCE_UPGRADE` code) | Version is blocked | Show blocking upgrade screen |

### Force Upgrade Flow

1. Backend returns `403` with `errorCode: "FORCE_UPGRADE"` and `storeUrl`.
2. App shows a full-screen modal: "A new version of {Company} is available. Please update to continue."
3. The CTA opens the App Store / Play Store listing.
4. The modal cannot be dismissed.

### Gradual Rollout

- **Play Console staged rollout:** 1% → 5% → 20% → 50% → 100%, with 24-hour soak at each stage.
- **TestFlight / App Store phased release:** 7-day automatic rollout (1%/day ramp for the first week).
- Rollout is paused automatically if crash rate exceeds 2x the baseline.

### Feature Flags for Mobile

Mobile features are gated by **LaunchDarkly** flags, not app versions:

```typescript
if (await ldClient.boolVariation('enable-scheduled-orders', false)) {
  showScheduledOrderUI();
}
```

This decouples feature delivery from app releases. A feature can be shipped in a binary and enabled remotely when ready.

### Over-the-Air Updates (CodePush)

| Policy | Rule |
|--------|------|
| **Allowed** | Bug fixes, copy changes, layout adjustments |
| **Not allowed** | New native modules, permission changes, navigation structure changes |
| **Rollout** | Staged: 10% → 50% → 100%, with 4-hour soak |
| **Rollback** | Automatic if crash rate > 1% on the updated cohort |
| **Mandatory vs Optional** | Critical fixes are mandatory; non-critical are optional (user can defer) |

---

## 9. Deep Linking

### URL Scheme

Internal deep links use the `{company}://` custom scheme:

| Link | Target |
|------|--------|
| `{company}://order/{orderId}` | Order detail screen |
| `{company}://request?dispatch={lat,lng}&delivery={lat,lng}` | Pre-filled order request |
| `{company}://profile` | User profile screen |
| `{company}://promo/{code}` | Apply promo code |
| `{company}://support/ticket/{ticketId}` | Support ticket detail |

### Universal Links (Sharing)

User-facing sharing uses HTTPS universal links routed through `link.{company}.app`:

| Link | Use Case |
|------|----------|
| `https://link.{company}.app/order/{orderId}/share` | Share order with a contact (live tracking) |
| `https://link.{company}.app/eta/{etaToken}` | Share ETA ("Estimated delivery in 7 minutes") |
| `https://link.{company}.app/invite/{referralCode}` | Referral invite link |

If the app is installed, the link opens in-app. If not, it opens a web fallback page with app store download links.

### Deferred Deep Linking

For install-then-navigate flows (e.g., a referral link clicked by someone without the app):

1. User clicks `https://link.{company}.app/invite/FRIEND25`.
2. User does not have the app → redirected to app store.
3. User installs and opens the app.
4. The deferred deep link payload is retrieved (via Firebase Dynamic Links).
5. The app navigates to the referral redemption screen and auto-applies `FRIEND25`.

---

## 10. Accessibility

### Compliance Target

{Company} targets **WCAG 2.1 Level AA** compliance across both Customer and Provider apps.

### Screen Reader Support

- All interactive elements have `accessibilityLabel` and `accessibilityHint`.
- Dynamic content updates announce via `AccessibilityInfo.announceForAccessibility()`.
- Map pins include descriptive labels: "Dispatch location: Main Street."
- Order status changes are announced: "Your provider has arrived."

### Touch Targets

- Minimum touch target size: **48 × 48 dp** (density-independent pixels).
- Interactive elements spaced at least **8 dp** apart.
- Primary CTAs (Place Order, Start Fulfillment) are at least **56 × 56 dp**.

### Color Contrast

- Text on backgrounds: minimum **4.5:1** contrast ratio (AA).
- Large text (18sp+): minimum **3:1** contrast ratio.
- Interactive elements: do not rely on color alone — pair with icons or labels.
- The {Company} brand primary color is validated against both light and dark backgrounds.

### RTL Support (Arabic)

- Layout mirroring is enabled globally via `I18nManager.forceRTL()`.
- Icons with directional meaning (arrows, chevrons) are mirrored.
- Text alignment follows the locale.
- All RTL layouts are tested on CI with `layoutDirection: "rtl"` on every PR.
- Map controls remain LTR (geographic convention).

---

## 11. Mobile CI/CD

### Pipeline

```mermaid
flowchart LR
    A["Pull Request"] --> B["Lint\n(ESLint + Prettier)"]
    B --> C["Unit Tests\n(Jest)"]
    C --> D["Build\n(iOS + Android)"]
    D --> E["E2E Tests\n(Detox)"]
    E --> F{"Target?"}
    F -- Beta --> G["TestFlight /\nPlay Console\n(Internal Track)"]
    F -- Production --> H["App Store /\nPlay Store\n(Staged Rollout)"]

    style A fill:#e3f2fd,stroke:#1565c0
    style E fill:#fff3e0,stroke:#e65100
    style G fill:#e8f5e9,stroke:#2e7d32
    style H fill:#e8f5e9,stroke:#2e7d32
```

### Build Automation (Fastlane)

| Lane | Purpose |
|------|---------|
| `fastlane ios beta` | Build, sign, upload to TestFlight |
| `fastlane android beta` | Build AAB, upload to Play Console internal track |
| `fastlane ios release` | Build, sign, submit for App Store review |
| `fastlane android release` | Build AAB, upload to Play Console production |
| `fastlane match` | Sync iOS signing certificates from encrypted git repo |

### CI Environment

- **Runner:** macOS (GitHub Actions self-hosted) for iOS builds; Linux for Android.
- **Simulators/Emulators:** Detox E2E tests run on iPhone 15 simulator (iOS 17) and Pixel 7 emulator (API 34).
- **Build caching:** Gradle and CocoaPods caches are shared across runs (30–40% build time reduction).
- **Secrets:** Signing keys and API tokens stored in GitHub Actions secrets, injected at build time.

---

## 12. Error Handling UX

### Network Timeout Patterns

| State | UX Treatment |
|-------|-------------|
| **Slow network** (> 5s response) | Show skeleton screens with shimmer animation |
| **Request failed** (retryable) | Inline retry banner: "Something went wrong. Tap to retry." |
| **Offline** | Persistent bottom bar: "You're offline. Some features may be limited." |
| **Back online** | Toast: "You're back online." (auto-dismiss after 3s) |

### Graceful Error Screens

Error screens follow a consistent structure:

1. **Illustration** — a branded, friendly illustration (not a raw error code).
2. **Title** — human-readable ("We couldn't load your orders").
3. **Description** — actionable ("Check your internet connection and try again.").
4. **Primary action** — "Try Again" button that retries the failed request.
5. **Secondary action** — "Go Home" or "Contact Support" where relevant.

**Never show** raw HTTP status codes, stack traces, or internal error IDs to users. Log them to Crashlytics instead.

### Crash Reporting (Firebase Crashlytics)

| Configuration | Value |
|--------------|-------|
| **Crash-free target** | ≥ 99.5% of sessions |
| **Non-fatal logging** | All caught exceptions + network errors logged as non-fatals |
| **User ID** | Attach anonymized user ID for crash correlation |
| **Breadcrumbs** | Log navigation events and key user actions (last 50) |
| **Alerting** | PagerDuty alert if crash-free rate drops below 99% in a 1-hour window |

---

← [Back to section](./README.md) · [Back to root](../README.md)
