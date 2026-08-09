---
name: scaffold
description: Start a new project on the Clean + package-by-component stance — interview for the calls a template can't make (bounded contexts, app topology, per-surface architecture), then emit ratified ADRs, scoped AGENTS.md files, and optionally the project files and boundary-lint wiring.
argument-hint: "[project-name] [path]"
disable-model-invocation: true
---

# Scaffold

Start a new project with the architecture decided, written down, and enforceable.

Scaffold's minimum output is **documentation** — the ADRs that fix the architecture and the scoped
`AGENTS.md` files that make them operative. Code is optional and additive on top of that.

Scaffolding is not implementing. Once the decisions are recorded and the project is green and empty,
stop. Features go through `specify` → `architect` → `implement`.

## The stance

**Pure Clean Architecture organized package-by-component**, with boundaries **mechanically
enforced**. That core is stack-agnostic and lives in `templates/adrs/` (ADR-001, ADR-002). Each
supported surface adds a profile under `templates/surfaces/<name>/` supplying ADR-003 and up. Read the
ADRs before scaffolding — they are the substance of this skill, not decoration.

In one paragraph: dependencies point inward only; the interfaces a use case needs live *with the use
case* in `application/`, not in `domain/`; the top level is capabilities, not layers, each behind a
facade with the Clean layers nested inside; a composition root per app binds implementations; and the
facades are enforced by tooling, wired on day one. The surface profile fills in *which* tooling.

Vocabulary matters and is deliberate. Say **gateway**, not "port" or "adapter". Interfaces are the
bare noun (`UserRepository`), implementations are qualified (`PostgresUserRepository`,
`DefaultClock`).

## Bundled templates

Stack-agnostic — every project gets these:

| Path | What it is |
| --- | --- |
| `templates/adrs/ADR-001-clean-architecture.md` | The dependency rule and the four layers |
| `templates/adrs/ADR-002-package-by-component.md` | Top-level organization; when to prefer p&a |
| `templates/adrs/_TEMPLATE.md` | Thin ADR template for new decisions |

Per surface, under `templates/surfaces/<name>/` — a `PROFILE.md`, an `AGENTS.md.skeleton`, and the ADRs
from 003 up. The `typescript-react` profile supplies DI (Inversify), frontend state (TanStack Query /
Zustand / `useState`), and boundary enforcement (`eslint-plugin-boundaries`).

These ship **Accepted**. They are this shop's decided architecture, not a menu — the project inherits
them. Stamp a real date on each. Amend one only when the interview requires it (below); if you amend,
edit the Context and Decision so the file reads as a decision *made for this project*, and say what
you changed in the final report.

## Surface catalog

Profiles are scoped to a **surface**, not to a kind of product. A project is a set of surfaces, and
it composes as many profiles as it has surfaces. "Desktop app" is not a profile — depending on where
its domain logic lives, it is either one web surface or a web surface plus a native one.

| Surface profile | Covers | Status |
| --- | --- | --- |
| `typescript-react` | TypeScript/Node backends; React frontends, web and Native — including Tauri, Electron, and RN apps whose native side is only wiring | **supported** |
| `rust-native` | Rust services, and Tauri shells that hold domain logic | planned |
| `kotlin-native` · `swift-native` | Android, iOS | planned |
| `dart-flutter` | Flutter, mobile and desktop | planned |
| `python` · `java` · `go` | backends and services | planned |

A profile is **supported** only when `templates/surfaces/<name>/PROFILE.md` is complete. Nothing else
counts — not a language this skill mentions, not one you know well, not one that is "basically the
same as" a supported profile.

Note what the first row does and doesn't claim. A Tauri desktop app or a React Native mobile app is
scaffoldable **today** when its native side is a thin shell; neither is when that side carries domain
logic. That distinction is a question for the user, not a guess from the directory listing — React
Native ships `android/` and `ios/` empty.

## Process

### 0. Describe and match — the gate

Ask the user to **describe the project in a few sentences**: what it does, what surfaces it has, what
it runs on. Then decompose it into surfaces and match each one against the catalog.

