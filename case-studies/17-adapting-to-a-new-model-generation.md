# Adapting the Setup When the Model Changes

> **In business terms**
>
> - **Problem:** Over five months I built a large layer of instructions around the AI: 275 reusable skills, 27 always-on rules and a compression plugin. Each piece made sense for the models of that moment. A newer model generation may not need them, and nobody sends a notice when that happens.
> - **What changed:** I tested the layer against the newer models instead of keeping it out of habit. Two blind tests, 60 runs in total. Then I removed what did not earn its place.
> - **Result:** The full workspace context and a stripped one scored the same, 7.30 out of 10. The only wrong price answers came from an out-of-date document in the context. On 24 October 2026 I retired both skill packs. 21 skills and 7 always-on rules remain.

Case 10 shows how I choose a model per task by measurement. This case is about the other side: the setup around the model. Instructions that helped an older model can cost tokens, or even mislead, on a newer one.

**Status (28 October 2026):** Both tests ran on 10 October 2026. The compression plugin went off the same day. The skill packs were retired on 24 October 2026. I have not re-measured the startup cost of a session since the retirement.

---

## What I had built

Between May and August 2026 I built a working environment around the AI. It had three layers.

- **Skills:** 275 reusable instructions, in two private packs. One pack held engineering doctrine. The other held domain knowledge, such as how the company prices a product line.
- **Rules:** 27 short rules that loaded in every session. They pointed to longer reference files.
- **A memory index:** one file that lists what the assistant should know about my projects.

The [context-budget pattern](../patterns/context-budget-engineering.md) describes how I cut the startup cost of that environment from about 101K tokens to 50,942. That work made the layer cheaper. It did not ask if the layer was still needed.

## Why I tested it

A newer model generation arrived in the summer. Public guidance on context for these models said that the references a model gets must be rich and current. That raised a question: did my large layer still help, or only cost tokens?

I did not adopt that as a fact. A claim about models in general says little about my tasks. So I measured it on my own work.

## Test one: full context against stripped context

The setup was 3 variants, 5 real tasks and 2 runs each: 30 runs.

- **Full:** the normal workspace, with rules, memory and skills.
- **Full without the compression plugin.**
- **Stripped:** a folder outside the workspace, so none of my rules, memory or skills loaded.

Every run was read-only. The tools could read and search files, nothing else. Nothing could be sent or written. A separate model judged the answers blind. The answers were shuffled and labelled A to F, and the judge had a rubric with the correct answer per task.

| Variant | Mean score (0–10) | Total cost |
|---|---|---|
| Full | 7.30 | $3.666 |
| Full, no plugin | 6.80 | $4.713 |
| Stripped | 7.30 | $3.484 |

The full context gave no measurable gain. It helped on the tasks that needed private facts: a mail draft that depended on a contract detail scored 7.5 with context and 6.0 without. On the self-contained tasks, a code review and a roadmap critique, stripped did as well or better.

## The stale document

One task asked for the price of a product configuration. The answers split sharply.

- 2 of the 4 full-context runs quoted an outdated price. They took it from a reference document in the context.
- Both stripped runs had no such document. They read the pricing code and got the correct price.

The document was not wrong when I wrote it. The pricing code changed on 15 September 2026, and the document did not follow. My own first rubric for this task was also wrong, because I took the answer from the same document. I re-judged the task after I checked the code.

The lesson is direct. **A stale reference is worse than no reference.** The model trusts what it receives up front. Without that doctrine, it goes to the source. I fixed the document, but the real fix is different: the document now names the code as the source of truth, instead of carrying its own copy of the numbers.

## Test two: the compression plugin on code

The plugin compresses the model's output. It cost 2,226 tokens at the start of every turn, in every session. The first test had only one code task, so I ran a second test: 5 code tasks, plugin on and off, 3 runs each. Another 30 runs, judged blind.

| Plugin | Mean score | Mean changed lines | Total cost |
|---|---|---|---|
| On | 7.53 | 20.5 | $2.910 |
| Off | 7.73 | 28.1 | $2.841 |

The plugin made diffs 27% smaller on 5 of 5 tasks. The quality was a tie, and the cost was the same. On one task the plugin runs also skipped a test that proved an edge case still worked. The best answer on that task came from a run without the plugin.

I switched the plugin off on 10 October 2026. Smaller diffs did not buy a better or a cheaper result, and the plugin loaded in sessions that had nothing to do with code.

## The retirement

On 24 October 2026 I retired both skill packs. One commit changed 789 files and deleted 59,564 lines.

- The two pack repositories are archived.
- 10 skills were removed outright. Their content already lived in the full reference files.
- The tools that switched pack profiles and searched the skill catalog went with them.
- Domain knowledge stays as a small set of skills. A skill only loads its full text when a task calls it.

What remains on 28 October 2026:

| Layer | Summer 2026 | 28 October 2026 |
|---|---|---|
| Skills | 275, in two packs | 21 |
| Rules that load in every session | 27 | 7 |
| Full reference files, read only when needed | not counted | 30 |

The rules went from 27 to 7 in August 2026, during the context-budget work. The rule text did not disappear: each full rule moved to a reference file that loads only when a task needs it. The skills went from 275 to 21 with the retirement.

The 7 rules that still load are the ones that apply to every task, such as how to treat a claim that has no source. Everything domain-specific moved behind a call.

## The model choice moves too

The same pattern showed up in model choice. Case 10 describes the method. Two results from the newer generation changed my defaults.

- **Less reasoning was better for review work.** For code-review subagents, the lowest reasoning setting found 10.9 real findings per pull request, against 7.4 for the previous model at the same setting. The higher settings cost more and found no more. The highest setting cut off 2 of 8 reviews before they finished.
- **A model alias pointed at an old model.** The short name for "the strongest model" resolved through a cached list that still named an older generation. Anything that used the alias would run on that older model, with no warning. I now use the full model identifier everywhere.

## What generalises

- **Re-test the scaffolding when the model changes.** Instructions written for one model generation are an assumption about that model. Measure them again on the next one.
- **A stale reference is worse than none.** If a document copies a number from code, it will drift. Point to the code instead.
- **Test on your own tasks, not on general claims.** Public guidance gave me the question. Only my own tasks gave me the answer.
- **Keep what carries private facts.** Memory about my projects helped on the tasks that needed it. Generic doctrine did not.
- **Pay per call, not per session.** A skill that loads only when called costs nothing on the days it is not needed.
- **Never trust an alias for a model.** Write the full identifier, so a change of model is a decision and not an accident.

## Limits

- Each cell in the first test had 2 runs. A difference under about 1.5 points is not a result.
- One model family generated and judged the answers.
- The tests ran without the interactive tools. They say nothing about the cost of those tools.
- I have not re-measured the startup cost of a session after the retirement. The before and after table counts files, not tokens.

---

## What's NOT in this case study

- The task prompts, the rubrics and the raw answers
- Product names, prices and the pricing rule behind the stale document
- The names of the skills, the rules and the plugin
- Model names and version numbers
- Repository paths and code

---

**Tests run:** 10 October 2026. **Compression plugin off:** 10 October 2026. **Skill packs retired:** 24 October 2026. **Counts measured:** 28 October 2026. **Last updated:** 28 October 2026.
