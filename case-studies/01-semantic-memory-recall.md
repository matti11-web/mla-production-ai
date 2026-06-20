# Semantic Memory for LLM Sessions (`/recall`)

A single-file semantic search index that lets me find what I wrote three months ago by *concept*, not keyword. Built because grep was missing things that mattered.

**Status:** In active daily use against my own ~7,300-chunk personal corpus (memory + rules + skills + project dev-docs; measured June 2026). Cohere `embed-v4.0` (1,536-dim) for embedding, optional `rerank-v3.5` for high-precision queries, numpy for brute-force cosine similarity.

---

## Problem

LLM sessions don't share state. Every new Claude Code session starts cold — no memory of what I decided last week, what bug I solved last month, what skills I already wrote that do exactly the thing I'm about to write again.

My workaround was a file-based memory system: `MEMORY.md` plus ~140 topic files (projects, feedback memories, references) totalling ~600 markdown chunks. Plus another ~140 skills, ~24 rules, ~430 project dev-docs (Mission Briefs, Roadmaps, Brainstorms).

**Grep solved 60% of the lookup problem.** For the other 40% I either remembered the wrong keyword, or the concept was phrased differently in past notes, or I didn't know what file to grep. I'd lose 10–20 minutes per session re-deriving things I'd already decided.

A semantic search index over the whole corpus would mean: query by concept, get top-5 matches, decide if they're relevant in under 30 seconds.

---

## Constraints that shaped the design

1. **Single-user, single-machine.** No need for a vector DB server. Postgres-pgvector, Pinecone, Weaviate — all overkill.
2. **Fast enough to invoke from any session.** Target <500 ms end-to-end for a query, including embedding the query text.
3. **Cheap to reindex.** I edit memory daily. A full reindex must cost cents, not dollars.
4. **No hosted state.** Must run from a script. No services to keep alive.
5. **Trial-tier API key.** Cohere trial caps at 100K tokens/min. Reindex of full corpus is ~1.3 M tokens. Must throttle gracefully.
6. **Diagnostic + observable.** I want to *see* the index: how chunks cluster, where duplicates are, what's stale.

---

## Architecture

```
                    ┌─────────────────────────────────────────┐
                    │ Corpus (~780 .md files, ~7.3 K chunks)  │
                    │ memory/ + rules/ + skills/ + dev-docs/  │
                    └────────────────┬────────────────────────┘
                                     │ reindex.py (--delta or full)
                                     │   ├─ Chunk per `##` section
                                     │   ├─ Prefix file-context + frontmatter
                                     │   ├─ Batch 96 chunks → Cohere embed-v4.0
                                     │   ├─ Sliding 60s rate-limit window
                                     │   └─ L2-normalize, write to disk
                                     ▼
                    ┌─────────────────────────────────────────┐
                    │ index.npz (27 MB) + chunks.jsonl (12 MB)│
                    │ 7,332 vectors × 1,536 dims, float32     │
                    └────────────────┬────────────────────────┘
                                     │ recall.py "<query>"
                                     │   ├─ Embed query (single Cohere call)
                                     │   ├─ Cosine vs all vectors (numpy)
                                     │   ├─ Optional --rerank cross-encoder
                                     │   └─ Return top-5 with previews
                                     ▼
                    ┌─────────────────────────────────────────┐
                    │ stdout (markdown) — top matches + scores │
                    └─────────────────────────────────────────┘
```

Sister scripts read the same index file:
- `visualize.py` — PCA scatter + cosine-pair histogram + cluster heatmap → PNG
- `find_duplicates.py` — flag near-duplicate chunks across files (merge candidates)
- `stale_check.py` — report files untouched >N days

All three are zero-setup beyond pointing at the index.

---

## Key technical decisions

### 1. Cosine over a vector DB

Brute-force cosine via numpy is ~7,300 vectors × 1,536 dims = ~11M multiply-accumulates per query. On a modern CPU: ~5 ms. Vector DBs are the right answer at 100K+ vectors; below that, a flat numpy array is faster, simpler, and has zero ops cost.

### 2. Chunk per `##` section, not per file or sliding window

A markdown `##` heading is a semantic boundary the author already chose. Sliding-window chunking ignores that. File-level chunking dilutes signal because a 2,000-word memory file mixes topics.

Fallback for sections >8K chars: split on `###`. Fallback for `###` >8K: sliding window 4K with 500-char overlap. <50-char chunks dropped as noise.

### 3. Prefix embed-text with file-context + frontmatter `description`

Each chunk is embedded as:

```
{file-stem} — {frontmatter.description}
## {section header}

{section body}
```

Without the prefix, Cohere often returns the right *section* but from the wrong *file* because two files have similar prose. Prefixing the file stem + description grounds the embedding to its source. Lifted top-1 accuracy noticeably on disambiguation queries ("which feedback note about X?").

### 4. Optional rerank, not always-on

Cohere `rerank-v3.5` is a cross-encoder that re-scores cosine top-(K×4) candidates against the query as a pair. Adds ~200 ms per query but dramatically improves precision on ambiguous queries — cosine alone puts marginal results in top-5 because it doesn't model query-document *joint* semantics.

