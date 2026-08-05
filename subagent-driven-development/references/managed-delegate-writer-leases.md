# Managed delegate writer leases

## Why process checks are insufficient

A `delegate_task` dispatch grants a logical writer lease. The managed worker may not expose the repository path in its local command line, may disappear temporarily from the UI, and may deliver filesystem writes or a completion callback later. Therefore none of these releases the lease:

- empty Agents panel / live-agent registry;
- no matching process from `psutil`;
- no expected files yet;
- two unchanged Git snapshots;
- a complete-looking file set;
- repeated user/keepalive messages asking to continue.

Using those signals to launch a replacement can create two serialized-but-overlapping writers. A delayed original may then overwrite files after controller verification or even after a commit.

## Correct recovery protocol

1. Record the delegation ID, accepted status, mode, allowlist, and dispatch time.
2. Treat the lease as active until the delegation runtime emits **completion, failure, timeout, or cancellation**, or the user explicitly cancels it.
3. While unresolved, remain read-only. One downstream planning step is useful; repeated Git/process snapshots are status polling, not progress.
4. After the terminal event, establish quiescence with two Git/filesystem snapshots separated by a short interval.
5. Mechanically enforce the allowlist and re-read every changed integration boundary.
6. Run focused tests, affected prior-task regressions, and required full-suite/static checks. Commit only when requested or required by the active workflow.
7. If any late callback or sibling-modification warning arrives during verification, stop, re-establish quiescence, re-read owned files, and rerun the complete gate.
8. Treat callbacks received after the corresponding work was already reconciled as stale claims; never reapply them blindly.

## If runtime status is unavailable

Do not infer abandonment from OS/process evidence. Report an unresolved control-plane lease and wait for an authoritative terminal event or explicit cancellation. If forward progress is urgent, ask the user to cancel the missing delegate before replacement.

## Controller repair boundary

After a valid handoff, the main agent owns review adjudication, all confirmed post-review corrections, integration fixes, regression coverage, and final verification. It must not edit while a worker lease is unresolved. If the original implementation remains fundamentally incomplete or requires redesign, preserve and classify the partial diff, replan the work, and dispatch a fresh bounded implementation job rather than silently absorbing an unbounded rewrite. Commit only when requested or required, and preserve unrelated user state.