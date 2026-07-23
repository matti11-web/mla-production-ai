# Choosing a Model With an Eval, Not a Vibe

A production summarization feature ran on the cheapest capable model. A blind evaluation on real inputs showed it was failing three out of four times — silently, in a way no error log would ever surface. This is the harness that found it.

**Status:** Eval run July 2026; the model swap it recommended is a configuration change awaiting sign-off.

---

## Why this eval existed at all

The feature: long, messy speech-to-text transcripts summarized into structured output for a business application. The model choice was inherited from an earlier evaluation on a *different* task (tool-augmented Q&A), where the cheap model had been fine.

That inheritance is the actual failure mode, and it's everywhere: **an eval result is bound to its task.** Summarizing a noisy 40,000-character transcript is not answering a question with tools. The quality dimension is different (faithfulness and coverage, not citation accuracy), the input length is different by an order of magnitude, and the failure modes are different. Carrying the old verdict over was a guess wearing an eval's clothes.

## The harness

| Step | What |
|---|---|
| Inputs | 4 real recordings, 24,000–56,000 characters per transcript. Not synthetic. |
| Transcription | Same speech-to-text provider, language and speaker-labelling as the production pipeline (EU-hosted) |
| Candidates | 4 model configurations, including the incumbent and two reasoning-effort variants of the newest model |
| Prompt | The **verbatim production prompt and schema**, copied from the live service — not a prompt written for the eval |
| Scoring | A blind jury: a separate model, not among the candidates, scoring anonymized outputs on faithfulness, coverage, structure and language |
| Hard metrics | Cost and latency computed from actual token usage, not list-price estimates |
| Total | 16 runs |

Four design choices did the heavy lifting:

1. **Real inputs, not synthetic ones.** A synthetic transcript is clean. Clean input is precisely where the cheap model looked fine. The whole finding lives in the mess: garbled words, half sentences, speaker overlap.
2. **The production prompt, copied verbatim.** If you evaluate with a nicer prompt than production runs, you've measured a system that doesn't exist.
3. **A blind judge that isn't a candidate.** A model grading its own output is not evidence. Anonymize the outputs, and use a model with no entry in the race.
4. **Cost and latency measured, not estimated.** Both come out of real token counts from the runs.

## What it found

The incumbent cheap model produced a usable summary for **one of four** real recordings. On two it refused outright, asking for "a correct transcript to be resubmitted." On a third it declared the text unreadable and returned loose numbers with no structure or action items. On the single clean recording where it did produce a summary, that summary contained an invented measurement and an invented role.

From the *same* transcripts, the mid-tier model produced four usable summaries out of four, with the highest coverage score in the field, and correctly extracted prices, timing, objections and next steps from the very text the cheap model had called unreadable.

The price of the fix: roughly **five cents more per summary**.

Two secondary findings worth recording:

- The newest, most capable model was **not** the answer — its runs failed technically on long inputs (truncated output) more often than the mid-tier model's did. Newer ≠ better for a specific task shape.
- The earlier eval's winner, carried over from the Q&A task, did not win here. The inheritance was wrong on the merits, not just in principle.

## The lesson that generalises

**Silent quality failure is the expensive kind.** Nothing was down. No error rate moved. No alert fired. The feature returned a 200 and a paragraph of text; the paragraph just happened to be a refusal or a fabrication. Traditional monitoring is blind to this by construction — which means for AI features, an eval *is* part of your monitoring, not a pre-launch formality.

Three practical rules I now apply:

1. **An eval is bound to its task.** New task shape → new eval. Reusing a verdict across tasks is a guess.
2. **Evaluate on your ugliest real inputs.** Model quality differences compress on clean data and explode on messy data. Your users bring messy data.
3. **Price the quality gap before arguing about it.** "Five cents per summary" ends a debate that "the bigger model feels better" would have kept alive for a month.

And the meta-point for anyone deploying AI in a business: the cheapest model that passes *your* eval is the right choice. The cheapest model that passes *someone else's* eval, on a different task, is a liability you won't notice until a customer does.

---

## What's NOT in this case study

- Recording contents, participants, or any customer-identifying material (transcripts and model outputs stay local and out of version control by design)
- The production prompt and schema
- Company-identifying detail about the pipeline this feeds
