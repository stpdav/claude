# Architecture: Slices, CQS & the Outbox

The `slices-cqs` overlay. Deployed to a repo, this document lives at the root as the
product's architecture position; the operational rules live in the `architecture`
skill and `.claude/rules/architecture.md`.

Status: **draft** - items marked *(assumed)* await ratification; see Assumptions.

## Principles

- **Code that changes together stays together** - the slice, not the layer, is the unit of organization
- **Edges are thin** - apps render, capture intent, and dispatch; no business logic, no database credentials
- **Contracts are explicit** - every boundary (slice, private API, events, public API) has a typed, validated, named contract
- **Commands are atomic** - a command's effects commit in one transaction or not at all, including the messages it emits
- **Everything external is asynchronous** - work beyond completing or starting the process leaves via the outbox
- **Every block has one job** - and an explicit list of things it must never do
- **One way to do things** - defaults with justified exceptions, never open choices;
  the less a developer must hold in their head, the better

## Topology

`apps/api` (Fastify) hosts the CQS application layer. Web and admin are
thin HTTP clients: they authenticate users, render, and dispatch through a typed
ts-rest client. They hold no database credentials - all data access, transactions,
and outbox writes live in one deployable.

**Private vs public API - separate deployables, never route namespaces.**
`apps/api` serves our own apps: network-restricted, internally authenticated,
contracts free to evolve because we deploy every consumer. `apps/public-api` exists
only when a customer-facing API becomes a product: API keys, rate limiting, WAF
exposure, versioned stable contracts. Both are thin adapters over the same slice
libs - the split is structural with zero logic duplication.

**Authentication.** AuthN at the edge: web/admin own login, sessions, MFA. Every
private-API call carries two identities: an *app credential* proving which app is
calling (audit, rate-limit, revoke per app) and a *verifiable user assertion* - raw
credentials never travel past the edge. AuthZ in the handlers: the API trusts the
edge for authentication and for nothing else. App identity never authorizes an
action - a leaked app credential must not be a skeleton key.

## Workspace Layout

```
apps/
  web/  admin/            # edge: authN, UI, typed client
  api/                    # Fastify: hosts every slice, one composition root
  public-api/             # only when a customer API is a product
  worker/                 # outbox relay + event consumers - always running
libs/
  slices/
    <slice>/
      contracts/          # own lib: commands, queries, results, published events
      slice/              # application/ · domain/ · infrastructure/ inside
  web/ | admin/           # edge-only UI composition libs (optional)
  shared/
    ui/  kernel/  outbox/  api-client/
```

- **Slices are app-agnostic.** A slice owns a business capability and serves every
  caller: `PlaceOrder` (web), `RefundOrder` (admin), `GetOrder` (both) all belong to
  the same slice, protecting the same entity and invariants. Admin-only commands are
  enforced by handler authorization (user roles), never by which app called. Only
  presentation is per-app. If a slice splits, it splits by capability and
  change-cadence (e.g. an `order-reporting` read-model slice), never by calling app
- **Cross-slice needs go through events**, never imports of another slice's
  internals. Whoever publishes, owns: a published event's contract (schema +
  versioning) belongs to the publishing slice, in its contracts lib - consumers adapt
- **Don't scaffold empty layers** - a CRUD-ish slice is contracts + application +
  infrastructure; `domain/` appears the day real rules appear

## The Dependency Rule

Imports point one way; nothing imports upward:

| Who                          | May import                                       |
|------------------------------|--------------------------------------------------|
| Edge apps (web/admin)        | contracts, shared ui/util, api-client - nothing else |
| application/ (handlers, ports) | contracts, domain                              |
| infrastructure/              | the ports it implements (repos, gateways, outbox writer) |
| domain/ and contracts        | nothing                                          |
| Composition root (api/worker)| everything - the only place that knows concrete implementations |

