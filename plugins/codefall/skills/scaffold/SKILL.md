---
name: scaffold
description: Start a new project on the Clean + package-by-component stance — interview for the calls a template can't make (bounded contexts, app topology, per-surface architecture), then emit ratified ADRs, scoped AGENTS.md files, and optionally the project files and boundary-lint wiring.
argument-hint: "[project-name] [path]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - AskUserQuestion
---

# Scaffold

Start a new project with the architecture decided, written down, and enforceable.

Scaffold's minimum output is **documentation** — the ADRs that fix the architecture and the scoped
`AGENTS.md` files that make them operative. Code is optional and additive on top of that.

Scaffolding is not implementing. Once the decisions are recorded and the project is green and empty,
stop. Features go through `specify` → `architect` → `implement`.

## Scope — how, not what

Scaffold decides **how this project will be built**. It does not decide **what it does**.

| In scope | Out of scope |
| --- | --- |
| Layering, the dependency rule, where interfaces live | What the features are, or how they work |
| DI, composition roots, naming | The data model, schemas, entity fields |
| Boundary enforcement, lint, CI | API design, endpoints, message shapes |
| Rendering strategy, topology, depth | Business rules, workflows, edge cases |

This is a hard line, and the most common way this skill goes wrong is crossing it. You are not
designing the application. Do not propose entities, sketch a schema, reason about how a feature will
behave, or ask questions whose only purpose is to understand the product. If the user describes what
they're building, that is context for matching a profile and judging shape — not an invitation to
design it.

**Not having named the domains yet is normal and fine**, and it is not the same as not having any.
A project can be scaffolded before anyone has picked names; step 1 judges whether the thing is one
cohesive domain or several separable capabilities, which is what actually chooses between
package-by-component and ports-and-adapters. Missing names decide nothing. Ask once, accept "not yet"
the first time it is said, and move on.

**The stack is yours; the stance is not.** Codefall has no opinion about which supported language or
framework a project uses and will not steer toward one — see the stack question. It does not bend on
the architecture. The ADRs ship Accepted because installing them is the entire point of this skill,
so a user who wants different layering, package-by-layer, or no boundary enforcement is asking for
something `scaffold` does not do. Say so plainly rather than compromising the stance to fit.

**Keep the session short.** Prefer defaults over questions, accept vague answers, and stop asking the
moment you have enough to emit the docs. A scaffold that takes four exchanges is working correctly.
If you find yourself on a long thread about how the thing will work, you are in `specify` and
`architect` territory — say so, and finish scaffolding.

Template paths in this document are relative to `${CLAUDE_PLUGIN_ROOT}/skills/scaffold/templates/`.
Resolve them against that root — they are not relative to the user's project.

## The stance

**Pure Clean Architecture organized package-by-component**, with boundaries **mechanically
enforced**, and **splittable** — a monolith whose components stay cheap to extract. That core is
stack-agnostic and lives in `templates/adrs/` (ADR-BASE-01 through ADR-BASE-03). Each
supported surface adds a profile under `templates/surfaces/<name>/` supplying its own ADRs. Read the
ADRs before scaffolding — they are the substance of this skill, not decoration.

In one paragraph: dependencies point inward only; the interfaces a use case needs live *with the use
case* in `application/`, not in `domain/`; the top level is capabilities, not layers, each behind a
facade with the Clean layers nested inside; a composition root per app binds implementations; the
facades are enforced by tooling, wired on day one; and components stay independently extractable —
each owns its data, no transaction spans two, and contracts cross facades rather than entities. The
surface profile fills in *which* tooling.

Vocabulary matters and is deliberate. Say **gateway**, not "port" or "adapter". Interfaces are the
bare noun (`UserRepository`), implementations are qualified (`PostgresUserRepository`,
`DefaultClock`).

## Bundled templates

Stack-agnostic — every project gets these:

| Path | What it is |
| --- | --- |
| `templates/adrs/ADR-BASE-01-clean-architecture.md` | The dependency rule and the four layers |
| `templates/adrs/ADR-BASE-02-package-by-component.md` | Top-level organization; when to prefer p&a |
| `templates/adrs/ADR-BASE-03-extraction-readiness.md` | Keeping components cheap to split out later |
| `templates/adrs/_TEMPLATE.md` | Thin ADR template for new decisions |

