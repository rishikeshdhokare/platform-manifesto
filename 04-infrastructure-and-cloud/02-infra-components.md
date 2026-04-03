# Common Infrastructure Components

> **Status:** Mandated  
> **Owner:** Platform Engineering  
> **Last Updated:** 2025

---

## 1. Overview

These are the platform-level infrastructure components that every service benefits from. Teams do not build these themselves — they consume them. Platform Engineering owns their operation, availability, and upgrade lifecycle.

---

## 2. Service Mesh — Istio

### 2.1 What It Does

Istio handles cross-cutting network concerns without application code changes:
- **mTLS between all pods** — zero-trust service-to-service encryption
- **Traffic management** — canary routing, retries, timeouts, circuit breaking at the mesh level
- **Observability** — distributed tracing and traffic metrics without instrumentation
- **Authorisation policies** — service-to-service access control by identity

### 2.2 What Teams Need to Do

Pods are automatically enrolled in the mesh via the `istio-injection: enabled` label on their namespace. Teams need to:
- Annotate their `Deployment` with Istio-compatible health check paths
- Define a `VirtualService` if they need custom traffic routing (e.g. canary)
- Define `DestinationRules` for connection pool settings (provided by platform templates)

### 2.3 Teams Must NOT

- Implement their own mTLS — Istio handles this
- Set `PeerAuthentication` to `PERMISSIVE` mode in production — all production namespaces require `STRICT`

---

## 3. Secrets Management — AWS Secrets Manager

### 3.1 How Secrets Are Consumed

Secrets are **never** passed as environment variables or stored in Git. The approved pattern:

**Pattern A: External Secrets Operator (preferred)**
- The `ExternalSecret` CRD syncs secrets from AWS Secrets Manager into Kubernetes Secrets
- Pods mount the Kubernetes Secret as a volume (not env vars)
- Rotation happens automatically — pods restart on secret rotation via Reloader

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: orders-db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: orders-db-credentials
  data:
  - secretKey: db-password
    remoteRef:
      key: /production/orders-service/db-password
```

**Pattern B: Spring Cloud AWS (for runtime secret fetching)**
- For secrets that need to be fetched at runtime, not startup
- Use `software.amazon.awssdk:secretsmanager` via Spring Cloud AWS autoconfiguration

### 3.2 Secret Naming Convention

```
/{environment}/{service-name}/{secret-name}

Examples:
  /production/orders-service/db-password
  /production/payments-service/stripe-api-key
  /staging/notifications-service/twilio-auth-token
```

### 3.3 Access Control

- Each service has an IRSA role granting `secretsmanager:GetSecretValue` only for its own secrets path
- No service can read another service's secrets
- Secret access is logged in CloudTrail

---

## 4. Configuration Management — AWS Systems Manager Parameter Store

For **non-sensitive** configuration values (feature toggles backup, service URLs, tunable parameters):

```
/{environment}/{service-name}/{parameter-name}

Examples:
  /production/fulfillment-service/max-search-radius-km
  /production/pricing-service/base-price
```

- Parameters are loaded at startup via Spring Cloud AWS
- Configuration changes do not require a redeployment — services poll for changes every 5 minutes
- **Secrets must never go in Parameter Store** — use Secrets Manager

---

## 5. Service Mesh Observability — Kiali

Kiali provides a topology view of the service mesh — which services are talking to which, with health and traffic metrics. It is deployed in the platform namespace and accessible to all engineers via SSO.

Use Kiali to:
- Visualise traffic flows and latency between services
- Identify misconfigured or unhealthy services in the mesh
- Debug mTLS handshake failures

---

## 6. Internal Developer Portal — Backstage

Backstage is the central catalog of all services, APIs, documentation, and tooling.

### 6.1 What Lives in Backstage

- **Service catalog** — every microservice registered with owner, tech stack, runbook links
- **API catalog** — all OpenAPI specs published from CI
- **Documentation** — TechDocs (Markdown in repo, rendered in Backstage)
- **Templates** — Golden path scaffolding (new service wizard)
- **Tech radar** — approved vs trial vs deprecated technologies

### 6.2 Service Registration (Mandatory)

Every service must have a `catalog-info.yaml` at the repo root:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: orders-service
  description: Manages the lifecycle of an order
  tags:
    - java
    - spring-boot
    - kafka
  links:
    - url: https://grafana.{company}.internal/d/orders
      title: Grafana Dashboard
    - url: https://wiki.{company}.internal/runbooks/orders-service
      title: Runbook
  annotations:
    github.com/project-slug: {company}/orders-service
    backstage.io/techdocs-ref: dir:.
    pagerduty.com/service-id: P1234567
spec:
  type: service
  lifecycle: production
  owner: team-orders
  system: platform
  dependsOn:
    - component:payments-service
    - component:provider-profile-service
  providesApis:
    - orders-api-v1
```

