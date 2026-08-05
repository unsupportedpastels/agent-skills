# Security pipeline plan orchestration

Use this reference when executing a long security/DAST plan in a shared main worktree.

## Unit design

- Convert the plan into dependency-safe units; do not assign the entire roadmap to one orchestrator child.
- Prefer leaf delegates for pure modules/tests. Serialize every unit touching a shared scanner, policy, runner, fixture, reporting file, or Dockerfile.
- Every brief must include: exact owned files, objective, acceptance criteria, exact focused tests, explicit non-goals, traffic/publication gates, and an explicit no-commit policy unless the user requested worker-owned commits.
- Split oversized features along pure interfaces first (for example: template index/safety; fingerprint graph/ranking; platform profiles; serial scanner/image integration).

## Controller acceptance

1. Check actual Git paths against the allowlist before reading the completion summary as evidence.
2. Re-read modified files and rerun focused tests from the controller workspace.
3. Review security invariants independently: authorized origin, redirects, physical byte/request caps, secret retention, mutation gates, deterministic zero-record artifacts, and truthful partial/truncation state.
4. Re-run prior-task suites whenever a shared helper is reused. Watch for regressions such as whole-file reads replacing streaming, response buffering despite caps, or filtered unsafe inputs disappearing from accounting.
5. Restore out-of-scope child edits. If useful, reapply them explicitly as controller-owned work with regression tests.
6. Commit only when requested or required by the active workflow, and then stage only explicit verified paths; always preserve unrelated untracked files.

## Network helper contract

- Pass the engagement origin explicitly; never authorize each candidate against its own origin.
- Count attempts immediately before dispatch.
- Disable redirects and validate the final URL.
- Request streaming, consume at most `cap + 1`, and close in `finally`.
- Persist only hashes, counts, shape classes, structural names, and terminal reasons—not response bodies, variables, error messages, headers, cookies, or credentials.
- A logical row/event cap is not a physical byte cap; enforce both.

## Artifact finalization and acceptance contracts

For scanners that emit reports plus a checksum inventory, treat finalization order and acceptance propagation as security invariants rather than cosmetic implementation details:

1. Write every deterministic zero-record artifact and aggregate summary even when no findings exist.
2. Build fail-closed coverage from explicit terminal summaries, never from file presence or finding count. A supported, uncapped phase may be complete with zero records; missing, malformed, unsupported, safety-gated, or capped phases must remain truthful partial states.
3. Derive conditional proof requirements from a closed upstream classification taxonomy—not arbitrary labels or the presence of a proof file.
4. Write coverage and disposition artifacts before report compilation.
5. Generate the artifact inventory last. If an error path rewrites an inventoried summary after inventory generation, refresh the inventory afterward or reorder the summary write; add a regression that recalculates every recorded hash.
6. Review pure aggregate APIs for bounded persisted counts, non-promotion of control evidence (WAF/public/auth-required), and idempotent rendering of already-normalized data.
7. Trace every new coverage contract through the complete acceptance chain: phase summary → normalized coverage → canonical `coverage-state` → host validation/process exit. A separate truthful JSON artifact is insufficient if canonical acceptance can still return success. Add a regression where legacy coverage is complete but the new required coverage is partial, and require the final state/exit to fail closed.
8. Preserve state precedence during reconciliation: new partial coverage may downgrade canonical complete, but must not overwrite stronger blocked/failed reasons; explicitly non-applicable modes remain neutral.

## Delegate/runtime failure handling

- Timeout or missing summary means incomplete, even if files exist.
- Registry absence is not filesystem quiescence. Verify process state when warranted and take two stable Git/filesystem snapshots before controller edits.
- Treat sibling-modification warnings as active-concurrency signals; stop, re-read, and reconcile rather than overwriting.
- A delegate's green tests are not a security review. Parent-side adversarial review and focused regression tests remain mandatory.

## Human gates

Keep external target traffic and live skill publication as named escalation gates. Complete local implementation, pure tests, fixtures, and packaged-image validation first; stop and request renewed authorization at the gate.