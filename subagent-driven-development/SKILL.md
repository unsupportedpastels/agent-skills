---
name: subagent-driven-development
description: "Use for controller-led coding with focused implementation subagents. Load when orchestrating delegated coding work: partitioning bounded jobs, dispatching parallel workers, integrating results, verifying diffs, and adjudicating review."
license: MIT
---

# Subagent-Driven Development

## Overview

Use the main agent as the **orchestrator and adjudicator** and subagents as narrowly targeted implementation workers.

The main agent owns repository discovery, decomposition, dependency routing, integration judgment, review adjudication, corrective edits, verification, and delivery. Each worker owns one bounded job with an explicit source-and-test allowlist and enough task context to finish without wandering into adjacent work.

**Core principle:** workers add focused implementation capacity; they do not replace the main agent's architectural context or decision authority. Worker summaries and reviewer findings are evidence, never proof or automatic decisions.

## When to Use

Use this skill when coding work can be divided into bounded jobs and fresh worker context improves focus or throughput. Keep the main agent in control of integration and verification.

Do not delegate work that is cheaper to do directly, depends on unresolved user/product decisions, overlaps an active worker, or primarily consists of shared-file integration or architectural adjudication.

## Role Contract

### Main agent: orchestrator and adjudicator

The main agent owns:

1. Repository inspection, dependency tracing, and sidecar requirements.
2. Worker objectives, dependency-closed allowlists, non-goals, acceptance criteria, and focused tests.
3. Parallelism decisions, writer-lease tracking, and serialized shared integration.
4. Diff inspection and verification from the stable main workspace.
5. Review adjudication against the spec and repository invariants.
6. Confirmed post-review corrections and regression coverage; the main agent has broader context and, in this workflow, the more capable model.
7. Final project gates, changed-path accounting, todo state, and delivery.
8. Commits only when requested or required by the active workflow.

### Subagents: targeted implementation workers

Each leaf worker receives one outcome that fits its context and timeout, with:

- A dependency-closed source-and-test allowlist.
- Full task semantics: required behavior, invariants, verification, non-goals, and stop condition.
- Paths—not copied bodies—for large source or reference files.
- A handoff requiring changed paths, commands/results, assumptions, and unresolved concerns.
- No authority to broaden scope, adjudicate peers, integrate shared work, or commit unless explicitly requested.

If a brief requires broad rediscovery, many shared files, unrelated suites, or architectural decisions, split it further or keep it main-agent-owned.

## Pre-flight Gate

Before dispatching a wave, the main agent verifies:

- The working tree and branch are understood; pre-existing user changes are identified and preserved.
- Relevant project instructions and matching skills are loaded.
- Required files, symbols, imports, dependencies, and test commands were inspected rather than guessed.
- Every job has an allowlist and a checkable finish line.
- Parallel jobs are mechanically disjoint.
- Delegate provider/model/reasoning settings match any user-specified configuration. Configuration changes affect only newly dispatched workers.
- Todos exist, and jobs being dispatched are marked `in_progress`.

If a precondition fails, resolve it before dispatch. Do not create partial worker ownership while the plan is still ambiguous.

## Decomposition and Parallelism

### Dependency-closed jobs

A narrow allowlist is a containment boundary, not permission to omit known production requirements. Before freezing ownership, trace:

- Production definitions and all direct call sites.
- Dedicated tests, fixtures, snapshots, and test setup.
- Schemas, migrations, grants, manifests, lockfiles, generated contracts, exports, and documentation required by project rules.
- Prior-task invariants in shared helpers.

Either include a required dependency in one worker's ownership or reserve it explicitly for serialized main-agent integration.

### Concurrent workers

Multiple implementation workers may run concurrently only when their complete ownership is disjoint—not merely when test filenames differ. Check for overlap in:

- Production files and symbols.
- Tests, fixtures, snapshots, mocks, setup, and golden files.
- Schemas, migrations, manifests, lockfiles, generated outputs, exports, and registries.
- Shared integration modules or monolithic files.
- Databases, ports, containers, test accounts, snapshots, or other mutable runtime resources.

Dispatch independent jobs together in one bounded batch, capped by the runtime concurrency limit. If any ownership cannot be made mechanically disjoint, serialize it. Never manufacture parallelism by assigning two workers different regions of the same shared file.