Do the decomposition explicitly — it is the step that decides everything downstream.

**A surface is defined by where domain logic lives, not by what languages are present.** A language
that only implements gateways is not a surface; it is the outer ring of a surface that already has a
profile. Ask of each part: *does this hold entities and use cases, or does it only reach out to
something on their behalf?*

This is what makes a **Tauri desktop app one surface, not two.** If `src-tauri/` is stock boilerplate
plus a handful of thin commands wrapping OS APIs — file dialogs, notifications, the tray — those
commands are gateway implementations. Per ADR-001 they belong in the `infrastructure/` ring of the
TypeScript surface, and the app is plain `typescript-react`. Rust being present doesn't make it a
Rust surface, any more than a Postgres driver makes SQL one. The same reasoning covers an Electron
main process, a React Native native module, and a thin native wrapper around a web view.

It becomes two surfaces only when the shell holds **real domain logic** — heavy compute, native
integrations with their own rules, security-sensitive work that must not live in the renderer. Then
Rust has entities and use cases of its own, needs its own profile, and there is a genuine seam.

So ask directly: *is there business logic in the shell, or is it just wiring?* Don't infer it from
the file listing.

A Flutter app with a Go API is two surfaces — both sides own domain logic. A plain Node service is
one.

**If every surface matches a supported profile**, name the profiles you matched and continue.

**If any surface does not, stop** — the project isn't scaffoldable, even if the other surfaces are.
Do not proceed. Specifically, do not:

- substitute the nearest supported profile for the unsupported surface;
- hand-author ADR-003/004/005 for an unsupported language from your own knowledge — the point of a
  profile is that those decisions were made deliberately, once, and reviewed;
- emit a partial scaffold and note the gap in passing.

Name which surfaces fit and which don't, say what's supported, and offer to record the request. A
clean refusal is the correct outcome — an improvised profile is worse than none, because it produces
ADRs marked Accepted that nobody actually decided.

If the user, having been told, explicitly asks for what *is* covered — the stack-agnostic core, or
the supported surfaces alone — that is theirs to choose. Emit it, and state plainly which surfaces
were left undecided and that their boundary-enforcement obligation is unmet.

### 0b. Seams — for composed projects only

Skip this when the decomposition found one surface. A thin-shell Tauri app has an IPC boundary but
not a seam in this sense: the domain lives in one place, and `invoke` is just a gateway
implementation like any HTTP client.

Two or more **domain-bearing** surfaces is a different matter, and profiles deliberately don't decide
the boundary. Where they meet, settle:

- **Which side owns the domain.** In a fat-shell Tauri app or a Flutter-plus-API project the entities
  can live on either side, or — badly — on both. Pick one and write it down.
- **What the gateway across the seam looks like.** Per ADR-001 this is just another gateway: an
  `invoke` command, an HTTP call, and a platform channel are the same shape to a use case, which
  never learns which one it got. The inner rings must not name the transport.

Record the answers as a project-specific ADR (ADR-006+, from `_TEMPLATE.md`). This is the one place
`scaffold` writes a genuinely new decision rather than instantiating a template — so draft it, then
have the user confirm it before writing.

### 1. Interview

The templates deliberately don't decide three things — they are per-project calls, and they change
what gets emitted. Ask; do not guess. Batch the questions rather than interrogating one at a time,
and offer a recommendation with each.

1. **App topology** — how many apps, and monorepo or single? Surfaces are already known from step 0;
   this is how they're laid out on disk. Drives directory layout and, per ADR-003, how many
   composition roots exist — and therefore how many `AGENTS.md` files.
2. **Bounded contexts** — what are the capabilities? These become the top-level component folders
   and make the structure scream the domain. Push for **coarse** boundaries: ADR-002 says start
   coarse and split, because re-slicing an existing boundary is the expensive case. Two or three is
   a fine start. If the user genuinely doesn't know yet, that is itself the answer — see #3.
