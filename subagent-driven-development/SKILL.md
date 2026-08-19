---
name: subagent-driven-development
description: "Use for cost-efficient coding: delegate implementation, orchestrator adjudicates."
version: 3.0.0
author: Hermes Agent (adapted from obra/superpowers)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [delegation, subagent, implementation, workflow, cost-control]
    related_skills: [requesting-code-review]
---

# Subagent-Driven Development

## Purpose

Use the expensive main model as the **planner, orchestrator, and adjudicator**. Use the configured delegate model as the **implementation worker**.

Default loop:

> Plan and bound the task → dispatch implementation worker(s) → inspect the actual diff → adjudicate correctness → fix confirmed issues → run final verification → deliver.

The goal is not maximum agent count. The goals are (a) to move implementation tokens to a cheaper model without surrendering architectural judgment or final correctness, and (b) to keep every session's context small: narrow briefs keep worker sessions bounded, and fan-out keeps implementation churn out of the orchestrator's context. Fan-out is a context-budget and speed tool, not just a cost tool.

## Cost-Control Rules

1. **Do not delegate trivial work.** If explaining and dispatching the task costs more than doing a small mechanical edit directly, do it directly.
2. **Actively capitalize on safe delegation.** For a non-trivial task, decompose into independent, bounded workstreams instead of reflexively using one worker. Dispatch all dependency-ready, disjoint slices concurrently when their ownership and mutable resources are disjoint, or when parallel work is read-only (for example, independent contract audits). The concurrency bound is disjointness and runtime limits, not a fixed agent count. This is the default optimization: it reduces elapsed time and keeps each session's context small without weakening review.
3. **Serialize overlapping writers.** Do not run concurrent implementation workers that can edit the same files, symbols, generated outputs, snapshots, databases, ports, accounts, or other mutable resources. Partition first, then parallelize only proven-disjoint ownership.
4. **Do not duplicate the worker's implementation.** The orchestrator should inspect and reason about the resulting diff, not independently reimplement the same feature.
5. **Do not spawn reviewers by habit.** Main-agent inspection is the default review. Independent review requires a specific trigger.
6. **Do not bounce routine fixes between agents.** The orchestrator directly fixes small, confirmed defects because it has the broader context and stronger model.
7. **Verify proportionally.** Rerun focused tests and project-required gates. Do not run every possible suite or review stage merely because it exists.
8. **Keep prompts bounded.** Pass task semantics, paths, invariants, and commands—not copied repositories, giant file bodies, or irrelevant conversation history.

Delegate provider/model/reasoning are configured outside this skill. Confirm them when the user changes delegation settings or when a mismatch is suspected; do not revalidate the model registry on every task.

## When to Use

Use when:

- The implementation is substantial enough that delegation saves main-model tokens.
- The outcome can be stated with concrete acceptance criteria.
- A worker can inspect and edit a bounded area without unresolved product decisions.
- Several jobs are mechanically disjoint and parallel execution provides real value.

Do not use when:

- The change is tiny or purely mechanical.
- The task is primarily architecture, product judgment, or review adjudication.
- Required user decisions are unresolved.
- The implementation depends on an active worker's files or mutable runtime resources.
- The worker brief would require reproducing most of the orchestrator's context.

## Roles

### Main model: plan, orchestrate, adjudicate

The main agent owns:

- Understanding the user's outcome and non-negotiable constraints.
- Inspecting enough of the repository to identify boundaries, invariants, and acceptance tests. Avoid exhaustive rediscovery when the worker can perform local discovery safely.
- Choosing direct work versus delegation and one worker versus a disjoint batch.
- Writing self-contained worker briefs.
- Preserving user changes and managing writer leases.
- Inspecting every changed path and the actual diff after handoff.
- Adjudicating behavior, security, compatibility, and scope.
- Directly correcting small confirmed defects and adding regressions where useful.
- Running authoritative final verification and reporting only verified results.

### Delegate model: implement the bounded outcome

A worker owns one concrete implementation job. It may perform local discovery needed inside its owned area, then edit and test the result.

The worker must receive:

- Objective and observable acceptance criteria.
- Owned paths or a clearly bounded component. Exact file allowlists are required for concurrent writers; a single worker may use a component boundary when exact dependencies are not yet known.
- Relevant invariants and compatibility/security constraints.
- Non-goals and explicit stop conditions.
- Focused verification commands.
- A handoff requiring changed paths, test results, assumptions, and unresolved concerns.

Workers do not commit, broaden scope, adjudicate peers, or perform unrelated cleanup unless explicitly instructed.

## Lightweight Preflight

Before dispatch:

- Understand branch/worktree state and preserve pre-existing changes.
- Load relevant repository instructions and task skills.
- Identify the acceptance boundary and likely affected component.
- Ensure concurrent jobs have no file, symbol, generated-output, database, port, container, account, or snapshot overlap.
- Use todos only when the overall task has enough moving parts to benefit from them; they are not mandatory delegation ceremony.

Do enough discovery to write a safe brief. Do not spend expensive-model tokens tracing every implementation detail that the bounded worker can discover.

## Worker Brief

```text
OBJECTIVE
One concrete implementation outcome.

ACCEPTANCE
Observable behavior and required invariants.

OWNERSHIP
Bounded component or exact source/test paths. For parallel writers, use disjoint allowlists.

CONTEXT
Relevant symbols, architecture decisions, repository instructions, and paths to inspect.

NON-GOALS
Adjacent behavior the worker must not change.

WORKFLOW
Use focused RED→GREEN TDD when behavior is testable. Make the smallest complete change.

VERIFY
Focused commands that prove the owned behavior. Do not run the entire repository matrix unless required.

HANDOFF
Report changed paths, commands/results, assumptions, and unresolved concerns. Do not commit. Stop when the bounded outcome is complete.
```

