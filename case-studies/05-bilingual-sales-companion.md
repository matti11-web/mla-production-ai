# Bilingual Sales Companion

A CRM and sales-operations companion for a bilingual construction sales team: leads, contacts, pipeline stages, quotes, AI classification, related-record discovery, signing flows, and post-signature automation.

**Status:** Production system for internal use. Public details are sanitized around architecture and product decisions; customer records, employee-specific performance details, schemas, and proprietary workflows are not included.

---

## Problem

The sales workflow had outgrown a simple lead table. The real operating questions were:

- Is this lead a duplicate?
- What product category and urgency does it belong to?
- Which past records are related?
- What stage is it in, and what should happen next?
- Is the quote signed, declined, expired, or waiting?
- Does the customer need Dutch or French communication?
- Which follow-up should happen after signature?

Generic CRMs can track some of that. The missing layer was construction-specific sales intelligence and bilingual workflow automation that matched how the team actually works.

---

## Constraints that shaped the design

1. **Bilingual data.** Lead text, customer communication, and internal labels mix Dutch and French.
2. **Legacy data quality.** Imported leads are incomplete, inconsistent, and sometimes duplicated.
3. **Small team, high workflow specificity.** Enterprise CRM customization would cost more than the problem warranted.
4. **AI must suggest, not silently mutate.** Classification and deduplication can assist agents; final decisions stay visible.
5. **Mobile-first field use.** Sales work happens in the showroom, on the road, and between customer conversations.
6. **Low operating cost.** AI features must be cheap enough to run continuously.

---

## Architecture

```text
          Lead creation
 manual / webhook / email
               |
               v
        Normalize input
               |
     +---------+----------+
     |                    |
     v                    v
 Embedding model       LLM classifier
 duplicate check      product, urgency,
 related leads        source suggestions
     |                    |
     +---------+----------+
               |
               v
            D1 CRM
 leads, contacts, quotes,
 activities, signing state
               |
               v
       React sales companion
 mobile CRM, pipeline,
 quote actions, follow-ups
```

The CRM is not just a database front-end. It is a workflow system: every stage transition, quote event, and signature state can trigger a next action.

---

## Key technical decisions

### 1. Vector-based deduplication for messy bilingual leads

Traditional duplicate checks work on exact email, phone, or fuzzy name matching. That misses variants across Dutch/French place names, company suffixes, abbreviations, and partial address data.

The design uses an embedded composite string: contact, company, address, project location, product type, notes, phone, and email. Similarity search then catches records that are semantically close even when fields differ.

### 2. Stored embeddings for read-time cost control

Related-lead lookup should not re-embed the lead every time a user opens the detail view. The embedding is generated when key fields change and stored. The related-record panel queries using that existing vector.

That turns a potentially expensive AI feature into a cheap retrieval feature.

### 3. LLM classification as suggestion metadata

The classifier proposes product category, urgency, source, and routing hints. It stores suggestions separately from the canonical CRM fields until a human accepts or overrides them.

This is slower than blind automation, but it produces a feedback loop and prevents hidden model errors from corrupting pipeline data.

### 4. Quote and signing state are workflow events

The CRM treats quote sending, viewing, signing, decline, expiry, and post-signature follow-up as state transitions. That allows reminders, cascades, and dashboards to be driven by events rather than manual memory.

### 5. NL/FR is a product requirement, not translation polish

Customer-facing surfaces and internal labels have to reflect the language of the customer and the team. Treating bilingual output as a late translation pass would break trust.

---

## Outcomes

| Metric | Value |
|---|---:|
| Historical leads considered in AI-intelligence design | ~8,500 |
| New lead volume modeled | ~50/day |
| Data language mix modeled | ~65% NL / ~30% FR / ~5% mixed or EN |
| Related-record pattern | vector search over stored lead embeddings |
| Classification model pattern | constrained JSON suggestions |
| Cost target for AI intelligence | low single-digit euros/month depending on provider |

**Real win:** the CRM stops being only a place where sales activity is recorded. It becomes a system that notices context the salesperson would otherwise have to remember.

---

## Lessons

### AI belongs where the data is messy

Duplicate detection and lead classification are good AI surfaces because the input is ambiguous: language variants, incomplete notes, typos, and mixed source channels.

Stage transitions, contract state, and post-signature cascades are not good AI surfaces. Those should be explicit rules.

### Suggestions need an acceptance model

If an AI classifier writes directly into canonical fields, it creates invisible data debt. Storing suggestion fields separately makes the system auditable and gives a path to measure accept/reject rates.

### Small-team CRMs should be workflow-specific

For a 10-person team, the advantage is not more generic CRM features. The advantage is encoding the exact few workflows that matter: lead intake, quote flow, signature state, follow-up, and bilingual customer handling.

### Legacy imports are not clean training data

Historical CRM data is useful, but not automatically trustworthy. Missing sources, inconsistent notes, and old statuses can reduce embedding and classification quality. The system has to tolerate lower confidence on older records.

---

## What's NOT in this case study

- Customer records, examples, or screenshots
- Salesperson-level performance data
- Internal pipeline-stage names where they reveal proprietary process
- Prompt examples containing company-specific routing rules
- Source code or database schema

The shareable pattern is the product architecture: vector relatedness, constrained classification, human acceptance, explicit sales workflow state, and bilingual UX as a first-order requirement.

---

**Built:** 2026-02 onward, with AI intelligence design wave in late March 2026.
