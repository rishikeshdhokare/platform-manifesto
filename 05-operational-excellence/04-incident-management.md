# 🚨 Incident Management

![Status: Mandated](https://img.shields.io/badge/status-Mandated-blue?style=flat-square) ![Owner: Engineering Leadership + Platform](https://img.shields.io/badge/owner-Engineering_Leadership_+_Platform-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 🎯 1. Philosophy

**Incidents are normal.** Complex distributed systems fail. The measure of an engineering organisation is not whether incidents happen — it is how quickly they are detected, how calmly they are managed, and how thoroughly they are learned from.

Two rules that govern everything:
1. **Restore service first. Investigate second.** Never let debugging delay a fix you can already apply.
2. **Blameless.** Incidents are system failures, not human failures. The goal is to find what made the system fragile — not who made the mistake.

---

## 🚨 2. Severity Levels

| Severity | Definition | Examples | Response SLA |
|----------|-----------|---------|-------------|
| **P1 — Critical** | Production down or severely degraded; large-scale user impact | Customers cannot create orders; providers cannot accept; payments failing broadly | Acknowledge in 5 min; update every 15 min; resolve ASAP |
| **P2 — Major** | Significant degradation; subset of users affected | Dynamic pricing not working; notifications delayed > 5 min; specific region down | Acknowledge in 15 min; update every 30 min; resolve in 2 hours |
| **P3 — Minor** | Limited impact; most users unaffected | Single provider unable to go online; reporting dashboard slow | Next business day |
| **P4 — Low** | No user impact; early warning | Elevated error rate on non-critical path; approaching a resource limit | Weekly review |

---

## 🚨 3. Incident Response Playbook

### Phase 1: Detection (0–5 minutes)

Incidents are detected via:
- **PagerDuty alert** — automatic from Grafana alerts
- **User report** — via support ticket or Slack
- **Proactive monitoring** — on-call engineer notices anomaly

**Immediate steps:**
```
1. Acknowledge the PagerDuty alert (stops escalation)
2. Join the incident Slack channel: #incident-{date}-{short-description}
   (create it if it doesn't exist)
3. Post your first status update:
   "I am investigating. [Service/symptom]. Will update in 10 minutes."
4. Declare severity (P1/P2/P3)
```

### Phase 2: Triage (5–15 minutes)

Determine: **what is broken, who is affected, and how bad is it?**

Triage questions:
```
- Which service(s) are affected?
- What is the user-visible impact? (Can customers create orders? Can providers accept?)
- When did it start? (Check Grafana — look for the inflection point)
- What changed recently? (Recent deployments? Config changes? Infra changes?)
- Is it getting worse, stable, or recovering on its own?
```

Useful commands:
```bash
# Check pod status
kubectl -n orders-production get pods
kubectl -n orders-production describe pod {pod-name}

# Check recent logs
kubectl -n orders-production logs -l app=orders-service --tail=100 --since=15m

# Check if the service was recently deployed
argocd app history orders-service

# Check Grafana — error rate and latency for the affected service
open https://grafana.{company}.internal/d/orders-service
```

### Phase 3: Mitigation (varies — minutes to hours)

**Goal: restore service as fast as possible.** Use the simplest fix available first.

Mitigation toolkit (fastest to slowest):

| Action | When to Use | How |
|--------|------------|-----|
| **Turn off a feature flag** | New feature is causing the issue | LaunchDarkly → turn off flag |
| **ArgoCD rollback** | Recent deployment caused the issue | `argocd app rollback {service} --revision {n-1}` |
| **Scale up** | Service is under load and running out of capacity | `kubectl scale deployment orders-service --replicas=10` |
| **Restart pods** | Pods are in a bad state (memory leak, deadlock) | `kubectl rollout restart deployment/orders-service` |
| **Circuit breaker** | A downstream service is causing cascading failures | Turn on manual circuit breaker flag in LaunchDarkly |
| **Disable a non-critical feature** | A non-critical path is consuming too many resources | Feature flag off |

**Don't:**
- Make ad-hoc code changes under pressure — this introduces more bugs
- Bypass CI/CD to push a "quick fix" directly to production
- Make database schema changes manually to fix a live issue

### Phase 4: Communication

While the incident is ongoing, communicate continuously.

**Internal (Slack #incident channel):**
```
[14:32] 🔴 P1 INCIDENT: Orders service returning 500 errors for ~40% of requests.
         Investigating. On-call: @john.doe

[14:38] Update: Issue started ~14:25. Correlated with orders-service v2.14.5 deployment.
         Rolling back to v2.14.4. ETA 3 minutes.

[14:41] ✅ Rollback complete. Error rate returning to normal. Monitoring.

[14:55] ✅ RESOLVED. Normal operation confirmed. PIR to follow.
```

**External (status page) — for P1/P2:**
```
14:30 — Investigating reports of issues with order requests
14:45 — We have identified the issue and are implementing a fix
15:00 — Service has been restored. We are monitoring for stability
```

### Phase 5: Resolution

An incident is resolved when:
- The user-visible impact has stopped
- The service metrics have returned to baseline
- You have monitored for at least 15 minutes post-fix

When resolved:
```
1. Post resolved status in #incident channel
2. Update external status page
3. Resolve PagerDuty incident
4. Schedule PIR (within 5 business days for P1; 10 days for P2)
```

---

## 🚨 4. Roles During an Incident

| Role | Responsibility | Who |
|------|--------------|-----|
| **Incident Commander (IC)** | Coordinates the response; makes decisions; drives to resolution | On-call engineer (or most senior available) |
| **Technical Investigator** | Digs into root cause; proposes mitigations | Domain expert(s) |
| **Comms Lead** | Writes updates to Slack and status page | IC or nominated engineer |
| **Executive Escalation** | Notified for P1 incidents affecting >20% of users | CTO / VP Engineering |

For small teams, one person may play IC + Technical Investigator. That's fine — just be clear about who is doing what.

---

## 🔄 5. Post-Incident Review (PIR)

The PIR is **not a blame session**. It is an engineering exercise to understand what made the system fragile and how to prevent recurrence.

**Mandatory for:** All P1 incidents; all P2 incidents lasting > 30 minutes.

**Timeline:** Within 5 business days of resolution.

**Attendees:** Incident responders + Tech Lead + Platform Engineering rep.

### PIR Template

```markdown
# Post-Incident Review — {Title}

**Date of incident:** YYYY-MM-DD
**Duration:** {HH:MM} (from first alert to resolution)
**Severity:** P1 / P2
**Services affected:** [list]
**User impact:** [describe what users could not do]
**Authored by:** [name(s)]
**PIR date:** YYYY-MM-DD

---

## Timeline of Events

| Time (UTC) | Event |
|------------|-------|
| 14:23 | Grafana alert fires: orders-service error rate > 1% |
| 14:25 | On-call engineer acknowledges PagerDuty alert |
| 14:32 | Root cause identified: null pointer in new price calculation code |
| 14:38 | Rollback initiated |
| 14:41 | Rollback complete; error rate recovering |
| 14:55 | Incident resolved |

---

## What Happened

[2-3 paragraphs: what failed, what the user experienced, what caused it]

Example:
At 14:23 UTC, the orders-service began returning HTTP 500 errors for approximately 
40% of order completion requests. This was caused by a null pointer exception in 
the new price calculation code introduced in v2.14.5, deployed at 13:45 UTC.

The exception occurred when an order had no assigned service type — a case that 
exists for orders created before service types were introduced, and which was not 
handled in the new code path.

---

## Root Cause

[One clear sentence]

Example: The v2.14.5 release of orders-service introduced a null pointer exception 
when completing orders created before the service_type field was added.

---

## Contributing Factors

[What made this possible — the system conditions, not the person]

- The new code path had no test coverage for orders with null service_type
- The staging environment only contained recent test data; no old orders existed to trigger the bug
- The deployment went to 100% traffic without a canary phase (canary was bypassed for a "low-risk" change)

---

## What Went Well

[Honest assessment of things that worked]

- PagerDuty alert fired within 2 minutes of the error rate rising
- On-call engineer identified the deployment correlation quickly using Grafana
- Rollback was completed in under 4 minutes

---

## What Went Poorly

[Honest assessment of things that didn't work]

- No integration test existed for orders with legacy data shape
- Canary deployment was bypassed — "it's a small change" is not sufficient justification
- External status page update was 8 minutes late

---

## Action Items

| Action | Owner | Due Date | Tracking |
|--------|-------|---------|---------|
| Add test coverage for orders with null service_type | @jane.doe | 2024-12-01 | ORD-9876 |
| Enforce canary deployment for ALL releases — no bypass | @platform-team | 2024-11-20 | PLAT-4321 |
| Add data seeding to staging with legacy order shapes | @john.doe | 2024-12-15 | ORD-9877 |
| Update on-call runbook to include status page update step | @john.doe | 2024-11-18 | ORD-9878 |

---

## Metrics

| Metric | Value |
|--------|-------|
| Time to detect | 2 minutes |
| Time to acknowledge | 7 minutes |
| Time to mitigate | 16 minutes |
| Time to resolve | 32 minutes |
| Users affected (estimated) | ~4,200 customers |
| Orders failed | ~380 |
```

---

## 🚨 6. On-Call Expectations

- **Response time:** Acknowledge P1 PagerDuty alerts within 5 minutes, at any hour
- **Handoff:** On-call rotation is weekly; hand off with a written status of any open issues
- **Runbooks:** Every P1/P2 alert has a runbook — use it, update it if it's wrong
- **Escalation:** If you're stuck after 20 minutes, escalate to Tech Lead or another engineer — do not solo-fight a P1
- **After a hard incident:** If you were on-call through a difficult P1, you're entitled to take time off — tell your manager

---

## 📋 7. Runbook Template

Every service must have a runbook at `docs/runbook.md`. Use this template:

```markdown
# {Service Name} Runbook

## Service Overview
[One paragraph: what it does, what it owns]

## Architecture
[Link to architecture diagram or brief description]

## Common Alerts

### {Alert Name}
**Severity:** P1 / P2
**What it means:** [What is happening]
**User impact:** [What users are experiencing]

**Immediate checks:**
1. Check the Grafana dashboard: [link]
2. Check recent deployments: `argocd app history {service}`
3. Check pod status: `kubectl -n {namespace} get pods`

**Common causes and fixes:**

**Cause 1: Database connection exhausted**
- Check: `kubectl -n {namespace} logs -l app={service} | grep "HikariPool"`
- Fix: Scale up pods or restart to clear connections
  ```bash
  kubectl -n {namespace} rollout restart deployment/{service}
  ```

**Cause 2: Downstream service {X} is unavailable**
- Check: [link to downstream service dashboard]
- Fix: Circuit breaker should auto-open. If not:
  ```
  LaunchDarkly → disable flag: {service}-circuit-breaker-{downstream}
  ```

## Escalation
- On-call: PagerDuty → team-{domain}
- Tech Lead: @{name}
- Downstream services: [list with on-call contacts]

## Useful Commands
```bash
# View live logs
kubectl -n {namespace}-production logs -l app={service} -f

# Check pod resource usage
kubectl -n {namespace}-production top pods

# Describe a specific pod
kubectl -n {namespace}-production describe pod {pod-name}

# Force restart
kubectl -n {namespace}-production rollout restart deployment/{service}
```
```

---

## 🔄 8. Incident Pattern Analysis

Individual PIRs catch specific failures. Pattern analysis catches systemic weaknesses.

**Quarterly process:**
1. Aggregate all P1/P2 incidents from the quarter
2. Classify by failure category:

| Category | Examples |
|----------|---------|
| Deployment-related | Canary bypass, config drift, missing feature flag |
| Data/schema | Migration bug, null field, schema incompatibility |
| Capacity | Connection pool exhaustion, Kafka lag, OOM |
| Dependency | Downstream timeout, circuit breaker not configured |
| Operational | Runbook missing, alert fatigue, delayed response |

3. Identify patterns: are 3+ incidents in the same category? That's a systemic issue requiring a platform-level fix, not a service-level patch.
4. Output: top 3 systemic risks → assigned to Platform Engineering or relevant Tech Lead as quarterly OKR items.

---

## 🔄 9. Reliability Review Board

Monthly cross-team review of production reliability health.

**Attendees:** All Tech Leads + Platform Lead + CTO (optional).

**Agenda:**
1. SLO compliance per service (Grafana dashboard review)
2. Error budget burn rate — any service that consumed >50% of budget in 30 days
3. Incident pattern analysis results (quarterly)
4. Open P1/P2 action items from PIRs — are they being closed?
5. Upcoming high-risk changes (schema migrations, major features)

**Output:** Reliability risk register updated. Teams in "reliability mode" (error budget exhausted) report progress on recovery plan.

---

## 📋 10. Production Readiness Review (PRR)

Mandatory for any new service before it receives production traffic. This is the gateway from "deployed" to "production-ready."

**Reviewer:** Platform Engineering team member.

### PRR Checklist

```
Observability
[ ] Structured JSON logging with correlation IDs verified
[ ] Prometheus metrics endpoint active and scraped
[ ] Grafana dashboard with RED metrics + business metrics
[ ] P1/P2 alert rules configured with runbook links
[ ] Distributed tracing (OTel agent) confirmed in traces

Resilience
[ ] Circuit breakers on all outbound sync calls
[ ] Timeouts configured (no unbounded waits)
[ ] Kafka consumers have DLQ configured
[ ] Graceful shutdown (30s drain period)
[ ] Load tested at 2x expected peak

Operations
[ ] Runbook complete at docs/runbook.md
[ ] PagerDuty service configured and routing tested
[ ] On-call rotation assigned (minimum 4 engineers)
[ ] SLO defined and tracked in Grafana

Security
[ ] Snyk scan: 0 critical/high vulnerabilities
[ ] IRSA role scoped to minimum permissions
[ ] No secrets in Git (Gitleaks clean)
[ ] Authorisation enforced at service level (not just gateway)

Deployment
[ ] Canary deployment configured (Argo Rollouts)
[ ] Feature flags for all unreleased features
[ ] Rollback tested and verified
[ ] HPA configured (min 3 replicas)
```

A service that fails PRR cannot receive production traffic until all items are resolved. There are no exceptions.

---

## 👁️ 11. Error Budget Enforcement

When a service exhausts its monthly error budget:

1. **Team enters "reliability mode"** — no new feature work until budget recovers
2. Tech Lead reports recovery plan to Reliability Review Board
3. CTO approves any exceptions (rare — only for business-critical launches)
4. Feature work resumes only after the service has sustained >50% remaining budget for 7 consecutive days

This is not punitive — it is a structural incentive to invest in reliability before it becomes an incident.

---

## 🚨 12. On-Call Structure

### 12.1 Rotation Model

Every team operating production services must maintain an on-call rotation with **primary and secondary** engineers.

| Role | Responsibility |
|------|---------------|
| **Primary** | First responder — receives all PagerDuty alerts, acknowledges and begins investigation |
| **Secondary** | Backup — receives escalation if primary does not acknowledge within 5 minutes |

### 12.2 Rotation Schedule

- **Rotation length:** Weekly
- **Handoff time:** Monday 10:00 AM local time
- **Handoff format:** Synchronous (Slack huddle or brief call) using the handoff checklist

### 12.3 Handoff Checklist

At every rotation handoff, the outgoing on-call walks the incoming on-call through:

```
[ ] Open incidents — any active or recently resolved incidents
[ ] Active alerts — any alerts currently firing or recently resolved
[ ] Recent deployments — services deployed in the last 48 hours (check ArgoCD)
[ ] Known issues — degraded services, upcoming maintenance, ongoing investigations
[ ] Pending PIR actions — any action items assigned to the team that are in progress
```

The checklist is posted in the team's Slack channel as a written record.

### 12.4 PagerDuty Escalation Policy

| Elapsed Time | Escalation Step |
|-------------|-----------------|
| 0 min | Alert sent to **primary on-call** |
| 5 min (no ack) | Escalate to **secondary on-call** |
| 15 min (no ack) | Escalate to **Engineering Manager** |
| 30 min (no ack) | Escalate to **VP Engineering** |

All escalation policies are configured in PagerDuty and tested quarterly to verify contact details and routing.

### 12.5 Follow-the-Sun

Follow-the-sun on-call is **not required** until a team spans more than 8 timezone hours. Until then:

- Single-timezone rotation with after-hours coverage
- After-hours expectations: engineer must be reachable (not necessarily at a desk) and able to respond within 15 minutes
- If the team grows to span > 8 TZ hours, transition to follow-the-sun with regional handoffs

### 12.6 Compensation

On-call engineers receive **time-off-in-lieu** for after-hours pages:

| Scenario | Compensation |
|----------|-------------|
| Page outside business hours (resolved in < 30 min) | 1 hour time off |
| Page outside business hours (resolved in > 30 min) | Actual time spent, rounded up to next hour |
| Weekend/holiday page | 1.5× the time spent |

Time off is tracked by the Engineering Manager and taken within the same quarter. Unused compensation does not roll over.

### 12.7 Burnout Prevention

| Guardrail | Policy |
|-----------|--------|
| **Max consecutive rotations** | 2 — no engineer may be on-call for more than 2 consecutive weeks |
| **Page volume target** | Max 10 pages per rotation — if exceeded, team must review alert hygiene (see Observability Standards, Section 11) |
| **Monthly review** | Engineering Managers review page volume per team member monthly; uneven distribution is addressed |
| **Post-incident rest** | After a P1 incident requiring > 2 hours of active response, the on-call engineer may take the next business day off |

### 12.8 Staffing Requirement

Every on-call rotation must have a **minimum of 4 engineers** to ensure sustainable coverage. Teams with fewer than 4 engineers share on-call with a related team or escalate to the Platform Engineering on-call as secondary.

---

## 🔄 13. PIR Action Item Governance

### 13.1 Tracking

All action items from Post-Incident Reviews are:

- Created as **Jira tickets** immediately after the PIR meeting
- Labelled with `pir-action` for tracking and reporting
- Linked to the PIR document (Confluence page or Git-hosted markdown)
- Assigned to a specific owner with a due date

### 13.2 SLA to Close

| Action Priority | Closure SLA | Determination |
|----------------|-------------|---------------|
| **P1 action** | 30 calendar days | Action prevents recurrence of a P1 incident |
| **P2 action** | 60 calendar days | Action prevents recurrence of a P2 incident or addresses a contributing factor |

Priority is determined during the PIR meeting based on the severity of the original incident and the criticality of the action item.

### 13.3 Verification

Action items are marked as **done** only after verification confirms the fix is effective:

| Verification Method | When to Use |
|--------------------|-------------|
| Automated test added and passing | Code-level fixes (null handling, validation) |
| Alert rule confirmed firing correctly | Observability improvements |
| Monitoring confirms no recurrence for 7 days | Operational changes (scaling, config) |
| Load test passing | Capacity-related fixes |

"I deployed the fix" is not sufficient — verification evidence must be linked in the Jira ticket.

### 13.4 Escalation

| Condition | Escalation |
|-----------|-----------|
| P1 action overdue by > 7 days | Flagged in weekly engineering leadership sync |
| P2 action overdue by > 14 days | Flagged in weekly engineering leadership sync |
| Any action overdue by > 30 days beyond SLA | Escalated to VP Engineering |

The Platform Engineering team maintains a dashboard of all open PIR action items, visible to engineering leadership.

### 13.5 Quarterly PIR Review

Every quarter, the Reliability Review Board conducts a PIR retrospective:

1. **Completion rate:** What percentage of PIR actions were closed within SLA?
2. **Systemic patterns:** Are multiple PIRs identifying the same root cause category? (e.g., "missing tests", "no canary", "missing circuit breaker")
3. **Effectiveness:** Did completed actions prevent recurrence? (Check: has a similar incident occurred since the action was completed?)
4. **Manifesto updates:** If a pattern is systemic, update this manifesto with a new standard or guardrail to prevent the class of failure

Target: **> 90% of PIR actions closed within SLA** each quarter.

---

## 🔄 14. Post-Launch Reliability Review

Every new service undergoes a **30-day post-launch reliability review** after receiving production traffic. This review is mandatory before removing the enhanced monitoring put in place during the Production Readiness Review (PRR).

### Review Criteria

| Metric | Threshold | Source |
|--------|-----------|--------|
| SLO adherence | ≥ target SLO for 30 consecutive days | Grafana SLO dashboard |
| Error rate | Stable at or below baseline | Prometheus |
| P99 latency | Within defined SLO | Prometheus |
| On-call burden | < 5 pages attributable to the new service | PagerDuty |
| Open P1/P2 incidents | 0 unresolved incidents | Incident tracker |

### Process

1. At day 30, the owning tech lead schedules a review with Platform Engineering
2. Review covers the criteria above plus a walkthrough of any incidents that occurred
3. If all criteria are met, enhanced PRR monitoring (additional alerts, tighter thresholds) is removed
4. If any criteria are not met, enhanced monitoring continues for another 15 days with a follow-up review

---

## 🔄 15. Change Management

### 15.1 Change Freeze Calendar

A change freeze calendar is published **quarterly** covering:

- National and regional holidays
- Major promotional events
- End-of-quarter financial close periods
- Any company-wide events with high customer visibility

During freeze windows, only emergency hotfixes are permitted. All other deployments are paused.

The calendar is published to **#platform-announcements** on Slack and maintained in the team wiki.

### 15.2 Change Risk Rubric

| Risk Level | Characteristics | Required Controls |
|------------|----------------|-------------------|
| **Low** | Configuration change; auto-rollback enabled; single service | Standard canary deployment; no additional approval |
| **Medium** | Single-service code change with canary deployment | Canary with extended observation window (30 min); service owner approval |
| **High** | Schema migration, multi-service coordinated change, manual approval gates, or changes with no automated rollback | Manual approval from tech lead + platform engineering; announced 48 hours in advance on #platform-announcements; rollback plan documented and tested |

### 15.3 High-Risk Change Announcements

All high-risk changes must be announced at least **48 hours in advance** in the **#platform-announcements** Slack channel with:

- Description of the change
- Expected impact window
- Rollback plan
- Point of contact during the change

---

## 📊 16. MTTx Targets

### 16.1 Organisation-Wide Targets

| Severity | MTTD (Mean Time to Detect) | MTTR (Mean Time to Resolve) | Tracking |
|----------|---------------------------|----------------------------|----------|
| **P1** | < 5 minutes | < 30 minutes | PagerDuty + Grafana |
| **P2** | < 15 minutes | < 4 hours | PagerDuty + Grafana |
| **P3** | Best effort | Next business day | Jira |
| **P4** | Best effort | Weekly review | Dashboard |

### 16.2 Measurement

- MTTD is measured from the onset of user impact (determined retrospectively from metrics) to the first alert firing or human detection
- MTTR is measured from first alert to incident resolution (user impact ceased)
- Both metrics are tracked per incident in the PIR and aggregated in the **monthly SRE review**
- Trends are reported to the Reliability Review Board quarterly

---

## 🔄 17. Service Dependency Mapping

### 17.1 Source of Truth

| Dependency Type | Source of Truth | Tooling |
|-----------------|----------------|---------|
| Synchronous (HTTP/gRPC) | Backstage service graph | `catalog-info.yaml` → `dependsOn` field |
| Asynchronous (Kafka) | EventCatalog | Event YAML definitions (producers/consumers) |
| Infrastructure | Terraform dependency graph | Terraform state |

### 17.2 Backstage Service Graph

Every service's `catalog-info.yaml` must declare its dependencies:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: orders-service
spec:
  type: service
  owner: team-orders
  dependsOn:
    - component:payment-service
    - component:fulfillment-engine
    - resource:orders-database
    - resource:orders-kafka-topic
```

The Backstage service graph is the primary tool for visualising service relationships during incidents.

### 17.3 Blast-Radius Analysis

Every **Tier 1 service** must have a documented blast-radius analysis answering:

| Question | Documentation Location |
|----------|----------------------|
| Which upstream services depend on this service? | Backstage reverse dependency graph |
| Which downstream services does this service call? | `catalog-info.yaml` `dependsOn` |
| What is the user impact if this service is completely unavailable? | Service runbook |
| What is the user impact if this service is degraded (high latency)? | Service runbook |
| What circuit breakers or fallbacks exist? | Service runbook + architecture diagram |

Blast-radius documentation is reviewed during the Production Readiness Review (Section 10) and updated after any architectural change.

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
