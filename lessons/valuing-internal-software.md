# Valuing Internal Software — A Three-Lens Methodology

Most engineers shrug when a CFO or board asks "what is this internal platform worth?" Here's the three-lens approach I used to put a defensible number on a 136K-LOC platform I built for an industrial SME.

The number is rarely the point. The methodology is.

---

## Why this matters

When a compensation discussion happens, a board review lands, an M&A scenario opens, or a recruiter asks "what have you actually built?" — you should be able to back up "I built X" with "X has value Y, here's how I derived it."

If you don't put a defensible number on the asset you created, someone else will assign one — usually lower than you'd expect, and almost never adjusted for what was unbuyable.

---

## The hard data you need first

You can't value what you can't count. Before anything, get these from your codebase:

| Metric | How to get it |
|---|---|
| LOC by language | `cloc` or `wc -l` on `.ts/.tsx/.sql/.py/.go` (whatever your stack), excluding `node_modules/`, build artifacts, vendored deps |
| Files, functions, classes | Knowledge-graph tool (e.g. `understand-anything`) or per-language AST parsers |
| API routes / endpoints | Count handlers in your routing layer |
| DB migrations | Files in `migrations/` directories |
| UI components | `*.tsx` / `*.vue` / `*.svelte` count, excluding tests |
| Services / business-logic modules | Files in `services/` or equivalent layer |

Example from a real platform (Hub + CRM + companion module):

| Layer | LOC | Files | Functions | Routes | Services | UI | Migrations |
|---|---:|---:|---:|---:|---:|---:|---:|
| Total | **~136K** | **824** | **1,626** | **222** | **191** | **369** | **~348** |

That's your foundation. Everything below builds on it.

---

## Lens 1: Replacement Cost

What it would cost to rebuild from scratch with an external agency.

### Formula

```
LOC ÷ blended dev velocity = dev-days
dev-days × blended fully-loaded rate = rebuild cost
```

### Calibration

**Dev velocity (mid-senior, modern AI-augmented workflow):**
- 50–75 production-quality LOC/day blended (code + tests + SQL + integration debug + review)
- Faster pure-generation rates exist but don't survive review at this caliber

**Fully-loaded blended rate ranges (mid-senior full-stack, 2025–2026):**

| Region / tier | €/dev-day | Notes |
|---|---|---|
| US senior consultancy (Bay Area, NYC) | €1,300–€1,800 | Highest, often requires 6-month minimum engagement |
| Western EU agency (NL, DE, UK, Nordics) | €900–€1,300 | Strong mid-senior pool, modern stack fluency |
| Southern EU agency (ES, IT, PT) | €700–€1,000 | Solid quality, lower bench costs |
| Eastern EU / Balkan agency | €500–€800 | Strong technical, adjust velocity down 20–30% for coordination |
| India / SEA outsourcing | €350–€600 | Cheapest sticker, velocity penalty 30–50% for ramp + back-and-forth |

### Worked example

136K LOC ÷ 60 LOC/day = **2,270 dev-days**

| Region | Raw cost | After 30% adjustment for scaffolding/CRUD/duplication |
|---|---|---|
| US senior | €3.0M–€4.1M | **€2.1M–€2.9M** |
| Western EU | €2.0M–€3.0M | **€1.4M–€2.1M** |
| Southern EU | €1.6M–€2.3M | **€1.1M–€1.6M** |
| Eastern EU | €1.5M–€2.3M | **€1.1M–€1.6M** (with velocity penalty) |

### Caveats that lower the number

- 30–40% of any LOC count is scaffolding, configs, generated types, repeated CRUD. Strip that.
- Agencies fail on this scope ~40% of the time. The "cost to ship successfully" is closer to 1.5× the nominal estimate. So nominal €1.5M ≈ €2.0–€2.5M to actually deliver working software.
- Nobody actually rebuilds 136K LOC of working internal software. The "rebuild cost" is a paper number unless an acquirer demands escrow.

---

## Lens 2: License-Stack Avoidance

What you'd pay annually to buy off-the-shelf SaaS equivalents.

### Process

1. Map each module of your platform to its closest commercial SaaS competitor
2. Price for actual seat count at list price
3. Apply 3–5 year horizon with 5–7% annual increase + churn risk
4. Add one-off integrator costs for connecting tools that don't natively talk to each other

### Worked example (40-user organization)

| Module | SaaS equivalent | Annual at 40 seats |
|---|---|---:|
| CRM | HubSpot Sales Pro / Pipedrive Advanced | €18–€30K |
| DMS + sync with existing on-prem vendor DMS | M-Files / DocuWare + custom integrator | €15–€25K + €30K one-off |
| Knowledge base + AI Q&A | Notion AI + Glean Lite | €12–€20K |
| BI / executive dashboard | Power BI Premium + dev time | €8–€12K |
| Meeting recording + summary | Fathom Team / Read.ai | €6–€10K |
| eSignature | DocuSign Business Pro | €4–€8K |
| Pricing engine / quote calculators | PriceBeam / Pricefx SME tier | €15–€30K |
| Auth + RBAC + i18n | Auth0 B2B | €6–€10K |

**Total: €85K–€145K/yr → 5-yr horizon: €500K–€900K**

Plus one-off integrator cost to make them talk to each other: another €60–€120K.

### What this lens misses

Some modules have **no commercial substitute at SME scale**. Those count as "buy not available" — value them separately as moats (next section).

