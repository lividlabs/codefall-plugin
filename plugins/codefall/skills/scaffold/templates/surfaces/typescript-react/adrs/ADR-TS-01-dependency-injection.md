# ADR-TS-01: Dependency Injection

## Status

Accepted — <date>

## Context

Clean Architecture (ADR-BASE-01) has inner layers depend on interfaces that outer layers implement;
something must bind adapters to those interfaces at a composition root. Options are manual wiring or
a container. For TypeScript, **Inversify** is the mature IoC container and handles the common
"one interface, multiple implementations, rebind in tests" case cleanly.

## Decision

### Container

- **Inversify** across all TypeScript packages. **One composition root per app**; library packages
  export `ContainerModule`s the app's root loads. The composition root is the only place that names
  concrete implementations.
- On the **Next.js topology** the root is a module, not an entry point, because every route handler
  is its own entry. Build the container once at module load and cache it on `globalThis`, so dev
  hot-reload does not rebuild it per request:

  ```ts
  declare global { var __container: Container | undefined }
  export const container: Container = (globalThis.__container ??= buildContainer())
  ```

  A **route handler is the only container-aware file**. It resolves what it needs and passes those
  use cases as arguments into a handler in `presentation/`, which never imports the container. That
  keeps the service-locator rule below intact by confining the locator to the boundary.
  This container is deliberately module-global; see the profile's per-request trap for why the query
  client and the stores must not be.

### Injection discipline

- **Inject only interfaces** — every injected dependency is typed as an interface, never a concrete
  class. (Practical exception: config/value objects injected by token.)
- **Always name the token explicitly** — `@inject(TYPES.Thing)` on every constructor parameter, with
  the parameter's own type imported as `import type`. Never rely on Inversify inferring the type from
  emitted metadata.

  This is a portability rule, not a style one. Implicit resolution needs `design:paramtypes`, which
  only `tsc` emits; Metro transpiles with Babel and Next.js with SWC, and under either the inference
  silently fails at runtime rather than at build. An explicit token needs no metadata, so the same
  code resolves identically under every toolchain.

### Naming

- **Interface** = the bare Thing, no `I`-prefix — `UserRepository`, not `IUserRepository`.
- **Implementation** = `<Qualifier>Thing` — `PostgresUserRepository`, `InMemoryUserRepository`.
- Use **`Default`** as the qualifier for the single canonical implementation (`DefaultClock`); an
  edge-case sibling (`SystemClock`) does not force the original to be renamed.

### Tokens

- DI tokens live in a **`types.ts`** exporting `TYPES = { X: Symbol.for("X") }` (Inversify's
  convention). Tests **rebind** a token to a mock — the standard test seam.

### Frontend boundary (important)

- The container wires **framework-agnostic logic** (use cases, gateways, services). **React-native
  tools stay idiomatic** — the query client via its own provider, stores as hooks — and are **not**
  registered in the container. React reaches the container via context + a `useInjection`-style
  hook. Never use the container as a service locator sprinkled inside components.

### Non-TypeScript layers

- A non-TS layer (e.g. a Rust/Tauri shell) uses idiomatic **constructor injection / trait objects**,
  not Inversify.

## Consequences

- Decorators require `reflect-metadata` and tsconfig `experimentalDecorators`. Set
  `emitDecoratorMetadata` too — it costs nothing and helps tooling — but nothing may *depend* on it,
  because explicit tokens are mandatory above.
- Rebind-to-mock is a uniform, clean test seam across packages.
- Some bundle-size cost on the frontend; acceptable for app surfaces.
- The "magic" of a container is centralized in one composition root, not scattered.

## Related

- ADR-BASE-01 — Clean Architecture (the interfaces being bound)
- ADR-BASE-02 — Package-by-component (per-component modules the root loads)
- ADR-TS-02 — Frontend state (why the query client / stores stay outside the container)
