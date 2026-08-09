---
name: requesting-code-review
description: "Use when independent pre-commit review is explicitly warranted by user request, formal policy, or unresolved high-consequence uncertainty; not for routine delegated work."
license: MIT
---

# Independent Pre-Commit Review

## Purpose

Provide a fresh, read-only review when independent evidence is explicitly useful. This is an **optional escalation**, not a routine tax on every implementation.

The main agent always inspects and verifies delegated work. Use this skill only when a separate reviewer has a concrete reason to exist.

## Use When

- The user explicitly asks for an independent review.
- A security, compliance, release, or repository rule requires one.
- A high-consequence change leaves a specific uncertainty that main-agent inspection and deterministic tests cannot close.
- A complex integrated diff benefits from one fresh-context challenge before delivery.

## Do Not Use Merely Because

- A subagent implemented the change.
- Two or more files changed.
- The user said commit, push, ship, done, or verify.
- A delegated task or wave completed.
- The change is labeled moderate/high risk but direct evidence establishes correctness.
- A prior reviewer offered only suggestions.

Routine path: inspect diff → run affected tests/gates → adjudicate → fix confirmed defects directly → deliver.

## Procedure

### 1. Establish the review boundary

Review the final integrated diff, not every worker's intermediate diff. Determine the correct source:

```bash
git status --short
git diff --check
git diff
```

Use `git diff --cached` when the intended review boundary is staged. Use an explicit commit range for committed work. Never guess the boundary.

If the diff is large, give the reviewer repository paths and the exact range rather than pasting a huge diff into the prompt.

### 2. Run authoritative checks first

Before dispatching, the main agent runs focused tests and project-required checks. The reviewer should challenge semantic correctness, not substitute for builds or tests.

Perform targeted static/security inspection appropriate to the language and changed surface. Do not run generic grep cargo-cult scans that produce untriaged noise.

### 3. Dispatch one combined read-only reviewer

Use at most one reviewer by default. Brief it with:

- User-visible intent and non-goals.
- Exact diff boundary and repository path.
- Relevant invariants and risk trigger.
- Tests/checks already run and their real results.
- Specific uncertainty the independent review should challenge.
- Read-only constraint: no edits, staging, commits, or side effects.

Ask for structured findings separated into:

- Concrete security concerns.
- Concrete logic/correctness defects.
- Non-blocking suggestions.
- Evidence: path, symbol/line, violated invariant, and failure scenario.

A reviewer conclusion is evidence, not truth.

### 4. Adjudicate every finding

The main agent classifies each item:

- **Confirmed** — supported by code, spec, invariant, reproduction, or test.
- **False positive** — contradicted by repository evidence.
- **Out of scope** — valid but not required for this change.
- **Decision required** — materially different valid product interpretations.

Do not implement suggestions solely because a reviewer listed them. Do not report a reviewer “pass” as stronger evidence than the actual diff and executed checks.

### 5. Correct confirmed defects

For localized fixes, the main agent performs the correction directly and adds or strengthens regression coverage when appropriate. Then rerun focused tests and affected project gates.

Do not spawn a third fix agent by default. Delegate again only when the finding requires a substantial newly bounded implementation slice that genuinely saves main-model tokens.

### 6. Re-review only when justified

Do not automatically re-review ordinary fixes. Re-review only when:

- A formal process requires it.
- The correction materially redesigns the solution.
- The original uncertainty remains unresolved after direct inspection and tests.

### 7. Commit only when requested

Independent review does not authorize an automatic commit and does not require a `[verified]` commit prefix. Preserve the repository's normal commit conventions.

## Integration with Subagent-Driven Development

`subagent-driven-development` already requires main-agent diff inspection, adjudication, correction, and final verification. This skill adds an independent reviewer only when its trigger is documented. It must never be invoked after every worker or every multi-file change by default.

## Completion Checklist

- [ ] A concrete independent-review trigger was documented.
- [ ] The final integrated diff boundary was explicit.
- [ ] Focused tests and required gates ran before review.
- [ ] One read-only reviewer received intent, invariants, risk, and evidence.
- [ ] Every finding was adjudicated rather than accepted automatically.
- [ ] Confirmed defects were fixed directly unless substantial redelegation was justified.
- [ ] Re-review occurred only for a formal gate, material redesign, or unresolved uncertainty.
- [ ] No automatic commit or special commit prefix was added.
