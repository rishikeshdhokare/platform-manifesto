# 🔔 Alerting Design Standards

![Status: Mandated](https://img.shields.io/badge/status-Mandated-blue?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

---

## 📋 Table of Contents

1. [Overview](#-1-overview)
2. [Alert Severity Levels](#-2-alert-severity-levels)
3. [Routing Rules](#-3-routing-rules)
4. [Alert Fatigue Prevention](#-4-alert-fatigue-prevention)
5. [On-Call Notification Channels](#-5-on-call-notification-channels)
6. [Alert Lifecycle](#-6-alert-lifecycle)

---

## 🎯 1. Overview

Alerts exist to notify the right person about the right problem at the right time. Poorly designed alerts erode trust, cause fatigue, and result in real incidents being missed. Every alert at {Company} must be actionable, routed correctly, and reviewed regularly.

> **Rule:** Every alert must have a linked runbook. Alerts without runbooks must not be promoted to production.

Cross-references: [SLO Framework](./10-slo-framework.md) for burn rate alerting, [Incident Severity](./13-incident-severity.md) for severity classification.

---

## 🚨 2. Alert Severity Levels

| Severity | Criteria | Response Time | Notification |
|:---------|:---------|:--------------|:-------------|
| **P1** | Customer-facing outage or data loss risk | Immediate (< 5 min) | Page on-call, auto-create incident |
| **P2** | Degraded experience or SLO burning fast | < 15 minutes | Page on-call |
| **P3** | Non-customer-facing issue needing attention | < 4 hours | Slack notification to team channel |
| **P4** | Informational or minor anomaly | Next business day | Slack notification, ticket created |

### Severity selection

| Question | If Yes | If No |
|:---------|:-------|:------|
| Customers currently impacted? | P1 or P2 | P3 or P4 |
| Data being lost or corrupted? | P1 | Check other criteria |
| Error budget burning > 6x? | P2 or higher | P3 or lower |
| Can wait until next business day? | P4 | P3 or higher |

---

## 🗺️ 3. Routing Rules

Alerts route based on service ownership in the [Service Catalog](../01-platform-standards/04-service-catalog.md).

| Route Type | When Used |
|:-----------|:----------|
| **Service owner** | Default for all service-specific alerts (PagerDuty escalation policy) |
| **Platform on-call** | Infrastructure alerts (Kubernetes, networking, CI/CD) |
| **Security on-call** | WAF triggers, credential leaks, CVE detections |
| **Database on-call** | Replication lag, connection pool exhaustion, storage |

### Escalation timing

| Step | Timing | Action |
|:-----|:-------|:-------|
| Primary on-call | T+0 | Phone call and push notification |
| Secondary on-call | T+5 min | Auto-escalate if primary does not ack |
| Engineering lead | T+15 min | Auto-escalate if no ack |
| VP Engineering | T+30 min | Auto-escalate for P1 only |

---

## 😴 4. Alert Fatigue Prevention

| Guardrail | Rule |
|:----------|:-----|
| **Actionability** | Every alert must require a human action. Remove info-only alerts from paging |
| **Deduplication** | Same root cause grouped into a single incident (PagerDuty grouping) |
| **Maintenance suppression** | Scheduled windows suppress expected alerts |
| **Flap detection** | Alerts firing/resolving > 3 times in 1 hour are auto-suppressed |
| **Noise budget** | Target < 2 non-actionable pages per shift |
| **Auto-resolve** | Alerts must auto-resolve when condition clears |

> **Rule:** If a shift generates more than 5 non-actionable pages, the team must conduct an alert quality review within 7 days.

---

## 📡 5. On-Call Notification Channels

| Channel | Used For |
|:--------|:---------|
| **Phone call** | P1 and P2 pages (PagerDuty voice) |
| **Push notification** | P1 and P2 parallel with call (PagerDuty mobile) |
| **SMS** | P1 fallback if phone and push fail |
| **Slack** | P3, P4, and coordination (`#alerts-{team-name}` via webhook) |
| **Email** | Daily digest and ticket notifications |

Alert messages must include: service name, severity, one-line summary, runbook link, and dashboard deep link. On-call engineers must have a verified phone number and PagerDuty mobile app installed.

---

## 🔄 6. Alert Lifecycle

| Phase | Activity | Owner |
|:------|:---------|:------|
| **Define** | Create alert rule with severity, routing, and runbook | Service team |
| **Review** | Peer review in pull request | Team lead or SRE |
| **Deploy** | Via Terraform or Prometheus rules-as-code | CI/CD pipeline |
| **Respond** | Acknowledge and follow runbook | On-call engineer |
| **Retro** | Evaluate quality in monthly review | Team |

### Monthly alert quality targets

| Metric | Target |
|:-------|:-------|
| Pages per shift | < 5 |
| Actionable page rate | > 80% |
| Median time to ack | < 5 min |
| Alerts without runbooks | 0 |
| Flapping alerts per week | 0 |

> **Rule:** Persistent noise (> 2 months below target) is escalated to the engineering lead.

---

<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
