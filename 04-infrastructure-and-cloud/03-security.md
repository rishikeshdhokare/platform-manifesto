# Security Standards

> **Status:** Mandated  
> **Owner:** Platform Engineering + Security  
> **Last Updated:** 2025

---

## 1. Philosophy

Security is not a phase, a team, or a checklist. It is a practice that runs through every commit, every pipeline, and every architecture decision. **Shift left** — find and fix security issues at development time, not after they reach production.

The platform enforces security automatically where possible. Where human judgment is required, this document provides the rules.

---

## 2. Secrets Management

This is the most commonly violated security practice. Rules are absolute:

| Rule | Detail |
|------|--------|
| **No secrets in Git** | Ever. Not even in private repos. Gitleaks scans every commit. |
| **No secrets in environment variables** | Use AWS Secrets Manager + External Secrets Operator |
| **No secrets in logs** | Log masking is configured in the platform logging template |
| **No secrets in container images** | Never `COPY` a credentials file into a Dockerfile |
| **No secrets in Kubernetes manifests** | `Secret` objects in Git must only hold references, not values |

Secret violation = P1 security incident. Rotate immediately, then investigate.

---

## 3. Shift-Left Security in CI

### 3.1 Pipeline Security Stages (Mandatory)

| Stage | Tool | What It Catches | Blocks PR? |
|-------|------|----------------|-----------|
| Secret detection | **Gitleaks** | Credentials, API keys, tokens in code | Yes |
| Dependency scan | **Snyk** | CVEs in Maven dependencies | Yes (critical/high) |
| Static analysis | **SonarCloud** | SQL injection, XSS, deserialization risks | Yes (critical) |
| Container scan | **Snyk Container** | CVEs in base image and OS packages | Yes (critical/high) |
| IaC scan | **tfsec + Checkov** | Misconfigured AWS resources | Yes (high) |
| OWASP check | **OWASP Dependency-Check** | Known vulnerable libraries | Weekly, not per-PR |

### 3.2 Dependency Vulnerability Policy

| Severity | Action | Timeline |
|----------|--------|---------|
| Critical | Block deployment immediately | Fix within 24 hours |
| High | Block new deployments | Fix within 7 days |
| Medium | Tracked in Snyk | Fix within 30 days |
| Low | Logged | Fix in next maintenance window |

Snyk PRs for dependency updates are automatically raised — teams must not ignore them.

### 3.3 Base Image Policy

- All production container images must use `amazoncorretto:21-alpine` or an approved hardened base
- Images are rebuilt weekly to pick up OS-level security patches (automated pipeline)
- `latest` tag is **never** used for base images in Dockerfiles — always pin to a digest or version
- Images run as non-root (enforced by Kubernetes PodSecurityPolicy)

---

## 4. Authentication & Authorisation

### 4.1 External Authentication

- All external API endpoints require **JWT Bearer token** authentication
- JWTs are signed with RS256 (asymmetric); public key published at `/.well-known/jwks.json`
- Token validation happens at the **BFF layer** — downstream services receive a forwarded, already-validated identity context
- Token expiry: **15 minutes** for access tokens; no exceptions for convenience

### 4.2 Internal Service Authentication

- **All service-to-service calls use mTLS** via Istio — no exceptions
- Services do not implement their own mTLS; Istio's `STRICT` peer authentication mode enforces it
- `AuthorizationPolicy` objects restrict which services may call which — principle of least privilege

### 4.3 AWS IAM — IRSA

- Every pod has an associated IAM role via **IRSA (IAM Roles for Service Accounts)**
- Each role grants only the permissions that service needs — no wildcard permissions
- No static AWS credentials in any service (no `AWS_ACCESS_KEY_ID` env vars)
- IAM policies are defined in Terraform and reviewed in PRs

