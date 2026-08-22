# codefall skills, for `.agents` harnesses

The entries beside this file are **symlinks**. The skills themselves live at
`plugins/codefall/skills/<verb>/`, which is the only canonical location — that path is what the
Claude Code plugin ships, and editing a skill means editing it there.

The links point inward, never outward. `.claude-plugin/marketplace.json` publishes the plugin as a
`git-subdir` rooted at `plugins/codefall`, so an installed user receives that subtree and nothing
else. A symlink placed inside `plugins/codefall` and aimed back at this directory would dangle for
every real install.

## Adding a skill

A new skill ships its symlink here in the same PR that adds it:

```sh
ln -s ../../plugins/codefall/skills/<verb> .agents/skills/<verb>
```

## What this is, and is not

It makes the skills discoverable to any harness that reads `.agents/skills/` from a checkout of
this repo — that is how a contributor dogfoods `scaffold` and `graft` outside Claude Code. It is
not a distribution channel. `git archive` and the GitHub tarballs preserve a symlink as a symlink,
and the target sits outside whatever path a skills CLI would extract, so a link copied out of this
repo resolves to nothing.

`disable-model-invocation: true` is a Claude Code field. Harnesses reading these skills through
`.agents/` ignore it, so they may invoke `scaffold` and `graft` on their own initiative. Claude
Code still requires a deliberate invocation.
