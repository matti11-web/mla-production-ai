# Autonomous Multi-PR Pipelines

A local orchestration pattern for turning a roadmap into several independent pull requests, with each PR built, reviewed, and left ready for human merge. Built because one long LLM session is the wrong shape for multi-workstream delivery.

**Status:** In active use across application and infrastructure work. The pattern has been validated across six roadmap runs: a wrapper loop spawns one fresh large-context implementation session per iteration, each iteration works one item on its own branch and opens a PR behind reviewer-agent gates, and state is handed between iterations through a progress artifact. I merge; the loop does not.

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

1. **Human merge remains mandatory.** Automation can propose and prepare PRs; it cannot decide that shared-state changes are safe.
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

The automation pushes branches and opens PRs itself — behind reviewer-agent gates. What it does not own: merging, deploying, moving live services, or deleting shared state. Those remain conscious human steps.

That sounds slower. In practice it is faster, because it preserves trust. I can review five clean, scoped PRs in less time than one ambitious mess.

---

## Outcomes

| Metric | Value |
|---|---:|
| Validated roadmap runs | 6 |
| PRs shipped per run | 5-10, sequential iterations |
| Target PR shape | one work item, one branch, one review surface |
| Human merge gate | always |
| Best-fit work | scoped feature work with per-WI acceptance criteria, refactors, testable infrastructure fixes |
| Poor-fit work | ambiguous product strategy, live service migration, destructive cleanup, legal/commercial decisions |

**Real win:** the bottleneck moved from "write all the code myself" to "make good merge decisions." That is the right bottleneck. It lets AI increase throughput without making the repository feel knowable only to the agent that produced it.

---

## Lessons

### "Autonomous" is an easy word to oversell

The system is autonomous only inside a fenced work item. It is not autonomous in strategy, risk acceptance, or merge authority. Calling it autonomous without that qualifier would be misleading.

The useful framing is: autonomous implementation, human release judgment.

### Sequencing only works after dependencies are honest

If work item B depends on work item A, the roadmap has to say so — otherwise iteration B builds on a branch that does not exist yet, or quietly duplicates work. The decomposition step is where speed is actually won or lost, not the implementation step.

### Small PRs are the safety mechanism

The review burden scales with the conceptual span of a diff. A 300-line focused PR can be safer than a 40-line cross-cutting patch if the latter changes a hidden contract. The pipeline optimizes for bounded reasoning, not raw lines changed.

---

## What's NOT in this case study

- The wrapper script source and local paths
- Private repository names beyond public infrastructure examples
- Agent prompts that include machine-specific rules
- Any claim that the system can safely merge or deploy without human review

The reusable pattern is the contract: roadmap decomposition, branch isolation, explicit review gates, and human-owned shared actions.

---

**Codified:** May 2026, as a reusable skill after three validated overnight runs; six runs validated to date (April–May 2026).
