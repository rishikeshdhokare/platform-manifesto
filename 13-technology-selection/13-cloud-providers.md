# ☁️ Cloud Providers Selection Guide

![Status: Reference](https://img.shields.io/badge/Status-Reference-blue) ![Owner: Architecture/Platform Engineering](https://img.shields.io/badge/Owner-Architecture%2FPlatform%20Engineering-purple) ![Updated: 2026](https://img.shields.io/badge/Updated-2026-green)

## 🎯 1. Overview

Cloud provider selection is a high-stakes, long-term decision that affects pricing, service availability, talent requirements, and vendor lock-in. While multi-cloud is often discussed, most organizations achieve better outcomes by going deep on a single provider and using portable abstractions only where justified.

Evaluate based on your dominant workload type, regional availability, existing contracts, and the managed services your architecture depends on.

## 📊 2. Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Service breadth | High | Range of managed services relevant to your workloads |
| Regional presence | High | Data center availability in your operating regions |
| Pricing and discounts | High | On-demand rates, reserved capacity, and EDP/commit deals |
| Kubernetes support | High | Managed Kubernetes quality and integration depth |
| AI/ML services | Medium | LLM hosting, training, and inference infrastructure |
| Compliance certs | High | SOC 2, ISO 27001, HIPAA, PCI, and regional compliance |
| Talent availability | High | Ease of hiring engineers experienced in the platform |
| Enterprise support | Medium | TAM quality, response times, and escalation paths |

## 🔄 3. Comparison Matrix

| Feature | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Market share | Largest | Second | Third |
| Service count | 200+ | 200+ | 150+ |
| Kubernetes (managed) | EKS | AKS | GKE (strongest) |
| Serverless compute | Lambda, Fargate | Functions, Container Apps | Cloud Run, Functions |
| Database options | RDS, DynamoDB, Aurora | Cosmos DB, SQL Database | Cloud SQL, Spanner |
| AI/ML platform | Bedrock, SageMaker | Azure OpenAI, ML | Vertex AI, Gemini |
| Data analytics | Redshift, Athena, EMR | Synapse, Fabric | BigQuery (strongest) |
| Networking | VPC, Transit Gateway | VNet, ExpressRoute | VPC, Cloud Interconnect |
| IaC support | CDK, CloudFormation | Bicep, ARM | Deployment Manager |
| Startup credits | Activate program | Founders Hub | Startup program |

## 🧭 4. Decision Framework

Select based on your primary workload and organizational context:

- **AWS** - Best default choice for most organizations
  - Choose when you need the broadest service catalog and largest partner ecosystem
  - Strongest for microservices, serverless, and event-driven architectures
- **Azure** - Best for Microsoft-centric organizations
  - Choose when you have significant Microsoft 365, Active Directory, or .NET investment
  - Azure OpenAI provides dedicated GPT model access with enterprise compliance
- **GCP** - Best for data-intensive and Kubernetes-native organizations
  - Choose when BigQuery, GKE, or Vertex AI are central to your architecture
  - Strongest managed Kubernetes experience and data analytics platform

Multi-cloud is justified only for regulatory requirements, M&A integration, or specific best-of-breed services. Avoid multi-cloud for "avoiding lock-in" alone.

## 📏 5. Recommendation by Scale

| Scale | Recommendation | Notes |
|-------|---------------|-------|
| Startup (< 50 eng) | Single cloud (AWS or GCP) | Maximize credit programs |
| Growth (50-200 eng) | Single cloud, deep investment | Negotiate committed spend discounts |
| Enterprise (200-1000 eng) | Primary + secondary for specific needs | Portable where practical |
| Hyperscale (1000+ eng) | Primary cloud + strategic multi-cloud | Dedicated cloud platform team |

Negotiate committed spend agreements early. Cloud provider discounts of 20-40% are standard for one to three year commitments and significantly affect total cost of ownership.

Invest in FinOps practices from the start. Unmanaged cloud spend grows faster than infrastructure needs, and retroactive cost optimization is significantly harder than proactive governance.

Document the cloud provider selection and commitment terms in an Architecture Decision Record (ADR).

## 📚 6. Related Documents

- [Container Orchestration](./14-container-orchestration.md)
- [IaC Tools](./15-iac-tools.md)
- [CDN Platforms](./18-cdn-platforms.md)

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