---

## Lens 3: Strategic / Productized SaaS

If spun out as vertical SaaS for the same industry, what could the asset be worth?

### Formula

```
TAM × penetration × ARPU × revenue multiple
```

### Worked example

Industry: industrial SMEs in 30–200 employee band, in Western EU only.

- TAM: ~300–500 prospects
- Realistic ARPU at SME tier: €25K–€45K/yr (vertical SaaS pricing benchmark)
- 5-year 10% penetration: 30–50 customers
- ARR at maturity: €800K–€2.2M
- Vertical-SaaS revenue multiples (low-TAM vertical): 4–8× ARR

**Standalone product valuation: €4M–€12M**

### Why this is the most flattering and most fragile lens

Conditional on **€300K–€500K productization spend**:
- Multi-tenancy hardening
- SOC 2 Type II (or sector-specific equivalent)
- GDPR DPO sign-off
- ToS, support tooling, billing infrastructure
- Sales motion + onboarding flows
- Customer support tooling

Never quote this number as **the** valuation. It's strategic upside conditional on a specific investment that may or may not happen.

---

## Reconciling the three

| Lens | Range (example platform) | Confidence |
|---|---|---|
| Rebuild cost (Western EU agency) | **€1.4M–€2.1M** | High |
| License-stack avoided (5-yr) | **€500K–€900K** | Medium-high |
| Productized SaaS (conditional) | €4M–€12M | Low |

**Defensible internal valuation = Rebuild Cost lens ± strategic moats with no buy alternative.**

For board / CFO / comp conversations, lead with rebuild cost. Mention the avoided-SaaS number as cross-check. Only bring up the productization lens if asked.

---

## What this number does NOT include

Always state these explicitly when presenting:

| Hidden cost | Magnitude |
|---|---|
| **Bus-factor concentration risk** — single-developer system. Replacing the IC = 3–6 months onboarding for a senior. | Discount asset value 25–35% |
| **Ongoing maintenance** — ~0.5 dev-day/month per 5K LOC industry standard | €60K–€120K/yr ongoing |
| **Cloud / vendor lock-in** — porting off your chosen cloud provider | €150K–€350K rewrite |
| **Productization gap** — no multi-tenancy, no compliance, no sales motion | €300K–€500K to close |

These are real costs, not sunk assets. State them or lose credibility.

---

## What I built that the buyer cannot acquire

These are where commercial software has nothing equivalent in the target industry. Each one is a moat — accidental, not designed.

Each was 2–12 weeks of work. An agency would have refused them or charged €40K–€100K each. Together they explain why the platform's defensible value sits at the top of the rebuild-cost range, not the middle.

1. **Sync with existing on-premise DMS software from sector vendor.** The vendor's API requires institutional knowledge to navigate (undocumented response envelopes, ID-namespace quirks, size-aware streaming for files >80MB). Multi-bug cascade fixed across 3 weeks. No SaaS bridges this.

2. **Bridge between modern web stack and a C++-coded legacy 3D-configurator.** Reverse-engineering effort on a 20-year-old proprietary tool. Nobody else can do it without your codebase analysis.

3. **Reconciliation: PDF order line items ↔ engineering bill-of-materials with image normalization.** Industry-specific. Generic SaaS handles drawings or BOMs but not the reconciliation between them.

4. **Domain-specific pricing logic codified across every calculator with leak detection.** Enterprise SaaS competitors (Vendavo, Pricefx) exist at €80K+/yr but nothing at SME scale. You codified the company's pricing doctrine in code — that doctrine itself is the moat.

5. **Customer Intelligence Layer — PDF extraction tuned to industry-specific document schemas.** Generic PDF extraction is widely available; schema-awareness for this industry's order forms / invoices / specs is yours.

6. **Genuine multilingual UX (NL/FR/EN, or equivalent for your market) on every screen and every PDF output.** Most international vertical SaaS sells English-primary with machine-translated other languages. Human-curated multilingual on every key is rare in the SME segment.

7. **Event-driven task auto-derivation from domain-state transitions.** Asana/Linear are generic and event-blind. Yours fires off domain-specific lifecycle events that only make sense for this industry.

---

## Where I might be too generous

Three honest caveats that should always travel with the number:

1. **Rebuild cost assumes successful delivery.** Agencies fail on this scope ~40% of the time. So the true cost to replace successfully is closer to the upper bound × 1.5.

2. **LOC includes redundancy.** Test files, duplicate model definitions, migration up/down pairs, generated types. Real "novel business logic + integration" is probably 50–60% of total LOC, not 100%.

3. **The productized-SaaS lens depends on a market that may not exist.** TAM for niche verticals is notoriously optimistic on paper and brutal in reality. The €4M–€12M number is only real with a buyer in hand.

---

## Why this matters for your career narrative

If you've built internal software at any scale, you should know its value before someone else assigns one.

When a compensation discussion happens, a board review lands, or a recruiter asks "what have you built?" — having a defensible methodology backed by hard numbers (LOC, modules, comparable SaaS prices) reframes the conversation from "do you deserve this?" to "here's the asset's value, here's my marginal cost vs. the alternative."

The number is rarely the point. The methodology is.

---

**Built:** 2026-05-28, co-authored methodology with Claude Opus 4.7. Grounded in real knowledge-graph metrics + filesystem audit of a platform I built for an industrial SME (sanitized — no client identifiers).
