# 🔑 Identity Providers Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Identity providers (IdPs) handle authentication, authorization, and user management across your applications. The choice between managed SaaS, open-source self-hosted, and cloud-native options affects security posture, operational burden, and vendor lock-in.

For most organizations, a managed IdP provides the best balance of security, compliance, and developer experience. Self-hosted options make sense when data sovereignty or cost at scale are primary constraints.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Protocol support | High | OAuth 2.0, OIDC, SAML, and SCIM coverage |
| MFA options | High | Passkeys, TOTP, SMS, push notification support |
| Social login | Medium | Google, Apple, GitHub, and other provider federation |
| Enterprise SSO | High | SAML and OIDC federation for B2B customers |
| User management API | Medium | Programmatic user lifecycle and role management |
| Customization | Medium | Login page branding and flow customization |
| Compliance | High | SOC 2, HIPAA, and GDPR compliance posture |
| Pricing model | Medium | Monthly active user versus flat-rate pricing |

## 🔄 3. Comparison Matrix

| Feature | Okta | Auth0 | Keycloak | Cognito |
|---------|------|-------|----------|---------|
| Managed service | Yes | Yes | Self-hosted | AWS managed |
| Social login | Excellent | Excellent | Good | Good |
| Enterprise SSO | Excellent | Strong | Good | Limited |
| MFA options | Excellent | Strong | Good | Good |
| Custom login flows | Actions | Actions | Flows | Lambda triggers |
| Machine-to-machine | Yes | Yes | Yes | Yes |
| User federation | LDAP, AD | LDAP, AD | LDAP, AD, Kerberos | LDAP (limited) |
| Passwordless | Strong | Strong | Good | Limited |
| Multi-tenancy | Organizations | Organizations | Realms | User pools |
| Pricing | Per MAU (high) | Per MAU (medium) | Free (infra cost) | Per MAU (low) |

## 🧭 4. Decision Framework

Select based on your deployment model and customer requirements:

- **Okta** - Best for enterprises needing workforce and customer identity
  - Choose when you need unified employee SSO and customer-facing authentication
  - Deepest enterprise federation and directory integration capabilities
- **Auth0** - Best developer experience for customer-facing applications
  - Choose when rapid integration and extensible login flows are priorities
  - Strong SDK ecosystem with Actions for custom authentication logic
- **Keycloak** - Best self-hosted option for full control and data sovereignty
  - Choose when you must keep identity data within your own infrastructure
  - No per-user licensing costs but requires operational investment
- **Cognito** - Best for AWS-native applications with simple auth needs
  - Choose when your stack is fully on AWS and requirements are straightforward
  - Lowest cost per MAU but limited customization and migration options

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Auth0 | Fast integration, generous free tier |
| Growth (50-200 eng) | Auth0 or Okta | Enterprise SSO for B2B sales |
| Enterprise (200-1000 eng) | Okta or Keycloak | Workforce + customer identity |
| Hyperscale (1000+ eng) | Okta + Keycloak hybrid | Managed for workforce, self-hosted for scale |

Enforce MFA for all internal applications and offer it for customer-facing products. Passwordless authentication via passkeys should be the long-term direction.

Plan token expiration, refresh strategies, and session management policies before selecting a provider. These architectural decisions are harder to change later than the provider itself.

## 📚 6. Related Documents

- [Secret Management](./36-secret-management.md)
- [API Gateways](./16-api-gateways.md)
- [Compliance Automation](./42-compliance-automation.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
