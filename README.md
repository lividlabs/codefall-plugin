codefall-plugin
---------------

**Codefall — Opinionated skills for the software development lifecycle.**

A plugin for Claude Code (and, later, Codex and friends). Every skill is a **verb**.

## Skills

| Skill | Does | Status |
| --- | --- | --- |
| [`conceptualize`](plugins/codefall/skills/conceptualize/SKILL.md) | Get an idea onto paper before anyone specifies or scaffolds it: a numbered concept document under `docs/concepts/` that carries the problem, the rough shape of an answer, and what nobody has decided yet. | in progress |
| [`scaffold`](plugins/codefall/skills/scaffold/SKILL.md) | Start a new project on the Clean + package-by-component stance: ratified ADRs, scoped `AGENTS.md`, optionally project files and boundary lint. | in progress |
| [`graft`](plugins/codefall/skills/graft/SKILL.md) | Bring a scaffolded project's docs up to date with the current templates: report what changed since its version, with per-file provenance, and apply only what the user takes. Also handles first-time adoption of the stance. | in progress |
| [`specify`](plugins/codefall/skills/specify/SKILL.md) | Turn a feature idea into a specification another session can implement: user stories with numbered, testable acceptance criteria, written into the issue tracker. | in progress |

### Concepts

`conceptualize` writes the *why* down first — the problem, who feels it, the rough shape of an answer,
and the open questions — as `docs/concepts/CONCEPT-001-slug.md`. It is deliberately informal, and its
length is proportional to what you put in: two paragraphs is a valid concept, and so is a page.
Unknowns stay in the document as unknowns rather than being invented away.

**`scaffold` requires a concept**, and offers to run this skill when there isn't one. That requirement
exists because scaffolding without any idea of what is being built is where scaffolds go wrong — the
architecture questions get answered by defaults picked from a one-sentence description. With a concept
in hand most of those questions are already answered, so the scaffold session is shorter *and* the
answers are better. You can decline, and the scaffold records that it ran without one.

`specify` may draw on a concept and never requires one, because a concept carries the *why* and a
specification carries the *what* — they are different documents, and plenty of features need only the
second.

A concept is `Draft` while you are still adding to it, `Ready` once it is written and agreed, and
`Active` once work starts against it. Replacing part of one adds a `Revised by` line; replacing it
whole archives it to `docs/concepts/archive/`, where the identifier stays valid and the citations still
resolve.

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

`scaffold` reads your concept, or asks you to describe the project when you declined one, decomposes
it into surfaces, and matches each against this table. **If any surface has no profile, it stops** — it won't improvise ADRs for an unsupported
language or scaffold only the half that fits. A profile counts as supported once
`templates/surfaces/<name>/PROFILE.md` is complete.

## Roadmap

The verbs chain: `conceptualize` frames the idea, `scaffold` makes the project, `specify` states the
problem, `mock-up` shows what it looks like, `design` decides the shape, `implement` writes it,
`review` checks it.

| Verb | Does |
| --- | --- |
| `mock-up` | Import a mockup exported from a design tool, or author one when there isn't one. Runs before or after `specify`; an issue labelled `requires-mockup` is blocked until it does. |
| `design` | Turn a specification into a technical design and a work breakdown; the tickets land in Beads, with their dependencies, as a graph. |
| `implement` | Write code for a tech spec. |
| `review` | Review specs or code. Eventually multi-harness. |
| `migrate` | Restructure code to a changed stance — architecture moves, component extraction. The code-refactor counterpart to `graft`, which moves only documents. |

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
    shared/
      import-mockup.md            # one import procedure, used by specify and mock-up
    skills/
      conceptualize/
        SKILL.md                    # the concept template and the status lifecycle
      graft/
        SKILL.md
        lineage.md                  # what every template used to be called; graft's rename record
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
      specify/
        SKILL.md
        trackers/
          github/
            PROFILE.md            # capabilities, field mapping, two-pass issue creation
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
