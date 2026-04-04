# 🧑‍💻 Developer Experience

![Status: Mandated](https://img.shields.io/badge/status-Mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 🎯 1. Philosophy

Developer experience is a force multiplier. An engineer who can go from zero to running code locally in under an hour, get a PR reviewed in under a day, and deploy to production confidently in under 15 minutes is more effective than one wrestling with environment setup and opaque tooling.

**Our DX goal:** Every engineer, on their first week, should be able to run any service locally, submit a PR, and watch it go to production — with no hand-holding.

---

## 🎯 2. Onboarding Target

A new engineer must be able to reach these milestones independently:

| Milestone | Target Time |
|-----------|------------|
| Laptop setup complete | Day 1 |
| Any service running locally | End of Day 1 |
| First PR submitted | End of Day 2 |
| First PR merged | End of Day 3 |
| First feature deployed to production | End of Week 1 |
| First feature PR merged, attend sprint ceremonies, complete security orientation (see [security-operations.md](../04-infrastructure-and-cloud/10-security-operations.md)) | End of Week 2 |
| On-call shadow shift completed (pair with primary on-call), domain deep-dive with tech lead, attend architecture clinic | End of Week 3 |
| Solo on-call eligible (added to PagerDuty rotation), buddy graduation (feedback form), 30-day check-in with EM | End of Week 4 |

If these targets are not met, it is a **platform problem**, not an engineer problem.

**Visual overview:**

```mermaid
gantt
    title Onboarding Journey
    dateFormat X
    axisFormat %s
    section Week 1
        Laptop setup            :done, d1, 0, 1d
        First PR merged         :done, d2, 1d, 2d
        Local service running   :done, d3, 2d, 2d
    section Week 2
        Feature PR merged       :active, d4, 5d, 3d
        Sprint ceremonies       :d5, 5d, 5d
    section Week 3
        On-call shadow          :d6, 10d, 3d
        Domain deep-dive        :d7, 12d, 2d
    section Week 4
        Solo on-call eligible   :d8, 15d, 3d
        Buddy graduation        :d9, 18d, 2d
```

---

## 💻 3. Local Development

### 3.1 Prerequisites

The platform provides a `setup.sh` script that installs all required tools on a fresh macOS or Ubuntu machine:

```bash
curl -s https://platform.{company}.internal/setup.sh | bash
```

This installs:
- SDKMAN + Amazon Corretto 21
- Docker Desktop
- Gradle (via wrapper in each repo)
- AWS CLI v2 + SSO configuration
- `kubectl` + `kubectx` + `kubens`
- `helm`
- `argocd` CLI
- `gh` (GitHub CLI)
- `pre-commit` (Git hooks)
- LocalStack CLI

### 3.2 Running a Service Locally

Every service in the platform follows the same pattern:

```bash
# Clone the repo
gh repo clone {company}/orders-service
cd orders-service

# Start all dependencies (Postgres, Redis, Kafka via Docker Compose)
docker compose up -d

# Run the service
./gradlew bootRun --args='--spring.profiles.active=local'
```

This should work. If it doesn't, it's a bug in the service's developer setup.

### 3.3 Docker Compose Standard

Every service ships a `docker-compose.yml` at the repo root that starts all its dependencies:

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    ports: ["5432:5432"]
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orders_user
      POSTGRES_PASSWORD: local_password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports: ["9092:9092"]
    environment:
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_NODE_ID: 1
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      CLUSTER_ID: "local-cluster-id"

volumes:
  postgres_data:
```

### 3.4 Application Local Profile

`application-local.yml` configures the service to use local Docker dependencies and disables cloud integrations:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/orders
    username: orders_user
    password: local_password
  kafka:
    bootstrap-servers: localhost:9092

# Use LocalStack for AWS services in local dev
aws:
  endpoint-override: http://localhost:4566
  region: eu-west-1

# Feature flags in offline mode locally
launchdarkly:
  offline: true
  default-flags:
    fulfillment-ml-model-release: false
```

### 3.5 AWS Services Locally — LocalStack

For services that use S3, SQS, Secrets Manager, etc., use **LocalStack**:

```bash
# Start LocalStack
localstack start -d

# Create local S3 bucket (from service's local-setup.sh)
awslocal s3 mb s3://{company}-orders-documents
```

Each service's `local-setup.sh` script provisions all required local AWS resources.

---

## 🛤️ 4. Git Hooks (pre-commit)

Every repository ships with a `.pre-commit-config.yaml` that runs lightweight checks before commit:

```yaml
repos:
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.18.0
  hooks:
  - id: gitleaks           # Secret detection before commit

- repo: local
  hooks:
  - id: checkstyle
    name: Checkstyle
    entry: ./gradlew checkstyleMain --quiet
    language: system
    pass_filenames: false

  - id: commitlint
    name: Conventional Commits lint
    entry: npx commitlint --edit
    language: node
    stages: [commit-msg]
```

Install hooks: `pre-commit install --hook-type commit-msg`

---

## 💻 5. IDE Setup

### 5.1 Recommended IDE: IntelliJ IDEA

The platform provides a shared IntelliJ settings repository with:
- Code style configuration (Google Java Style)
- Run configurations for all services
- Checkstyle plugin configuration
- SonarLint plugin configuration

Import: `File → Manage IDE Settings → Import Settings → platform-ide-settings.zip`

### 5.2 Required Plugins

| Plugin | Purpose |
|--------|---------|
| **Checkstyle-IDEA** | Inline style violations |
| **SonarLint** | Inline security and quality issues |
| **EnvFile** | Load `.env.local` for run configurations |
| **AWS Toolkit** | AWS resource browsing and Lambda testing |
| **OpenAPI (Swagger) Editor** | Spec validation and preview |

---

## 🛤️ 6. Service Scaffolding — Golden Path Template

New services are created using the Backstage service template — not by copying an existing service.

```
Backstage → Create → Java Microservice Template
```

The template generates:
- GitHub repository with correct branch protection rules
- Standard project structure (hexagonal architecture)
- Pre-configured `build.gradle` with platform BOM
- `docker-compose.yml` and `application-local.yml`
- GitHub Actions workflows (CI pipeline)
- ArgoCD application manifest (in platform-config)
- `catalog-info.yaml` (Backstage registration)
- Standard `logback-spring.xml`
- ArchUnit test class
- README template

From template creation to running locally: **< 15 minutes**.

---

## 🏗️ 7. Platform BOM (Bill of Materials)

All services import the **platform BOM** which pins all dependency versions. Teams never specify versions for platform-managed dependencies:

```kotlin
// build.gradle.kts
dependencies {
    implementation(platform("com.{company}:platform-bom:2024.11"))

    // No versions needed — managed by the BOM
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("io.micrometer:micrometer-registry-prometheus")
    implementation("org.springframework.kafka:spring-kafka")
    implementation("io.opentelemetry:opentelemetry-api")

    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.testcontainers:postgresql")
    testImplementation("org.testcontainers:kafka")
    testImplementation("com.github.tomakehurst:wiremock-jre8")
    testImplementation("au.com.dius.pact.provider:junit5spring")
}
```

The platform BOM is updated quarterly. Security patches trigger out-of-band updates.

---

## ⚡ 8. Inner Loop Speed Targets

The inner loop is the cycle: **write code → compile → test → run**. Slow inner loops kill focus.

| Activity | Target |
|----------|--------|
| Clean build (no cache) | < 60 seconds |
| Incremental build | < 10 seconds |
| Unit test suite (single module) | < 30 seconds |
| Integration test suite | < 3 minutes |
| Hot reload (Spring DevTools) | < 3 seconds |
| Application startup (local) | < 15 seconds |

If a service consistently misses these targets, the team must invest in improving build performance.

---

## 📏 9. Documentation Standards

### 9.1 What to Document

| Document | Location | Audience |
|----------|---------|---------|
| Service overview | `README.md` | All engineers |
| Local dev setup | `README.md#local-development` | New joiners |
| API reference | `api/openapi.yaml` → Backstage | API consumers |
| Architecture decisions | `docs/adr/` | Team + future engineers |
| Runbook | `docs/runbook.md` + Backstage | On-call engineers |
| Performance baseline | `docs/performance.md` | Platform + team |

### 9.2 README Standard

Every service README must contain:

```markdown
# Service Name

One sentence description of what this service does.

## Responsibilities
- What this service owns
- What it does NOT own

## Architecture
Brief description + link to architecture diagram

## Local Development
Prerequisites and steps to run locally (must work!)

## Configuration
Key configuration properties and where they come from

## API
Link to OpenAPI spec / Backstage API page

## Dependencies
- Upstream services this depends on
- Downstream services that depend on this

## On-call
Link to runbook and PagerDuty service
```

---

## 📈 10. Feedback & Platform Improvement

The platform exists to serve engineering teams. Teams should provide feedback via:
- **Monthly DX survey** — NPS-style, 3 questions, 2 minutes
- **#platform-feedback** Slack channel — for immediate issues
- **Backstage → Feedback** — for feature requests

Platform team SLA for responding to feedback: 2 business days.

---

## 🤝 11. Onboarding Buddy Program

Every new joiner is assigned an **onboarding buddy** from their team. The buddy is not the new joiner's manager.

### Buddy Responsibilities

| Period | Cadence | Activities |
|--------|---------|------------|
| Week 1 | Daily check-in | Orient to codebase, team rituals, tooling; answer questions; pair programming on first PR |
| Week 2 | 2–3 check-ins | First feature PR merged; attend sprint ceremonies (planning, standup, retro); complete security orientation (see [security-operations.md](../04-infrastructure-and-cloud/10-security-operations.md)) |
| Week 3 | Weekly check-in | On-call shadow — pair with primary on-call for one shift; domain deep-dive session with tech lead; attend architecture clinic |
| Week 4 | Weekly check-in | Solo on-call eligible (added to PagerDuty rotation); buddy graduation (feedback form submitted); 30-day check-in with engineering manager |

### Program Details

| Parameter | Value |
|-----------|-------|
| Duration | 4 weeks |
| Selection | Rotating among team members |
| Engineering ladder | Counts toward the "mentoring" criterion |

### Feedback

At the end of the 4-week period:
- Both the new joiner and the buddy fill out a feedback form
- Results are reviewed by the engineering manager
- Aggregate feedback is used to improve the onboarding process quarterly

---

## 📏 12. Documentation Standards

### Required Metadata

Every markdown document in the platform manifesto and service documentation must include:

| Field | Requirement |
|-------|-------------|
| `Last Updated` | Date in the document header; updated with every substantive change |
| `Owner` | Team or individual responsible for the document's accuracy |

### Automated Quality Checks

| Check | Frequency | Action on Failure |
|-------|-----------|-------------------|
| Broken link detection | Monthly CI job | Broken links filed as Jira issues assigned to document owner |
| Staleness detection | Monthly scan | Docs not updated in > 6 months flagged for review by owner |

### Behavior Change Documentation

Any PR that changes **observable behavior** (API response format, configuration defaults, error handling, etc.) must update the relevant documentation. This is enforced via the PR template checklist:

```markdown
## PR Checklist
- [ ] If this PR changes observable behavior, I have updated the relevant documentation
```

### API Documentation

- **OpenAPI spec** is the source of truth for all API documentation
- Backstage TechDocs pages are **auto-generated** from the OpenAPI spec
- Manual API documentation is not maintained separately — it drifts and becomes a liability

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