---

## 7. GitOps — ArgoCD

### 7.1 Access Model

- ArgoCD UI is accessible to all engineers (read access)
- Sync and rollback operations require `developer` role or above
- Production namespace sync requires `senior-engineer` role
- All actions in ArgoCD are logged and auditable

### 7.2 Application Structure in platform-config

```
platform-config/
├── apps/
│   ├── dev/
│   │   ├── orders-service.yaml
│   │   ├── fulfillment-service.yaml
│   │   └── ...
│   ├── staging/
│   └── production/
├── helm-charts/
│   ├── java-service/        # Shared Helm chart for all Java services
│   └── ...
└── argocd/
    └── app-of-apps.yaml     # ArgoCD App of Apps pattern
```

### 7.3 Helm Chart — java-service

The platform ships a shared Helm chart `java-service` that handles all standard Kubernetes resources (Deployment, Service, HPA, PodDisruptionBudget, ServiceMonitor). Teams override values, not the chart:

```yaml
# values-production.yaml
image:
  repository: 123456789.dkr.ecr.eu-west-1.amazonaws.com/orders-service
  tag: "sha-abc1234"

replicaCount: 3

resources:
  requests:
    cpu: 500m
    memory: 1Gi
  limits:
    cpu: 2000m
    memory: 2Gi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 30
  targetCPUUtilizationPercentage: 60

env:
  SPRING_PROFILES_ACTIVE: production
  AWS_REGION: eu-west-1
```

---

## 8. Container Registry — Amazon ECR

- One ECR repository per service: `{service-name}`
- Images are tagged: `{git-sha}` (immutable), `{semver}`, `latest` (mutable, points to last main build)
- **Image scanning:** ECR enhanced scanning (on push) — critical and high vulnerabilities block deployment
- **Lifecycle policy:** Keep last 50 images; expire images older than 90 days
- Cross-account access: Staging and production accounts pull from the shared services ECR account

---

## 9. Internal DNS & Service Discovery

- **Internal DNS:** Route 53 Private Hosted Zone — `{service}.internal.{company}.com`
- **In-cluster:** Kubernetes DNS + Istio service registry — services address each other as `http://orders-service.orders.svc.cluster.local`
- **Cross-cluster (future):** Istio multi-cluster mesh with cross-cluster service discovery

---

## 10. Certificate Management

- **External TLS:** AWS Certificate Manager (ACM) — certificates auto-renewed, attached to ALB/CloudFront
- **Internal mTLS:** Istio manages certificates via its CA — teams never handle internal certs manually
- **No self-signed certs in any environment** (SCP enforced at org level for public endpoints)

---

## 11. Certificate Lifecycle Management

### 11.1 ACM Certificates

AWS Certificate Manager certificates attached to ALBs and CloudFront distributions are **auto-renewed** by AWS. No manual action is required. These cover the majority of the platform's external TLS.

### 11.2 Non-ACM Certificates

Third-party certificates (partner mTLS, vendor-issued certificates, client certificates for external integrations) require manual tracking:

- All non-ACM certificates are recorded in a **certificate inventory spreadsheet** (linked from the platform runbook in Backstage)
- Each entry includes: domain/CN, issuer, expiry date, owning team, renewal procedure, and associated service
- Expiry is monitored via a **CloudWatch custom metric** (`certificate.days_until_expiry`) emitted daily by a scheduled Lambda

