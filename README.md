# agent-skills

Personal repository of [Agent Skills](https://agentskills.io/) — the open-standard
skill format (a `SKILL.md` per directory) that works with GitHub Copilot (VS Code,
Copilot CLI, cloud agent), Claude Code, and other skills-compatible agents.

Designed to be **grabbed from machines that cannot run Hermes** — e.g. a
Copilot Enterprise-only workstation.

## Layout

One directory per skill, exactly like the [Agent Skills spec](https://agentskills.io/specification) requires:

```
agent-skills/
├── subagent-driven-development/
│   ├── SKILL.md          ← required: YAML frontmatter (name, description) + markdown body
│   └── references/       ← optional: on-demand playbooks linked from SKILL.md
├── requesting-code-review/
│   └── SKILL.md          ← optional independent-review escalation policy
└── <your-next-skill>/
    └── SKILL.md
```

## Installing on another machine

### With gh CLI (recommended)

```bash
gh skill install unsupportedpastels/agent-skills            # browse interactively
gh skill install unsupportedpastels/agent-skills subagent-driven-development   # install one skill
gh skill install unsupportedpastels/agent-skills requesting-code-review
```

`gh` places the skill in the correct directory for your agent host automatically.
Requires gh ≥ 2.90.

### Without gh (plain clone + copy)

```bash
git clone https://github.com/unsupportedpastels/agent-skills.git
# project scope (one repo):
cp -r agent-skills/subagent-driven-development .github/skills/
cp -r agent-skills/requesting-code-review .github/skills/
# personal scope (all projects):
cp -r agent-skills/subagent-driven-development ~/.copilot/skills/
cp -r agent-skills/requesting-code-review ~/.copilot/skills/
```

Or point VS Code's `chat.agentSkillsLocations` setting at the cloned checkout.

### Updating

```bash
gh skill update                 # interactive
gh skill update --all           # update everything
```

## Adding a new skill

1. Create `<skill-name>/SKILL.md` — frontmatter needs `name` (lowercase + hyphens,
   must match the directory name) and `description` (what it does **and** when to
   use it; Copilot matches on this).
2. Optionally add `references/`, `scripts/`, or `examples/` subdirectories and
   link them from `SKILL.md` with relative markdown paths.
3. Validate before pushing:

```bash
gh skill publish --dry-run      # validates against the agentskills.io spec
```

## Notes


- Skills carry their own attribution; see `LICENSE` for repo-level licensing.