Example IRSA policy for orders-service:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["secretsmanager:GetSecretValue"],
      "Resource": "arn:aws:secretsmanager:eu-west-1:*:secret:/production/orders-service/*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::{company}-orders-documents/*"
    }
  ]
}
```

### 4.4 Authorisation in Services

- Services must authorise the **requesting principal** against the **requested resource**
- Never trust data in the request body for identity — use the claims in the forwarded JWT
- Implement authorisation at the **service/domain layer**, not just at the API gateway:
  ```java
  // Bad: relies solely on gateway auth
  public Order getOrder(String orderId) {
      return orderRepository.findById(orderId);
  }

  // Good: checks the calling principal owns this resource
  public Order getOrder(String orderId, Principal caller) {
      Order order = orderRepository.findById(orderId).orElseThrow();
      if (!order.getCustomerId().equals(caller.getId()) && !caller.hasRole("ADMIN")) {
          throw new AccessDeniedException("Cannot access order " + orderId);
      }
      return order;
  }
  ```

---

## 5. Data Security

### 5.1 Encryption

| Layer | Standard |
|-------|---------|
| Data in transit | TLS 1.2 minimum; TLS 1.3 preferred |
| Data at rest (RDS) | AES-256 (AWS-managed KMS key) |
| Data at rest (S3) | SSE-S3 or SSE-KMS |
| Data at rest (EBS/node volumes) | AES-256 (AWS-managed) |
| Backups | Encrypted with same KMS key as source |

TLS 1.0 and 1.1 are disabled on all ALBs and API Gateways via SCP.

### 5.2 PII Handling

The platform handles customer and provider personal data. All engineers must understand our data classification:

| Class | Examples | Rules |
|-------|---------|-------|
| **Highly Sensitive** | Payment card data, national ID | Never store raw; tokenise |
| **Personal** | Name, email, phone, location history | Encrypt at rest; access logged |
| **Operational** | Order IDs, transaction statistics | Standard controls |
| **Public** | Provider ratings (aggregated) | No special controls |

- PII must never appear in logs — log masking is configured in the platform logging template for common field names
- PII must never be used as cache keys or URL parameters
- GDPR right-to-erasure requests are handled via a data deletion service (not manual DB ops)

### 5.3 Database Security

- RDS instances are **not publicly accessible** (enforced by SCP)
- Connections require TLS (enforced by `rds.force_ssl` parameter)
- Database credentials are rotated automatically every 30 days via Secrets Manager
- Read-only service accounts for read-only use cases
- No direct developer access to production databases — all queries go through audit-logged bastion access

---

## 6. Network Security

### 6.1 Zero-Trust Posture

No service trusts another based on network location alone. Every request must be authenticated:

- External → BFF: JWT authentication
- BFF → Internal service: mTLS (Istio) + forwarded JWT claims
- Service → Service: mTLS (Istio) + AuthorizationPolicy

### 6.2 Web Application Firewall

**AWS WAF** is deployed in front of all public endpoints with the following managed rule groups enabled:
- AWS Managed Rules — Common Rule Set
- AWS Managed Rules — Known Bad Inputs
- AWS Managed Rules — SQL Database
- IP reputation list (blocks known malicious IPs)
- Rate-based rules (anti-scraping, anti-credential-stuffing)

### 6.3 DDoS Protection

- **AWS Shield Standard** — on all infrastructure by default
- **AWS Shield Advanced** — on Tier 1 critical services (additional cost, justified for platform safety criticality)

---

## 7. Security Testing

### 7.1 SAST (Static Analysis)

- **SonarCloud** runs on every PR — blocks on critical security hotspots
- Findings are tracked and must be resolved within the SLA above

### 7.2 DAST (Dynamic Analysis)

- **OWASP ZAP** runs weekly against the staging environment
- Findings are triaged by the security team and assigned to owning teams

### 7.3 Penetration Testing

- Annual external penetration test of the full platform
- Scope includes API layer, mobile applications, admin portals
- Findings are treated as P1/P2 depending on severity and resolved within SLA

### 7.4 Dependency Audit

- Snyk scans all repositories daily
- OWASP Dependency Check runs weekly in CI
- An internal Dependabot configuration auto-raises PRs for dependency updates in all repos

---

## 8. Incident Response

### 8.1 Security Incident Classification

| Level | Example | Response |
|-------|---------|---------|
| **Critical** | Data breach, credential exposure | Immediate: Platform + Security + CTO within 30 min |
| **High** | Exploited vulnerability, account takeover | Same-day response team |
| **Medium** | Unpatched CVE, misconfiguration found | 7-day remediation |

### 8.2 Credential Exposure Protocol

If a secret is detected in a commit or log:
1. **Rotate the credential immediately** — do not wait for investigation
2. Revoke the exposed credential in the relevant system
3. Scan Git history for any downstream propagation
4. Open a P1 security incident ticket
5. Conduct a post-mortem within 5 business days

### 8.3 AWS GuardDuty

GuardDuty is enabled in all AWS accounts and monitors for:
- Credential theft and unusual API calls
- Cryptocurrency mining on EC2
- Unusual network traffic
- S3 bucket compromise

GuardDuty HIGH/CRITICAL findings page on-call immediately via PagerDuty.

---

## 9. Compliance

| Framework | Status | Scope |
|-----------|--------|-------|
| **GDPR** | Ongoing | All customer/provider data processing |
| **PCI-DSS** | Level 2 | Payment processing (via tokenisation — offloaded to payment provider) |
| **SOC 2 Type II** | In progress | Platform infrastructure |

All compliance controls are mapped to the engineering practices in this manifesto. Audit evidence is collected automatically via AWS Config, CloudTrail, and Security Hub.

---

*← [Back to section](./README.md) · [Back to root](../README.md)*