A worker report is a claim. The actual workspace and main-agent tool output are authoritative.

## Execution Loop

### 1. Plan and partition

Decompose the plan into the smallest independently-verifiable jobs. Never hand one worker the entire todo list — one worker owns one bounded slice with very specific acceptance criteria. Dispatch all dependency-ready, mechanically disjoint slices concurrently; serialize only overlapping writers or true dependencies. Use a single worker only when the task genuinely has one indivisible slice.

### 2. Dispatch

Send self-contained briefs. A managed worker holds a writer lease until the runtime reports completion, failure, timeout, or cancellation. While active, do not edit, test, stage, or commit overlapping files or mutable resources.

### 3. Reconcile

After authoritative completion:

- Inspect `git status` and every changed path.
- Compare the actual diff with ownership, acceptance criteria, and non-goals.
- Reject scope drift, hidden generated changes, unrelated cleanup, or unverifiable claims.
- Rerun the worker's focused tests from the stable main workspace.

Inspect evidence-proportionally to keep review cheap: start from `git diff --stat` and the worker's focused test results; read full hunks only where risk concentrates — shared surfaces, security/compatibility boundaries, protocol/contract code, or anything touching paths outside the worker's ownership. For mechanical or well-tested changes, passing acceptance tests plus a stat-level scan is sufficient primary evidence. Every changed path must still be accounted for; "accounted for" does not require reading every line. When reconciling a wide concurrent batch, process one worker's result at a time rather than loading all diffs simultaneously.

Do not reread the entire repository or redo the implementation merely to prove the orchestrator was involved.

### 4. Review and adjudicate

Main-agent review is mandatory and normally sufficient. Classify each concern as:

- **Confirmed:** supported by code, spec, reproduction, invariant, or test.
- **False positive:** contradicted by repository evidence.
- **Out of scope:** valid observation not required for this task.
- **Decision required:** materially different valid product choices; ask the user.

Do not turn suggestions into mandatory work without evidence that they affect acceptance or safety.

### 5. Fix confirmed issues

For a small or localized correction, the main agent edits it directly, adds or strengthens regression coverage when appropriate, and reruns affected tests.

If adjudication reveals a substantial redesign or a large new implementation slice:

1. Stop accepting the affected result.
2. Reassess the design and ownership boundary.
3. Keep architecture and shared integration main-agent-owned.
4. Dispatch a fresh bounded implementation job if delegation still saves tokens.

Do not reflexively return routine feedback to the original worker or spawn a separate fix agent.

### 6. Final verification

Run:

- Focused worker tests from the stable workspace.
- Regressions for changed shared behavior.
- Affected integration/type/lint/static/build checks.
- The repository's explicitly required completion gate.

Run a full repository suite only when project instructions require it, the blast radius justifies it, or focused evidence cannot establish safety.

Then account for all changed paths, update useful todos, and commit only when requested or required by the active workflow.

## Independent Review Policy

An independent reviewer is **not part of the default loop**. The expensive orchestrator already reviews and adjudicates the delegate's work.

Add at most one combined read-only reviewer for the integrated diff only when:

- The user explicitly requests independent review.
- A formal security/compliance/release gate requires it.
- The change is high consequence and the orchestrator identifies a concrete uncertainty that code inspection and tests cannot close.
- Competing implementations or subtle domain semantics benefit from a genuinely independent perspective.

Do **not** add an independent reviewer merely because:

- A subagent wrote the code.
- More than one or two files changed.
- A task, wave, commit, or push is complete.
- The change touches a category labeled “moderate” or “high” but the orchestrator can establish correctness directly.
- A previous reviewer produced only non-blocking suggestions.

If used, review once at the highest integrated boundary. Re-review only for a formal requirement or a material redesign—not after ordinary localized fixes.

A reviewer is read-only evidence, not an acceptance authority. The main agent adjudicates every finding before changing code.

## Failure and Recovery

- Never replace or overwrite an unresolved managed worker lease.
- On timeout, delayed callback, cancellation, or uncertain writer state, load `references/managed-delegate-writer-leases.md`.
- For external CLI workers or isolated worktrees, load `references/external-cli-process-trees.md`.
- If a worker fails without writing, replan or take over only after the lease ends.
- If a worker returns an incomplete but safe partial diff, either finish a small remainder directly or define a fresh bounded job. Do not create an endless feedback loop.

## Progressive References

Load only when triggered:

| Trigger | Reference |
| --- | --- |
| Large roadmap, many workers, or context pressure | `references/context-budget-discipline.md` |
| Explicit revision/escalation/abort gates | `references/gates-taxonomy.md` |
| Security/DAST or bounded evidence capture | `references/security-pipeline-orchestration.md` |
| Managed worker timeout or uncertain lease | `references/managed-delegate-writer-leases.md` |
| External CLI process or isolated worktree | `references/external-cli-process-trees.md` |

## Completion Checklist

- [ ] Delegation saved more work than it added.
- [ ] The plan was partitioned into narrow slices; no single worker received the entire todo list.
- [ ] Each worker had one bounded, checkable outcome.
- [ ] Concurrent writers had disjoint ownership and mutable resources.
- [ ] Worker leases ended authoritatively before reconciliation.
- [ ] The main agent inspected every changed path and reran focused tests.
- [ ] The main agent adjudicated correctness without duplicating implementation.
- [ ] Confirmed localized defects were fixed directly.
- [ ] Independent review was used only with a documented trigger.
- [ ] Required affected and project-mandated gates passed.
- [ ] Every changed path is explained and user state preserved.
- [ ] No commit was created unless requested or required.
