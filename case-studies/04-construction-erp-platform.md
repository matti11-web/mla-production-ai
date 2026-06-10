# Construction-Industry ERP Platform

A Cloudflare-native internal platform for a European construction SME: CRM, document signing, knowledge vault, customer intelligence, calculators, task engine, and preparation workflows under one operating model.

**Status:** Built solo with AI assistance from late February 2026 onward. Public details are intentionally architectural: module breadth, stack choices, delivery constraints, and lessons. No proprietary schemas, customer data, commercial rules, or source code are published.

---

## Problem

The company had the classic SME software gap: several specialized tools, spreadsheets, file storage, legacy calculators, and human memory holding the process together.

Buying a single ERP was not realistic. Off-the-shelf tools could cover fragments, but not the construction-specific workflow:

- sales pipeline and quote status
- project dossier preparation
- document signing and auditability
- knowledge retrieval
- calculators and margin governance
- bilingual Dutch/French output
- bridges to legacy and vendor systems

The alternative was to build a platform layer that kept existing systems where they made sense and filled the operational gaps around them.

---

## Constraints that shaped the design

1. **Solo builder.** The architecture had to be maintainable by one person, not a department.
2. **Small internal audience.** Designed for a 10-40 person organization; team adoption is ongoing. Enterprise architecture would have been ceremony.
3. **Cloudflare-first.** Workers, D1, R2, Queues, and service bindings fit the budget and operational profile.
4. **Existing systems stay.** The platform integrates with vendor DMS and legacy calculation workflows instead of pretending they can be replaced overnight.
5. **Bilingual by default.** Dutch-first, French-second, with English only where useful for developer/admin surfaces.
6. **No active external customer traffic assumption.** Internal tools are tested and shipped based on code and workflow readiness, not imaginary enterprise rollout rituals.

---

## Architecture

```text
                 Unified platform shell
                 auth, nav, app registry
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
       CRM          Pre-analysis        Knowledge/Vault
 leads, quotes      dossier checks      docs, search, AI Q&A
        |                 |                 |
        +-----------------+-----------------+
                          |
                          v
                 Platform API layer
          shared auth, admin, audit, tasks
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
       D1                R2             External systems
 relational data        PDFs/assets     vendor DMS, email,
                                         legacy config bridge
```

The platform is federated, not monolithic. Domain apps keep their own concerns, while shared concerns move into a platform layer: identity, admin, audit, knowledge, app access, and cross-app navigation.

---

## Key technical decisions

### 1. Cloudflare Workers + D1 over traditional servers

The platform did not need Kubernetes, managed Postgres, or a permanent server. The workload is bursty, internal, and EU-oriented. Workers + D1 gave fast deploys, low fixed cost, and enough relational structure for the domain.

The trade-off is real: D1 and Workers impose constraints around memory, SQL behavior, local tooling, and portability. Those constraints are acceptable for this scale.

### 2. Federated databases instead of one giant schema

CRM, pre-analysis, and platform services have different domain lifecycles. Forcing everything into a single central schema would make early development feel clean and later changes brittle.

The chosen pattern is shared identity and platform services, with per-app domain data where that app owns the workflow.

### 3. Integrate before replacing

The platform does not try to replace every existing system. It wraps, syncs, and augments:

- existing cloud-based vendor DMS for files
- C++ legacy 3D-configurator bridge for calculation context
- email services for notifications and signing flows
- static knowledge repositories for internal documentation

That choice made the platform useful sooner. Replacement can be a later decision, not a precondition.

### 4. Business rules live in workflows, not just forms

The hard part is rarely a CRUD screen. It is the rule: when should a dossier be blocked, when is a signed document valid, which mismatch needs motivation, which task should appear after a stage change, which language should the output use?

Those decisions are the product. The software exists to make them repeatable.

### 5. AI assists the platform, but does not own it

AI is used for classification, summarization, search, code generation, and requirement extraction. Deterministic business rules remain deterministic. That separation is why the system can be debugged.

---

## Outcomes

| Metric | Value |
|---|---:|
| Platform scope measured | Hub + CRM + Pre-analysis |
| LOC measured | ~136K TypeScript/TSX/SQL |
| Files measured | 824 |
| Functions measured | 1,626 |
| API routes measured | 222 |
| Services measured | 191 |
| UI components measured | 369 |
| DB migrations measured | ~348 |
| Build window | roughly 3 months for the core platform wave |

Module breadth includes CRM, document signing, DMS sync, knowledge vault, pre-analysis, calculators, customer intelligence, meetings, product sheets, training, feedback, planning, and director workflows.

---

## Lessons

### Internal software value is not measured only by seats

For an SME of this size, a SaaS-style per-seat comparison understates value. The platform replaces workflow gaps that SaaS tools do not cover at SME scale. The better valuation lenses are rebuild cost, avoided license stack, and strategic optionality.

### The hidden asset is domain compression

An outside agency could write code. It would not automatically know the sales process, document flow, exception cases, language requirements, margin doctrine, legacy quirks, and political constraints.

The speed came from compressing domain knowledge and implementation into one loop.

### Cloudflare is excellent until it is not

Workers, D1, and R2 are a strong fit for internal tools. But lock-in is real. A future migration off Cloudflare would be a rewrite project, not a config change. That is acceptable only because the current operational leverage is high.

### One-person platforms need ruthless scope discipline

Every extra abstraction becomes a future maintenance obligation. The architecture has to stay boring wherever possible: Hono routes, D1 migrations, React components, typed services, explicit tests around risky flows.

---

## What's NOT in this case study

- Client code, database schemas, or migrations
- Internal commercial rules, margin formulas, or commercial policy
- Internal screenshots with customer or employee data
- Vendor-specific implementation details
- Claims that the platform is productized SaaS

As-is, this is an internal platform, not a multi-tenant product. Productizing it would require a separate hardening track: tenancy, GDPR/DPO review, support tooling, contracts, onboarding, and security posture.

---

**Built:** 2026-02 to 2026-05 core wave; ongoing platform evolution after that.
