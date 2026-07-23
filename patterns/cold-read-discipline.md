# Cold-Read Discipline

A verification pass for anything an AI generated that you're about to act on or publish. It exists because the dangerous failure mode of a capable model isn't nonsense — it's plausible, specific, confident detail that happens to be wrong.

**Status:** A standing rule in my own workflow, applied to brainstorms, roadmaps, drafts, and analyses before they inform a decision.

---

## The problem it solves

An LLM asked to summarize, plan, or draft will produce fluent output with the *texture* of fact: named people, exact figures, specific scenarios, cited artifacts. Some of that texture is real. Some is invented, and the invented parts are written in exactly the same confident register as the true ones. There is no stylistic tell.

Concrete example from my own work: an AI drafted a set of case studies about projects I had actually built. Four of them contained false claims — an architecture diagram describing an execution model that never ran that way, a pull request cited as evidence that didn't exist, a throughput number about ten times reality, a file count off by an order of magnitude. Every one was the kind of claim I'd have nodded at, because they were about *my* projects, phrased the way I'd phrase them. Fluency plus familiarity is exactly the blind spot.

The naive response — "I'll just read it and I'll notice if something's wrong" — fails, because reading for comprehension and reading for verification are different acts. Comprehension rewards flow; the errors ride along inside the flow.

## The discipline

Before AI-generated content informs an action or goes public, run a separate pass with a different posture:

1. **Read adversarially, cold.** Not "does this make sense?" but "as if a competitor wrote this to mislead me — where is it wrong?" The shift in stance is the whole mechanism. You are hunting for the error, not following the argument.

2. **Treat specifics as unverified until checked.** Every named person, company, exact scenario, and uncited number is a claim, not a fact, until traced to a source. Specificity is not evidence — a model invents specific detail as readily as general detail, and specific detail is more persuasive, which makes it more dangerous.

3. **Verify against the source, not against plausibility.** "That sounds right" is the failure. Open the actual PR, count the actual files, check the actual figure. If a claim can't be traced to something checkable, it doesn't ship as fact — it gets softened to a claim or cut.

4. **Add dated errata; don't silently repair.** When you find a hallucination, record it — what was claimed, what was true, when you caught it — rather than quietly overwriting. The record of what the model got wrong is itself signal: it tells you which kinds of claims to distrust next time, and it keeps the correction honest.

## Why this is non-negotiable, not optional polish

The productivity gain from AI is real, and it comes from *not* re-doing by hand what the model produced. That same gain is precisely why the verification step is the one you're tempted to skip when you're moving fast — and skipping it is where a fast workflow ships a confident falsehood into a decision or a public artifact.

So the rule is asymmetric on purpose: the verification pass is cheapest exactly when it feels most redundant (the output looks great) and most valuable exactly then too (great-looking output is the kind that gets trusted without checking). You don't get to earn out of it by having a good model. A better model produces *more convincing* wrong claims, not fewer.

## What it costs, honestly

- **Time.** A real cold-read of a substantial document is not free — it's a second pass with full attention. The trade is that a single confident falsehood reaching a customer or a decision costs far more than the pass.
- **It doesn't catch everything.** A claim that's false but that you *can't* trace to a checkable source still slips through as "unverified — softened." The discipline reduces confident-wrong output; it doesn't guarantee omniscience.
- **It requires a genuine stance change.** If you run the "adversarial" pass in the same comprehension mode as the first read, you get nothing. The value is entirely in actually adopting the skeptical posture, which takes deliberate effort each time.

## The general principle

Fluency is not accuracy, and a capable model widens the gap rather than closing it, because it makes the inaccurate output more fluent. Any workflow that puts AI-generated content in front of a decision or an audience needs a verification step that is structurally separate from generation — different posture, source-checked, and never skipped on the strength of how good the output looks.

---

*This pattern is applied throughout this repository: the case studies were cold-read against their sources before publication, and known corrections were made openly rather than silently.*
