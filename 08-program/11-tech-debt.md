# 🧹 Technical Debt Tracking and Remediation

![Status: Mandated](https://img.shields.io/badge/status-Mandated-blue?style=flat-square) ![Owner: Engineering Leads](https://img.shields.io/badge/owner-Engineering_Leads-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 🎯 1. Overview

Technical debt is an intentional or unintentional deviation from the ideal technical state that creates ongoing maintenance burden, slows delivery, or increases risk. At {Company}, technical debt is not ignored or treated as a backlog afterthought - it is tracked, categorized, prioritized, and budgeted for remediation.

> **Rule:** Every team must allocate a minimum of 20% of sprint capacity to technical debt remediation. This is a floor, not a ceiling.

---

## 🗂️ 2. Debt Categories

| Category | Description | Examples |
|----------|-------------|---------|
| **Architecture debt** | Structural issues in system design | Monolith that should be decomposed, missing async boundaries |
| **Code debt** | Implementation shortcuts or quality gaps | Missing tests, duplicated logic, dead code |
| **Dependency debt** | Outdated or vulnerable dependencies | Libraries behind by major versions, EOL frameworks |
| **Infrastructure debt** | Manual processes or outdated infrastructure | Hand-configured servers, missing IaC, legacy CI pipelines |
| **Documentation debt** | Missing, stale, or inaccurate documentation | Outdated runbooks, missing ADRs, stale API docs |
| **Test debt** | Insufficient or unreliable test coverage | Flaky tests, missing integration tests, low coverage |

---

## 📊 3. Debt Scoring

Every tech debt item is scored on two dimensions.

| Dimension | Scale | Description |
|-----------|-------|-------------|
| **Impact** | 1 - 5 | How much does this debt slow delivery or increase risk? |
| **Effort** | 1 - 5 | How much effort is required to remediate? |

**Priority matrix:**

| Impact / Effort | Low Effort (1-2) | Medium Effort (3) | High Effort (4-5) |
|-----------------|------------------|--------------------|--------------------|
| **High Impact (4-5)** | Fix immediately | Schedule this quarter | Plan and decompose |
| **Medium Impact (3)** | Fix in next sprint | Schedule this quarter | Evaluate ROI |
| **Low Impact (1-2)** | Fix opportunistically | Backlog | Likely not worth fixing |

---

## 📋 4. Tracking Process

| Step | Action | Owner |
|------|--------|-------|
| **Identify** | Any engineer can file a tech debt ticket | Individual contributor |
| **Categorize** | Assign category, impact, and effort scores | Tech lead |
| **Prioritize** | Rank against other debt items using priority matrix | Team lead |
| **Schedule** | Assign to sprint within the 20% debt budget | Team lead |
| **Execute** | Implement remediation, verify improvement | Assigned engineer |
| **Validate** | Confirm debt is resolved, close ticket | Tech lead |

> **Rule:** Tech debt tickets must follow the same definition of done as feature tickets - including tests, documentation, and code review.

---

## 📈 5. Metrics and Reporting

| Metric | Target | Measurement |
|--------|--------|-------------|
| Debt items per service | Trending downward | Count of open debt tickets per service |
| Debt remediation velocity | >= 20% of sprint points | Sprint velocity attributed to debt work |
| Dependency currency | < 30 days behind latest patch | Automated dependency scan |
| Critical debt age | < 90 days | Time from identification to resolution for high-impact items |
| Debt-to-feature ratio | 20 - 30% of total work | Ratio of debt points to total sprint points |

**Visual overview:**

```mermaid
flowchart LR
    Identify[Identify Debt] --> Score[Score Impact + Effort]
    Score --> Prioritize[Prioritize]
    Prioritize --> Schedule[Schedule in Sprint]
    Schedule --> Execute[Remediate]
    Execute --> Validate[Validate + Close]
    Validate --> Report[Quarterly Report]
```

---

## 🚫 6. Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|-------------|------|------------|
| **Debt denial** | "We will fix it later" - later never comes | Mandatory 20% allocation enforced in sprint planning |
| **Gold plating** | Rewriting working systems for aesthetic reasons | Debt must have measurable impact on delivery or risk |
| **Invisible debt** | Debt not tracked, so it is not addressed | All debt must have a ticket in the project tracker |
| **Big bang rewrites** | Attempting to fix all debt at once | Decompose into incremental improvements |

---

## 🔗 7. Cross-References

- [Engineering Metrics](./04-engineering-metrics.md) - Delivery velocity and quality metrics
- [Engineering KPIs](./07-engineering-kpis.md) - Platform adoption and dependency currency targets

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
