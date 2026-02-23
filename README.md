# 🚀 OrderFlow — Architect's Coding Comeback

> **Owner**: scanpratik (`scan.pratik@gmail.com`) · GitHub: [`scanpratik`](https://github.com/scanpratik)  
> **Methodology**: SDD (Spec-Driven Development) — spec first, then AI codes, then you review  
> **AI Pair**: Antigravity (VS Code extension)  
> **Started**: Feb 22, 2026 · **Cadence**: ~6 hrs/week

---

## ⚡ How to Resume a Session (Read This Every Time)

Open VS Code, then tell Antigravity one of the following:

```
Option A (fastest):   @[conversation:"Architect's Coding Comeback"]
Option B (fallback):  "retrieve my context"
Option C (simplest):  "read the README and let's continue"
```

> Antigravity will read this file and know exactly where we left off.  
> End every session with: **"update my progress tracker"** so this file stays current.

---

## 📍 Current Status — Feb 23, 2026

| Phase | Status | What's Happening |
|-------|--------|-----------------|
| **Phase 1 — Foundation** | 🟡 Week 1 Done → **Week 2-4 Starting** | Spec written ✅ → About to scaffold code |
| **Phase 2 — OrderFlow** | ⬜ Not started | Microservices + EDA |
| **Phase 3 — AI + OSS** | ⬜ Not started | Bedrock + Open Source PRs |

### ✅ Immediately Up Next
**Scaffold `order-service`** from the approved spec → say:
```
scaffold the order service from the spec
```

---

## 🧠 Who You Are & Why This Project Exists

- **Senior architect**, 15–16 years experience, Java/Spring Boot + AWS
- Returning to **hands-on coding** after a 3–4 year break
- Goals: sharpen coding skills · contribute to open source · land hands-on coding roles
- Building **OrderFlow** as a portfolio project to demonstrate modern skills

---

## 🏗️ OrderFlow — What We're Building

A **multi-service event-driven e-commerce system** on AWS, built with SDD methodology.

```
Customer → API Gateway → Lambda (order-service)
                              ↓
                        RDS Aurora (Postgres)
                              ↓
                        EventBridge → SQS → inventory-service
                                          → notification-service
                              ↓
                        Step Functions (fulfillment workflow)
                              ↓ (Phase 3)
                        AWS Bedrock (AI recommendations)
```

---

## 🔧 Locked-In Tech Stack

| Layer | Choice | Notes |
|-------|--------|-------|
| Language | **Java 21** | Virtual threads enabled |
| Framework | **Spring Boot 3.4.x** + Spring Cloud Function | Lambda-compatible |
| Build | **Gradle** (Kotlin DSL) | |
| Database | **DynamoDB** (AWS managed, serverless) | No VPC needed |
| ORM | AWS DynamoDB Enhanced Client (SDK v2) | |
| Auth | Spring Security + JWT (stateless) | |
| Docs | SpringDoc OpenAPI (Swagger UI) | |
| Testing | JUnit 5 + Mockito + Testcontainers | |
| Cloud | **AWS Lambda** + API Gateway (HTTP API) | |
| IaC | **AWS CDK** (Java) | |
| Methodology | **SDD** | Spec → Review → AI codes → Test → Ship |

---

## ✅ Dev Environment — All Done

| Tool | Version | Status |
|------|---------|--------|
| Java | 21.0.10 (`JAVA_HOME` set) | ✅ |
| Gradle | 9.3.1 | ✅ |
| Docker Desktop | 29.2.1 | ✅ |
| AWS CLI | v2.33.27 | ✅ |
| Git | Configured (`scanpratik`) | ✅ |
| GitHub SSH | `ssh -T git@github.com` → authenticated | ✅ |
| VS Code | Java + Spring Boot + AWS Toolkit extensions | ✅ |

---

## 📁 This Repo — Files

| File | Purpose |
|------|---------|
| `README.md` | **← YOU ARE HERE** — session anchor, full context |
| [`sdd_spec_order_service.md`](./sdd_spec_order_service.md) | SDD Spec v2 (Lambda) — approved, ready to scaffold |
| [`progress.md`](./progress.md) | Session-by-session progress tracker |
| [`comeback_plan.md`](./comeback_plan.md) | Full 6-month roadmap |

---

## 🗂️ `~/working/` Layout

```
~/working/
├── orderflow-docs/        ← YOU ARE HERE — planning & specs
├── order-service/         ← Spring Boot Lambda app (Phase 1) — TO BE CREATED
├── inventory-service/     ← (Phase 2)
└── orderflow-cdk/         ← AWS CDK infra stack (Phase 1–2) — TO BE CREATED
```

---

## 📋 Phase 1 Checklist — Foundation (Weeks 1–4)

### Week 1 — Setup ✅ Complete
- [x] Dev environment (Java 21, Gradle, Docker, AWS CLI, Git)
- [x] VS Code extensions (Java, Spring Boot, AWS Toolkit)
- [x] GitHub SSH authenticated

### Week 2–4 — Spring Boot Reboot 🟡 In Progress
- [x] Write SDD Spec → [`sdd_spec_order_service.md`](./sdd_spec_order_service.md) ✅ Approved
- [ ] **Scaffold** `order-service` (Spring Boot 3.x + Spring Cloud Function + Gradle Kotlin DSL)
- [ ] Add JPA + PostgreSQL + validation + JWT security + exception handling
- [ ] Write tests (JUnit 5 / Mockito / Testcontainers)
- [ ] Deploy to AWS Lambda via CDK (`cdk deploy`)

---

## 📖 The SDD Loop (How Every Feature Gets Built)

```
1. SPEC    → Write spec with Antigravity (API, data model, business rules, acceptance criteria)
2. REVIEW  → You read and approve the spec (like a senior reviewing a design doc)
3. CODE    → Antigravity generates code from the spec (95%+ correct on first pass)
4. TEST    → Tests are derived directly from acceptance criteria in the spec
5. SHIP    → Deploy; the spec becomes living documentation
6. ITERATE → New features: update spec first, then code
```

> **Rule**: Never update code without updating the spec first. Spec drift = tech debt.

---

## 🚀 Quick Git Commit After Each Session

```bash
cd ~/working/orderflow-docs
git add -A
git commit -m "session: <date> — <what changed>"
git push
```

---

*Last updated: 2026-02-23 by Antigravity + scanpratik*
