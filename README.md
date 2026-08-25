# Matthias Labeeuw — Production AI

Production AI projects, architecture patterns, and lessons learned from building a 14-module internal business platform and a 275-skill AI toolkit — solo, since February 2026.

I'm Matthias Labeeuw — **a business strategist and coach, not a developer.** For 15 years I built companies, coached teams, and drove commercial strategy. Then AI made software buildable by people who deeply understand business — and I had exactly the missing half. This repo is the proof: a 14-module internal platform, autonomous build pipelines, and 275 reusable AI skills, built solo with AI — documented as case studies, architecture patterns, and verifiable numbers.

→ **[How I work](how-i-work.md)** — the business-strategy × AI cocktail, and why the bottleneck was never the code.

No source code from client work is published here. Each case study describes the architecture, technical decisions, outcomes, and lessons in a sanitized form. The goal is to show *how I think* and *what works*, not to leak proprietary implementations.

---

## Featured case studies

| # | Project | What it does | Stack |
|---|---|---|---|
| [01](case-studies/01-semantic-memory-recall.md) | **Semantic Memory (`/recall`)** | Cohere-embedded, RAG-style search over 19K+ personal-knowledge chunks. Replaces grep for fuzzy concept lookup across rules, skills, and project notes. Auto-reindexes on session close. | Python · Cohere `embed-v4.0` + `rerank-v3.5` · numpy |
| [02](case-studies/02-autonomous-multi-pr-pipelines.md) | **Autonomous Multi-PR Pipelines** | Roadmap-to-PR orchestration pattern: isolated feature branches, local checks, reviewer-agent gates, and human-owned merge decisions. | PowerShell · Claude Code · gh CLI · reviewer agents |
| [03](case-studies/03-customer-intelligence-layer.md) | **Customer Intelligence Layer** | PDF extraction pipeline that turns operational documents into comparable, queryable workflow data. Content-stream parser, no OCR. | Cloudflare Workers · D1 · TypeScript · custom PDF parser |
| [04](case-studies/04-construction-erp-platform.md) | **Construction-Industry ERP Platform** | 14-module internal platform (core wave built in ~3 months): CRM, document signing, knowledge vault, customer intelligence, calculators. | Cloudflare Workers · D1 · Hono · React · Claude |
| [05](case-studies/05-bilingual-sales-companion.md) | **Bilingual Sales Companion** | CRM with AI-classified leads, NL/FR workflows, vector related-record discovery, pipeline automation, and post-signature flows. | React · Zustand · React Query · D1 · Anthropic API |
| [06](case-studies/06-hash-chain-document-signing.md) | **Hash-Chain Document Signing** | E-signing flow with signed-PDF generation, completion certificates, hash-chain audit events, and signed-artifact preference. | Workers · D1 · R2 · Web Crypto · Resend |
| [07](case-studies/07-english-media-brand-solo.md) | **English-Language Media Brand, Solo** | Full media-brand build for a sim-racing team: identity system, Astro website, shorts-first content pipeline, YouTube-API analytics loop. Capability case, not a growth story. | Astro · Cloudflare · Python · YouTube APIs |
| [08](case-studies/08-compound-systems.md) | **Compound Systems** | What happens after ~5 months of building on shared data layers: one email triggers coordinated updates across document management, knowledge vault, public website, and document generation. | Workers · D1 · Vectorize · MCP · Claude Code |
| [09](case-studies/09-legacy-erp-unlock.md) | **Unlocking a 15-Year-Old ERP** | Read-only gateway + nightly mirror over a vendor ERP (213 tables, ~4.3M rows) — no writes, no migration project. Plus two bugs that produced confident wrong answers. | FastAPI gateway · Workers cron · D1 · Vectorize |
| [10](case-studies/10-model-choice-by-eval.md) | **Choosing a Model With an Eval** | Blind-jury evaluation on real, messy inputs: the incumbent cheap model was silently failing 3 of 4 production cases. Cost of the fix: ~5 cents per run. | Node · Anthropic API · blind LLM jury |
| [11](case-studies/11-intake-first-document-architecture.md) | **Killing the Folder Tree** | Intake-first architecture for two document stores that were drifting apart: one controlled door, two stores, one shared topic code, everything lands as a draft. Includes the silent truncation bug the design review surfaced. | Workers · D1 · R2 · Vectorize · Claude |
| [12](case-studies/12-verifying-an-irreversible-cutover.md) | **Verifying an Irreversible Cutover** | Proving a ~1,000-page site migration *before* the DNS switch, because there is no diff afterwards. Path, content and pixel parity gates — and the gate that ran green against the wrong artifact. | Astro · headless CMS · Cloudflare · Python · Playwright |