## Worker Brief Template

Every implementation brief should include:

```text
OBJECTIVE
One concrete outcome.

OWNED PATHS
Exact dependency-closed source and test allowlist.

REQUIRED BEHAVIOR
Observable acceptance criteria and repository invariants.

CONTEXT
Where the change fits, inspected symbols, dependencies, and paths to relevant large files.

NON-GOALS
Adjacent behavior and files the worker must not change.

WORKFLOW
Use focused TDD when appropriate: establish a meaningful failure, implement, then verify.

VERIFICATION
Exact focused commands. Do not run the full repository suite unless this job uniquely requires it.

HANDOFF
Report changed paths, commands and results, assumptions, and unresolved concerns. Do not commit.
Stop after the bounded job; do not perform shared integration or unrelated cleanup.
```

A worker report is a claim. The repository diff and main-agent execution results are authoritative.

## Implementation Wave

For each dependency-ready wave:

1. **Partition:** define one or more dependency-closed jobs and reserve shared integration for the main agent.
2. **Dispatch:** mark todos in progress and dispatch one worker or one disjoint parallel batch.
3. **Protect leases:** while workers are active, do not edit, test, stage, or commit overlapping files or mutable runtime resources. Use the interval for non-overlapping read-only orchestration.
4. **Reconcile:** after authoritative completion, inspect actual changed paths before accepting the summary. Stop on scope drift or concurrency signals.
5. **Verify workers:** read each diff, enforce its allowlist, and rerun its focused tests from the main workspace.
6. **Integrate:** apply shared/cross-cutting edits serially under main-agent ownership.
7. **Review by risk:** use the policy below; do not launch reviewers by habit.
8. **Adjudicate and correct:** the main agent resolves confirmed findings and adds regression coverage.
9. **Run combined gates:** execute affected integration suites, type/lint/static checks, build, and full tests required by the project.
10. **Close:** account for every changed path, update todos immediately, and commit only if requested.

With a standing goal, continue into the next dependency-ready wave after verification. Stop only for a real blocker, explicit human/external gate, unresolved worker lease, or completion.

## Risk-Based Review Policy

Main-agent diff inspection and verification are always mandatory. An **independent reviewer** is additional evidence and should be selected by risk.

| Risk | Typical characteristics | Independent review |
| --- | --- | --- |
| Low | Localized behavior; one bounded module; no persistent-data, security, concurrency, or public-contract impact; strong deterministic regression tests | None by default |
| Moderate | Several interacting modules; client/server boundary; meaningful public behavior; performance or lifecycle logic; tests leave semantic uncertainty | One combined spec-and-quality reviewer after integration when the main agent cannot fully establish correctness from code and tests |
| High | Authentication/authorization/session/secret handling; migrations or destructive data operations; lease fencing/concurrency/transactions; security or outbound-network boundaries; risk/SLA policy semantics; silent corruption/data-loss potential; compatibility-sensitive public interfaces | One mandatory combined read-only reviewer after integration |
| Formal gate | User, security plan, compliance process, or release procedure explicitly requires independent review | Follow the named gate; add re-review only when it requires it |

When uncertain between tiers, choose the higher tier. Explain the risk trigger in the reviewer brief so the review targets the actual failure modes.

### Review budget

- Use at most **one combined independent reviewer per implementation wave** by default.
- Do not launch separate spec and quality reviewers for every worker.
- Review the integrated diff at the highest useful boundary rather than reviewing every intermediate boundary.
- A reviewer is read-only and reports evidence; it does not edit, commit, or decide acceptance.
- Re-review only when a formal gate requires it, a correction materially changes the design, or unresolved uncertainty cannot be closed by direct inspection and tests.

## Review Adjudication and Corrections

For every reviewer finding, the main agent classifies it as:

- **Confirmed:** supported by the spec, code, invariant, reproduction, or test.
- **False positive:** contradicted by repository evidence.
- **Out of scope:** valid observation but not required for this change; report rather than silently expanding scope.
- **Decision required:** multiple valid product interpretations materially change the implementation; ask the user.

For confirmed findings:

1. The main agent applies the correction directly.
2. Add or strengthen regression coverage where appropriate.
3. Rerun focused tests, affected prior-task tests, and required integration gates.
4. Do not send ordinary review feedback back to the original implementation worker. Keeping workers single-purpose avoids context drift and uses the main agent's stronger model and broader context for reconciliation.

