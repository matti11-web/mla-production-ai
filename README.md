# Matthias Labeeuw — Production AI

Production AI projects, architecture patterns, and lessons learned from building a 14-module internal business platform and the AI working environment around it — solo, since February 2026.

I'm Matthias Labeeuw — **a business strategist and coach, not a developer.** For 15 years I built companies, coached teams, and drove commercial strategy. Then AI made software buildable by people who deeply understand business — and I had exactly the missing half. This repo is the proof: a 14-module internal platform, autonomous build pipelines, and an AI working environment that I re-test when the models change, built solo with AI — documented as case studies, architecture patterns, and verifiable numbers.

→ **[How I work](how-i-work.md)** — the business-strategy × AI cocktail, and why the bottleneck was never the code.

No source code from client work is published here. Each case study describes the architecture, technical decisions, outcomes, and lessons in a sanitized form. The goal is to show *how I think* and *what works*, not to leak proprietary implementations.

---

## Featured case studies

| # | Project | What it does | Stack |
|---|---|---|---|
| [01](case-studies/01-semantic-memory-recall.md) | **Semantic Memory (`/recall`)** | Cohere-embedded, RAG-style search over ~30,000 distinct personal-knowledge chunks. Replaces grep for fuzzy concept lookup across rules, skills, and project notes. Auto-reindexes on session close. | Python · Cohere `embed-v4.0` + `rerank-v3.5` · numpy |
| [02](case-studies/02-autonomous-multi-pr-pipelines.md) | **Autonomous Multi-PR Pipelines** | Roadmap-to-PR orchestration pattern: isolated feature branches, local checks, reviewer-agent gates, and a merge gate that moved in November 2026: a rule-bound loop merges the low-risk pull requests, and I keep the risky ones. | PowerShell · Claude Code · gh CLI · reviewer agents |
| [03](case-studies/03-customer-intelligence-layer.md) | **Customer Intelligence Layer** | PDF extraction pipeline that turns operational documents into comparable, queryable workflow data. Content-stream parser, no OCR. | Cloudflare Workers · D1 · TypeScript · custom PDF parser |
| [04](case-studies/04-construction-erp-platform.md) | **Construction-Industry ERP Platform** | 14-module internal platform (core wave built in ~3 months): CRM, document signing, knowledge vault, customer intelligence, calculators. | Cloudflare Workers · D1 · Hono · React · Claude |
| [05](case-studies/05-bilingual-sales-companion.md) | **Bilingual Sales Companion** | CRM with AI-classified leads, NL/FR workflows, vector related-record discovery, pipeline automation, and post-signature flows. | React · Zustand · React Query · D1 · Anthropic API |
| [06](case-studies/06-hash-chain-document-signing.md) | **Hash-Chain Document Signing** | E-signing flow with signed-PDF generation, completion certificates, hash-chain audit events, and signed-artifact preference. | Workers · D1 · R2 · Web Crypto · Resend |
| [07](case-studies/07-english-media-brand-solo.md) | **English-Language Media Brand, Solo** | Full media-brand build for a sim-racing team: identity system, Astro website, shorts-first content pipeline, YouTube-API analytics loop. Capability case, not a growth story. | Astro · Cloudflare · Python · YouTube APIs |
| [08](case-studies/08-compound-systems.md) | **Compound Systems** | What happens once systems built since February 2026 share data layers: one email triggers coordinated updates across document management, knowledge vault, public website, and document generation. | Workers · D1 · Vectorize · MCP · Claude Code |
| [09](case-studies/09-legacy-erp-unlock.md) | **Building on a Vendor ERP** | Read-only gateway + two-speed mirror (nightly full pass, 15-minute daytime pass) over a vendor ERP (213 tables, ~4.3M rows) — no writes, no migration project. Plus two bugs that produced confident wrong answers. | FastAPI gateway · Workers cron · D1 · Vectorize |
| [10](case-studies/10-model-choice-by-eval.md) | **How I Know the AI Answer Is Right** | A checking method for AI output that looks fine but is wrong: real messy inputs, the verbatim production prompt, a blind judge, real token costs, and a new test per task. Worked example: a cheap model silently failed 3 of 4 real documents, fixed for ~5 cents per run. | Node · Anthropic API · blind LLM jury |
| [11](case-studies/11-intake-first-document-architecture.md) | **Killing the Folder Tree** | Intake-first architecture for two document stores that were drifting apart: one controlled door, two stores, one shared topic code, everything lands as a draft. Includes the silent truncation bug the design review surfaced. | Workers · D1 · R2 · Vectorize · Claude |
| [12](case-studies/12-verifying-an-irreversible-cutover.md) | **Verifying an Irreversible Cutover** | A website that carries most of the won sales, ~1,000 pages, one DNS switch. Path, content and pixel checks ran before the switch, and found 110 missing buttons in the build that would ship, after the same checks ran green against staging. | Astro · headless CMS · Cloudflare · Python · Playwright |
| [13](case-studies/13-salon-off-saas.md) | **Taking a One-Person Salon Off a Paid SaaS** | Custom booking software for a one-person grooming salon, on real data four days after the first commit. The switch took seven more weeks, and the first leak came from my own links. | Workers · D1 · React · Astro · Playwright |
| [14](case-studies/14-shipped-is-not-used.md) | **From Shipped to Used: The Non-AI Part of Four Rollouts** | Four internal tools that people now use daily, and the non-AI work that got them there: the idea, the new workflow, an end date for the old way, and follow-up until users were convinced. | Workers · D1 · SQL · ffprobe |
| [15](case-studies/15-assistant-with-hard-limits.md) | **An AI Assistant With Its Limits in Code, Not in the Prompt** | A WhatsApp assistant that replies on its own in a few group chats. The model only proposes. A double live gate, regex hard-stops, allowlisted chats and read-only sources decide what goes out. | Python · SQLite · Claude · WhatsApp bridge |
| [16](case-studies/16-minutes-without-uploading-audio.md) | **Meeting Minutes Without Uploading the Audio** | Whisper on a local GPU transcribes the recording and only the text goes to the cloud for the summary: about 4 minutes for a 30-minute meeting, and the audio never leaves the building. | Python · faster-whisper · local GPU · Claude |
| [17](case-studies/17-adapting-to-a-new-model-generation.md) | **Adapting the Setup When the Model Changes** | 275 skills and 27 always-on rules, re-tested against a newer model generation in 60 blind runs. Full and stripped context scored the same, and the only wrong price answers came from a stale document. Both skill packs retired, 21 skills left. | Claude Code · blind LLM judge · Python |

