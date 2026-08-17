# Surface profile: typescript-react

**Status:** supported.
**ADR prefix:** `TS` — this profile's ADRs are `ADR-TS-01` upward.

TypeScript on Node, and React web frontends. Covers either surface alone or both together.

## Fits when

- A backend or service written in TypeScript on Node, **or**
- A React frontend — web or **React Native** — regardless of what it talks to, **or**
- A **Tauri or Electron desktop app whose shell is only wiring** — the whole app, not half of it.

The frontend does not care what is on the other end of its gateways. Per ADR-BASE-01 a gateway is "an
interface to any external system" — an HTTP API, a Tauri `invoke` command, an Electron IPC channel,
and a WebSocket are all the same shape to a use case. A React frontend is this profile whether it is
served over HTTP or embedded in a native shell.

### Render targets

React is the component model; the renderer is an outer-ring detail. The same domain, use cases,
gateways, DI, and boundary rules serve React DOM and React Native alike — ADR-TS-02's three-tier split
is unchanged, since TanStack Query, Zustand, and `useState` all run on React Native. What varies is
toolchain, and one delta is load-bearing:

| Target | Delta |
| --- | --- |
| React DOM (web) | The baseline. |
| React Native | **Metro transpiles with Babel, not `tsc`.** |

**The React Native decorator trap.** ADR-TS-01 relies on tsconfig `emitDecoratorMetadata`, which is a
`tsc` feature. Metro never reads it — Babel strips types without emitting `design:type` metadata, so
Inversify cannot infer constructor parameter types and injection fails at runtime, not at build. Add
`babel-plugin-transform-typescript-metadata` (ordered before the decorators plugin) in
`babel.config.js`. Setting the tsconfig flags alone looks correct and does nothing.

Verify this the way step 4 requires: resolve one decorated class from the container in a smoke test.
A container that fails only on the first real injection is the failure mode here.

### Thin-shell desktop and mobile apps

When `src-tauri/`, the Electron main process, or a React Native **native module** is stock
boilerplate plus a few commands wrapping OS APIs — file dialogs, notifications, the tray, the camera,
secure storage — those commands are **gateway implementations**, and the app is entirely this
profile. Kotlin or Swift in `android/` and `ios/` no more makes it a native surface than a Postgres
driver makes SQL one.

Concretely: the `invoke` call or bridge module lives in the `infrastructure/` ring of whichever
component needs it, behind an interface owned by that component's `application/` layer. Use cases
never import Tauri or `NativeModules`, and ADR-TS-03 rule 5 already forbids it. Swapping the desktop
shell for a web deployment, or Android for iOS, then touches one gateway implementation and nothing
else.

## Does not fit

Non-Node backends and non-React frontends.

A shell or native module that carries **real domain logic** — heavy compute, native integrations with
their own rules, security-sensitive work kept out of the JS thread — is a second surface, not wiring.
It needs its own profile, and the project needs a seam ADR deciding which side owns the domain.
ADR-TS-01 anticipates this: non-TS layers use idiomatic constructor injection and trait objects, not
Inversify.

The test is **where domain logic lives**, and it is a question for the user. Do not infer it from the
presence of a `src-tauri/` or `android/` directory — React Native ships those empty.

## Visibility model

TypeScript has **no `package-private`**, so component facades are unenforceable by the compiler. This
is the fact ADR-TS-03 is built around, and it is why this profile requires a boundary linter rather
than merely recommending one. Profiles for languages that *do* have real visibility (Kotlin
`internal`, Rust `pub(crate)`, Go's `internal/` packages, Java modules) need a much lighter
boundary-enforcement ADR of their own — do not copy this one across. `ADR-GO-02` is the worked
example: the compiler covers the facade rules outright, and only the layer rule needs a linter.

## Toolchain

| Concern | Choice | ADR |
| --- | --- | --- |
| DI container | Inversify, one composition root per app | ADR-TS-01 |
| Server state | TanStack Query | ADR-TS-02 |
| Shared client state | Zustand | ADR-TS-02 |
| Local UI state | `useState` / `useReducer` | ADR-TS-02 |
| Boundary enforcement | `eslint-plugin-boundaries`, `dependency-cruiser` optional | ADR-TS-03 |

## ADRs this profile supplies

- `adrs/ADR-TS-01-dependency-injection.md`
- `adrs/ADR-TS-02-frontend-state.md` — **backend-only projects skip this one.**
- `adrs/ADR-TS-03-boundary-enforcement.md`

Shared ADR-BASE-01 and ADR-BASE-02 come from `templates/adrs/` and apply to every profile.

## Depth notes

Beyond docs, this profile's **project files** tier owes: `package.json`, a `tsconfig` with
`experimentalDecorators` and `emitDecoratorMetadata` (ADR-TS-01 needs both), ESLint including the
`eslint-plugin-boundaries` rules, formatter, test runner, and CI running all of it.

On **React Native**, those tsconfig flags are not sufficient — see the decorator trap above. Add the
Babel metadata plugin as well, or DI compiles clean and fails at runtime.

The **runnable skeleton** tier adds the component folders with their `index.ts` facades and nested
Clean layers, plus one Inversify composition root per app that boots with no features in it.

---

## Adding a new profile

Copy this file's shape. A profile is complete when it states: an **ADR prefix**, what it fits, what it
explicitly does not, its **visibility model** (which determines how much its boundary-enforcement ADR
has to do), its toolchain choices, the ADRs it supplies, an `AGENTS.md.skeleton`, and its depth notes.
Until all of that exists, the surface stays `planned` in the catalog and `scaffold` refuses projects
that need it.

**Pick a short uppercase ADR prefix** — `TS`, `GO` — and check that no existing profile has claimed
it. Numbering restarts at 01 inside each profile, so `ADR-<PREFIX>-01` is simply the first ADR this
profile supplies and nothing has to be coordinated with any other profile. Two profiles in one
project never collide, and a profile with no use for a given concern just doesn't ship that ADR —
there is no gap to explain and nothing to renumber.

Scope a profile to **one surface**, not to one kind of product. "Desktop app" is not a profile — it
is a web surface plus a native shell, and each of those is reusable on its own. Profiles that
describe products rather than surfaces duplicate each other and rot.