### 11.3 Expiry Alerting

| Days Before Expiry | Action |
|---------------------|--------|
| **30 days** | Slack notification to owning team + `#platform-alerts` |
| **14 days** | PagerDuty low-urgency alert to owning team |
| **7 days** | PagerDuty high-urgency alert to owning team + engineering manager escalation |

### 11.4 Istio CA Rotation

Istio's **intermediate CA certificate** (issued by a root CA managed by Platform Engineering) is rotated **annually**. The rotation procedure is documented in the Istio CA rotation runbook and includes:

- Generating a new intermediate CA signed by the root CA
- Configuring Istio to serve both old and new CA certificates during a 48-hour overlap window
- Verifying mTLS connectivity across all namespaces before removing the old CA
- Post-rotation validation via automated mesh health checks

### 11.5 Certificate Pinning Coordination

Any server certificate change that affects mobile clients requires coordination with the mobile team:

- The mobile team must be notified **60 days before** any server certificate change (issuer change, key rotation, or domain migration)
- This allows time to ship an app update that trusts the new certificate before the old one expires
- Cross-reference: see [09-mobile-and-frontend/01-mobile-standards.md](../09-mobile-and-frontend/01-mobile-standards.md) for mobile certificate pinning implementation standards

---

## 12. Job Scheduling — Amazon EventBridge Scheduler

All scheduled tasks (previously cron jobs) are managed via EventBridge Scheduler:

- Schedules are defined in Terraform (not in application code)
- Targets are Lambda functions or EKS CronJobs
- Failed executions trigger CloudWatch alarms
- No scheduled tasks run in application containers — this creates unpredictable scaling behaviour

---

## 13. Platform Component Health

The platform team publishes a **Platform Status Page** (internal) showing the health of all shared components. Teams can subscribe to incident notifications for components they depend on.

| Component | SLA (Uptime) | Owner |
|-----------|-------------|-------|
| EKS Clusters | 99.95% | Platform |
| MSK (Kafka) | 99.9% | Platform |
| Aurora (shared infra) | 99.99% | Platform |
| ElastiCache | 99.9% | Platform |
| ArgoCD | 99.9% | Platform |
| Backstage | 99.5% | Platform |
| Secrets Manager (AWS) | 99.99% | AWS |

---

## 14. ArgoCD Service Onboarding

To onboard a new service to ArgoCD, follow these steps in order:

| Step | Action | Detail |
|------|--------|--------|
| 1 | **Create Application manifest** | Add a new `{service-name}.yaml` in `platform-config/apps/{env}/` for each target environment (dev, staging, production). Reference the shared `java-service` Helm chart and set service-specific values. |
| 2 | **Add to app-of-apps** | Register the new Application manifest in `platform-config/argocd/app-of-apps.yaml` so ArgoCD discovers it automatically. |
| 3 | **Configure sync policy** | Set `automated` sync with `selfHeal: true` for dev and staging. Production uses manual sync or automated with canary gates. Set `prune: true` to remove orphaned resources. |
| 4 | **Verify in ArgoCD UI** | After merging the PR to `platform-config`, confirm the application appears in ArgoCD UI with status `Synced` and `Healthy`. Trigger a manual sync if needed. |
| 5 | **Add to Backstage catalog** | Ensure the service's `catalog-info.yaml` includes the ArgoCD annotation (`argocd/app-name: {service-name}`) so the Backstage UI links directly to the ArgoCD application view. |

---

## 15. Platform Monitoring Ownership

The Platform Engineering team is on-call for all shared infrastructure components. Stream-aligned teams are **not** responsible for platform component health — they are consumers.

| Component | On-Call Owner |
|-----------|--------------|
| Amazon MSK (Kafka) | Platform Engineering |
| ElastiCache (Redis) | Platform Engineering |
| Aurora (shared infra databases) | Platform Engineering |
| ArgoCD | Platform Engineering |
| Backstage | Platform Engineering |
| Grafana / Observability stack | Platform Engineering |
| EKS Clusters | Platform Engineering |

