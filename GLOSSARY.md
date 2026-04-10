# 📖 Glossary

![Status: Reference](https://img.shields.io/badge/status-Reference-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

Quick reference for acronyms and terms used across this manifesto.

---

| Term | Full Name | Definition | Where to Learn More |
|------|-----------|-----------|-------------------|
| **ABAC** | Attribute-Based Access Control | Access control model that evaluates attributes (user, resource, environment) to grant or deny access | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **ADR** | Architecture Decision Record | A document capturing a significant architectural decision, its context, and consequences | [01-team-topology.md](./07-ways-of-working/01-team-topology.md) |
| **Agent Identity** | Agent Identity | A dedicated service account, GitHub App, or OIDC principal assigned to an AI agent so its actions are auditable and distinct from human engineer identities | [ONBOARDING.md](./ONBOARDING.md) |
| **Agent-Native Organisation** | Agent-Native Organisation | An engineering organisation where AI agents are first-class participants - planning, coding, reviewing, testing, deploying, and operating services alongside humans, governed by the same standards | [README.md](./README.md#-agent-native-by-design) |
| **Agent-Operated Service** | Agent-Operated Service | A service where the majority of code changes, deployments, or operational tasks are performed by AI agents under human oversight. Tagged `agent-operated` in the service catalog for visibility | [04-service-catalog.md](./01-platform-standards/04-service-catalog.md) |
| **BFF** | Backend for Frontend | A server-side component tailored to serve a specific frontend client's needs | [01-system-architecture.md](./02-architecture-and-api/01-system-architecture.md) |
| **BOM** | Bill of Materials | A dependency management construct that aligns versions of related libraries | [04-android-standards.md](./09-mobile-and-frontend/04-android-standards.md) |
| **CAB** | Change Advisory Board | A governance body that reviews and approves high-risk changes to production systems | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **CDC (Data)** | Change Data Capture | A pattern that captures row-level changes from a database and publishes them as events for downstream consumers | [05-data-platform.md](./06-developer-guides/05-data-platform.md) |
| **CDC (Testing)** | Consumer-Driven Contract | A testing pattern where the API consumer defines the expected contract (often Pact-style) and the provider verifies compliance | [01-testing-pyramid.md](./03-engineering-practices/01-testing-pyramid.md) |
| **CD** | Continuous Delivery / Deployment | The practice of automatically deploying code changes to production after passing all quality gates | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **CI** | Continuous Integration | The practice of frequently merging code changes into a shared branch with automated build and test validation | [02-ci-practices.md](./03-engineering-practices/02-ci-practices.md) |
| **CMDB** | Configuration Management Database | A centralised repository of information about IT assets and their relationships | [04-configuration-management.md](./04-infrastructure-and-cloud/04-configuration-management.md) |
| **Context Engineering** | Context Engineering | The discipline of structuring code, architecture, and standards as machine-readable context so AI tools generate output aligned with team conventions | [01-context-engineering.md](./12-ai-engineering/01-context-engineering.md) |
| **CQRS** | Command Query Responsibility Segregation | An architecture pattern that separates read and write data models for a service | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **CSP** | Content Security Policy | An HTTP header that restricts which resources a web page can load to mitigate XSS attacks | [02-web-frontend-standards.md](./09-mobile-and-frontend/02-web-frontend-standards.md) |
| **CVE** | Common Vulnerabilities and Exposures | A unique identifier for a publicly disclosed cybersecurity vulnerability | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **CVSS** | Common Vulnerability Scoring System | A numerical scoring system (0-10) for rating the severity of security vulnerabilities | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **DAST** | Dynamic Application Security Testing | Security testing that analyses a running application for vulnerabilities by simulating attacks | [11-qa-standards.md](./03-engineering-practices/11-qa-standards.md) |
| **DLQ** | Dead Letter Queue | A message queue that stores messages that could not be processed successfully for later inspection | [04-kafka-patterns.md](./06-developer-guides/04-kafka-patterns.md) |
| **DORA** | DevOps Research and Assessment | A set of four key metrics (deployment frequency, lead time, change failure rate, MTTR) measuring software delivery performance | [04-engineering-metrics.md](./08-program/04-engineering-metrics.md) |
| **DPA** | Data Processing Agreement | A legal contract between a data controller and data processor governing the handling of personal data | [08-privacy-engineering.md](./04-infrastructure-and-cloud/08-privacy-engineering.md) |
| **dSYM** | Debug Symbol | A file containing debug symbols for iOS builds, used to symbolicate crash reports | [05-ios-standards.md](./09-mobile-and-frontend/05-ios-standards.md) |
| **E2E** | End-to-End (Testing) | Tests that validate entire user flows through the full application stack | [01-testing-pyramid.md](./03-engineering-practices/01-testing-pyramid.md) |
| **ETA** | Estimated Time of Arrival | The predicted time at which a delivery or order will reach its destination | [01-mobile-standards.md](./09-mobile-and-frontend/01-mobile-standards.md) |
| **GitOps** | GitOps | An operational model where infrastructure and application configuration are declared in Git and applied via automated reconciliation | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **gRPC** | gRPC Remote Procedure Call | A high-performance RPC framework using Protocol Buffers for service-to-service communication | [05-grpc-standards.md](./02-architecture-and-api/05-grpc-standards.md) |
| **HPA** | Horizontal Pod Autoscaler | A Kubernetes resource that automatically scales the number of pod replicas based on observed metrics | [06-capacity-planning.md](./04-infrastructure-and-cloud/06-capacity-planning.md) |
| **Human Gate** | Human-in-the-Loop Gate | A point in an automated or agent-driven workflow where human approval is required before proceeding (e.g., merging to main, production deploys, architecture decisions, security-sensitive changes) | [README.md](./README.md#-agent-native-by-design) |
| **IaC** | Infrastructure as Code | Managing infrastructure through version-controlled configuration files rather than manual processes | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **IAM** | Identity and Access Management | AWS service for controlling who can do what across cloud resources | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **IRSA** | IAM Roles for Service Accounts | An AWS mechanism that allows Kubernetes pods to assume IAM roles via service account annotations | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **JIT** | Just-In-Time | An access provisioning model where permissions are granted temporarily and only when needed | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **Kafka** | Apache Kafka | A distributed event streaming platform used for building real-time data pipelines and event-driven architectures | [04-kafka-patterns.md](./06-developer-guides/04-kafka-patterns.md) |
| **KMS** | Key Management Service | A managed service for creating and controlling encryption keys used to encrypt data | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **KPI** | Key Performance Indicator | A measurable value that demonstrates how effectively a team is achieving key business objectives | [04-engineering-metrics.md](./08-program/04-engineering-metrics.md) |
| **LD** | LaunchDarkly | A feature flag management platform for progressive rollouts and experimentation. Reference implementation at {Company}: LaunchDarkly | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **LLM** | Large Language Model | A neural network trained on large text corpora capable of generating code, text, and structured output (e.g., Claude, GPT) | [02-ai-governance.md](./10-ai-ml-platform/02-ai-governance.md) |
| **MSK** | Amazon Managed Streaming for Apache Kafka | AWS-managed Apache Kafka service for event streaming. Reference implementation at {Company}: Amazon MSK | [04-kafka-patterns.md](./06-developer-guides/04-kafka-patterns.md) |
| **mTLS** | Mutual Transport Layer Security | A protocol where both client and server authenticate each other using TLS certificates | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **OKR** | Objectives and Key Results | A goal-setting framework linking qualitative objectives to measurable key results | [10-product-operations.md](./07-ways-of-working/10-product-operations.md) |
| **OTel** | OpenTelemetry | A vendor-neutral observability framework for collecting traces, metrics, and logs | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **PAM** | Privileged Access Management | Security controls and tools for managing and monitoring access to critical systems by privileged accounts | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **PIR** | Post-Incident Review | A structured retrospective conducted after an incident to identify root causes and preventive actions | [04-incident-management.md](./05-operational-excellence/04-incident-management.md) |
| **PRR** | Production Readiness Review | A checklist-driven review ensuring a service meets operational standards before its first production deployment | [02-golden-path.md](./06-developer-guides/02-golden-path.md) |
| **RACI** | Responsible, Accountable, Consulted, Informed | A matrix that clarifies roles and responsibilities for tasks and decisions | [09-engineering-management.md](./07-ways-of-working/09-engineering-management.md) |
| **RAG** | Retrieval-Augmented Generation | A pattern that grounds LLM responses in retrieved documents, reducing hallucination and enabling domain-specific answers | [02-ai-governance.md](./10-ai-ml-platform/02-ai-governance.md) |
| **RBAC** | Role-Based Access Control | An access control model where permissions are assigned to roles and users are assigned to roles | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **RFC** | Request for Comments | A proposal document circulated for feedback before making a significant technical decision | [05-rfc-process.md](./07-ways-of-working/05-rfc-process.md) |
| **RLS** | Row-Level Security | A database feature that restricts which rows a user can access based on policies | [09-multi-tenancy.md](./04-infrastructure-and-cloud/09-multi-tenancy.md) |
| **SAST** | Static Application Security Testing | Security testing that analyses source code for vulnerabilities without executing the program | [11-qa-standards.md](./03-engineering-practices/11-qa-standards.md) |
| **SBOM** | Software Bill of Materials | A comprehensive inventory of all components, libraries, and dependencies in a software product | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **SIEM** | Security Information and Event Management | A system that aggregates and analyses security events from across the infrastructure for threat detection | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **SLA** | Service Level Agreement | A commitment between a service provider and consumer defining expected performance and availability | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **SLI** | Service Level Indicator | A quantitative measure of a specific aspect of service performance (e.g., latency, error rate) | [01-observability-standards.md](./05-operational-excellence/01-observability-standards.md) |
| **SLO** | Service Level Objective | A target value or range for an SLI that the service aims to meet (e.g., 99.9% availability) | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **SOC** | Security Operations Centre | A centralised team and facility responsible for monitoring and responding to security incidents | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **STRIDE** | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege | A threat modelling framework that categorises security threats into six types | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **VPA** | Vertical Pod Autoscaler | A Kubernetes component that automatically adjusts CPU and memory requests/limits for pods | [06-capacity-planning.md](./04-infrastructure-and-cloud/06-capacity-planning.md) |
| **WAF** | Web Application Firewall | A firewall that filters, monitors, and blocks HTTP traffic to and from a web application | [07-api-gateway-strategy.md](./04-infrastructure-and-cloud/07-api-gateway-strategy.md) |

---
<div align="center">

🏠 [Back to root](./README.md)

</div>
