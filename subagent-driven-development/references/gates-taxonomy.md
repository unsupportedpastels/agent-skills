# Gates Taxonomy

Canonical gate types for validation checkpoints across any workflow that spawns subagents, runs review loops, or has human-approval pauses. Every validation checkpoint maps to one of these four types — naming them explicitly makes the workflow legible and prevents "what happens when this check fails?" confusion.

Adapted from the GSD (Get Shit Done) project's gates reference — MIT © 2025 Lex Christopherson ([gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)).

## The four gate types

### 1. Pre-flight gate

**Purpose:** Validates preconditions before starting an operation.

**Behavior:** Blocks entry if conditions unmet. No partial work created — bail before anything changes.

**Recovery:** Fix the missing precondition, then retry.

**Examples:**
- Implementation phase checks that the plan file exists before it starts writing code.
- Delegated subagent checks that required env vars are set before making API calls.
- Commit checks that tests passed before pushing.

### 2. Revision gate

**Purpose:** Evaluates output quality and routes to revision if insufficient.

**Behavior:** A checker reports specific evidence. In subagent-driven coding, the main agent adjudicates the findings and applies confirmed corrections directly. Bound the loop (typically at most one combined review per wave); re-review only when a formal gate requires it or a correction materially changes the design.

**Recovery:** The main agent rejects false positives, fixes confirmed issues, reruns focused and integration verification, and either closes the gate or escalates unresolved product ambiguity. If the review exposes a fundamental redesign, stop acceptance and repartition fresh implementation jobs instead of bouncing feedback to the original worker.

**Examples:**
- Plan review identifies an ambiguous product decision; the main agent escalates it to the user.
- A combined code reviewer finds a missing authorization invariant; the main agent confirms it, adds the correction and regression test, then reruns the gate.
- Test inspection shows coverage does not execute the new path; the main agent repairs the test before acceptance.

### 3. Escalation gate

**Purpose:** Surfaces unresolvable issues to the human for a decision.

**Behavior:** Pauses workflow, presents options, waits for human input. Never guesses, never picks a default.

**Recovery:** Human chooses action; workflow resumes on the selected path.

**Examples:**
- Revision loop exhausted after 3 iterations.
- Merge conflict during automated worktree cleanup.
- Ambiguous requirement — two reasonable interpretations and the choice changes the approach.
- Subagent reports "the plan says X but the codebase actually does Y" — human decides which is right.

### 4. Abort gate

**Purpose:** Terminates the operation to prevent damage or waste.

**Behavior:** Stops immediately, preserves state (checkpoint current progress), reports the specific reason.

**Recovery:** Human investigates root cause, fixes, restarts from checkpoint.

**Examples:**
- Context window critically low during execution (POOR tier, >70%) — abort cleanly rather than produce truncated output.
- Critical dependency unavailable mid-run (network down, API key revoked).
- Unrecoverable filesystem state (disk full, permissions lost).
- Safety invariant violated (agent attempted an irreversible destructive action outside approved scope).

## How to use this in a skill

When you write an orchestration skill that has validation checkpoints, **name each checkpoint by its gate type explicitly** and answer three questions:

1. **What condition triggers this gate?** (e.g., "plan file missing", "issue count didn't decrease", "context >70%")
2. **What happens when it fails?** (block / loop back / ask human / abort)
3. **Who resumes, and from where?** (fix precondition + retry, revise + re-check, human decision, restart from checkpoint)

Answering these three up front means your skill never hits "what do we do now?" at runtime.

## Example — a review loop with all four gate types

```
[Pre-flight] task contract and disjoint ownership valid?  → no: fix decomposition before dispatch
                 ↓ yes
[Execute]    targeted worker implements bounded job
                 ↓
[Revision]   main agent verifies; optional reviewer adds evidence
                 ↓ findings
[Adjudicate] main agent rejects false positives and fixes confirmed issues
                 ↓
[Pre-flight] focused + integration gates pass?            → no: diagnose or escalate blocker
                 ↓ yes
[Deliver]    report verified result; commit only if requested

(on material redesign)
[Abort wave] preserve/classify diff → replan → fresh bounded job

(on unresolved product ambiguity)
[Escalation] present the decision and wait for the user

(on critical context pressure)
[Abort] checkpoint verified state and stop cleanly
```

The vocabulary is small on purpose. Every gate in every workflow should fit one of these four. If you find yourself inventing a fifth, it's probably a revision gate with extra branching, or an escalation gate in disguise.
