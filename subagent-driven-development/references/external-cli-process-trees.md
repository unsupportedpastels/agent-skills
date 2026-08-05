# External CLI Delegate Process Trees and Worktree Isolation

Use this when a subagent is implemented as an external CLI process (Codex, Claude Code, OpenCode, etc.), especially on Windows or with unrestricted execution such as `--yolo`.

## Enforcement boundary

A prompt saying “read only,” “do not spawn subagents,” or “do not commit” is not an OS-enforced boundary. An unrestricted reviewer can launch another agent, edit files, or commit. Do not rely on prose constraints to protect a shared worktree.

## Isolation rules

1. Give every writer an isolated worktree, disposable clone, or Git-backed staging copy.
2. Never run an unrestricted reviewer in the canonical writable worktree. Review a disposable snapshot or use an enforced read-only sandbox.
3. Serialize tasks touching the same monolithic file. Parallelize only disjoint files or isolated worktrees.
4. Record the known-good HEAD and worktree state before launch.
5. Merge or copy back only after parent-side tests and review.

## Windows process-tree caveat

Hermes may track a shell/wrapper while `codex.exe`, `codex-code-mode-host.exe`, Node, or PowerShell descendants continue after the wrapper is killed. If files keep changing, callbacks are stale, or a stopped delegate later commits:

1. Stop the tracked wrapper.
2. Inspect OS process command lines and parent PIDs for the unique task/prompt fragment.
3. Terminate only the matching descendant tree; never kill all agent processes on a shared machine.
4. Re-query to prove the tree is gone.
5. Inspect `git status`, `git log`, and changed files before cleanup.

Example Windows inspection:

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.Name -in @('codex.exe','codex-code-mode-host.exe') } |
  Select-Object ProcessId,ParentProcessId,Name,CommandLine
```

Terminate an identified tree with `taskkill /PID <pid> /T /F` only after matching its command line to the current task.

## Recovery and acceptance

- Restore only files whose agent provenance is established; never discard user changes.
- Treat unexpected commits/files as untrusted candidates, even when tests were claimed green.
- Re-run focused tests, full tests, syntax checks, and `git diff --check` after all descendants are stopped.
- A task is complete only when no matching descendant remains, the diff matches assigned ownership, repository history is explained, and parent-side verification passes on the stable snapshot.
