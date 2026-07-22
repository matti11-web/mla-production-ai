# Compound Systems: One Email, Five Systems

What building on shared data layers for ~5 months actually buys you. This is not a case study about one system — it's about what happens *between* the systems documented in cases 01–07 once they share infrastructure.

**Status:** Describes a working production flow, July 2026. Deliberately neutral in tone: the individual pieces are ordinary; the point is the composition.

---

## The trigger

A supplier email arrives containing updated maintenance and warranty documents for a product line. The instruction to the AI session is a single paragraph, roughly:

> "This email contains the maintenance and warranty documents. They need to go into the DMS. Their content needs to go into the knowledge vault. The website content about warranty and maintenance needs to be aligned with these documents. And the warranty terms need to carry into the new order-form version we're working on."

One email, one instruction — five systems affected.

## What executes

| Step | System | What happens |
|---|---|---|
| 1 | Mail (MCP) | The session reads the email thread and pulls the linked documents directly — no manual download/upload step. |
| 2 | Document management ([case 06](06-hash-chain-document-signing.md) infrastructure) | Documents are filed into the DMS with metadata and an auditable event trail. |
| 3 | Knowledge vault | Document content is chunked and embedded into a vector index — the same index that serves semantic search in two different internal frontends. Sales staff asking "what does the warranty cover for X?" now hit the current documents. |
| 4 | Public website | The site's warranty/maintenance content is diffed against the new documents and updated where it disagrees — the website is treated as a downstream artifact of the source documents, not a separately-maintained copy. |
| 5 | Order-form generation | The warranty terms flow into the document-generation rebuild in progress, so the next version of the customer-facing order form references the current terms. |
| — | Persistent memory ([case 01](01-semantic-memory-recall.md)) | The decision and its context are indexed, so a session three weeks later that touches warranty text finds this change instead of resurrecting the old wording. |

A human reviews the externally-visible steps (website content, customer-facing documents) before they ship. The filing, ingestion, and drafting run without intervention.

## Why this works — and why it took months before it could

None of the five steps is impressive alone. The composition works because of decisions made system-by-system over months:

1. **Shared data layer.** The platform modules ([case 04](04-construction-erp-platform.md)) run on the same edge database family (Cloudflare D1), so "put this here and reference it there" is a query, not an integration project.
2. **One vector index, multiple frontends.** Semantic search was built once and shared. Adding a document to the vault upgrades every app that queries the index, for free.
3. **A 15-year-old vendor ERP, mirrored read-only.** Operational data (articles, dossiers, invoices) from the legacy ERP is mirrored on a schedule into the same edge database. The AI session can join "what the supplier says" against "what we actually sell" without touching the ERP itself.
4. **Skills as routing.** Recurring operations (filing conventions, brand rules, pricing logic) are encoded as reusable skills, so the session doesn't re-derive process from scratch — it invokes it.
5. **Persistent memory.** Cross-session state ([case 01](01-semantic-memory-recall.md)) is what turns five one-off actions into a durable change the next session builds on.

The honest framing: each system was built to solve its own problem, with shared infrastructure chosen deliberately but without a grand integration blueprint. The compounding showed up as a property of those choices, roughly at the point where most new work started touching two or more existing systems.

## The general claim

Point solutions cap out at the value of the point. Systems that share a data layer, an embedding index, and a memory substrate start to *compound*: each new capability multiplies against the existing ones instead of adding to them.

For a small company this inverts the usual build-vs-buy logic. Five separate SaaS tools would each handle their step fine — and step 4→5 handoffs would be a human copying data between browser tabs forever. The integration layer is where the value lives, and it's exactly the part you can't buy.

## Limits, stated plainly

- **Single operator.** One person orchestrates this. The flow above is repeatable, but it is not (yet) a process others in the organisation trigger themselves.
- **Human gates are load-bearing.** Externally-visible output goes through review. Removing those gates would make the flow faster and worse.
- **Composition is fragile at the edges.** When one system changes its data shape, downstream steps degrade quietly. Monitoring (8 production services under synthetic checks) catches outages, not semantic drift — that's an open problem.

---

## What's NOT in this case study

- Company names, product lines, supplier identities, or the actual document contents
- Pricing data, warranty terms, or anything commercially sensitive
- Source code of the connecting glue (the composition pattern is the shareable part)
