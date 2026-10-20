# Building on a Vendor ERP (Read-Only)

The operational truth of the business — 213 tables, ~4.3 million rows — lived in a vendor ERP built around a Windows desktop application. No public API, no export pipeline, and no reason to make replacing it the first step. This is how that data became queryable by modern applications without touching the system of record.

**Status:** Read-only gateway and mirror in production, 2026. Semantic search over the article catalog live in two internal frontends. Updated 20 October 2026: the mirror now runs at two speeds (see below).

---

## The situation most SMEs are actually in

The AI-transformation conversation usually assumes a modern data stack. The reality in mid-sized industrial companies is a vendor ERP that has run the business for years, holds every customer, job, invoice and article, and is known inside out by the people who use it daily. In many software-renewal stories, replacing it becomes step one, and nothing else starts until it is done. I treated it as a base to build on instead.

Constraints I had to accept as fixed:

1. **Not my system.** The ERP is vendor-maintained. Writing to it, or asking for schema changes, is a negotiation — not a task.
2. **Availability isn't guaranteed.** The access gateway runs on a machine that isn't always-on. Any consumer must tolerate the source being unreachable.
3. **Read-only, permanently.** The system of record stays the system of record. Nothing I build is allowed to become a second source of truth for operational data.

## Architecture: gateway → scheduled mirror → applications

```
   Vendor ERP (SQL Server, desktop app)
              │  read-only gateway (token auth, parameterized,
              │  identifier-validated, rate-limited)
              ▼
   Scheduled worker (nightly full snapshot + reconcile,
                     plus a 15-minute daytime pass on planning
                     and after-sales service data)
              │
              ▼
   Edge database (D1)  ──►  internal apps, reporting, margin analysis
              │
              └─────────►  embedding index ──► semantic catalog search
```

Three decisions carried the design:

**Mirror, don't proxy.** Applications never call the ERP live. A flaky source behind a tunnel would make every downstream feature as unreliable as the weakest link. A scheduled mirror converts an availability problem into a freshness problem — and freshness is negotiable, availability isn't.

**Full snapshot + delete-reconcile as the backbone, incremental only on top.** The obvious design is incremental sync on a change-timestamp column. On its own it doesn't survive contact with the data: the change column is NULL on roughly two-thirds of rows in the highest-volume tables, and the ERP hard-deletes rows without any tombstone. Incremental sync would silently miss both. So the nightly run stages the primary keys it saw, then deletes mirror rows that no longer exist upstream — gated so a partial run can never trigger a mass delete. Slower, and correct regardless of upstream metadata quality. Since late September it also writes only the rows that actually changed, which cut database writes sharply.

Nightly turned out to be too slow for planning, so a second, faster pass was added: every 15 minutes on working days, it fetches only the changes for the planning data that teams need the same day. On 20 October 2026 the after-sales service tables joined that pass: the service team had been working on data from the day before. Before adding them, we checked live that both tables stamp their change column on every write, a status change included. The two passes never mix. The daytime pass can never delete anything, it stands down while the nightly pass owns the mirror, and the next nightly run corrects anything it missed.

**Embedding index on top, not inside.** The article catalog (~19K items) is embedded into a vector index that several frontends query. Semantic search — "wide sliding door with a thermal break" instead of an article code — became a shared capability rather than a per-app feature.

## Two bugs worth the write-up

### The pagination cap that silently ate 60% of a table

The API caps page size at 200 rows. Request 500, and it returns 200 — with no error and no indication that your request was reduced.

My scanning loop stepped the offset by the limit it *requested* rather than the number of rows it *received*. Result: it read rows 0–199, then jumped to offset 500, skipping 300 rows on every page. Roughly 60% of the table was never seen, and the scan reported success.

The damage wasn't a crash — it was a **confident wrong conclusion**. I concluded that a set of article codes didn't exist in the ERP. They did; my scan had stepped over them.

The fix is one line: advance the offset by `len(returned_items)`, never by the requested limit. The lesson is bigger than the fix — **a paginating API you haven't probed is an untrusted API.** Before scanning anything, request an oversized page and count what actually comes back. Silent truncation is common and is indistinguishable from "there's nothing there."

### The taxonomy column that meant something else

A handover document stated a rule for separating physical products from services: one type column, one value = product, everything else = service. Clean, plausible, and wrong.

Checked empirically against the full ~19K-row catalog, that column turned out to be an eight-value taxonomy that mixes products and services across its values — applying the rule mislabeled genuinely physical products as services. The actual product/service discriminator was a different column entirely, where a NULL (not a value) marks the non-stock items.

Generalisable lesson: **in a long-lived schema, column semantics are archaeology, not documentation.** Names drift, meanings get repurposed, and the handover doc records what someone believed in a meeting. Verify any classification rule against the full distribution before you build on it — it costs one query and prevents a category of wrong answer that looks completely reasonable in a report.

## Outcome

- Operational ERP data is queryable by modern applications, with no writes to the system of record and no migration project.
- Semantic search over the catalog, shared across frontends via one index.
- Margin and reporting analysis run against the mirror instead of against exports emailed around as spreadsheets.
- The ERP vendor's surface area stayed small: a read-only gateway and, later, one added filter parameter.

## Limits, stated plainly

- **Freshness differs per table, and none of it is real-time.** Planning and after-sales service data refresh every 15 minutes on working days. Everything else is nightly, which is correct for reporting and analysis and wrong for anything transactional. Older service lines from before the change column was filled still only update with the nightly run.
- **Availability of the source still gates the sync window.** Runs align to when the gateway host is actually up.
- **Access credentials remain broader than they should be** on the ERP side. A dedicated read-only database login is requested and still open — worth naming, because "the mirror is read-only" describes my layer, not the whole chain.

---

## What's NOT in this case study

- The ERP product, vendor, or hosting details
- Schema: table names, column names, keys
- Any customer, article, pricing or margin data
- The gateway's source code
