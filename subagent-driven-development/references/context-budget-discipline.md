# Context Budget Discipline

Practical rules for keeping orchestrator context lean when spawning subagents or reading large artifacts. Use these whenever you're running a multi-step agent loop that will consume significant context — plan execution, subagent orchestration, review pipelines, multi-file refactors.

Adapted from the GSD (Get Shit Done) project's context-budget reference — MIT © 2025 Lex Christopherson ([gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)).

## Universal rules

Every workflow that spawns agents or reads significant content must follow these:

1. **Do not duplicate auto-loaded definitions.** If the runtime already injects an agent or project definition, do not read and inline it again.
2. **Pass complete task semantics, not large file bodies.** Put the objective, acceptance criteria, invariants, allowlist, and non-goals directly in the worker brief; point to large source/reference files by path so the worker can read only what its job needs.
3. **Read depth scales with context window.** See the table below.
4. **Delegate bounded implementation; retain judgment centrally.** Workers perform focused jobs. The main agent retains decomposition, review adjudication, corrections, integration, and final verification.
5. **Checkpoint before degradation.** When context pressure becomes material, summarize verified state, remaining dependencies, active leases, and next gates before continuing.

## Read depth by context window

Check the model's actual context window (not "it's Claude so 200K"). Some Sonnet deployments are 1M, some are 200K. If you don't know, assume the smaller one — err toward leanness.

| Context window | Subagent output reading | Summary files | Verification files | Plans for other phases |
|----------------|-------------------------|---------------|--------------------|-----------------------|
| < 500k (e.g. 200k) | Frontmatter only | Frontmatter only | Frontmatter only | Current phase only |
| >= 500k (1M models) | Full body permitted | Full body permitted | Full body permitted | Current phase only |

"Frontmatter only" means: read enough to see the final status/verdict/conclusion. If the subagent wrote a 3000-line debug log, read the summary section it produced, not the log.

## Four-tier degradation model

Monitor your context usage and shift behavior as you climb the tiers. The point is to notice *before* you hit the wall, not when responses start truncating.

| Tier | Usage | Behavior |
|------|-------|----------|
| **PEAK** | 0 – 30% | Full operations. Read bodies, spawn multiple agents in parallel, inline results freely. |
| **GOOD** | 30 – 50% | Normal operations. Prefer frontmatter reads. Delegate aggressively. |
| **DEGRADING** | 50 – 70% | Economize. Frontmatter-only reads, minimal inlining, **warn the user** about budget. |
| **POOR** | 70%+ | Emergency mode. **Checkpoint progress immediately.** No new reads unless critical. Finish the current task and stop cleanly. |

## Early warning signs (before panic thresholds fire)

Quality degrades *gradually* before hard limits hit. Watch for these:

- **Silent partial completion.** Subagent claims done but implementation is incomplete. Self-checks catch file existence, not semantic completeness. Always verify subagent output against the plan's must-haves, not just "did a file appear?"
- **Increasing vagueness.** Agent starts using phrases like "appropriate handling" or "standard patterns" instead of specific code. This is context pressure showing up before budget warnings fire.
- **Skipped protocol steps.** Agent omits steps it would normally follow. If success criteria has 8 items and the report covers 5, suspect context pressure, not "the agent decided 5 was enough."

When these signs appear, checkpoint the work and either reset context or hand off to a fresh subagent.

## Semantic verification boundary

A worker summary establishes neither structural nor semantic correctness. The main agent must inspect the actual diff, execute the relevant behavior, and compare the result with explicit must-have truths. An independent reviewer can add evidence at a risk-based gate, but it does not replace main-agent adjudication or execution.

**Mitigation:** put concrete must-haves in every worker brief and require the handoff to name changed paths, commands, results, assumptions, and unresolved concerns. Read detailed logs only when verification fails or a claim needs evidence.