### Redesign escape hatch

If review reveals that the original architecture or decomposition is fundamentally wrong, do not turn the main agent's correction into an unbounded rewrite and do not bounce a growing feedback list to the original worker.

1. Stop acceptance of the affected wave.
2. Preserve and classify the current diff.
3. Reassess the design and dependency graph.
4. Keep architectural decisions and shared integration main-agent-owned.
5. Repartition any substantial new implementation into fresh bounded jobs.
6. Dispatch a new worker only for a newly defined implementation job, not as a continuation of the old review loop.
7. Re-run the complete acceptance gate on the integrated result.

Use this escape hatch for genuine redesign, not routine corrections.

## Testing Policy

### Worker tests

Workers run the smallest commands that prove their owned behavior:

- Dedicated unit or component tests.
- Directly affected neighboring tests.
- File/module-level syntax, type, or lint checks when available.
- A meaningful RED step before implementation when TDD is appropriate.

Workers should not independently run the entire repository test/build matrix unless their task uniquely requires it. Parallel full-suite runs waste time and can contend on shared resources.

### Main-agent tests

After reconciliation and integration, the main agent runs:

1. Every focused worker suite from the stable main workspace.
2. Prior-task regressions for reused shared modules.
3. Cross-module integration tests.
4. Project-required type, lint, static-analysis, build, and full-test gates.
5. Diff/static checks and any database-backed verification required by the change.

Do not claim completion from a worker's green tests alone.

## Failure and Recovery

A managed dispatch holds a logical writer lease until the runtime reports completion, failure, timeout, or cancellation, or the user explicitly cancels it. Missing UI entries, process searches, unchanged Git snapshots, or complete-looking files do not release that lease.

- For a timeout, missing worker, late callback, sibling-modification warning, or uncertain writer ownership, see [references/managed-delegate-writer-leases.md](references/managed-delegate-writer-leases.md) before taking over or dispatching a replacement.
- For an unrestricted external CLI worker, isolated worktree, Windows descendant process, or unexpected worker commit, see [references/external-cli-process-trees.md](references/external-cli-process-trees.md).
- Never run a replacement writer over an unresolved lease.
- After a valid handoff, the main agent may correct reviewed work and integrate it. If the initial implementation itself remains fundamentally incomplete, replan it into a fresh bounded job rather than silently absorbing broad implementation.

## Progressive-Disclosure References

These files hold the failure-mode playbooks. Read the file whose trigger matches the current situation (open the file when the trigger applies — do not preload them):

| Trigger | Reference |
| --- | --- |
| Large roadmap, many workers, large artifacts, or rising context pressure | [context-budget-discipline](references/context-budget-discipline.md) |
| Workflow needs explicit pre-flight, revision, escalation, or abort gates | [gates-taxonomy](references/gates-taxonomy.md) |
| Security/DAST pipeline, outbound traffic, bounded capture, or security artifact acceptance | [security-pipeline-orchestration](references/security-pipeline-orchestration.md) |
| Managed worker timeout, missing UI state, delayed callback/write, cancellation, or unresolved writer lease | [managed-delegate-writer-leases](references/managed-delegate-writer-leases.md) |
| Codex/Claude/OpenCode or another external CLI process, unrestricted reviewer, worktree isolation, or orphan descendant | [external-cli-process-trees](references/external-cli-process-trees.md) |

## Verification Checklist

Before final delivery, confirm:

- [ ] Every worker had one bounded job and an enforced source-and-test allowlist.
- [ ] Concurrent workers had no file, symbol, integration, or mutable-runtime overlap.
- [ ] Every worker lease ended authoritatively before reconciliation.
- [ ] The main agent inspected every changed path and reran focused tests.
- [ ] Shared integration was serialized under main-agent ownership.
- [ ] Review level matched the documented risk policy.
- [ ] Reviewer findings were adjudicated; confirmed fixes were main-agent-owned.
- [ ] Required integration, check, build, and test gates passed on the stable final state.
- [ ] Every changed path is explained and unrelated user state is preserved.
- [ ] Todos reflect the verified result.
- [ ] No commit was created unless requested or required.
