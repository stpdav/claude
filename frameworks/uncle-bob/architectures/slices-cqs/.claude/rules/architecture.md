# Architecture Rules (slices-cqs)

Supplied by the `slices-cqs` overlay. Always follow, alongside the `architecture` skill.

## Non-Negotiables

- Edge apps (web/admin) are thin: authN, UI, typed client calls - no business logic, no DB credentials
- Every boundary has an explicit, zod-validated contract
- All reads and writes go through the application layer - query handlers authorize before reading
- All domain logic is written as deciders (`decide` / `evolve` / `initialState`) - one style for state-stored and event-sourced aggregates alike
- Domain is pure: no I/O, no ORM types, no direct clock/randomness - inject them
- State-stored is the default; event sourcing is earned per aggregate (compliance, audit trail, replay) and recorded in a short ADR
- Commands are atomic. State-stored: evolved state + outbox messages in one transaction. Event-sourced: the stream append is the commit - one aggregate (stream) per command
- Repositories retrieve and store objects only - no business logic, no event emission
- Reactions are async by default, everywhere: a command commits its own aggregate plus its recorded events; processors do the rest, at-least-once, idempotent - regardless of persistence style
- Same-transaction reactions are an earned exception: state-stored slices only, a real cross-aggregate invariant, one-line justification in review - fix the aggregate boundary first
- The command result is the read: edges render from the handler's returned outcome, never an immediate re-query; cross-slice read models are eventual and labeled as such
- Errors have fixed homes: shape failures die at the contract (422); scoped-load misses return not-found (never forbidden - a 403 confirms existence); role failures return forbidden; business refusals are typed domain errors thrown by the decider and translated once by the handler factory; anything else is a bug (500 + alert)
- Integration events are versioned translations - delivered via the outbox (state-stored) or event-store consumers (event-sourced); domain events never leave the process
- Handlers are built with the shared handler factory so state-stored and event-sourced write paths look identical; queries read a uniform read-model port
- Drizzle is the ORM - repositories, outbox, projections, read-model queries. Read models are relational tables by default, a jsonb document column when key-fetched and screen-shaped; no Pongo
- A slice's own read model (event-sourced) is projected inline in the append transaction - read-your-writes on your own aggregate; all other projections are async
- Scaffold slices and commands with the workspace generators - structure is generated, not remembered
- No DI container: ports in the application layer, environment selection in plain code, per-slice wiring functions composed by each app's composition root
- Tests use fakes from `libs/shared/testing` (and Emmett's in-memory event store) passed directly to handlers - deciders need no doubles at all
- Query handlers read projections for event-sourced aggregates - never fold streams at query time
- Event consumers and processors are idempotent - delivery is at-least-once
- Emmett never leaks past the slice boundary - contracts and edges see nothing of it
- Waiting states live in the aggregate (status) with reactors and, for deadlines, scheduled sweeps - a dedicated process manager is an earned exception, modeled as a decider (events in, commands out)
- Cross-slice communication happens via events, never imports of another slice's internals
- Don't scaffold empty layers - `domain/` appears when real rules appear