---

## Architecture patterns

| Pattern | Why it works |
|---|---|
| Cloudflare Workers + D1 + Hono | Edge-first, EU data-location options (D1 location hints), sub-50ms cold start, zero servers to manage. Tradeoff: 128 MB memory cap forces stream-based file handling for anything >80 MB. |
| Claude Code skills as plugin marketplaces | Skills compose like Unix tools. Domain knowledge (e.g. construction pricing) lives next to engineering skills (e.g. Hono routing) without conflict. |
| Multi-agent orchestration via Task tool | Use Opus for architecture + synthesis, Sonnet for parallel execution. Single agent always loses to specialized agents on long-context refactors. |
| [Cold-read discipline](patterns/cold-read-discipline.md) | LLM output is fluent, not accurate — hallucinations arrive as plausible, specific, confident detail. A separate adversarial verification pass before action is non-negotiable. Full write-up. |
| [Context-budget engineering](patterns/context-budget-engineering.md) | Always-on context (skill descriptions, rules, memory index) is a measurable tax on every session. Measured ~101K tokens, engineered down to ~71K, then to ~51K — without losing routing accuracy. Full write-up. |

---

## By the numbers

All figures measured 25 August 2026 against my own repos and GitHub account (first commit in this body of work: 1 March 2026 — 177 days).

| Metric | Value | Change since 19 August |
|---|---|---|
| Merged pull requests | 1,814 (of 1,847 opened — 98%) | +90 |
| Commits | 5,965 across 30 repositories | +303 |
| Production D1 database migrations (forward-only) | 305 | +14 |
| Reusable AI skills authored/curated (deduplicated) | 275 | unchanged |
| Production services under 24/7 synthetic monitoring | 8 | unchanged — re-counted this time, previously carried forward |
| Semantic memory corpus | 19,991 chunks across 2,489 files | +706 chunks |

How each is counted: merged PRs from the GitHub Search API across my account; commits via `git rev-list --count` per repository, own repos only, excluding vendored and reference checkouts; migrations by counting forward-only migration files in production services, excluding `.down.sql` rollbacks and worktree copies; skills as the deduplicated union of my two private skill marketplaces; the memory corpus as lines in the live embedding index.

Honest framing: commits are AI-co-authored (Claude/Codex) with a single human operator — that *is* the claim, not a caveat. PRs pass CI gates (typecheck, lint, tests, security scans) plus adversarial AI review; there is no second human reviewer. And 1,814 PRs ≠ 1,814 features — the number demonstrates cadence and process discipline; the case studies demonstrate substance.

---

## What I'm not publishing here

- Client source code, schemas, customer data, pricing tables
- Anything covered by an employer/client IP arrangement
- Internal business decisions, vendor negotiations, or financials

Some detail stays private by design.

---

## About me

- 36, Belgian. Native Dutch, professional French and English.
- 15 years across entrepreneurship (founded multiple SMEs), business-strategy consulting, sales leadership, and coaching (sales teams, real-estate agents, founders).
- Today: Commercial Director at a European construction SME — and turning business strategy into working software with AI.
- The fusion: deep commercial judgment × coaching/requirement-extraction × AI-and-software fluency. See [How I work](how-i-work.md).
- [LinkedIn](https://www.linkedin.com/in/matthias-labeeuw-6576694b) · Plus a 275-skill Claude Code toolkit organised in domain packs — kept private (it encodes client-specific domain knowledge); the reusable patterns are documented in the case studies here.