Trade-off resolved by making it a flag: default cosine (snappy), `--rerank` for niche lookups. Output shows both scores so I can judge whether reranking changed the ordering.

### 5. Rate-limit + 429-retry built into the embedder

Cohere trial keys cap at 100K tokens/min. Naive batching trips that on every reindex. Implemented a sliding 60-second window that tracks tokens billed per batch and sleeps to stay under 90K/min, plus exponential 429-retry (15s → 30s → 60s → 120s). Full reindex now runs reliably without manual restarts.

### 6. ID-prefix per source to avoid stem collisions

Memory files use `path.stem` as chunk ID (`project_recall::00-architecture`). Skills, rules, and dev-docs would collide on shared stems (every skill's file is `SKILL.md`; multiple projects have `README.md`). Solved by namespacing per source: `rule-{stem}`, `skill-local-{folder}`, `skill-mat-{pack}-{folder}`, `dev-{project-slug}-{stem}`.

---

## Outcomes

| Metric | Value |
|---|---|
| Corpus indexed | 7,332 chunks across 781 files (measured June 2026) |
| Sources | personal memory (~670) · project rules (~145) · local skills (~55) · plugin-marketplace skills (~2,790) · project dev-docs (~3,670) |
| Index size on disk | 27 MB `index.npz` + 12 MB `chunks.jsonl` |
| Cost: single query | ~30 query tokens = $0.000004 |
| Cost: query with `--rerank` | + one Cohere rerank call ≈ $0.001 |
| Cost: full reindex | ~1.3 M tokens ≈ $0.16 |
| Cost: delta reindex (typical session) | ~50 K tokens ≈ $0.006 |
| Query latency, cosine-only | ~150–300 ms (mostly the Cohere embed RTT) |
| Query latency, with `--rerank` | ~350–500 ms |
| Lookups per week (informal estimate) | 15–25 |

**Real win:** I find things I'd otherwise miss. Concrete example — this case study exists because `/recall` *failed* to find a brainstorm document about an important upcoming negotiation. The doc was correctly stored per the file-routing rule (in `{Project}/dev/`) but the index only covered `memory/` + `rules/` + `skills/`. Discovering the blindspot, extending `reindex.py`, and re-running cost about 30 minutes and $0.16 — and now an entire class of high-signal documents (Mission Briefs, Roadmaps, Brainstorms) is searchable. Without `/recall`, I'd have continued mentally relying on grep and missing that doc forever.

---

## Lessons

### Trial-key rate limits force you to make the script robust

A production API key with no rate limit would have hidden a class of bugs. The trial cap forced me to implement proper sliding-window throttling, exponential 429-retry, and progress logging. The result is a script that survives Cohere outages, hangs, and 429-storms without manual intervention. Constraints make code better.

### Index freshness is a real problem

The index does not refresh itself, so freshness has to be engineered — and the honest history is that I shipped this with a manual gap and only later closed it. Two layers handle it: (1) a `--delta` flag that only re-embeds files with `mtime > index.npz mtime` (reuses cached embeddings for unchanged files — ~99% of the corpus on a typical edit), and (2) **a session-end hook that runs the delta automatically** after any session that touched the corpus, with a weekly cron as a backstop. Originally layer (2) was just a verbal habit of running `reindex.py --delta` by hand — that was the weak link, and it's now removed.

**Resolved (2026-06-19):** reindex is triggered on session-close via a Stop hook (plus the weekly cron safety-net), not left to manual habit. The API-cost-vs-freshness trade-off lands fine because `--delta` makes a typical refresh ~$0.006. *(This case study originally listed this as an open question with a manual workflow; updating it to match the running system — the changelog said "manual", the build now auto-updates.)*

### Rerank score ≠ cosine score

A chunk can have cosine 0.40 (mediocre) but rerank 0.65 (strong match). Reverse also happens. Reporting both scores in the output lets me see when reranking flips a result up — and trust the rerank score more for ambiguous queries.

### What got missed without the fix

This isn't an abstract lesson — a 19-day-old document that was load-bearing for a real upcoming decision was invisible to the system. The file-routing rule was correct (dev docs in `{Project}/dev/`); the index scope was incomplete. The fix took 30 minutes once spotted, but I'd been operating without it for weeks. The takeaway: an index is only as useful as its scope is correct, and the scope decays as the corpus shape changes.

---

## What's NOT in this case study

- The actual content being searched (personal memory, project notes, brainstorms)
- Specific search results or queries that touched client data
- The full source of `recall.py` and `reindex.py` (the architecture and decisions above are the shareable part; the implementation specifics include paths, exclusions, and tuning for my corpus)

If the design is interesting and you'd want a similar system for your own corpus, the patterns above translate directly — Cohere or OpenAI embeddings, a flat numpy index, chunk per heading, sliding-window throttling. The implementation is the easy part once the design is clear.

---

**Built:** 2026-05-24, extended 2026-05-25 (rules+skills indexing, visualization, duplicate-detection, stale-check, reranker), extended 2026-05-27 (project dev-docs).
