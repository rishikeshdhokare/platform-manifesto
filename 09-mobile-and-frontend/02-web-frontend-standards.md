# Web Frontend Standards

> **Status:** Mandated · **Owner:** Platform Engineering · **Last Updated:** 2026

---

## Table of Contents

1. [SPA vs SSR Decision Guide](#1-spa-vs-ssr-decision-guide)
2. [Microfrontend Strategy](#2-microfrontend-strategy)
3. [Design System](#3-design-system)
4. [Core Web Vitals](#4-core-web-vitals)
5. [Accessibility](#5-accessibility)
6. [Browser Support](#6-browser-support)
7. [CSP & Frontend Security](#7-csp--frontend-security)
8. [Bundle Size Budget](#8-bundle-size-budget)
9. [State Management](#9-state-management)
10. [Testing Strategy](#10-testing-strategy)
11. [Error Tracking](#11-error-tracking)

---

## 1. SPA vs SSR Decision Guide

Not every web surface has the same requirements. The rendering strategy should be chosen based on the nature of the content, SEO needs, and interactivity model.

### 1.1 Decision Matrix

| Criterion | SPA (Vite + React) | SSR (Next.js) |
|-----------|-------------------|---------------|
| **Primary use case** | App-like experiences: dashboards, internal tools, ops consoles | Content-heavy pages: marketing site, help center, blog |
| **SEO requirement** | None or minimal — behind authentication | Critical — pages must be crawlable and indexable |
| **Interactivity** | High — complex state, real-time updates, drag-and-drop | Moderate — mostly read, occasional forms |
| **First Contentful Paint** | Acceptable delay (user is authenticated, expects app loading) | Must be fast — users arrive from search engines |
| **Data fetching** | Client-side via TanStack Query | Server-side via `getServerSideProps` / React Server Components |
| **Deployment** | Static assets to S3 + CloudFront | Containerized (ECS/EKS) or Vercel |

### 1.2 Decision Flowchart

```mermaid
flowchart TD
    Start["New web surface"] --> Q1{"Does the page need\nto be indexed by\nsearch engines?"}

    Q1 -->|"Yes"| Q2{"Is the content\nhighly dynamic\n(per-user)?" }
    Q1 -->|"No"| Q3{"Is it an app-like\nexperience with complex\nclient state?"}

    Q2 -->|"Yes"| SSR_DYNAMIC["SSR with Next.js\n(getServerSideProps)"]
    Q2 -->|"No"| SSG["SSG with Next.js\n(getStaticProps + ISR)"]

    Q3 -->|"Yes"| SPA["SPA with Vite + React"]
    Q3 -->|"No"| SSR_SIMPLE["SSR with Next.js\n(simpler pages, better FCP)"]

    SPA --> DONE["Proceed to\nproject scaffolding"]
    SSR_DYNAMIC --> DONE
    SSG --> DONE
    SSR_SIMPLE --> DONE
```

### 1.3 Default

When in doubt, prefer **SPA** for authenticated internal surfaces and **Next.js SSR** for public-facing surfaces. Hybrid is permitted — a Next.js app can serve SSR marketing pages alongside client-rendered dashboard routes.

---

## 2. Microfrontend Strategy

### 2.1 When Microfrontends Are Justified

Microfrontends (via Module Federation) introduce build complexity, runtime overhead, and shared-dependency versioning challenges. They are only justified when the coordination cost of a monorepo exceeds the cost of the microfrontend infrastructure.

| Signal | Threshold | Action |
|--------|-----------|--------|
| Number of teams contributing to the same web surface | > 3 teams | Consider microfrontends |
| Deployment coupling | Teams cannot ship independently | Consider microfrontends |
| Conflicting framework versions | One team needs React 18, another needs React 19 | Module Federation |
| Shared surface, shared cadence | ≤ 3 teams, coordinated releases | **Monorepo** (default) |

### 2.2 Decision Tree

```mermaid
flowchart TD
    Start["Multiple teams\ncontributing to\none web surface"] --> Q1{"> 3 teams?"}

    Q1 -->|"Yes"| Q2{"Can teams align on\na single React version\nand shared deps?"}
    Q1 -->|"No"| MONO["Monorepo\n(Nx or Turborepo)"]

    Q2 -->|"Yes"| MF_SHARED["Module Federation\nwith shared runtime"]
    Q2 -->|"No"| MF_ISOLATED["Module Federation\nwith isolated builds\n(version pinning per remote)"]

    MONO --> DONE["Proceed"]
    MF_SHARED --> DONE
    MF_ISOLATED --> DONE
```

### 2.3 Monorepo Default

The **default** for all new web projects is a monorepo managed with **Nx** or **Turborepo**:

- Shared component library as an internal package
- Per-app build targets
- Affected-only CI (only build/test what changed)
- Single `package.json` lockfile for dependency consistency

### 2.4 Module Federation Rules (When Used)

| Rule | Detail |
|------|--------|
| Shell app | One shell app owns the layout, routing, and authentication |
| Remote contracts | Each remote exposes a typed interface; breaking changes follow the deprecation lifecycle |
| Shared dependencies | React, React DOM, and the design system are shared singletons — never bundled per remote |
| Versioning | Each remote is independently versioned and deployed |
| Fallback | If a remote fails to load, the shell renders a graceful fallback, not a white screen |

---

## 3. Design System

### 3.1 Shared Component Library

All {Company} web surfaces consume the **{Company} Design System** (`@{company}/ui`), a shared React component library. No team builds custom buttons, modals, or form inputs.

| Layer | Contents | Examples |
|-------|----------|----------|
| **Primitives** | Atomic UI elements | `Button`, `Input`, `Checkbox`, `Badge`, `Avatar` |
| **Composites** | Multi-element patterns | `DataTable`, `Modal`, `Sidebar`, `CommandPalette` |
| **Layout** | Page structure | `PageShell`, `ContentArea`, `SplitPane` |
| **Icons** | Unified icon set | SVG-based, tree-shakeable, accessible labels |

### 3.2 Design Tokens

Design tokens are the single source of truth for visual consistency. They are defined in JSON and consumed by both Figma and code.

```json
{
  "color": {
    "brand": {
      "primary": { "value": "#6C3FC5" },
      "secondary": { "value": "#06AC38" }
    },
    "semantic": {
      "error": { "value": "#E53935" },
      "warning": { "value": "#F9A825" },
      "success": { "value": "#06AC38" },
      "info": { "value": "#1E88E5" }
    }
  },
  "spacing": {
    "xs": { "value": "4px" },
    "sm": { "value": "8px" },
    "md": { "value": "16px" },
    "lg": { "value": "24px" },
    "xl": { "value": "32px" },
    "2xl": { "value": "48px" }
  },
  "typography": {
    "fontFamily": {
      "sans": { "value": "Inter, system-ui, sans-serif" },
      "mono": { "value": "JetBrains Mono, monospace" }
    },
    "fontSize": {
      "xs": { "value": "12px" },
      "sm": { "value": "14px" },
      "base": { "value": "16px" },
      "lg": { "value": "18px" },
      "xl": { "value": "20px" },
      "2xl": { "value": "24px" }
    }
  },
  "borderRadius": {
    "sm": { "value": "4px" },
    "md": { "value": "8px" },
    "lg": { "value": "12px" },
    "full": { "value": "9999px" }
  }
}
```

### 3.3 Storybook

**Storybook** is the component documentation and development environment. Every component in `@{company}/ui` has a corresponding story.

| Requirement | Detail |
|-------------|--------|
| Story coverage | 100% of exported components must have at least one story |
| Interaction tests | All interactive components (forms, modals, dropdowns) include Storybook interaction tests |
| Visual regression | Chromatic runs on every PR; visual diffs require approval |
| Accessibility addon | `@storybook/addon-a11y` enabled — violations fail the CI build |
| Hosted instance | Storybook is deployed to an internal URL, linked from Backstage |

### 3.4 Figma-to-Code Workflow

```mermaid
flowchart LR
    A["Designer creates\ncomponent in Figma"] --> B["Design tokens\nsynced via\nTokens Studio"]
    B --> C["Tokens exported\nto JSON"]
    C --> D["Style Dictionary\ngenerates CSS variables\nand TS constants"]
    D --> E["Developer implements\nReact component\nusing tokens"]
    E --> F["Storybook story\ncreated and reviewed"]
    F --> G["Chromatic visual\nregression test"]
    G --> H["Component published\nto @{company}/ui"]
```

**Rules:**
- Designers use only design tokens defined in the shared Figma library — no one-off colors or spacing values.
- Developers reference CSS custom properties (`var(--color-brand-primary)`) or TypeScript token constants — never hardcoded hex values.
- New tokens require design system team approval.

---

## 4. Core Web Vitals

### 4.1 Targets

| Metric | Target | What It Measures |
|--------|--------|-----------------|
| **Largest Contentful Paint (LCP)** | < 2.5 seconds | Loading performance — when the main content becomes visible |
| **First Input Delay (FID)** | < 100 ms | Interactivity — delay between user input and browser response |
| **Cumulative Layout Shift (CLS)** | < 0.1 | Visual stability — unexpected layout shifts during loading |
| **Interaction to Next Paint (INP)** | < 200 ms | Responsiveness — latency of all user interactions |

### 4.2 Monitoring

| Layer | Tool | Purpose |
|-------|------|---------|
| **CI** | Lighthouse CI | Automated audit on every PR; scores below threshold block merge |
| **Synthetic** | Lighthouse scheduled runs | Daily synthetic tests against staging and production |
| **Real User** | Datadog RUM | Field data segmented by device, connection speed, geography |

### 4.3 Lighthouse CI Configuration

```json
{
  "ci": {
    "collect": {
      "url": ["http://localhost:3000/", "http://localhost:3000/dashboard"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "categories:accessibility": ["error", { "minScore": 0.95 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "first-input-delay": ["error", { "maxNumericValue": 100 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }]
      }
    }
  }
}
```

### 4.4 Performance Budget Enforcement

PRs that regress Lighthouse performance score below **0.9** or violate any Core Web Vital threshold are **automatically blocked**. The PR author must optimize before merging or request an exception from the frontend platform team with a documented justification.

---

## 5. Accessibility

### 5.1 Compliance Target

{Company} web surfaces target **WCAG 2.1 Level AA** compliance. This is mandatory for all customer-facing and internal surfaces.

### 5.2 Automated Enforcement

| Layer | Tool | Enforcement |
|-------|------|-------------|
| **Component development** | `@storybook/addon-a11y` (axe-core) | Violations visible during development |
| **CI — static** | `eslint-plugin-jsx-a11y` | Lint errors block PR merge |
| **CI — runtime** | axe-core via `@axe-core/playwright` | Automated axe scan in E2E test suite; violations fail the build |
| **E2E tests** | Playwright a11y assertions | `expect(page).toPassAxe()` assertion in critical user flows |
| **Manual audit** | Third-party audit | Annual audit by certified accessibility consultants |

### 5.3 Requirements

| Area | Requirement |
|------|-------------|
| **Keyboard navigation** | All interactive elements reachable and operable via keyboard; visible focus indicators |
| **Screen reader** | All images have `alt` text; all form inputs have associated labels; ARIA roles used correctly |
| **Color contrast** | Minimum 4.5:1 for normal text, 3:1 for large text (18px+ bold or 24px+ regular) |
| **Motion** | Respect `prefers-reduced-motion`; no auto-playing animations without user control |
| **Focus management** | Focus trapped in modals; focus returned to trigger on close; skip-to-content link on every page |
| **Error identification** | Form errors identified by more than color alone; error messages associated with inputs via `aria-describedby` |

### 5.4 Playwright A11y Assertion Example

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('dashboard has no accessibility violations', async ({ page }) => {
  await page.goto('/dashboard');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

## 6. Browser Support

### 6.1 Support Matrix

| Browser | Supported Versions | Notes |
|---------|--------------------|-------|
| **Google Chrome** | Last 2 major versions | Primary development and testing browser |
| **Mozilla Firefox** | Last 2 major versions | Tested in CI via Playwright |
| **Apple Safari** | Last 2 major versions | Tested on macOS; iOS Safari tested via BrowserStack |
| **Microsoft Edge** | Last 2 major versions | Chromium-based; covered by Chrome testing in most cases |
| **Internet Explorer** | Not supported | No polyfills, no workarounds, no exceptions |

### 6.2 Browserslist Configuration

```
last 2 Chrome versions
last 2 Firefox versions
last 2 Safari versions
last 2 Edge versions
not dead
not ie 11
```

### 6.3 Polyfill Policy

- **No IE polyfills.** Period.
- Modern API polyfills (e.g., `ResizeObserver`, `IntersectionObserver`) are added only when required by a supported browser version and only via dynamic import to avoid bloating the main bundle.
- The `browserslist` configuration drives Babel/SWC transpilation targets. Do not override manually.

---

## 7. CSP & Frontend Security

### 7.1 Content Security Policy

All {Company} web applications enforce a **strict Content Security Policy** via HTTP headers.

| Directive | Value | Rationale |
|-----------|-------|-----------|
| `default-src` | `'self'` | Only load resources from the same origin by default |
| `script-src` | `'self'` (no `'unsafe-inline'`, no `'unsafe-eval'`) | Prevents inline script injection (XSS) |
| `style-src` | `'self' 'unsafe-inline'` | Inline styles permitted for CSS-in-JS; nonce-based where possible |
| `img-src` | `'self' data: https://cdn.{company}.app` | Allow images from CDN and data URIs |
| `font-src` | `'self' https://fonts.gstatic.com` | Google Fonts or self-hosted |
| `connect-src` | `'self' https://api.{company}.app wss://ws.{company}.app` | API and WebSocket endpoints |
| `frame-ancestors` | `'none'` | Prevent clickjacking — no embedding in iframes |
| `base-uri` | `'self'` | Prevent `<base>` tag injection |

### 7.2 Additional Security Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `X-Content-Type-Options` | `nosniff` | Prevent MIME-type sniffing |
| `X-Frame-Options` | `DENY` | Legacy clickjacking protection |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer information leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(self)` | Restrict browser API access |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains; preload` | Enforce HTTPS |

### 7.3 Subresource Integrity (SRI)

All scripts and stylesheets loaded from CDNs must include SRI hashes:

```html
<script
  src="https://cdn.{company}.app/vendor/analytics.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxAh6VgnSY3lEp9cGvR2mBJC0SxFhD"
  crossorigin="anonymous"
></script>
```

SRI hashes are generated during the build pipeline and injected automatically. Manual hash management is not required.

### 7.4 XSS Prevention

| Control | Implementation |
|---------|---------------|
| No `dangerouslySetInnerHTML` | Lint rule blocks usage; exceptions require security team review |
| Input sanitization | All user input rendered via React's built-in escaping; DOMPurify for rich-text rendering |
| URL validation | `href` attributes validated against allowlist schemes (`https:`, `mailto:`) — no `javascript:` |
| CSP as safety net | Even if a bypass occurs, strict CSP blocks execution of injected scripts |

---

## 8. Bundle Size Budget

### 8.1 Thresholds

| Metric | Limit | Enforcement |
|--------|-------|-------------|
| **Initial JS bundle (gzipped)** | < 200 KB | CI blocks PR if exceeded |
| **Per-route chunk (gzipped)** | < 80 KB | CI warning; blocking at 120 KB |
| **CSS (gzipped)** | < 50 KB | CI warning |
| **Total transfer size (initial load)** | < 500 KB | Lighthouse budget assertion |

### 8.2 CI Enforcement

Bundle size is measured on every PR using `size-limit`:

```json
[
  {
    "name": "Initial JS",
    "path": "dist/assets/index-*.js",
    "gzip": true,
    "limit": "200 KB"
  },
  {
    "name": "CSS",
    "path": "dist/assets/index-*.css",
    "gzip": true,
    "limit": "50 KB"
  }
]
```

If the bundle exceeds the limit, the PR is blocked with a clear message showing the current size, the limit, and the delta.

### 8.3 Optimization Techniques

| Technique | Detail |
|-----------|--------|
| **Code splitting** | React.lazy + Suspense for route-level splitting; dynamic imports for heavy components |
| **Tree shaking** | Only import what you use — `import { Button } from '@{company}/ui'`, not `import * as UI` |
| **Image optimization** | WebP/AVIF formats, responsive `srcset`, lazy loading via `loading="lazy"` |
| **Font subsetting** | Only include character ranges actually used; `font-display: swap` |
| **Dependency audit** | `bundlephobia` check before adding any new dependency; prefer smaller alternatives |

---

## 9. State Management

### 9.1 Standard Stack

| State Type | Tool | Use Case |
|------------|------|----------|
| **Server state** | TanStack Query (React Query) | API data fetching, caching, background refetching, optimistic updates |
| **Client state** | Zustand | UI state, form state, local preferences, ephemeral state |
| **URL state** | React Router search params | Filters, pagination cursors, tab selection — anything linkable |
| **Form state** | React Hook Form + Zod | Complex forms with validation |

### 9.2 What Not to Use

| Tool | Status | Reason |
|------|--------|--------|
| **Redux** | Not permitted for new projects | Excessive boilerplate; TanStack Query + Zustand cover all use cases with less ceremony |
| **MobX** | Not permitted | Implicit reactivity model conflicts with React's explicit rendering |
| **Context API (for global state)** | Discouraged | Causes unnecessary re-renders; use Zustand instead |
| **Local component state** | Permitted | Fine for truly local, ephemeral state (open/closed toggles, hover state) |

### 9.3 TanStack Query Conventions

```typescript
const { data, isLoading, error } = useQuery({
  queryKey: ['orders', { status: 'active', page }],
  queryFn: () => ordersApi.listOrders({ status: 'active', page }),
  staleTime: 30_000,
  gcTime: 5 * 60_000,
});
```

| Convention | Rule |
|------------|------|
| Query keys | Structured arrays: `[resource, filters]` — never plain strings |
| Stale time | Set explicitly per query; default `0` is rarely appropriate |
| Error handling | Global error handler via `QueryClient` for 401/403; per-query for domain errors |
| Mutations | Use `useMutation` with `onSuccess` invalidation — never manually set query data after mutation |

---

## 10. Testing Strategy

### 10.1 Testing Layers

| Layer | Tool | What to Test | Coverage Target |
|-------|------|-------------|----------------|
| **Unit** | Jest + React Testing Library (RTL) | Component behavior, hooks, utility functions | > 80% of business logic |
| **Component** | Storybook Interaction Tests | Interactive component behavior (dropdowns, modals, forms) | 100% of `@{company}/ui` components |
| **Integration** | Playwright | Page-level flows with mocked API | Critical user journeys |
| **E2E** | Playwright | Full end-to-end flows against staging | Top 5 user journeys per surface |
| **Visual regression** | Chromatic | Unintended visual changes | All Storybook stories |

### 10.2 Testing Principles

- **Test behavior, not implementation.** Query by accessible roles and labels (`getByRole`, `getByLabelText`), not by CSS class or test ID.
- **No snapshot tests.** They are brittle and produce meaningless diffs. Use Chromatic for visual regression instead.
- **Mock at the network boundary.** Use MSW (Mock Service Worker) for API mocking in unit and integration tests — never mock internal modules.
- **Playwright for E2E.** Playwright runs in CI against staging. Tests cover the top 5 user journeys per web surface.

### 10.3 RTL Example

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('submitting the form calls onSubmit with form data', async () => {
  const onSubmit = vi.fn();
  render(<OrderForm onSubmit={onSubmit} />);

  await userEvent.type(screen.getByLabelText('Shipping address'), '123 Main St');
  await userEvent.type(screen.getByLabelText('Billing address'), '456 Oak Ave');
  await userEvent.click(screen.getByRole('button', { name: 'Place order' }));

  expect(onSubmit).toHaveBeenCalledWith({
    shippingAddress: '123 Main St',
    billingAddress: '456 Oak Ave',
  });
});
```

---

## 11. Error Tracking

### 11.1 Sentry Integration

**Sentry** is the mandatory error tracking platform for all {Company} web frontends. Every web surface must integrate Sentry before production deployment.

| Configuration | Value |
|---------------|-------|
| **DSN** | Per-project, stored in environment variable (`VITE_SENTRY_DSN`) |
| **Environment** | `development`, `staging`, `production` |
| **Release** | Git commit SHA, set during CI build |
| **Sample rate** | `1.0` for errors, `0.1` for performance transactions |
| **Source maps** | Uploaded to Sentry during CI build; not served publicly |

### 11.2 Source Map Upload (CI)

```bash
npx @sentry/cli releases new "$GIT_SHA"
npx @sentry/cli releases files "$GIT_SHA" upload-sourcemaps ./dist --url-prefix "~/"
npx @sentry/cli releases finalize "$GIT_SHA"
```

Source maps are uploaded to Sentry and then **deleted from the deployment artifact**. They must never be served to end users.

### 11.3 Error Boundary

Every web surface wraps its root component in a Sentry error boundary:

```typescript
import * as Sentry from '@sentry/react';

function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={<FullPageError />}
      showDialog={false}
    >
      <RouterProvider router={router} />
    </Sentry.ErrorBoundary>
  );
}
```

### 11.4 Alert Rules

| Rule | Condition | Action |
|------|-----------|--------|
| **New issue spike** | > 10 occurrences of a new error in 5 minutes | Slack alert to owning team channel |
| **Regression** | A previously resolved issue reoccurs | Slack alert + PagerDuty if P1 |
| **Error rate threshold** | > 1% of sessions have unhandled errors | PagerDuty alert |

---

← [Back to section](./README.md) · [Back to root](../README.md)
