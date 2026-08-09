codefall-plugin
---------------

A plugin for Claude Code (and, later, Codex and friends).

Every skill in Codefall is a **verb**.

## Skills

| Skill | Does | Status |
| --- | --- | --- |
| [`scaffold`](skills/scaffold/SKILL.md) | Start a new project on the Clean + package-by-component stance: ratified ADRs, scoped `AGENTS.md`, optionally project files and boundary lint. | in progress |

Skills are **explicitly invoked** — `/scaffold`, `/specify`, and so on. Each carries
`disable-model-invocation: true`, so none of them fire on their own; scaffolding a project or filing
an issue is a deliberate act, not something inferred from a passing remark.

### The stance

**Pure Clean Architecture organized package-by-component**, with boundaries **mechanically
enforced**. Dependencies point inward only; the interfaces a use case needs live with the use case in
`application/`, not in `domain/`; the top level is capabilities, each behind a facade with the Clean
layers nested inside; a composition root per app binds implementations.

That core is stack-agnostic ([ADR-001, ADR-002](skills/scaffold/templates/adrs/)). Each surface adds
a profile supplying ADR-003 and up — for `typescript-react`, that's Inversify, the TanStack Query /
Zustand / `useState` split, and `eslint-plugin-boundaries`.

### Surfaces

Profiles are scoped to a **surface**, not to a kind of product — a project composes as many profiles
as it has surfaces. And a surface is defined by **where domain logic lives**, not by which languages
appear in the repo.

| Surface profile | Covers | Status |
| --- | --- | --- |
| `typescript-react` | TypeScript/Node backends; React frontends, web and Native — including Tauri, Electron, and RN apps whose native side is only wiring | **supported** |
| `rust-native` | Rust services, and Tauri shells that hold domain logic | planned |
| `kotlin-native` · `swift-native` | Android, iOS | planned |
| `dart-flutter` | Flutter, mobile and desktop | planned |
| `python` · `java` · `go` | backends and services | planned |

So a **Tauri desktop app and a React Native mobile app are both scaffoldable today**, whole, when
their native side is boilerplate plus a few commands wrapping OS APIs. Those commands are gateway
implementations living in the `infrastructure/` ring — per ADR-001, a Tauri `invoke`, a bridge call,
an HTTP request, and a platform channel are all just gateways, and the use case never learns which
one it got. Rust or Kotlin in the repo doesn't make it a native surface, any more than a Postgres
driver makes SQL one.

It's two surfaces only when the native side holds real domain logic. Then it needs its own profile
plus a **seam ADR** deciding which side owns the domain.

React is the component model; the renderer is an outer-ring detail, so React DOM and React Native
share one profile. What differs is toolchain — and the profile records the trap: Metro transpiles
with Babel, not `tsc`, so ADR-003's `emitDecoratorMetadata` is inert and Inversify fails at runtime
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

```
.claude-plugin/
  plugin.json         # plugin manifest
  marketplace.json    # lets this repo be added as a marketplace directly
skills/
  scaffold/
    SKILL.md
    templates/
      adrs/                     # stack-agnostic: ADR-001, ADR-002, _TEMPLATE
      surfaces/
        typescript-react/
          PROFILE.md            # fit, visibility model, toolchain, depth notes
          adrs/                 # ADR-003..005
          AGENTS.md.skeleton    # scoped per-surface rules doc
```

`skills/`, `commands/`, `agents/`, and `hooks/hooks.json` are auto-discovered by
Claude Code — no manifest entries needed when adding new ones.

## Install (local development)

```
/plugin marketplace add /Users/djensen/code/lividlabs/codefall-plugin
/plugin install codefall@codefall
```

Or, once pushed:

```
/plugin marketplace add lividlabs/codefall-plugin
/plugin install codefall@codefall
```

## License

[MIT](LICENSE) — Copyright (c) 2026 Livid Labs, LLC, authored by Dave Jensen.

The templates under `skills/scaffold/templates/`, and everything `scaffold` copies from them into
your project, are additionally available under [0BSD](LICENSE): no attribution, no notice, no
obligation. Your architecture documents are yours.
