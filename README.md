codefall-plugin
---------------

**Codefall — Opinionated skills for the software development lifecycle.**

A plugin for Claude Code (and, later, Codex and friends). Every skill is a **verb**.

## Skills

| Skill | Does | Status |
| --- | --- | --- |
| [`scaffold`](plugins/codefall/skills/scaffold/SKILL.md) | Start a new project on the Clean + package-by-component stance: ratified ADRs, scoped `AGENTS.md`, optionally project files and boundary lint. | in progress |

Skills are **explicitly invoked** — `/scaffold`, `/specify`, and so on. Each carries
`disable-model-invocation: true`, so none of them fire on their own; scaffolding a project or filing
an issue is a deliberate act, not something inferred from a passing remark.

### The stance

**Pure Clean Architecture organized package-by-component**, with boundaries **mechanically
enforced** and components that stay **cheap to extract**. Dependencies point inward only; the
interfaces a use case needs live with the use case in `application/`, not in `domain/`; the top level
is capabilities, each behind a facade with the Clean layers nested inside; a composition root per app
binds implementations.

The output is a **monolith on purpose** — one deployable, no network between use cases. What it is
not is a monolith you are stuck with: each component owns its own data, no transaction spans two of
them, and what crosses a facade is a contract rather than an entity. Pulling a component out later is
a deployment change, not a redesign.

That core is stack-agnostic ([ADR-BASE-01 through ADR-BASE-03](plugins/codefall/skills/scaffold/templates/adrs/)) — Clean
Architecture, package-by-component, and keeping the resulting monolith cheap to split. Each surface adds
a profile supplying its own ADRs under its own prefix — `ADR-TS-01` and up for `typescript-react`,
which is Inversify, the TanStack Query / Zustand / `useState` split, and `eslint-plugin-boundaries`;
`ADR-GO-01` and up for `go`. Numbering restarts per profile, so two profiles never collide, and a
project's own ADRs are a separate sequence starting at `ADR-001`.

### Surfaces

Profiles are scoped to a **surface**, not to a kind of product — a project composes as many profiles
as it has surfaces. And a surface is defined by **where domain logic lives**, not by which languages
appear in the repo.

| Surface profile | Covers | Status |
| --- | --- | --- |
| `typescript-react` | TypeScript/Node backends; React frontends, web and Native — including Tauri, Electron, and RN apps whose native side is only wiring | **supported** |
| `go` | Go services, APIs, workers, daemons, and CLIs that hold domain logic | **supported** |
| `rust-native` | Rust services, and Tauri shells that hold domain logic | planned |
| `kotlin-native` · `swift-native` | Android, iOS | planned |
| `dart-flutter` | Flutter, mobile and desktop | planned |
| `python` · `java` | backends and services | planned |

So a **Tauri desktop app and a React Native mobile app are both scaffoldable today**, whole, when
their native side is boilerplate plus a few commands wrapping OS APIs. Those commands are gateway
implementations living in the `infrastructure/` ring — per ADR-BASE-01, a Tauri `invoke`, a bridge call,
an HTTP request, and a platform channel are all just gateways, and the use case never learns which
one it got. Rust or Kotlin in the repo doesn't make it a native surface, any more than a Postgres
driver makes SQL one.

It's two surfaces only when the native side holds real domain logic. Then it needs its own profile
plus a **seam ADR** deciding which side owns the domain.

React is the component model; the renderer is an outer-ring detail, so React DOM and React Native
share one profile. What differs is toolchain — and the profile records the trap: Metro transpiles
with Babel, not `tsc`, so ADR-TS-01's `emitDecoratorMetadata` is inert and Inversify fails at runtime
unless `babel-plugin-transform-typescript-metadata` is added.

`scaffold` asks you to describe the project, decomposes it into surfaces, and matches each against
this table. **If any surface has no profile, it stops** — it won't improvise ADRs for an unsupported
language or scaffold only the half that fits. A profile counts as supported once
`templates/surfaces/<name>/PROFILE.md` is complete.

## Roadmap

The verbs chain: `scaffold` makes the project, `specify` states the problem,
`architect` decides the shape, `implement` writes it, `review` checks it.

| Verb | Does |
| --- | --- |
| `specify` | Write a GitHub issue for a bug or feature. |
| `architect` | Write ADRs and tech specs, as GitHub issues or equivalent. |
| `implement` | Write code for a tech spec. |
| `review` | Review specs or code. Eventually multi-harness. |

## Layout

This repo is a **marketplace** at the root and a **plugin** in `plugins/codefall/`. The two are
versioned independently on purpose: the catalog stays current on `main` while the entry pins the
plugin to a released commit.

```
.claude-plugin/
  marketplace.json    # the catalog — pins plugins/codefall to a released commit
plugins/
  codefall/           # the plugin; this subtree is what gets installed
    .claude-plugin/
      plugin.json     # plugin manifest
    skills/
      scaffold/
        SKILL.md
        templates/
          adrs/                     # stack-agnostic: ADR-BASE-01..03, _TEMPLATE
          surfaces/
            typescript-react/
              PROFILE.md            # prefix, fit, visibility model, toolchain, depth notes
              adrs/                 # ADR-TS-01..03
              AGENTS.md.skeleton    # scoped per-surface rules doc
            go/
              PROFILE.md
              adrs/                 # ADR-GO-01..03
              AGENTS.md.skeleton
```

Within the plugin, `skills/`, `commands/`, `agents/`, and `hooks/hooks.json` are auto-discovered by
Claude Code — no manifest entries needed when adding new ones.

## Install

```
/plugin marketplace add lividlabs/codefall-plugin
/plugin install codefall@codefall
```

The catalog is read from `main`, so it is always current, but the `codefall` entry is pinned to the
commit of the most recent release. Merges to `main` do not reach installs; merging a release PR does.

### Local development

Point the marketplace at your checkout and install the `codefall-dev` entry, which resolves to
`plugins/codefall/` in your working tree rather than to a published commit:

```
/plugin marketplace add /path/to/codefall-plugin
/plugin install codefall-dev@codefall
```

Installing `codefall` from a local checkout would still fetch the pinned commit from GitHub, so use
`codefall-dev` when you want to see your edits.

## License

[MIT](LICENSE) — Copyright (c) 2026 Livid Labs, LLC, authored by Dave Jensen.

The templates under `plugins/codefall/skills/scaffold/templates/`, and everything `scaffold` copies from them into
your project, are additionally available under [0BSD](LICENSE): no attribution, no notice, no
obligation. Your architecture documents are yours.
