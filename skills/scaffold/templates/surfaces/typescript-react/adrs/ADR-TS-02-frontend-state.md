# ADR-TS-02: Frontend State Management

## Status

Accepted — <date>

## Context

A frontend juggles four genuinely different kinds of state. Conflating them — e.g. one global store
holding server responses, derived UI flags, and form text — forces you to hand-roll cache
invalidation for server data and needlessly globalizes ephemeral state.

## Decision

Four categories, one tool each:

| State kind | Tool | Examples |
|---|---|---|
| Server state | **TanStack Query** | anything owned by an API / backend |
| Navigation state | **the router** | current screen, back stack, route params |
| Shared client state | **Zustand** | selection, layout, theme — client-owned, cross-component |
| Local UI state | **React `useState` / `useReducer`** | form fields, toggles, hover |

### Decision rule

Ask in order: server-owned? → TanStack Query. Router-owned? → the router. Shared client state? →
Zustand. Component-local? → React.

### Rules

- **Do not mirror server state into Zustand** (banned). Zustand may hold a *key/reference* (e.g. a
  selected id) that Query then resolves; the server data itself stays in Query.
- **Do not mirror navigation state into Zustand** (banned). The router owns the current route, back
  stack, and route params. Read them from the router; a store may hold state *keyed by* a route,
  never the route itself.
- Real-time transports (WebSocket, SSE) **patch or invalidate the Query cache** rather than living in
  a parallel store.
- Per ADR-TS-01, the query client and the stores are frontend adapter-ring pieces and stay
  framework-idiomatic (not in the DI container).

## Consequences

- Server-state concerns (caching, refetch, retry, loading/error) are the library's, not bespoke
  code.
- Four tools to learn, but each does one job; the decision rule removes "where does this go?".
- The no-mirroring rules are load-bearing and enforced in review.
- **Redux is deliberately not chosen** — heavier than this split needs, and it still requires a
  query layer for server state.

## Related

- ADR-BASE-01 — Clean Architecture (the frontend's inner rings)
- ADR-TS-01 — Dependency Injection (the frontend DI boundary)
