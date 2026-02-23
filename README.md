# 📋 orderflow-docs

> Living documentation for the **OrderFlow** portfolio project and **Architect's Coding Comeback** journey.  
> Updated each session by [Antigravity](https://antigravity.dev) + scanpratik.

---

## 📁 Files

| File | Purpose |
|------|---------|
| [comeback_plan.md](./comeback_plan.md) | Full 6-month roadmap — phases, tools, SDD methodology, weekly budget |
| [progress.md](./progress.md) | Session-by-session progress tracker — what's done, what's next |
| [sdd_spec_order_service.md](./sdd_spec_order_service.md) | SDD Spec for `order-service` — API contract, data model, acceptance criteria |

---

## 🧠 How This Repo Works

This repo is the **single source of truth** for planning and progress.  
Every coding session follows this loop:

```
1. Open this repo + VS Code
2. Tell Antigravity what you want to build or review
3. Antigravity updates docs here + scaffolds code in sibling repos
4. You commit + push after each session
```

---

## 🗂️ ~/working Layout

```
~/working/
├── orderflow-docs/        ← YOU ARE HERE — planning & specs
├── order-service/         ← Spring Boot Lambda (Phase 1)
├── inventory-service/     ← (Phase 2)
└── orderflow-cdk/         ← AWS CDK infra (Phase 1–2)
```

---

## 🚀 Quick Commit After Each Session

```bash
cd ~/working/orderflow-docs
git add -A
git commit -m "session: <date> — <what changed>"
git push
```

---

> **Stack**: Java 21 · Spring Boot 3.4.x · Spring Cloud Function · AWS Lambda · EventBridge · SQS · SNS · Step Functions · AWS Bedrock · Gradle · AWS CDK  
> **Methodology**: SDD (Spec-Driven Development) — spec first, then code with AI
