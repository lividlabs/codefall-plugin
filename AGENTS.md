# codefall-plugin — operative rules

Terse on purpose: the why lives in the linked docs. This is the same convention the plugin
installs into scaffolded projects — an `AGENTS.md` holds the operative rules and links back, and
never restates the reasoning.

## ADRs are history

- A ratified ADR is never rewritten — not in a user's project, not in this repo. A revision lands
  as a **new, superseding ADR**; the only in-place edit ever made to an existing ADR is flipping
  its Status line to `Superseded by <id> — <date>`.
- Successor naming: the current template's name when a rename is involved, an edition suffix
  (`ADR-BASE-02.2`) when it isn't. Mechanics: `plugins/codefall/skills/graft/SKILL.md`, step 5.
- This binds every verb that touches ADRs — `graft` today, `architect` and the rest as they land.

## Skills

- Named as **verbs** (`scaffold`, `graft`), one directory each:
  `plugins/codefall/skills/<verb>/SKILL.md`.
- Every skill carries `disable-model-invocation: true`. Running one is a deliberate act.
  The field is Claude Code's; harnesses that read the skills through `.agents/skills/`
  ignore it and may invoke them on their own initiative.
- A skill reports and offers; it applies only what the user takes. Nothing lands unrequested.
- Never present an option that would be refused — unsupported stacks and planned profiles are
  exits, not menu choices. See the stack question in `scaffold`'s SKILL.md.
- Renaming, moving, or retiring a template ships a row in
  `plugins/codefall/skills/graft/lineage.md`, in the same PR. `graft` can only tell a rename from
  a deletion plus an addition because that record exists.
- A new skill ships its `.agents/skills/<verb>` symlink in the same PR. The links point inward
  at `plugins/codefall/skills/<verb>`, never the reverse — the plugin publishes as a
  `git-subdir` rooted at `plugins/codefall`, so a link escaping that subtree dangles for
  every install. See `.agents/skills/README.md`.
- New skill prose matches the established register — declarative, reasons attached, refusals
  stated plainly. Read `scaffold`'s SKILL.md end to end before writing one.

## Workflow

- All work happens on a branch or worktree, never directly on `main` — a ruleset forbids pushing
  it anyway. Every change lands through a PR.
- Stacked branches are fine, and the right way to break large work into smaller reviewable
  pieces. Check `gh pr list` before basing new work.
- Conventional commits, with a body that says *why*. release-please reads the types
  (`release-please-config.json` maps them to changelog sections).
- release-please owns `CHANGELOG.md` and the `version` in
  `plugins/codefall/.claude-plugin/plugin.json`. Hand-editing either lies to the release process.

## Testing a skill locally

- Add this repo as a marketplace from its local path and install `codefall-dev` — it runs from
  the working tree (see `.claude-plugin/marketplace.json`). The `codefall` entry serves only the
  released commit it pins, so real installs never see unreleased work.
- Outside Claude Code, a harness that reads `.agents/skills/` picks both skills up straight
  from the checkout — those symlinks resolve into the working tree, so there is nothing to
  install.