---

## Architecture patterns

| Pattern | Why it works |
|---|---|
| Cloudflare Workers + D1 + Hono | Edge-first, EU data-location options (D1 location hints), fast cold starts, zero servers to manage. Tradeoff: 128 MB memory cap forces stream-based file handling for anything >80 MB. |
| Lean always-on context, references on demand | Only the rules that apply to every task load in every session. Domain knowledge loads when a task calls it, and it points to the code as the source of truth. Re-tested at each model generation, see [case 17](case-studies/17-adapting-to-a-new-model-generation.md). |
| Multi-agent orchestration via Task tool | Model and reasoning effort are chosen per task type by measured evals, not by habit; currently the most capable model runs subagent work at low reasoning effort. Single agent always loses to specialized agents on long-context refactors. |
| [Cold-read discipline](patterns/cold-read-discipline.md) | LLM output is fluent, not accurate — hallucinations arrive as plausible, specific, confident detail. A separate adversarial verification pass before action is non-negotiable. Full write-up. |
| [Context-budget engineering](patterns/context-budget-engineering.md) | Always-on context (skill descriptions, rules, memory index) is a measurable tax on every session. Measured ~101K tokens, engineered down to ~71K, then to ~51K — without losing routing accuracy. Full write-up. |

---

## By the numbers

All figures measured 26 November 2026 against my own repos and GitHub account (first commit in this body of work: 1 March 2026 — 270 days).

| Metric | Value | Change since 11 November |
|---|---|---|
| Merged pull requests | 3,459 (of 3,532 opened — 98%) | +420 |
| Commits I authored, on default branches | 7,141 across 52 repositories | +401 commits, +1 repository |
| Production D1 database migrations (forward-only) | 514 | +33 |
| Reusable AI skills that still load | 25 | +2. Down from 275 before both skill packs were retired on 24 October, see case 17 |
| Production checks under 24/7 synthetic monitoring | 12 | counted this time, in the target list of the monitor. The 8 published until now was carried forward from 25 August |
| Semantic memory corpus | 29,935 chunks across 3,393 files | −95,724 chunks. Correction: about three quarters of the 11 November count were copies, see below |

How each is counted: merged PRs from the GitHub Search API across my account; commits from the GitHub GraphQL API, default branch only, filtered to commits authored by my account; migrations as the forward-only `.sql` files in a `migrations/` folder on each repository's default branch, excluding `.down.sql` rollbacks; skills as the skill folders that still load; monitoring as the entries in the target list of the monitor; the memory corpus counted directly in the live embedding index (last written 26 November).

**Correction to the memory corpus.** On 11 November this page said 125,659 chunks. That count was right, and it was misleading. The index held the same documents several times: in release folders, in build outputs and in parallel git checkouts. The same day I removed the copies. The index went from 122,880 chunks to 29,841, and no distinct text left it. So the real size was never above about 30,000 distinct chunks. The 106,795 of 15 October was counted the same way and holds copies too. I did not go back to measure how many.

Monitoring: the 12 checks run on 11 addresses, because two checks read the same health response.

As a check, the same queries limited to 11 November give 3,056 merged PRs, 6,716 commits in 52 repositories and 482 migrations. The PR figure is 0.6% above the figure published that day, because that count ran in the morning. The migration figure is 1 above the published 481. The commit figure is 24 below the published 6,740, with one repository more. I could not find the cause of that gap. Against the check, the growth is +425 commits and no new repository.

Commits: most repositories sit in GitHub organisations that also hold code that colleagues and contractors write. The count asks GitHub directly and keeps only my own commits — 7,141 of the 10,196 on those default branches. Migrations: every file is counted from scratch. Two repositories with migrations are left out because someone else wrote that code.

Honest framing: commits are AI-co-authored (Claude/Codex) with a single human operator — that *is* the claim, not a caveat. PRs pass CI gates (typecheck, lint, tests, security scans) plus adversarial AI review; there is no second human reviewer. Since 21 November a rule-bound loop merges part of them without me: 26 so far, see [case 02](case-studies/02-autonomous-multi-pr-pipelines.md). And 3,459 PRs ≠ 3,459 features — the number demonstrates cadence and process discipline; the case studies demonstrate substance.

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
- [LinkedIn](https://www.linkedin.com/in/matthias-labeeuw-6576694b) · My Claude Code skills and rules stay private (they encode client-specific domain knowledge). The reusable patterns are documented in the case studies here.
