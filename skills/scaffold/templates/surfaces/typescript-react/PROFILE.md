# Surface profile: typescript-react

**Status:** supported.

TypeScript on Node, and React web frontends. Covers either surface alone or both together.

## Fits when

- A backend or service written in TypeScript on Node, **or**
- A React frontend — web or **React Native** — regardless of what it talks to, **or**
- A **Tauri or Electron desktop app whose shell is only wiring** — the whole app, not half of it.

The frontend does not care what is on the other end of its gateways. Per ADR-001 a gateway is "an
interface to any external system" — an HTTP API, a Tauri `invoke` command, an Electron IPC channel,
and a WebSocket are all the same shape to a use case. A React frontend is this profile whether it is
served over HTTP or embedded in a native shell.

### Render targets

React is the component model; the renderer is an outer-ring detail. The same domain, use cases,
gateways, DI, and boundary rules serve React DOM and React Native alike — ADR-004's three-tier split
is unchanged, since TanStack Query, Zustand, and `useState` all run on React Native. What varies is
toolchain, and one delta is load-bearing:

| Target | Delta |
| --- | --- |
| React DOM (web) | The baseline. |
| React Native | **Metro transpiles with Babel, not `tsc`.** |

**The React Native decorator trap.** ADR-003 relies on tsconfig `emitDecoratorMetadata`, which is a
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
never import Tauri or `NativeModules`, and ADR-005 rule 5 already forbids it. Swapping the desktop
shell for a web deployment, or Android for iOS, then touches one gateway implementation and nothing
else.

## Does not fit

Non-Node backends and non-React frontends.

A shell or native module that carries **real domain logic** — heavy compute, native integrations with
their own rules, security-sensitive work kept out of the JS thread — is a second surface, not wiring.
It needs its own profile, and the project needs a seam ADR deciding which side owns the domain.
ADR-003 anticipates this: non-TS layers use idiomatic constructor injection and trait objects, not
Inversify.

The test is **where domain logic lives**, and it is a question for the user. Do not infer it from the
presence of a `src-tauri/` or `android/` directory — React Native ships those empty.

## Visibility model

TypeScript has **no `package-private`**, so component facades are unenforceable by the compiler. This
is the fact ADR-005 is built around, and it is why this profile requires a boundary linter rather
than merely recommending one. Profiles for languages that *do* have real visibility (Kotlin
`internal`, Rust `pub(crate)`, Go's lowercase identifiers, Java modules) will need a much lighter
ADR-005 — do not copy this one across.

## Toolchain

| Concern | Choice | ADR |
| --- | --- | --- |
| DI container | Inversify, one composition root per app | ADR-003 |
| Server state | TanStack Query | ADR-004 |
| Shared client state | Zustand | ADR-004 |
| Local UI state | `useState` / `useReducer` | ADR-004 |
| Boundary enforcement | `eslint-plugin-boundaries`, `dependency-cruiser` optional | ADR-005 |

## ADRs this profile supplies

- `adrs/ADR-003-dependency-injection.md`
- `adrs/ADR-004-frontend-state.md` — **backend-only projects skip this one.**
- `adrs/ADR-005-boundary-enforcement.md`

Shared ADR-001 and ADR-002 come from `templates/adrs/` and apply to every profile.

## Depth notes

Beyond docs, this profile's **project files** tier owes: `package.json`, a `tsconfig` with
`experimentalDecorators` and `emitDecoratorMetadata` (ADR-003 needs both), ESLint including the
`eslint-plugin-boundaries` rules, formatter, test runner, and CI running all of it.

On **React Native**, those tsconfig flags are not sufficient — see the decorator trap above. Add the
Babel metadata plugin as well, or DI compiles clean and fails at runtime.

The **runnable skeleton** tier adds the component folders with their `index.ts` facades and nested
Clean layers, plus one Inversify composition root per app that boots with no features in it.

---

## Adding a new profile

Copy this file's shape. A profile is complete when it states: what it fits, what it explicitly does
not, its **visibility model** (which determines how much ADR-005 has to do), its toolchain choices,
the ADRs it supplies from ADR-003 up, an `AGENTS.md.skeleton`, and its depth notes. Until all of
that exists, the surface stays `planned` in the catalog and `scaffold` refuses projects that need it.

Scope a profile to **one surface**, not to one kind of product. "Desktop app" is not a profile — it
is a web surface plus a native shell, and each of those is reusable on its own. Profiles that
describe products rather than surfaces duplicate each other and rot.