3. **Per-surface architecture** — package-by-component (the default) or ports-and-adapters?
   ADR-002 names three conditions favoring p&a: boundaries genuinely unknown, one cohesive domain
   rather than separable capabilities, or no service-extraction goal. If the answer to #2 was "don't
   know", recommend p&a for that surface and amend ADR-002 accordingly.
4. **Depth** — docs only, docs + project files, or a runnable skeleton? Default to **docs only**
   unless the user wants more. See [Depth](#depth).

Also settle the project name and destination. Never scaffold into a non-empty directory without
saying so first, and never overwrite an existing path.

### 2. Emit the docs — always

- `docs/adrs/` — the shared ADRs (001, 002) plus every ADR the matched profiles supply, plus
  `_TEMPLATE.md`. Dated and Accepted, amended per the interview. Drop ADRs that don't apply — a
  backend-only project has no use for `typescript-react`'s ADR-004.
- `docs/decision-log.md` — the in-flight scratchpad, with `Locked` / `Open` / `Parking lot`
  sections. Seed `Open` with anything the interview surfaced but didn't settle.
- A scoped `AGENTS.md` per app/package, from the skeleton: fill the name, the one-line description,
  fix the ADR links to the right relative path, delete the `<frontend only>` blocks on backend
  surfaces, and fill in Gotchas. Keep it terse — it links the ADRs and never restates the why.

If the harness in use reads `CLAUDE.md` rather than `AGENTS.md`, add `CLAUDE.md` as a one-line
pointer to `AGENTS.md`. Do not maintain two copies of the rules.

### 3. Emit code — per the depth answer

#### Depth

**Docs only** (default). The decision layer, nothing else. Correct when dropping the stance into an
existing project, or when the project's shape isn't settled enough for code to be anything but
guesswork.

**Docs + project files.** The root scaffolding a project needs before any real code: manifest, build
config, linter **including the boundary rules**, formatter, test runner, and CI that runs all of it.

**Runnable skeleton.** The component folders from the interview, each with its facade and nested
`domain/` `application/` `infrastructure/` `presentation/`, plus one composition root per app that
starts the app end to end with no features in it.

The concrete file list for both tiers is the profile's business — see its **Depth notes**. Follow the
profile rather than reaching for what you'd reflexively pick for the language.

#### The ADR-005 obligation

ADR-005 says boundary enforcement is wired **at the project scaffold, day one** — deferring defeats
the purpose, and without it the package-by-component choice is cosmetic. Docs-only output cannot
satisfy that. When emitting docs only, say so plainly in the report and name it as the first task
the user owes the project.

The boundary config is the encoded architecture. Whenever you write it, it enforces all five ADR-005
rules — facade-only imports, inward-only layers, cross-component via facades only, shared modules
can't import components, and inner rings can't touch the UI framework or gateways.

How much tooling this takes depends on the language's visibility model, which is why ADR-005 is
per-profile: TypeScript needs a linter because it has no `package-private`, while Kotlin `internal`,
Rust `pub(crate)`, and Go's unexported identifiers do part of the job in the compiler.

### 4. Verify

Whatever you emitted must actually work. If there are project files, run install, lint, and test —
and confirm the boundary rules **fail on a deliberate violation**, because a lint config that
catches nothing is the common failure here. Do not assume; run it. If there are only docs, check
that every ADR cross-link and every `AGENTS.md` link resolves.

### 5. Report

- The surfaces identified and the profile matched to each.
- The interview answers, as the decisions now recorded.
- Any ADR amended, and what changed.
- Files created.
- What the user still owes the project — always including boundary enforcement if it isn't wired.

## Conventions

- Skills in this plugin are named as **verbs** (`scaffold`, not `scaffolder`).
- One skill per directory: `skills/<verb>/SKILL.md`.
- The doc workflow this seeds: **discuss → decision-log → ADR → scoped AGENTS.md → code.** The
  decision-log holds detail while a decision is moving; the ADR holds the settled decision *and its
  why*; `AGENTS.md` holds the terse operative rules and links back. `architect` picks up from here.