Per surface, under `templates/surfaces/<name>/` — a `PROFILE.md`, an `AGENTS.md.skeleton`, and the
profile's own ADRs. `typescript-react` supplies DI (Inversify), frontend state (TanStack Query /
Zustand / `useState`), and boundary enforcement (`eslint-plugin-boundaries`). `go` supplies DI
(`samber/do`), boundary enforcement (`internal/` packages plus `depguard`), and optional values
(`samber/mo`'s `Option`).

**ADR identifiers.** Three separate namespaces, and they never interact:

| Namespace | Who owns it | Examples |
| --- | --- | --- |
| `ADR-BASE-NN` | the stack-agnostic core in `templates/adrs/` | `ADR-BASE-01`, `ADR-BASE-02` |
| `ADR-<PREFIX>-NN` | a surface profile, prefix declared in its `PROFILE.md` | `ADR-TS-01`, `ADR-GO-02` |
| `ADR-NNN` | **this project's own** decisions, starting at `ADR-001` | `ADR-001` seam ADR |

Numbering restarts at 01 inside each profile, so nothing coordinates across profiles and a project
matching two of them gets `ADR-TS-01` and `ADR-GO-01` side by side with no collision. A profile that
has no use for a concern simply doesn't ship that ADR — there is no gap to explain.

**Project ADRs are a separate sequence starting at `ADR-001`**, never a continuation of the inherited
ones. That is deliberate: the three-digit bare form marks a decision made *here*, and the prefixed
forms mark stance inherited from Codefall. A reader can tell them apart at a glance.

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
| `go` | Go services, APIs, workers, daemons, and CLIs that hold domain logic | **supported** |
| `rust-native` | Rust services, and Tauri shells that hold domain logic | planned |
| `kotlin-native` · `swift-native` | Android, iOS | planned |
| `dart-flutter` | Flutter, mobile and desktop | planned |
| `python` · `java` | backends and services | planned |

A profile is **supported** only when `templates/surfaces/<name>/PROFILE.md` is complete. Nothing else
counts — not a language this skill mentions, not one you know well, not one that is "basically the
same as" a supported profile.

Note what the first row does and doesn't claim. A Tauri desktop app or a React Native mobile app is
scaffoldable **today** when its native side is a thin shell; neither is when that side carries domain
logic. That distinction is a question for the user, not a guess from the directory listing — React
Native ships `android/` and `ios/` empty.

## Process

### 1. Describe and match — the gate

Ask **one question**: what are you building, and what does it run on? A sentence or two.

That answer has to yield exactly **two** things, and nothing else learned here changes what gets
emitted:

1. **The surfaces**, so each can be matched to a profile.
2. **The shape** — whether this is *one cohesive domain* or *several separable capabilities*. This
   decides package-by-component versus ports-and-adapters in step 3, and it is answerable from a
   sentence without naming, defining, or designing anything.

This is the same question ADR-BASE-03 exists to keep answerable later — whether a capability could
be pulled out on its own. Asking it once at the start is cheaper than discovering the answer during
an extraction.

**Judging shape.** "A habit tracker where users log habits and see streaks" is one cohesive domain.
"An internal platform for billing, inventory, and shipping" is several separable capabilities. The
test is whether the thing decomposes into capabilities that could plausibly be owned, deployed, or
extracted separately — **not** whether the user has named them. Most greenfield projects have no
names *and* several capabilities; those are independent facts.

If the sentence genuinely doesn't say, ask once, flatly: *does this break into a few separate
capabilities, or is it one cohesive thing?* That is a question about shape, not about the product,
and it is the only follow-up this step is allowed.

**Most projects are not defined yet, and that is the normal case.** The user is starting something.
They do not owe you a product description, and a scaffold does not need one. Do not ask what the
features are, what the data looks like, who the users are, or how any of it behaves.

**When the user volunteers more than you asked for — and they will** — the failure to avoid is
*engaging* with it, not *hearing* it. Name the surfaces you found and carry on; that naming is the
acknowledgement, and it is in the only currency this skill trades in. A sentence of ordinary
acknowledgement alongside it is fine.

What is not fine: asking a follow-up about a feature, proposing entities or a schema, reasoning about
how something will behave, or letting product detail reach the ADRs. Those turn a description into a
design session, which is the most common way this skill fails.

If something in the description will genuinely matter to `specify` or `architect`, put it in the
decision-log's **Parking lot** and say you did. Recorded, not acted on.

Decompose the answer into surfaces, then confirm each one with the stack question below. Carry the
shape judgement forward to step 3 — it is an input to #3, not something to re-litigate there.

**Note any rendering signal without asking for one.** If the description mentions public pages,
sharing, browsing without an account, or search visibility, that decides a React surface's topology
in #1 below. Don't go looking for it — the question belongs there, and only if the description was
silent.

#### Decomposition

Do this explicitly — it is the step that decides everything downstream.

**A surface is defined by where domain logic lives, not by what languages are present.** A language
that only implements gateways is not a surface; it is the outer ring of a surface that already has a
profile. Ask of each part: *does this hold entities and use cases, or does it only reach out to
something on their behalf?*

This is what makes a **Tauri desktop app one surface, not two.** If `src-tauri/` is stock boilerplate
plus a handful of thin commands wrapping OS APIs — file dialogs, notifications, the tray — those
commands are gateway implementations. Per ADR-BASE-01 they belong in the `infrastructure/` ring of the
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

#### The stack question

**Name the profile the description already implies, and confirm it.** "A website" implies
`typescript-react`; "a CLI in Go" implies `go`. Say which you matched and give the user a plain way
to say otherwise.

**Do not present every supported profile as a co-equal menu.** Offering `go` beside
`typescript-react` for a website invites a choice the description already made, and implies the two
are equally indicated when they are not. Confirming one is faster and more honest than picking from
a list.

Ask with the full list **only when the description genuinely leaves it open** — "an API", "a
service", "a background worker" with no language named. Then there is a real choice, and it is:

- `typescript-react` — TypeScript/Node backends, React frontends (web and Native), and Tauri or
  Electron apps whose native side is only wiring
- `go` — Go services, APIs, workers, daemons, and CLIs
- **None of these**

That list is generated from the catalog. When a profile moves from planned to supported it gains an
option; until then it has none. The rule that matters is not *how many* supported profiles you show
— it is that **you never show one that isn't supported**.

**Never list a planned profile as an option.** `rust-native`, `python`, `dart-flutter` and the rest
are not available, so offering them and then refusing spends the user's choice on nothing. An option
annotated "this is unsupported, I would have to stop" is not a choice — it is a trap with extra
steps. This holds for every question in this skill, not just this one.

**"None of these" is the deliberate exit**, and the one option that leads to a stop. That is fine
because it is honest about being an exit — the user picks it knowing what it means, rather than
discovering it after choosing something that looked available.

When the user picks it, say plainly:

> Sorry — we don't support your stack yet.

Then name what *is* supported, offer to record the request, and stop. Do not ask follow-up questions
hunting for a way in, and do not steer the user toward a supported profile they didn't pick.

If the project has more than one surface, ask the question once per surface, each with the same
options.

**If every surface matches a supported profile**, name the profiles you matched and continue.

**If any surface does not, stop** — the project isn't scaffoldable, even if the other surfaces are.
Do not proceed. Specifically, do not:

- substitute the nearest supported profile for the unsupported surface;
- hand-author a profile's ADRs for an unsupported language from your own knowledge — the point of a
  profile is that those decisions were made deliberately, once, and reviewed;
- emit a partial scaffold and note the gap in passing.

Name which surfaces fit and which don't, say what's supported, and offer to record the request. A
clean refusal is the correct outcome — an improvised profile is worse than none, because it produces
ADRs marked Accepted that nobody actually decided.

If the user, having been told, explicitly asks for what *is* covered — the stack-agnostic core, or
the supported surfaces alone — that is theirs to choose. Emit it, and state plainly which surfaces
were left undecided and that their boundary-enforcement obligation is unmet.

### 2. Seams — only when a client owns domain logic

**Start from the default: the domain lives on the server.** A browser, a thin native shell, or any
client that renders and calls an API is not a second domain-bearing surface. It holds no entities and
no use cases of its own, so there is no seam — a `fetch`, an `invoke`, and a platform channel are all
gateway implementations, and ADR-BASE-01 already decides how they are treated.

That covers most projects, including every ordinary web app. **Skip this step unless a client holds
domain logic of its own**, which happens in three recognisable cases:

- **Offline-capable mobile or desktop apps.** A client that must decide, validate, and reconcile
  without asking the server is running domain rules, not rendering someone else's.
- **Games.** In-game simulation cannot round-trip, so the rules live on the client — while accounts,
  inventory, and progression usually live on the server. Two domains, genuinely.
- **No server at all.** A standalone SPA, a CLI, a local-only tool. Then the client *is* the whole
  app: one surface, and still no seam.

The third case is a reminder that two *processes* are not two domains, and one process is never two.
Ask directly rather than inferring it from the file listing: *does this client decide anything on its
own, or does it always ask?*

When a client genuinely does hold domain logic, the profiles deliberately don't decide the boundary.
Where the two meet, settle:

- **Which side owns the domain.** In a fat-shell Tauri app or a Flutter-plus-API project the entities
  can live on either side, or — badly — on both. Pick one and write it down.
- **What the gateway across the seam looks like.** Per ADR-BASE-01 this is just another gateway: an
  `invoke` command, an HTTP call, and a platform channel are the same shape to a use case, which
  never learns which one it got. The inner rings must not name the transport.

Record the answers as a project-specific ADR — `ADR-001`, the first in this project's own sequence,
from `_TEMPLATE.md`. This is the one place
`scaffold` writes a genuinely new decision rather than instantiating a template — so draft it, then
have the user confirm it before writing.

### 3. Interview

The templates deliberately don't decide four things — they are per-project calls, and they change
what gets emitted. Ask; do not guess. **Batch all of it into one round of questions**, not a
conversation. Every item below has a workable default, so a user who answers none of them still gets
a correct scaffold.

**One of the four is not a question.** #3 is read from step 1's shape judgement — say which
architecture you are using and why, do not ask the user to pick it. The other three are asked, along
with the destination below.

**Recommend only what an ADR supports.** Where a decision traces to an ADR, say so and name the
recommendation — that is the ADR doing its job. Where Codefall has no stance, present the options
flat, say plainly that there is no house opinion, and let the user choose. A "(Recommended)" label
with no ADR behind it invents an opinion this project does not hold, and the user cannot tell the
difference between a considered default and one you made up on the spot.

1. **App topology** — how many apps? Mostly answered already by step 1's surfaces. Per the matched
   profile's dependency-injection ADR each app gets its own composition root, and therefore its own
   `AGENTS.md`.
   Codefall has **no opinion on repo layout** — monorepo or separate repos, workspaces or a single
   package. No ADR covers it, so do not recommend one and do not label an option "Recommended". If
   there is only one app, default to a single package without asking. If there is more than one, ask
   where the user wants them, say there is no house preference, and follow the answer.
   **Also settle the topology of each React surface** — SPA plus a separate API, or a Next.js SSR
   shell. The profile describes both and the choice turns on one question: does anything need to be
   publicly reachable and worth indexing? If step 1's description already answered that, take it and
   do not ask again. If it was silent, ask exactly that question once — not "do you want Next.js",
   which invites a preference where there is a criterion. Default to SPA plus API when the answer is
   no.
2. **Bounded contexts** — do the capabilities already have obvious names? If so, they become the
   top-level component folders. Ask **once**, in one sentence, and offer "not yet" as a first-class
   answer. For a new project "not yet" is the **expected** answer, not the fallback — treat naming
   them as the special case, and phrase the question so declining costs the user nothing.
   **Names are colour, not a gate.** They seed folder names when offered and change nothing else. In
   particular they do **not** decide the architecture — step 1's shape judgement already settled that,
   and #3 reads it from there.
   If the user names some, keep them **coarse** — ADR-BASE-02 says start coarse and split, because
   re-slicing an existing boundary is the expensive case, and two or three is a fine start.
   If the answer is "not yet", vague, or hesitant: **take it and move on.** Do not push, do not
   suggest candidates, do not ask them to think it through. Package-by-component with components that
   are not named yet is still package-by-component; the folders get their names from the first
   capability that earns one.
3. **Per-surface architecture** — package-by-component (the default) or ports-and-adapters?
   Decide from **step 1's shape judgement**, never from whether #2 produced names. ADR-BASE-02 names
   two conditions favoring p&a: one cohesive domain rather than separable capabilities, or boundaries
   genuinely unknown. Several separable capabilities means package-by-component **even when nobody
   has named them yet**, and **even when the project will never be deployed as separate services** —
   the payoff is encapsulation, not deployment.
   **"We haven't named them" is not one of those conditions**, and reading it as one hands p&a to
   every greenfield project — which is every project this skill exists to scaffold. Not having names
   is the normal starting state, not evidence of a single cohesive domain.
   If p&a is chosen because the shape is genuinely *cohesive*, that is a real decision: amend
   ADR-BASE-02 for that surface. If it is chosen because the shape is genuinely *unclear*, that is a
   provisional call — leave ADR-BASE-02 alone and record it under `Open` in the decision log, per
   step 4.
4. **Depth** — docs only, docs + project files, or a runnable skeleton? Default to **docs only**
   unless the user wants more. See [Depth](#depth).

**Ask where it goes.** The destination is the user's to name, and the current working directory is
not a default — they may be standing in an unrelated repo. Include it in the same batch of
questions, and only then check the chosen target: never scaffold into a non-empty directory without
saying so first, and never overwrite an existing path. A warning about a directory the user never
nominated is noise.

### 4. Emit the docs — always

- `docs/adrs/` — `ADR-BASE-01` through `ADR-BASE-03` plus every ADR the matched profiles supply, plus
  `_TEMPLATE.md`. One flat directory: the prefixes keep them distinct, so a project matching two
  profiles needs no subdirectories. Dated and Accepted, amended per the interview. Drop ADRs that
  don't apply — a backend-only project has no use for `ADR-TS-02`, and a surface that could never be
  split into services at all (a CLI, a desktop or mobile app, a library) has no use for
  `ADR-BASE-03`. That gate is about the surface, not its layout: such a project may still be
  package-by-component, and should be if it has separable capabilities. Having no *current* plan to
  split an extractable backend is not a reason to drop it.
- `docs/decision-log.md` — the in-flight scratchpad, with `Locked` / `Open` / `Parking lot`
  sections. Seed `Open` with anything the interview surfaced but didn't settle — including a
  provisionally chosen ports-and-adapters, which belongs here rather than amended into ADR-BASE-02 as
  though it were settled. Seed `Parking lot` with the product detail step 1 heard but deliberately did
  not act on. Parking it is how that input reaches `specify` and `architect` instead of being lost.
- A scoped `AGENTS.md` per app/package, from the skeleton: fill the name, the one-line description,
  fix the ADR links to the right relative path, delete the `<frontend only>` blocks on backend
  surfaces, and fill in Gotchas. Keep it terse — it links the ADRs and never restates the why.

If the harness in use reads `CLAUDE.md` rather than `AGENTS.md`, add `CLAUDE.md` as a one-line
pointer to `AGENTS.md`. Do not maintain two copies of the rules.

### 5. Emit code — per the depth answer

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

#### The boundary-enforcement obligation

Every profile's boundary-enforcement ADR says the same thing: it is wired **at the project scaffold,
day one** — deferring defeats the purpose, and without it the package-by-component choice is
cosmetic. Docs-only output cannot satisfy that. When emitting docs only, say so plainly in the report
and name it as the first task the user owes the project.

The boundary config is the encoded architecture. Whenever you write it, it enforces all five rules —
facade-only imports, inward-only layers, cross-component via facades only, shared modules can't
import components, and inner rings can't touch the UI framework or gateways.

How much tooling this takes depends on the language's visibility model, which is why the
boundary-enforcement ADR is per-profile: TypeScript needs a linter because it has no
`package-private`, while Kotlin `internal`,
Rust `pub(crate)`, and Go's `internal/` packages do part of the job in the compiler. Part, not all —
Go rejects import cycles but not outward ones, so the facade rules come free and the layer rule still
needs a linter. Read the profile rather than assuming a language with real visibility needs nothing.

### 6. Verify

Whatever you emitted must actually work. If there are project files, run install, lint, and test —
and confirm the boundary rules **fail on a deliberate violation**, because a lint config that
catches nothing is the common failure here. Do not assume; run it. If there are only docs, check
that every ADR cross-link and every `AGENTS.md` link resolves.

### 7. Report

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