### Escalation Path

```
Platform Engineer on-call
    → Platform PagerDuty (5 min)
        → Platform Engineering Manager (15 min)
            → VP Engineering (30 min)
```

Platform incidents escalate within the platform team. Stream-aligned teams report platform issues via `#platform-support` Slack channel or by paging the platform on-call directly.

---

## 16. Platform Incident Runbooks

When a shared platform component fails, the platform on-call uses these runbooks as the first line of response.

| Component | Failure Mode | Mitigation |
|-----------|-------------|------------|
| **ArgoCD** | UI or sync controller unavailable | Manual `kubectl apply` against `platform-config` manifests; restore ArgoCD pods from its own Helm chart |
| **Grafana** | Dashboards inaccessible | Fall back to CloudWatch console for core metrics; Prometheus remains accessible via port-forward |
| **Backstage** | Portal unavailable | Direct GitHub repository access for catalog, documentation, and API specs |
| **MSK (Kafka)** | Broker failure or partition under-replication | FIS runbook: verify broker health, check ISR counts, trigger MirrorMaker failover if needed |
| **ElastiCache (Redis)** | Primary node failure | Automatic failover to replica; verify application reconnection; monitor cache miss spike |
| **Aurora** | Writer instance failure | Multi-AZ automatic failover; verify connection pool reconnection; check replication lag |
| **EKS** | Control plane degradation | AWS-managed recovery; monitor node readiness; reschedule workloads if nodes are unhealthy |

Detailed runbooks for each component are maintained in the platform team's Backstage TechDocs namespace.

---

## 17. Self-Service Boundaries

The platform provides self-service capabilities for common operations while requiring tickets for changes that affect shared infrastructure or incur cost.

| Category | Capability | Provisioning Method |
|----------|-----------|-------------------|
| **Self-serve** | Kubernetes namespaces | Backstage template |
| **Self-serve** | Feature flags | LaunchDarkly UI |
| **Self-serve** | Grafana dashboards | Dashboard JSON in service repo |
| **Self-serve** | Service scaffolding | Backstage golden path template |
| **Self-serve** | Preview environments | PR-triggered GitHub Actions |
| **Ticket-based** | New databases (Aurora clusters) | Platform Jira ticket |
| **Ticket-based** | New Kafka topics | Platform Jira ticket |
| **Ticket-based** | New AWS accounts | Platform Jira ticket |
| **Ticket-based** | Redis cluster changes | Platform Jira ticket |
| **Ticket-based** | DNS record changes | Platform Jira ticket |

**Platform SLA for ticket-based requests: 1 business day** for acknowledgement and initial assessment. Provisioning timelines vary by complexity and are communicated on the ticket.

---

## 18. Platform Capacity Planning

The platform team runs a **quarterly capacity planning review** aligned with the product roadmap to ensure shared infrastructure scales ahead of demand.

| Component | Metric | Headroom Target | Review Cadence |
|-----------|--------|----------------|---------------|
| **MSK (Kafka)** | Partition count, throughput (MB/s) | 30% headroom above peak | Quarterly |
| **Aurora** | IOPS, storage, connection count | 30% headroom above peak | Quarterly |
| **ElastiCache (Redis)** | Memory utilization, connection count | 30% headroom above peak | Quarterly |
| **EKS** | Node count, CPU/memory reservation | 30% headroom above peak | Quarterly |
| **S3** | Storage volume, request rate | Reviewed for cost, not capacity | Quarterly |

### Process

1. Product and engineering leadership share the **quarterly growth forecast** (projected transaction volume, new regions, expected user growth).
2. Platform Engineering translates the forecast into infrastructure capacity requirements.
3. Capacity gaps are identified and remediation is scheduled (scaling, upgrades, new clusters).
4. Results are presented in the quarterly FinOps review alongside cost implications.

The **30% headroom target** ensures the platform can absorb traffic spikes and seasonal peaks without emergency scaling.

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
