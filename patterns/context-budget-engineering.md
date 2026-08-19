# Context-Budget Engineering

How I got the always-on context load of my AI working environment from ~101K tokens down to 50,942 — measured, not estimated — without losing skill-routing accuracy. This is the pattern write-up promised in the README.

**Status:** Applied to my own Claude Code workspace, May–August 2026. Numbers below are from a measurement probe against real sessions, not vibes. Current baseline: **50,942 startup tokens**, measured 9 August 2026.

---

## The problem nobody prices in

Every LLM session pays a context tax before any work happens. In a mature Claude Code setup, that tax has three main line items:

| Line item | What it is | My measured baseline (May 2026) |
|---|---|---:|
| Skill descriptions | Every enabled plugin skill injects its name + description into every session, so the model can route to it | ~72K tokens |
| Standing rules | Always-on instruction files (coding standards, domain rules, safety conventions) | ~22.5K tokens |
| Memory index | The always-loaded index of persistent memory | ~7K tokens |
| **Total** | | **~101K tokens** |

That's a large fraction of the usable context window spent before the first user message — on every session, whether the session needs trading rules or not. The insidious part: each individual addition was rational. A skill here, a rule there. Nobody ever *decided* to spend 101K tokens; it accreted.

## Principle: measure first, then treat context like a budget

The single highest-leverage move was a script that reproduces what actually gets injected and totals it per source. Two reasons:

1. **Intuition is wrong here.** I would have guessed rules were the biggest item. They were less than a third of skill descriptions.
2. **What actually arrives ≠ what you wrote.** Example: HTML comments in the workspace instruction file are stripped before injection — discovered by planting marker sentinels and checking which ones surfaced in context. If you optimize files without measuring the injected result, you optimize the wrong thing.

## The five levers

### 1. Profiles: enable skill packs per work mode

Skills are grouped into domain packs (dev, commercial, trading, media, …). A switch script rewrites the enabled-plugin list from a named profile and can restore the previous state. Working on code? The trading and media packs' descriptions have no business costing context.

Result: skill descriptions ~72K → ~41K in the default dev profile. This was the whole −30K, essentially.

### 2. Rules as loaders: compact always-on core + searchable full reference

Each standing rule was split in two:

- A **loader**: ~15 lines, always on — the non-negotiable "must keep" bullets plus a pointer to the full reference.
- A **full reference**: the complete rule with examples and edge cases, living outside the always-on path, retrieved on demand with a search helper (`rg` wrapper) when the session actually enters that domain.

The rule stays *enforceable* (the core constraints are always present); the detail costs nothing until needed. This is progressive disclosure applied to instructions instead of UI.

### 3. Hard cap on the memory index

The always-loaded memory index has a size limit above which it partially loads — silently degrading recall of your own memory. Treat that limit as a hard budget: the index holds one-line pointers only; content lives in topic files loaded on demand. Discipline that follows: periodically de-index items that no longer earn their line, and *graduate* stable feedback into rules.

One non-obvious accounting note: graduating a memory item into an always-on rule is roughly token-neutral — both load every session. The real savings come from de-indexing and from moving detail behind retrieval, not from shuffling content between always-on buckets.

### 4. Retrieval instead of residence

The strongest form of the pattern: knowledge that lives in a semantic index costs **zero** always-on tokens. My `/recall` system (case study 01) searches ~17K chunks of memory, rules, skills, and project docs by concept. Anything searchable-on-demand doesn't need to be resident. The always-on budget should hold only what must *never* be missed — everything else is a retrieval problem.

### 5. Move domain rules into the skills that own them

The rule-loader split above (lever 2) still leaves every loader resident. The next cut asked a harder question: does this domain rule need to be always-on *at all*?

Twenty-seven standing rules became seven. The seven that stayed are the ones that apply regardless of task — output conventions, verification discipline, environment facts, file routing, memory-write boundaries, model selection, context budget itself. The other twenty were domain rules (API patterns, design system, testing conventions, commercial pricing doctrine). Each was folded into the skill that owns that domain, so the rule now arrives with the skill instead of before it. The full reference text was not touched — nothing was lost, only relocated off the always-on path.

Measured effect: **59,964 → 50,942 startup tokens (−9,022, −15.0%)**.

### 6. Deleting in git is not deleting

The twenty domain loaders were removed from version control on 5 August. They kept loading until 9 August.

The commit deleted them from the index; it left them in the working tree as untracked files. The harness reads the filesystem, not git. So for four days the measurement said the cut had shipped and the runtime disagreed — a −15% saving that existed entirely on paper.

The fix is a one-line regression check that treats any untracked file in the rules directory as a defect:

```
git status --porcelain <rules-dir>    # any "??" line is a loader that is loading but should not be
```

Generalised: **verify the saving in the channel that actually consumes the file.** A tool that reasons about your intent is blind to a defect in the artifact.

### 7. A discoverability catalog for what you disabled

The objection to disabling skill packs: "if the description isn't in context, the model won't know the skill exists." Valid — and solvable. A generated catalog (indexes per pack and per skill, plus a local search script) keeps disabled long-tail skills *findable* without keeping them *resident*. Disabling ≠ losing; it means moving from push (always injected) to pull (looked up when relevant).

## Results

| Measure | May 2026 | July 2026 | 9 August 2026 |
|---|---:|---:|---:|
| Always-on context | ~101K tokens | ~71K tokens | **50,942 tokens** |
| Skill descriptions | ~72K | ~41K | ~41K |
| Always-on standing rules | 27 files | 27 files | 7 files |
| Skill-routing accuracy | — | No observed regressions | No observed regressions; domain rules now arrive with their skill |

Total reduction from the May baseline: **~50%**. The July → August half came almost entirely from levers 5 and 6 — and lever 6 is the reason the July → August half took four extra days to actually happen.

## Lessons

- **Descriptions cost even when unused.** The price of an installed skill is paid every session, invoked or not. Install-time is the moment to ask "does this earn always-on placement, or catalog placement?"
- **The budget decays.** Every new skill, rule, and memory line erodes it. Re-measure on a schedule; the script makes that a 30-second check instead of an audit.
- **Startup cost is re-paid every turn.** It is not a one-time load — it is re-read on each turn of the session as cached context. A kilotoken saved is a kilotoken saved per turn, which is why a 15% cut is worth four days of chasing.
- **Verify the injection path empirically.** Marker-file spikes (plant a sentinel, check whether it surfaces in a fresh session's context) settle in minutes what documentation debates can't.
- **This is an ops discipline, not a one-off cleanup.** The interesting version of "prompt engineering" at system scale is deciding what *not* to load.

---

*Specific token counts are from my own workspace and will differ per setup; the levers transfer directly to any Claude Code (or comparable agent-harness) environment.*
