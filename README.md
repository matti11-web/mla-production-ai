# Matthias Labeeuw — Production AI

Production AI projects, architecture patterns, and lessons learned from building ~80 applied-AI tools in 2025–2026.

I'm Matthias Labeeuw — **a business strategist and coach, not a developer.** For 15 years I built companies, coached teams, and drove commercial strategy. Then AI made software buildable by people who deeply understand business — and I had exactly the missing half. This repo is the proof: ~80 production tools built solo with AI, documented as case studies, architecture, and lessons.

→ **[How I work](how-i-work.md)** — the business-strategy × AI cocktail, and why the bottleneck was never the code.

No source code from client work is published here. Each case study describes the architecture, technical decisions, outcomes, and lessons in a sanitized form. The goal is to show *how I think* and *what works*, not to leak proprietary implementations.

---

## Featured case studies

| # | Project | What it does | Stack |
|---|---|---|---|
| [01](case-studies/01-semantic-memory-recall.md) | **Semantic Memory (`/recall`)** | Cohere-embedded, RAG-style search over ~6K personal-memory chunks. Replaces grep for fuzzy concept lookup across rules, skills, and project notes. | Python · Cohere `embed-v4.0` + `rerank-v3.5` · numpy |
| 02 | *Autonomous Multi-PR Pipelines* | PowerShell wrapper + Claude Code spawning that ships 5–10 PRs overnight against a roadmap, with self-review gates and reviewer-agents. 6× validated. | PowerShell · Claude Code · gh CLI · code-reviewer agent |
| 03 | *Customer Intelligence Layer* | PDF extraction pipeline that turns unstructured supplier/customer documents into queryable D1 rows. Content-stream regex parser, no OCR. | Cloudflare Workers · D1 · TypeScript · custom PDF parser |
| 04 | *Construction-Industry ERP Platform* | 14-module internal platform built in 3 months: CRM, document signing, knowledge vault, customer intelligence, calculators. | Cloudflare Workers · D1 · Hono · React · Claude |
| 05 | *Bilingual Sales Companion* | CRM with AI-classified leads, multilingual offers (NL/FR), pipeline automation, post-signature workflows. | React · Zustand · React Query · D1 · Anthropic API |
| 06 | *Hash-Chain Document Management* | E-signing flow with X.509 cert validation, hash-chain audit log, sync with existing cloud-based vendor DMS, and per-document RBAC. | Workers · D1 · Web Crypto · Resend |

(Cases 02–06 written soon. Pinned project for now: case 01.)

---

## Architecture patterns

| Pattern | Why it works |
|---|---|
| Cloudflare Workers + D1 + Hono | Edge-first, EU residency by default, sub-50ms cold start, zero servers to manage. Tradeoff: 128 MB memory cap forces stream-based file handling for anything >80 MB. |
| Claude Code skills as plugin marketplaces | Skills compose like Unix tools. Domain knowledge (e.g. construction pricing) lives next to engineering skills (e.g. Hono routing) without conflict. |
| Multi-agent orchestration via Task tool | Use Opus for architecture + synthesis, Sonnet for parallel execution. Single agent always loses to specialized agents on long-context refactors. |
| Cold-read discipline | LLM-generated brainstorms contain hallucinations indistinguishable from facts. Adversarial cold-read before action is non-negotiable. |

(Full pattern write-ups under `patterns/` — coming.)

---

## Lessons

| | |
|---|---|
| [Valuing internal software — a three-lens methodology](lessons/valuing-internal-software.md) | How to put a defensible number on internal software you've built. Rebuild-cost / license-stack-avoidance / strategic-SaaS lenses with international wage benchmarks (US / W-EU / S-EU / E-EU / SEA). Grounded in a real 136K-LOC platform. |

---

## What I'm not publishing here

- Client source code, schemas, customer data, pricing tables
- Anything covered by an employer/client IP arrangement
- Internal business decisions, vendor negotiations, or financials

If a case study is interesting and you want more detail than is shareable here, the right path is a conversation, not more public detail.

---

## About me

- 35, Belgian. Native Dutch, professional French and English.
- 15 years across entrepreneurship (founded multiple SMEs), business-strategy consulting, sales leadership, and coaching (sales teams, real-estate agents, founders).
- Today: Commercial Director at Ostyn NV (aluminium, timber, container construction) — and turning business strategy into working software with AI.
- The fusion: deep commercial judgment × coaching/requirement-extraction × AI-and-software fluency. See [How I work](how-i-work.md).
- [LinkedIn](https://www.linkedin.com/in/matthiaslabeeuw) · [matthias-skills marketplace](https://github.com/matti11-web/matthias-skills) (140+ Claude Code skills across 13 domain packs)