Validation splits in two: **shape** validation in contracts (zod - "is this a
well-formed PlaceOrder?"); **business** validation in domain ("may this customer
place this order?"). Business rules in zod schemas are the subtle wrong place.

Errors have fixed homes - nothing crosses the API boundary as an unhandled
exception: shape failures die at the contract (422); scoped-load misses return
not-found (never forbidden - a 403 confirms the resource exists); role failures
return forbidden; business refusals are typed domain errors thrown by the decider
and translated once by the handler factory into the contract's typed error;
anything else is a bug (500 + alert).

## Responsibilities

Each block has one job and a "never" list. The full catalogue with smells is in the
`architecture` skill's review checklist; the short version:

| Block               | Responsible for                                        | Never                                             |
|---------------------|--------------------------------------------------------|---------------------------------------------------|
| Edge apps           | Rendering, input, authN, dispatching via typed client  | Business rules, DB access, authZ decisions        |
| Contracts           | Named intent, zod shape, results, published events     | Behavior; importing deeper layers                 |
| Command handlers    | One use case: authorize → load → domain → one tx (state + outbox) | HTTP concerns, calling handlers, slow external calls in-tx, owning business rules |
| Query handlers      | Authorize first, then read (scoped: `findForUser`)     | Mutations, outbox writes, any side effect         |
| Domain              | A decider: `decide` (rules → events), `evolve` (event → state), `initialState`; pure | I/O, ORM types, direct clock/randomness |
| Repositories        | Retrieve and store objects; persistence↔domain mapping; join the passed tx | Business logic, events, deciding when to save |
| Gateways            | One external system; typed errors, timeouts, retries   | Business decisions; being called from the edge    |
| Domain events       | Async reactions via processors (same-tx = earned exception, state-stored only) | External I/O in-tx; leaving the process |
| Integration events  | Versioned translations for other services, via outbox  | Carrying internal domain types; unversioned change |
| Outbox writer/relay | Persist in-tx; poll, publish, retry, dead-letter       | Inspecting or transforming business content       |
| Event consumers     | Dispatch local commands; idempotent (at-least-once)    | Direct DB writes; bypassing handlers              |
| Composition root    | Wiring implementations to ports, per app               | Logic                                             |

## Aggregates: One Domain Style, Two Persistence Strategies

All domain logic is written as a **decider** - three pure functions, identical for
every aggregate:

```typescript
decide:       (command, state) => event[]    // business rules; reject on violation
evolve:       (state, event)   => state      // how a fact changes state
initialState: ()               => state
```

Developers learn one way to write business rules; persistence is the only variable:

| | State-stored (default) | Event-sourced (earned) |
|---|---|---|
| Write | handler folds `decide`'s events into new state; saves state + outbox in one tx | Emmett `CommandHandler`: read stream → fold → `decide` → append (the append is the commit) |
| Read | query handler → scoped repository | query handler → projection - own read model inline (same tx), others async; never fold streams at query time |
| Integration events | outbox table → relay → RabbitMQ | event store → reactor → RabbitMQ |
| Local reactions | processors, async (same-tx = earned exception) | processors, async, at-least-once |

**Event sourcing is earned, per aggregate**, by compliance/audit-trail requirements,
"what did we know when" questions, or replay debugging - and each adoption is
recorded in a short ADR. Everything else stays state-stored.

**Emmett** provides the event-sourced machinery: the PostgreSQL event store,
`CommandHandler`, projections feeding query handlers, and checkpointed
consumers/processors (at-least-once; poison messages skip to dead-letter). Two
containment rules: Emmett never leaks past the slice boundary (contracts and edges
see nothing of it), and versions are pinned - it is pre-1.0.

**Consistency: async by default, everywhere.** One aggregate per command; all
reactions - local ones included - run via checkpointed processors, at-least-once,
idempotent, regardless of persistence style. The primary path stays strong: the
command result is the read (edges render from the handler's returned outcome, never
an immediate re-query), and reading your own aggregate is always consistent -
state-stored: the state itself; event-sourced: an inline projection maintained in
the append transaction.
Cross-slice read models are eventual and labeled as such. When eventual isn't good
enough, three tools in priority order: fix the aggregate boundary (most
cross-aggregate invariants are boundary mistakes); the earned same-transaction
exception (state-stored only, one-line justification in review); compensate.

**Consistency devices** keep the two persistence strategies feeling like one system:
a shared handler factory (`libs/shared`) so both write paths look identical -
developers supply authorize + decider, the composition root supplies persistence; a
uniform read-model port behind every query (repository query or Emmett projection,
the composition root decides); and Nx workspace generators (`nx g slice`,
`nx g command`) so slice structure is generated, never remembered.

Deciders are tested given/when/then - given past events, when command, then expected
events - as pure unit tests with no infrastructure.

## Atomicity & the Outbox

- **Atomicity's hard boundary is one database.** Within a product: real transactions,
  all-or-nothing, including the outbox write. Across the broker: eventual consistency -
  sagas and compensating actions, not rollbacks. No one should expect a distributed
  transaction
- Commit ⇒ the message will be delivered; rollback ⇒ it never existed. This only
  works because the outbox insert shares the command's transaction
- Delivery is **at-least-once** - consumer idempotency is a rule, not an option
- Bounded retries, then dead-letter - and DLQ monitoring ships with the first
  consumer, because an unwatched DLQ is silent data loss
- Start simple: outbox table + polling relay per product, publishing to RabbitMQ;
  dead-lettering maps to RabbitMQ's native DLX. CDC is a later optimization with
  the same contract

## Contracts

| Contract            | Audience                | Stability                                  |
|---------------------|-------------------------|--------------------------------------------|
| Slice contracts     | Our own apps via ts-rest| Evolves freely - we deploy every consumer  |
| Integration events  | Other services          | Versioned; additive changes preferred      |
| Public API contracts| Customers               | Versioned, documented, stable              |

ts-rest makes slice contracts a single source of truth: one zod definition generates
the typed Fastify router and the typed client - types on both ends of the wire.

## Decisions

Decided:

1. **Fastify** for `apps/api` and `apps/public-api`
2. **RabbitMQ** as the message broker - the relay publishes outbox messages to it;
   dead-lettering via RabbitMQ's native DLX
3. **Whoever publishes, owns** - a published event's contract (schema + versioning)
   belongs to the publishing slice, in its contracts lib; consumers adapt
4. **Deciders everywhere** - all domain logic as `decide`/`evolve`/`initialState`,
   one style regardless of persistence (this settles the former "domain richness"
   assumption: functional deciders, not OO entities)
5. **Emmett** for event-sourced aggregates - PostgreSQL event store, projections,
   checkpointed processors; pinned versions, contained behind the slice boundary
6. **Reactions async by default, everywhere** - same-transaction reactions are an
   earned exception (state-stored only, justified in review); the command result is
   the read
7. **Consistency devices** - shared handler factory, uniform read-model ports, Nx
   workspace generators for slices and commands
8. **Drizzle** as the ORM - repositories, outbox, projections, and read-model
   queries; SQL-first, explicit, interactive transactions. No Pongo - read models
   are relational tables by default, or a plain jsonb document column (still
   Drizzle) when a read model is genuinely key-fetched and screen-shaped
9. **A slice's own read model is inline** (event-sourced aggregates): maintained
   with Drizzle inside the append transaction, giving read-your-writes on your own
   aggregate. Every other projection is async. The Drizzle-inside-Emmett-tx
   mechanism (Drizzle over the tx client vs query-builder SQL on Emmett's executor)
   is written once in the shared projection helper - verify in the first spike

10. **Manual composition roots, no DI container** - ports (interfaces) in the
    application layer, implementations selected per environment in plain code, and
    **per-slice wiring functions** (`wireCheckoutSlice({ db, outbox, env })` returns
    the slice's handlers) so roots stay a page long and wiring lives with the code
    it wires. Testing needs no container: deciders are pure (zero doubles), handlers
    take fakes directly via the handler factory, and `libs/shared/testing` ships
    reusable in-memory fakes (plus Emmett's in-memory event store). Awilix is the
    future-ADR escape hatch if wiring ever genuinely hurts

11. **Waiting states without new machinery** - a process that waits lives as
    aggregate status + reactors + (for deadlines) a scheduled sweep in the worker.
    Canonical example, back-orders: order status `awaiting-stock`, a
    backorders-by-product read model, a reactor on `StockReceived` dispatching
    continue commands in FIFO order (the overselling invariant stays inside the
    inventory aggregate, so allocation is race-safe), an expiry sweep for deadlines.
    A dedicated process manager is the earned exception - when coordination state
    belongs to no single aggregate - and it is modeled as a decider (events in,
    commands out), so nothing new to learn.
    Richer example - multi-party order review (prescriber and pharmacist approve or
    reject items independently and out of order; warehouse dispenses in parallel and
    pulls rejections; clinical check gates packing): one event-sourced fulfilment
    aggregate owns per-item decision state, and **the join lives in the decider** -
    the command that completes the decision set also emits `ReadyForClinicalCheck`;
    reactors bridge to the warehouse; the never-ship-a-rejected-item invariant is a
    `PackOrder` guard. No orchestrator - completeness is a fold over the events

All items decided. One implementation verification remains for the first spike: the
Drizzle-inside-Emmett-transaction mechanism (decision 9).
