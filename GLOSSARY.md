# 📖 Glossary

Quick reference for acronyms and terms used across this manifesto.

---

| Term | Full Name | Definition | Where to Learn More |
|------|-----------|-----------|-------------------|
| **ABAC** | Attribute-Based Access Control | Access control model that evaluates attributes (user, resource, environment) to grant or deny access | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **ADR** | Architecture Decision Record | A document capturing a significant architectural decision, its context, and consequences | [01-team-topology.md](./07-ways-of-working/01-team-topology.md) |
| **BFF** | Backend for Frontend | A server-side component tailored to serve a specific frontend client's needs | [01-mobile-standards.md](./09-mobile-and-frontend/01-mobile-standards.md) |
| **BOM** | Bill of Materials | A dependency management construct that aligns versions of related libraries | [04-android-standards.md](./09-mobile-and-frontend/04-android-standards.md) |
| **CAB** | Change Advisory Board | A governance body that reviews and approves high-risk changes to production systems | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **CDC** | Consumer-Driven Contract | A testing pattern where the consumer defines the expected API contract and the provider verifies against it | [01-testing-pyramid.md](./03-engineering-practices/01-testing-pyramid.md) |
| **CD** | Continuous Delivery / Deployment | The practice of automatically deploying code changes to production after passing all quality gates | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **CI** | Continuous Integration | The practice of frequently merging code changes into a shared branch with automated build and test validation | [02-ci-practices.md](./03-engineering-practices/02-ci-practices.md) |
| **CMDB** | Configuration Management Database | A centralised repository of information about IT assets and their relationships | [04-configuration-management.md](./04-infrastructure-and-cloud/04-configuration-management.md) |
| **Context Engineering** | Context Engineering | The discipline of structuring code, architecture, and standards as machine-readable context so AI tools generate output aligned with team conventions | [01-context-engineering.md](./12-ai-engineering/01-context-engineering.md) |
| **CQRS** | Command Query Responsibility Segregation | An architecture pattern that separates read and write data models for a service | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **CSP** | Content Security Policy | An HTTP header that restricts which resources a web page can load to mitigate XSS attacks | [02-web-frontend-standards.md](./09-mobile-and-frontend/02-web-frontend-standards.md) |
| **CVE** | Common Vulnerabilities and Exposures | A unique identifier for a publicly disclosed cybersecurity vulnerability | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **CVSS** | Common Vulnerability Scoring System | A numerical scoring system (0–10) for rating the severity of security vulnerabilities | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **DAST** | Dynamic Application Security Testing | Security testing that analyses a running application for vulnerabilities by simulating attacks | [11-qa-standards.md](./03-engineering-practices/11-qa-standards.md) |
| **DLQ** | Dead Letter Queue | A message queue that stores messages that could not be processed successfully for later inspection | [04-kafka-patterns.md](./06-developer-guides/04-kafka-patterns.md) |
| **DORA** | DevOps Research and Assessment | A set of four key metrics (deployment frequency, lead time, change failure rate, MTTR) measuring software delivery performance | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **DPA** | Data Processing Agreement | A legal contract between a data controller and data processor governing the handling of personal data | [08-privacy-engineering.md](./04-infrastructure-and-cloud/08-privacy-engineering.md) |
| **dSYM** | Debug Symbol | A file containing debug symbols for iOS builds, used to symbolicate crash reports | [05-ios-standards.md](./09-mobile-and-frontend/05-ios-standards.md) |
| **ETA** | Estimated Time of Arrival | The predicted time at which a delivery or order will reach its destination | [01-mobile-standards.md](./09-mobile-and-frontend/01-mobile-standards.md) |
| **HPA** | Horizontal Pod Autoscaler | A Kubernetes resource that automatically scales the number of pod replicas based on observed metrics | [06-capacity-planning.md](./04-infrastructure-and-cloud/06-capacity-planning.md) |
| **IRSA** | IAM Roles for Service Accounts | An AWS mechanism that allows Kubernetes pods to assume IAM roles via service account annotations | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **JIT** | Just-In-Time | An access provisioning model where permissions are granted temporarily and only when needed | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **KMS** | Key Management Service | A managed service for creating and controlling encryption keys used to encrypt data | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **KPI** | Key Performance Indicator | A measurable value that demonstrates how effectively a team is achieving key business objectives | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **LD** | LaunchDarkly | The feature flag management platform used at {Company} for progressive rollouts and experimentation | [03-cd-practices.md](./03-engineering-practices/03-cd-practices.md) |
| **LLM** | Large Language Model | A neural network trained on large text corpora capable of generating code, text, and structured output (e.g., Claude, GPT) | [02-ai-governance.md](./10-ai-ml-platform/02-ai-governance.md) |
| **MSK** | Amazon Managed Streaming for Apache Kafka | AWS-managed Kafka service used for event streaming at {Company} | [04-kafka-patterns.md](./06-developer-guides/04-kafka-patterns.md) |
| **mTLS** | Mutual Transport Layer Security | A protocol where both client and server authenticate each other using TLS certificates | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **OKR** | Objectives and Key Results | A goal-setting framework linking qualitative objectives to measurable key results | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **OTel** | OpenTelemetry | A vendor-neutral observability framework for collecting traces, metrics, and logs | [01-cloud-architecture.md](./04-infrastructure-and-cloud/01-cloud-architecture.md) |
| **PAM** | Privileged Access Management | Security controls and tools for managing and monitoring access to critical systems by privileged accounts | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **PIR** | Post-Incident Review | A structured retrospective conducted after an incident to identify root causes and preventive actions | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **PRR** | Production Readiness Review | A checklist-driven review ensuring a service meets operational standards before its first production deployment | [02-golden-path.md](./06-developer-guides/02-golden-path.md) |
| **RACI** | Responsible, Accountable, Consulted, Informed | A matrix that clarifies roles and responsibilities for tasks and decisions | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **RAG** | Retrieval-Augmented Generation | A pattern that grounds LLM responses in retrieved documents, reducing hallucination and enabling domain-specific answers | [02-ai-governance.md](./10-ai-ml-platform/02-ai-governance.md) |
| **RBAC** | Role-Based Access Control | An access control model where permissions are assigned to roles and users are assigned to roles | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **RFC** | Request for Comments | A proposal document circulated for feedback before making a significant technical decision | [05-rfc-process.md](./07-ways-of-working/05-rfc-process.md) |
| **RLS** | Row-Level Security | A database feature that restricts which rows a user can access based on policies | [09-multi-tenancy.md](./04-infrastructure-and-cloud/09-multi-tenancy.md) |
| **SAST** | Static Application Security Testing | Security testing that analyses source code for vulnerabilities without executing the program | [11-qa-standards.md](./03-engineering-practices/11-qa-standards.md) |
| **SBOM** | Software Bill of Materials | A comprehensive inventory of all components, libraries, and dependencies in a software product | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **SIEM** | Security Information and Event Management | A system that aggregates and analyses security events from across the infrastructure for threat detection | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **SLA** | Service Level Agreement | A commitment between a service provider and consumer defining expected performance and availability | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **SLI** | Service Level Indicator | A quantitative measure of a specific aspect of service performance (e.g., latency, error rate) | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **SLO** | Service Level Objective | A target value or range for an SLI that the service aims to meet (e.g., 99.9% availability) | [01-maturity-model.md](./08-program/01-maturity-model.md) |
| **SOC** | Security Operations Centre | A centralised team and facility responsible for monitoring and responding to security incidents | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **STRIDE** | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege | A threat modelling framework that categorises security threats into six types | [03-security.md](./04-infrastructure-and-cloud/03-security.md) |
| **VPA** | Vertical Pod Autoscaler | A Kubernetes component that automatically adjusts CPU and memory requests/limits for pods | [06-capacity-planning.md](./04-infrastructure-and-cloud/06-capacity-planning.md) |
| **WAF** | Web Application Firewall | A firewall that filters, monitors, and blocks HTTP traffic to and from a web application | [07-api-gateway-strategy.md](./04-infrastructure-and-cloud/07-api-gateway-strategy.md) |

---
<div align="center">

🏠 [Back to root](./README.md)

</div>
