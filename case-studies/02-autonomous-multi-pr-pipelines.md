# Autonomous Multi-PR Pipelines

> **In business terms**
>
> - **Problem:** One long AI session working through a whole plan produces one huge change that is slow and risky to check.
> - **What changed:** Each plan item is built separately, checked by review agents, and handed over as its own small change.
> - **Result:** 5–10 reviewable changes per run. Since 21 November 2026 a loop merges the low-risk ones by fixed rules, and I still approve every change to the database, the login code and the build.

A local orchestration pattern for turning a roadmap into several independent pull requests, with each PR built, reviewed, and left ready to merge. Built because one long LLM session is the wrong shape for multi-workstream delivery.

**Status:** In active use across application and infrastructure work since May 2026. A wrapper loop spawns one fresh large-context implementation session per iteration; each iteration works one item on its own branch and opens a PR behind reviewer-agent gates; state is handed between iterations through a progress artifact. Since first validation the pattern has grown a scheduled front-end (see *The governance layer* below). Until 21 November 2026 one thing had not moved: I merged, the loop did not. That changed, see the [errata of 26 November 2026](#errata--26-november-2026-the-merge-gate-moved).

---

## Problem

AI-assisted coding is fast inside a narrow task, but a roadmap is rarely one task. It is usually a dependency graph:

- fix the broken foundation first
- split independent work into separate branches
- review each branch in context
- avoid mixing unrelated changes
- keep a human merge gate

The naive workflow is one very long session that reads everything, edits everything, and eventually hands you a massive diff. That feels productive until the review cost arrives. A 30-file diff with three unrelated goals is slower to trust than three clean PRs, even if the code was written quickly.

I needed a workflow that could safely run 5-10 small implementation tracks overnight without turning the morning review into archaeology.

---

## Constraints that shaped the design

1. **Human merge remains mandatory.** Automation can propose and prepare PRs; it cannot decide that shared-state changes are safe. *(Narrowed on 21 November 2026 to the changes that touch shared state, see the errata.)*
2. **One work item per branch.** Cross-contamination between tasks destroys reviewability.
3. **Reviewer agents must be real gates, not decoration.** A PR without review output is not ready.
4. **Windows-native environment.** The orchestration has to work from PowerShell, GitHub CLI, local repos, and Claude Code sessions.
5. **Recoverable by default.** If a worker fails halfway, it must leave logs and a branch, not a mystery.
6. **No hidden destructive operations.** Deletes, force-pushes, deployments, and other shared actions require explicit human approval.

---

## Architecture

```text
              Roadmap / work-item register
                         |
                         v
            PowerShell wrapper loop (orchestrator)
                         |
            +------------v-------------+
            |  Iteration N             |
            |  fresh large-context     |
            |  implementation session  |
            |  one WI, one branch      |
            |                          |
            |  local checks            |
            |  code-reviewer gate      |
            |  security-reviewer       |
            |  where needed            |
            |                          |
            |  push branch + open PR   |
            |  update progress file    |
            +------------+-------------+
                         |
              exit status -> next iteration
              (N+1 reads the progress file
               and picks the next WI)
                         |
                         v
            Human review, merge, and sequencing
```

The loop is sequential by design: one fresh session per iteration, handing off through a durable progress artifact instead of a shared chat context. The important part is the contract around each iteration: a bounded scope, a branch name, acceptance criteria, test commands, and a no-go list. The orchestrator is allowed to create work; it is not allowed to erase judgment.

---

## Key technical decisions

### 1. Roadmap-first decomposition

The input is a work-item register, not a vague prompt. Each work item has a title, effort, dependencies, risk level, autonomy classification, and gate reviewer. This makes the orchestrator behave more like a release manager than a brainstorming assistant.

### 2. Branch isolation over session isolation

Sessions forget. Branches do not. Each worker runs against its own feature branch so the durable artifact is the git diff, not the chat transcript.

That choice also makes failure less expensive. A failed worker branch can be inspected, amended, or abandoned without poisoning the rest of the roadmap.

### 3. Reviewer agents as explicit gates

The pipeline does not treat "tests passed" as enough. For code changes, a code-reviewer pass is expected. For security-sensitive surfaces, a security-reviewer pass is expected. For broad architecture changes, I use a higher-context review before implementation.

This catches a different class of issue than the compiler: scope creep, missing rollback, weak invariants, unsafe defaults, and "this works but should not exist."

### 4. Human-owned merge and shared actions

*This section describes the design until 21 November 2026. The errata of 26 November 2026 says what moved.*

The automation pushes branches and opens PRs itself — behind reviewer-agent gates. What it does not own: merging, deploying, moving live services, or deleting shared state. Those remain conscious human steps.

That sounds slower. In practice it is faster, because it preserves trust. I can review five clean, scoped PRs in less time than one ambitious mess.

---

## The governance layer: scheduling without surrendering the gate

The pattern later grew a scheduled front-end — the ability to set a build in motion for the night rather than kicking each run off by hand. The interesting part is everything I *refused* to automate on the way there.

- **Machine-checkable preconditions, not a mood.** A run only starts if four conditions hold: the work item is not blocked by an open human decision, the target repository has CI configured, the work list is non-empty, and the repository path resolves. All four are checked in code before anything spawns. None of them requires me to be awake.
- **Morning verification is part of the loop, not an afterthought.** A run only "counts" once a post-run audit confirms it cleared the gates. A run that fired but produced nothing mergeable is a failure the system has to surface, including the case where no run fired at all.
- **Unattended operation is gated on a scoped credential — deliberately.** An overnight loop running on a broad personal access token could, in principle, execute a generated-then-run script that touches production data. The unlock was a *read-mostly, scoped* credential: read access to the data and deploy surfaces the loop needs, and nothing that can delete or mutate production. That scoping is now in place, and it was the actual precondition for unattended nights — not a missing feature. A governance decision wearing the costume of a to-do item.

The through-line (true until 21 November 2026): automation earned more reach over time, but never the merge decision, and never a destructive capability it could reach unsupervised. Every increment in autonomy came with an explicit gate to match it.

### Errata — 19 August 2026: I removed a gate I had publicly defended

An earlier version of this section argued for two launch-time rules: **one build per night**, and **silence is a stop** (a scheduled proposal that waited for an explicit one-tap approval, so no reply meant no run). Both were real, both ran for roughly three months, and I have now removed both.

They were gating the wrong moment. The scarce resource is my attention at *merge* time, not at *launch* time — and a launch-time approval tap does not reduce merge-time load, it just adds a step that fails closed on a night I happen to be busy. In practice the rule mostly produced skipped nights, not safer ones. The one-build cap had a second problem: it capped the wrong unit. Iteration budget now scales with the size of the work list rather than being fixed at one build, so a five-item list gets a proportionally longer run instead of being truncated.

What did *not* change: the merge decision is still mine, destructive actions are still human-owned, and the credential the loop runs on is still scoped read-mostly. The gate moved nowhere. The ceremony in front of it went away.

I am leaving the original claim visible rather than editing it out. A pattern write-up that only ever shows the rules that survived is a worse artifact than one that shows which rule the author was wrong about.

### Errata — 26 November 2026: the merge gate moved

This page said in six places that I merge every pull request. That stopped being true on 21 November 2026. The old sentences stay above, marked, for the same reason as in August.

**What changed.** A loop now merges a pull request without me when all of these hold:

- The repository is one of three on a fixed list in the code.
- My own account opened the pull request.
- Every required check is green, and each one comes from its pinned workflow.
- No commit arrived for 20 minutes.
- No changed file is a database migration or schema file, login or access code, a CI file, build or dependency configuration, or a deploy file.
- The stop file does not exist.

The loop uses a squash merge, never admin rights, and at most three merges per run. It cannot merge a change to the repository that holds the loop itself: that repository is excluded in code.

After a merge, a second loop watches the deploy that the repository starts by itself. If that deploy fails, the guard writes the stop file and opens a draft revert pull request. Every loop then stops until I remove the file. The loop never deploys by hand and never runs a migration.

**What it did so far.** From 21 to 26 November 2026 the loop merged 26 pull requests in those three repositories. In the same six days I merged the other 80 by hand. 12 of the 26 started a deploy and all 12 succeeded. The other 14 had no deploy to watch. The guard opened no revert and never wrote the stop file. The most common reason for a refusal was a changed dependency file.

**Why.** Since March I merged 98% of the pull requests that were opened (3,459 of 3,532). A gate that lets 98% through does not decide much. The August errata said that my attention when I merge is the scarce resource. For a small change with green checks and no file on the deny list, that attention had become a click.

**What did not change.** Migrations, login code, CI, dependency and deploy configuration still wait for me. So do deletes, force-pushes and manual deploys.

**What I do not know yet.** Six days and 26 merges prove nothing. The guard catches a deploy that fails. It does not catch a wrong feature that deploys cleanly. That risk now rests on the checks and the AI review, with no human look before the merge.

---

## Outcomes

| Metric | Value |
|---|---:|
| PRs shipped per run | 5-10, sequential iterations |
| Target PR shape | one work item, one branch, one review surface |
| Human merge gate | always until 21 November 2026. Since then: always for migrations, login code, CI, dependency and deploy configuration |
| Merged by the loop | 26 of 106 merges in three repositories, 21–26 November 2026 — see errata |
| Launch-time human approval | removed August 2026 — see errata |
| Best-fit work | scoped feature work with per-WI acceptance criteria, refactors, testable infrastructure fixes |
| Poor-fit work | ambiguous product strategy, live service migration, destructive cleanup, legal/commercial decisions |

**Real win:** the bottleneck moved from "write all the code myself" to "make good merge decisions." That is the right bottleneck. It lets AI increase throughput without making the repository feel knowable only to the agent that produced it.

---

## Lessons

### "Autonomous" is an easy word to oversell

The system is autonomous only inside a fenced work item. It is not autonomous in strategy or risk acceptance, and since November 2026 its merge authority ends at a deny list that I wrote. Calling it autonomous without that qualifier would be misleading.

The useful framing was: autonomous implementation, human release judgment. Since November 2026 it is: autonomous implementation, and human judgment on which kinds of change the machine may release.

### Sequencing only works after dependencies are honest

If work item B depends on work item A, the roadmap has to say so — otherwise iteration B builds on a branch that does not exist yet, or quietly duplicates work. The decomposition step is where speed is actually won or lost, not the implementation step.

### Small PRs are the safety mechanism

The review burden scales with the conceptual span of a diff. A 300-line focused PR can be safer than a 40-line cross-cutting patch if the latter changes a hidden contract. The pipeline optimizes for bounded reasoning, not raw lines changed.

---

## What's NOT in this case study

- The wrapper script source and local paths
- Private repository names beyond public infrastructure examples
- Agent prompts that include machine-specific rules
- Any claim that the system can safely merge everything: it merges only what the deny list leaves, and it has done so for six days

The reusable pattern is the contract: roadmap decomposition, branch isolation, explicit review gates, and human-owned shared actions.

---

**Codified:** May 2026, as a reusable skill after the first validated overnight runs. Extended between May and August 2026 with a scheduled launcher front-end, whose launch-time approval tap was then removed in August 2026 (see errata). The merge decision stayed with me until 21 November 2026, when a rule-bound loop took over the low-risk merges (see the second errata). Destructive-action authority stayed with me throughout.
