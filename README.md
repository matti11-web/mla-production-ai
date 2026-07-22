# Matthias Labeeuw — Production AI

Production AI projects, architecture patterns, and lessons learned from building a 14-module internal business platform and a 275-skill AI toolkit — solo, since February 2026.

I'm Matthias Labeeuw — **a business strategist and coach, not a developer.** For 15 years I built companies, coached teams, and drove commercial strategy. Then AI made software buildable by people who deeply understand business — and I had exactly the missing half. This repo is the proof: a 14-module internal platform, autonomous build pipelines, and 275 reusable AI skills, built solo with AI — documented as case studies, architecture patterns, and verifiable numbers.

→ **[How I work](how-i-work.md)** — the business-strategy × AI cocktail, and why the bottleneck was never the code.

No source code from client work is published here. Each case study describes the architecture, technical decisions, outcomes, and lessons in a sanitized form. The goal is to show *how I think* and *what works*, not to leak proprietary implementations.

---

## Featured case studies

| # | Project | What it does | Stack |
|---|---|---|---|
| [01](case-studies/01-semantic-memory-recall.md) | **Semantic Memory (`/recall`)** | Cohere-embedded, RAG-style search over 16K+ personal-knowledge chunks. Replaces grep for fuzzy concept lookup across rules, skills, and project notes. Auto-reindexes on session close. | Python · Cohere `embed-v4.0` + `rerank-v3.5` · numpy |
| [02](case-studies/02-autonomous-multi-pr-pipelines.md) | **Autonomous Multi-PR Pipelines** | Roadmap-to-PR orchestration pattern: isolated feature branches, local checks, reviewer-agent gates, and human-owned merge decisions. | PowerShell · Claude Code · gh CLI · reviewer agents |
| [03](case-studies/03-customer-intelligence-layer.md) | **Customer Intelligence Layer** | PDF extraction pipeline that turns operational documents into comparable, queryable workflow data. Content-stream parser, no OCR. | Cloudflare Workers · D1 · TypeScript · custom PDF parser |
| [04](case-studies/04-construction-erp-platform.md) | **Construction-Industry ERP Platform** | 14-module internal platform (core wave built in ~3 months): CRM, document signing, knowledge vault, customer intelligence, calculators. | Cloudflare Workers · D1 · Hono · React · Claude |
| [05](case-studies/05-bilingual-sales-companion.md) | **Bilingual Sales Companion** | CRM with AI-classified leads, NL/FR workflows, vector related-record discovery, pipeline automation, and post-signature flows. | React · Zustand · React Query · D1 · Anthropic API |
| [06](case-studies/06-hash-chain-document-signing.md) | **Hash-Chain Document Signing** | E-signing flow with signed-PDF generation, completion certificates, hash-chain audit events, and signed-artifact preference. | Workers · D1 · R2 · Web Crypto · Resend |
| [07](case-studies/07-english-media-brand-solo.md) | **English-Language Media Brand, Solo** | Full media-brand build for a sim-racing team: identity system, Astro website, shorts-first content pipeline, YouTube-API analytics loop. Capability case, not a growth story. | Astro · Cloudflare · Python · YouTube APIs |
| [08](case-studies/08-compound-systems.md) | **Compound Systems** | What happens after ~5 months of building on shared data layers: one email triggers coordinated updates across document management, knowledge vault, public website, and document generation. | Workers · D1 · Vectorize · MCP · Claude Code |

---

## Architecture patterns

| Pattern | Why it works |
|---|---|
| Cloudflare Workers + D1 + Hono | Edge-first, EU data-location options (D1 location hints), sub-50ms cold start, zero servers to manage. Tradeoff: 128 MB memory cap forces stream-based file handling for anything >80 MB. |
| Claude Code skills as plugin marketplaces | Skills compose like Unix tools. Domain knowledge (e.g. construction pricing) lives next to engineering skills (e.g. Hono routing) without conflict. |
| Multi-agent orchestration via Task tool | Use Opus for architecture + synthesis, Sonnet for parallel execution. Single agent always loses to specialized agents on long-context refactors. |
| Cold-read discipline | LLM-generated brainstorms contain hallucinations indistinguishable from facts. Adversarial cold-read before action is non-negotiable. |
| [Context-budget engineering](patterns/context-budget-engineering.md) | Always-on context (skill descriptions, rules, memory index) is a measurable tax on every session. Measured ~101K tokens, engineered down to ~71K without losing routing accuracy. Full write-up. |

---

## By the numbers

All figures measured 23 July 2026 against my own repos and GitHub account (first commit in this body of work: 1 March 2026 — 144 days).

| Metric | Value |
|---|---|
| Merged pull requests | 1,325 (of 1,350 opened — 98%) |
| Commits | 4,400+ across 26 repositories |
| Production D1 database migrations (forward-only) | 240+ |
| Reusable AI skills authored/curated (deduplicated) | 275 |
| Production services under 24/7 synthetic monitoring | 8 |
| Semantic memory corpus | 16,865 chunks across 2,169 files |

Honest framing: commits are AI-co-authored (Claude/Codex) with a single human operator — that *is* the claim, not a caveat. PRs pass CI gates (typecheck, lint, tests, security scans) plus adversarial AI review; there is no second human reviewer. And 1,325 PRs ≠ 1,325 features — the number demonstrates cadence and process discipline; the case studies demonstrate substance.

---

## What I'm not publishing here

- Client source code, schemas, customer data, pricing tables
- Anything covered by an employer/client IP arrangement
- Internal business decisions, vendor negotiations, or financials

Some detail stays private by design.

---

## About me

- 35, Belgian. Native Dutch, professional French and English.
- 15 years across entrepreneurship (founded multiple SMEs), business-strategy consulting, sales leadership, and coaching (sales teams, real-estate agents, founders).
- Today: Commercial Director at a European construction SME — and turning business strategy into working software with AI.
- The fusion: deep commercial judgment × coaching/requirement-extraction × AI-and-software fluency. See [How I work](how-i-work.md).
- [LinkedIn](https://www.linkedin.com/in/matthias-labeeuw-6576694b) · Plus a 275-skill Claude Code toolkit organised in domain packs — kept private (it encodes client-specific domain knowledge); the reusable patterns are documented in the case studies here.
