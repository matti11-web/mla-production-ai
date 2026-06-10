# Customer Intelligence Layer

A document-intelligence layer that turns operational PDFs from a European construction SME into structured, queryable records. No OCR, no manual re-entry, and no generic "chat with your PDF" abstraction where deterministic parsing is the better tool.

**Status:** Production architecture proven across order-form parsing, comparison workflows, and CRM intelligence design. The public version is sanitized: vendor names, schemas, customer data, and proprietary document examples are intentionally omitted.

---

## Problem

The business process depended on PDFs that humans could read but software could not reason about:

- signed order forms
- calculation exports from a C++ legacy 3D-configurator bridge
- project-preparation documents
- supplier/customer attachments

The failure mode was not "we cannot find the PDF." The files existed in an existing cloud-based vendor DMS. The failure mode was subtler: the PDF said one thing, the calculation model might say another thing, and nobody compared them field-by-field before production work began.

A normal document search tool would not solve that. The system needed to extract domain fields, compare them, store results, and surface mismatches inside the workflow.

---

## Constraints that shaped the design

1. **PDFs are structured enough to parse.** OCR would add cost and errors. The source PDFs contain extractable text streams.
2. **Cloudflare Workers memory limit.** The parser has to survive a 128 MB runtime and avoid loading huge files wastefully.
3. **Exactness matters.** For order-vs-calculation comparison, a mismatch is either a real problem or a consciously accepted deviation. "Close enough" is not acceptable.
4. **Bilingual output.** The workflow and surfaced labels must work in Dutch and French.
5. **No schema leaks.** Public explanation can describe architecture and classes of fields, not internal field names or customer examples.
6. **Human override with accountability.** The system should block unmotivated deviations, not pretend every mismatch can be auto-resolved.

---

## Architecture

```text
       Existing cloud-based vendor DMS
                    |
                    v
        Download PDF / config export
                    |
          +---------+----------+
          |                    |
          v                    v
  PDF content-stream       Legacy config
  extraction parser        binary parser
          |                    |
          v                    v
  Normalized document     Normalized calc
  field model             field model
          |                    |
          +---------+----------+
                    |
                    v
          Comparison engine
          exact fields, warnings,
          blocking mismatches
                    |
                    v
        D1 persistence + workflow UI
```

The first principle was to separate extraction from judgment. The parser extracts what the document says. The comparison engine decides what matters. The UI forces the human to resolve deviations.

---

## Key technical decisions

### 1. Text-stream parsing over OCR

The PDFs looked unstructured to a human because the layout was document-like, but the actual PDF streams carried text in predictable regions. That made OCR the wrong default.

Parsing text streams is cheaper, faster, more deterministic, and easier to test. OCR remains a fallback category for scans, not the primary path.

### 2. Domain model before database model

The parser does not write directly into tables. It first produces a normalized domain object: parties, project metadata, dimensions, products, materials, totals, and warning flags.

That intermediate object is the contract between extraction, comparison, UI, and storage. It makes parser changes less dangerous because the rest of the system depends on a stable shape.

### 3. Exact comparison for contract-critical fields

For some categories, any difference is meaningful: dimensions, quantities, product choices, colors, and signed-document totals. The comparison layer treats those as exact-match fields.

When a mismatch is intentional, the user can mark it as conscious and explain why. Without that explanation, the workflow blocks the "ready" state.

### 4. "Not comparable" is a first-class result

Some fields cannot be reliably extracted from every document version. The system marks those as not comparable instead of inventing confidence. This matters because false precision is worse than an honest gap.

### 5. AI is used where language is the problem

AI is useful for classification, matching multilingual phrasing, and summarizing ambiguous free text. It is not useful for arithmetic, exact contract comparison, or deterministic field extraction when a parser can do the job.

That split keeps cost low and makes the system auditable.

---

## Outcomes

| Metric | Value |
|---|---:|
| Extracted order-form field classes | 50+ |
| Legacy config files analyzed during parser research | 1,500+ |
| Runtime target | Cloudflare Workers, 128 MB memory |
| Comparison style | exact match for critical fields |
| Human override | required motivation for deviations |
| Storage target | D1 rows plus JSON snapshots |

**Real win:** the tool changed a document from "attached evidence" into workflow data. That is the difference between a file archive and an operational system.

---

## Lessons

### Generic document AI would have been the expensive wrong solution

This did not need a chatbot over PDFs. It needed deterministic extraction for the parts that were structured and narrow AI use for the parts that were linguistic.

That is a recurring pattern in production AI systems: use AI to remove ambiguity, not to replace a reliable parser.

### The parser is only half the product

A parser that extracts fields into JSON is a demo. The product is the comparison, the UI, the readiness score, the blocked state, and the audit trail around conscious deviations.

The business value comes from closing the loop inside the workflow.

### "No tolerance" is a product decision, not a technical one

For contract-derived documents, approximate equality can hide the exact problem the tool is meant to catch. The decision to require exact matches on critical fields came from the business process, not from engineering preference.

---

## What's NOT in this case study

- Actual customer PDFs or screenshots
- Internal schemas, field names, vendor names, or document IDs
- Internal commercial rules
- Source code from the parser or comparison engine

The shareable pattern is the architecture: deterministic PDF extraction, normalized domain objects, exact comparison where the business requires it, and accountable human overrides.

---

**Built:** 2026-03 to 2026-05, extended from project-preparation workflows into broader customer intelligence patterns.
