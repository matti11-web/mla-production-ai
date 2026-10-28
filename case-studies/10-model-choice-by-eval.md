# How I Know the AI Answer Is Right

> **In business terms**
>
> - **Problem:** An AI answer can look complete and still be wrong. A busy owner who reads it has no way to see that.
> - **What changed:** I check AI output with a fixed test: real messy inputs, the exact production instructions, a blind judge, real costs, and a new test for each new task.
> - **Result:** The test found a cheap model that failed three of four real documents without any error. The fix costs about five cents more per summary.

Most AI mistakes do not look like mistakes. The answer arrives on time, it reads well, and nothing in the system reports a problem. This case study describes the method I use to find those mistakes before a customer does. The worked example is one evaluation of a summary feature. Two later evaluations show the same method at work.

**Status:** Eval run July 2026. At the time of writing (July 2026) the model swap it recommended was a configuration change awaiting sign-off. This case study does not track the rollout after that.
**Status (reframe, 28 October 2026):** Rewritten to present the July eval as an example of a general checking method. Two short examples from September 2026 were added. The July facts and figures are unchanged.

---

## The problem: wrong answers that look fine

A summary feature condensed long, noisy documents into structured output. The documents were machine-generated text of tens of thousands of characters. They contained the errors that automated extraction leaves behind: garbled words, half sentences and repeated fragments.

The feature ran on the cheapest capable model. That choice came from an earlier test on a different task (answering questions with tools), where the cheap model did well.

Nothing looked wrong. The service never went down. No error rate moved. No alert fired. Every request returned a paragraph of text. But some of those paragraphs were refusals, and one contained invented details. Normal monitoring cannot see this, because the system works and only the content is wrong.

## The method

I use five checks. Each one closes a specific way to fool yourself.

| Check | What I do | What it prevents |
|---|---|---|
| Real messy inputs | Use real production documents, prepared through the same pipeline as production | Clean test data hides the failures. Real users bring messy data. |
| The verbatim production prompt | Copy the exact instructions and output schema from the live service | A better prompt in the test measures a system that does not exist. |
| A blind judge that is not a candidate | A separate model scores anonymised outputs. It is not one of the models under test. | A model that grades its own output gives no evidence. |
| Cost from real token usage | Compute cost and latency from the token counts of the actual runs | List-price estimates are often wrong for long inputs. |
| Re-measure for each task | Run a new test when the task changes | A verdict from one task does not transfer to another task. |

## Worked example: the summary feature

### The test

| Step | What |
|---|---|
| Inputs | 4 real production documents, 24,000 to 56,000 characters each. Not synthetic. |
| Preparation | Generated through the same upstream pipeline as production, so the input noise matches reality |
| Candidates | 4 model configurations: the incumbent and others, including two reasoning-effort variants of the newest model |
| Prompt | The verbatim production prompt and schema, copied from the live service |
| Scoring | A blind jury: a separate model, not among the candidates, scored anonymised outputs on faithfulness, coverage, structure and language |
| Hard metrics | Cost and latency computed from actual token usage |
| Total | 16 runs |

### What it found

The incumbent cheap model produced usable output for **one of four** real documents.

- On two documents it refused. It asked for a corrected input.
- On a third document it called the text unreadable. It returned loose figures with no structure and no action items.
- On the cleanest document it did produce a summary. That summary contained two invented details.

The mid-tier model received the same inputs. It produced four usable outputs out of four, with the highest coverage score in the test. It correctly extracted the key figures, timing and follow-up items from the text that the cheap model had called unreadable.

The price of the fix was about **five cents more per summary**.

Two more findings:

- The newest and most capable model was not the best choice. On long inputs its runs failed more often than the mid-tier model's runs, because the output was cut off. A newer model is not automatically better for a specific task.
- The winner of the earlier question-answering test did not win here. The old verdict was wrong for this task in practice, not only in principle.

## Two later examples

The same method caught two more answers that looked fine but were wrong. Both come from September 2026.

**A French question answered in Dutch.** I tested a cheaper model for an internal chat assistant that answers colleagues in Dutch and French. The cheaper model matched the more expensive model on quality and cost 31% less. But it answered one of the five French test questions in Dutch, in all three effort settings. The fact in the answer (a 30% deposit) was correct. The language was wrong. A probable cause is a Dutch source document, but that cause was not checked. The system prompt already told the model to answer in the language of the question. A strong reminder added next to each French question fixed it: 0 of 3 French answers without the reminder, 5 of 5 with it. After deployment, all 5 French questions in a 36-question production check came back in French.

**A cheaper model for a different task.** For the automated bots, the same cheaper model scored 8.08 against 7.89 for the model in production, and cost 37% less per run. This result came from a separate test on bot tasks. It did not reuse the chat result.

## What generalises

1. **Treat the check as part of monitoring.** Wrong content does not raise errors. For AI features, a regular test is the only signal that the content is still right.
2. **A verdict belongs to one task.** When the task changes, run a new test. A result from someone else's test, or from your own test on another task, is a guess.
3. **Test on your worst real inputs.** Quality differences between models are small on clean data and large on messy data.
4. **Read the output in the language and form the user sees.** A correct fact in the wrong language is still a failure for the reader.
5. **Put a price on the quality gap.** "Five cents per summary" ends a discussion that "the bigger model feels better" would keep open for weeks.

For an owner who uses AI output: the right model is the cheapest one that passes a test on your own task and your own data. A model that passed a test on another task can fail on yours, and you may only learn it from a customer.

---

## What's NOT in this case study

- The evaluation inputs, their contents, or anything that identifies the people or business they concern (inputs and model outputs stay local and out of version control by design)
- The production prompts and schemas
- The application domain and any company-identifying detail about the pipelines these tests feed
- The full score tables of the September 2026 evaluations

*Originally written July 2026. Reframed 28 October 2026.*
