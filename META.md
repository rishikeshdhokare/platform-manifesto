# 📋 Repository Meta

![Status: Guidance](https://img.shields.io/badge/status-Guidance-orange?style=flat-square) ![Owner: Platform Engineering](https://img.shields.io/badge/owner-Platform_Engineering-purple?style=flat-square) ![Updated: 2026](https://img.shields.io/badge/updated-2026-green?style=flat-square)

This file explains how [Repository Standards](./01-platform-standards/03-repository-standards.md) apply to **this** repository.

---

## 1. 🎯 What this repository is

This repository is the **{Company} platform manifesto**: a **documentation repository**, not a deployable microservice. It holds Markdown, diagrams, and AI context files. Many requirements in [Repository Standards](./01-platform-standards/03-repository-standards.md) exist so runtime services are operable, discoverable, and consistent in production. Those requirements do not all apply here.

---

## 2. ✅ Standards and practices that apply

| Item | Notes |
|------|--------|
| **`README.md`** | Present. Describes the manifesto, navigation, and how to contribute. |
| **`AGENTS.md`** | Present. Rules for humans and AI agents editing this repository. |
| **Branch protection** | Same expectations as other org repositories (for example protected `main`, required checks where configured). |
| **Pull request reviews** | Changes land via PR with review, consistent with platform Git workflow. |
| **`.gitignore`** | Present. Keeps editor and OS noise out of version control. |

---

## 3. ❌ Service-level standards that do not apply

The following are **not** required for this documentation repository:

| Item | Why |
|------|-----|
| **`catalog-info.yaml`** | There is no Backstage **runtime service** or component to register. |
| **`Dockerfile`** | Nothing is containerized or deployed as a service from this repo. |
| **`Makefile`** | There is no compiled artifact or standard `make` build lifecycle. |
| **Runbooks** | No on-call operational surface; there are no production endpoints to run or restart. |
| **`docs/adr/`** | Product and service ADRs live with the systems they describe. Manifesto changes use normal PR rationale and Git history. |
| **Health endpoints** | Not applicable without a running service. |

---

## 4. 📝 Changelog

We do **not** maintain a root `CHANGELOG.md`. Every change is recorded in **Git history**; significant batches of work are summarized in **GitHub releases** when Platform Engineering cuts a release. For service repositories, [Repository Standards](./01-platform-standards/03-repository-standards.md) still requires `CHANGELOG.md`.

---

<div align="center">

🏠 [Back to root](./README.md)

</div>
