---
name: architecture
description: >
  The slices-cqs architecture: vertical slices hosting a CQS application layer
  behind a private API, atomic commands, and a transactional outbox. Supplied by
  the slices-cqs overlay. Use on every task touching application, domain, or
  infrastructure code, and when creating or restructuring slices.
allowed-tools: Read, Bash, Grep, Glob
---

# architecture - Slices, CQS & the Outbox

Full rationale, diagrams, and assumptions: see `ARCHITECTURE.md` at the repo root
(deployed with this overlay).

## Workspace Layout

```
apps/
  web/  admin/            # edge: authN, UI, typed client - no DB, no business logic
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

- A slice owns a business capability and serves every caller - web, admin, public-api.
  Admin-only commands are enforced by handler authorization (user roles), never by
  which app sent the request
- Slices split by capability and change-cadence, never by calling app
- `domain/` appears when real business rules appear - no empty layers

## The Dependency Rule

Imports point one way. Nothing imports upward.

- Edge apps import **contracts only** (plus shared ui/util and the api-client)
- `application/` imports contracts + domain, and defines ports (interfaces)
- `infrastructure/` implements the ports - repositories, gateways, outbox writer
- `domain/` and contracts import nothing
- Only the composition root (in `apps/api` / `apps/worker`) knows concrete implementations
- Validation splits: **shape** in contracts (zod), **business** in domain

## Deciders: One Domain Style

Every aggregate's domain logic is a decider - three pure functions, identical for
state-stored and event-sourced aggregates:

```typescript
// domain/order.ts - pure, synchronous, deterministic
export const initialState = (): OrderState => ({ status: "empty" });

export const decide = (cmd: OrderCommand, state: OrderState): OrderEvent[] => {
  // business rules live here; reject on violation
};

export const evolve = (state: OrderState, event: OrderEvent): OrderState => {
  // how a fact changes state
};
```

- Persistence is the only variable: state-stored aggregates persist the evolved
  state; event-sourced aggregates persist the events and fold state on load
- State-stored is the default. Event sourcing is earned per aggregate - compliance,
  audit trail, "what did we know when", replay debugging - and recorded in a short
  ADR (see the governance skill's template)
- Test deciders given/when/then: given past events, when command, then expected
  events - pure unit tests, no infrastructure

## The Write Path

State-stored aggregate (the default) - one command, one aggregate, commanded via its
own id:

```typescript
export async function checkoutBasketHandler(cmd: CheckoutBasket, caller: UserIdentity) {
  const basket = await basketRepo.findForUser(cmd.basketId, caller.userId);
  if (!basket) return notFound();                      // scoped miss = 404, never 403

  const events = decide({ type: "Checkout", data: {} }, basket.state);  // pure domain;
  const next = events.reduce(evolve, basket.state);    // business refusals THROW typed errors

  await db.transaction(async (tx) => {
    await basketRepo.save(next, tx);
    await outbox.write(events, tx);                    // reactions happen async, via processors
  });
  // nothing that can fail is allowed to matter past this line

  return ok({ basketId: cmd.basketId, status: "checked-out" });
}
```

Event-sourced aggregate (Emmett - the append is the commit):

```typescript
const handle = CommandHandler({ evolve, initialState });

export async function approveItemHandler(cmd: ApproveItem, caller: UserIdentity) {
  if (!(await canReview(caller, cmd.orderId))) return forbidden();   // role check = 403

  await handle(eventStore, reviewStreamId(cmd.orderId),
    (state) => decide({ type: "ApproveItem", data: cmd }, state));
  // read stream -> fold -> decide -> append, with optimistic concurrency

  return ok({ orderId: cmd.orderId });
}
```

Error taxonomy - every failure has a fixed home; nothing crosses the API boundary as
an unhandled exception:

- Malformed input: zod at the contract, 422 - before the handler runs
- Scoped-load miss (not yours / doesn't exist): 404 - never 403, a 403 confirms existence
- Missing role/permission: 403 from the handler's authorize step
- Business refusal: the decider throws a typed `DomainError`; the handler factory
  translates it once into the contract's typed error (e.g. 409)
- Anything else is a bug: 500 + alert, never mapped, never swallowed

- The handler does only what completes or starts the process - everything else
  (email, analytics, notifications, local reactions) happens after commit, via
  processors
- Production slices use the shared handler factory (`libs/shared`) so both write
  paths look identical: developers supply authorize + decider, the composition root
  supplies persistence
- Queries: authorize first, then read a uniform read-model port - a scoped
  repository query or an Emmett projection backs it, the composition root decides.
  Never fold streams at query time. No observable side effects
- Scaffold new slices and commands with the workspace generators (`nx g slice`,
  `nx g command`) - structure is generated, not remembered

## Events & Consistency

- **Reactions are async by default, everywhere.** A command commits its own aggregate
  plus its recorded events (outbox table or stream); processors do everything else -
  local reactions, read models, external effects - at-least-once, idempotent,
  regardless of persistence style
- **Same-transaction reactions are an earned exception**: state-stored slices only,
  a real cross-aggregate invariant, one-line justification in review. Priority order
  when eventual isn't good enough: fix the aggregate boundary (most cross-aggregate
  invariants are boundary mistakes), then the exception, then compensate. Even in the
  exception: cheap, local, DB-only - an external call in-transaction is a bug
- **The command result is the read**: handlers return the outcome; edges render from
  it, never an immediate re-query. Reading your own aggregate is always consistent -
  state-stored: the state itself; event-sourced: an inline projection maintained in
  the append transaction. Cross-slice read models are eventual and labeled as such
- **Read models are Drizzle tables**: relational by default, a jsonb document column
  when a read model is genuinely key-fetched and screen-shaped. No Pongo. A slice's
  own read model (event-sourced) is projected inline via the shared projection
  helper; every other projection is async
- **Domain events**: internal vocabulary, recorded in the command's commit; never
  leave the process
- **Integration events**: versioned translations published to RabbitMQ by a reactor;
  bounded retries, then dead-letter via DLX
- Processors dispatch local commands through the application layer and are idempotent
- **Waiting states**: aggregate status + a read model + reactors, plus a scheduled
  sweep for deadlines (back-orders: `awaiting-stock` status, backorders-by-product
  read model, reactor on `StockReceived` dispatching continue commands FIFO). A
  dedicated process manager is the earned exception when coordination state belongs
  to no aggregate - model it as a decider (events in, commands out)
- **Multi-party joins live in the decider**: when several actors decide independently
  and something unlocks "once all decisions are made", completeness is a derived
  condition over aggregate state - the command completing the set also emits the
  ready event (e.g. `ReadyForClinicalCheck`). No orchestrator watches for it
- Whoever publishes, owns: event contracts belong to the publishing slice's contracts lib

## Nx Tags

```jsonc
// eslint - @nx/enforce-module-boundaries
"depConstraints": [
  { "sourceTag": "type:app-edge",  "onlyDependOnLibsWithTags": ["type:contracts", "type:ui", "type:api-client", "type:util"] },
  { "sourceTag": "type:app-core",  "onlyDependOnLibsWithTags": ["type:slice", "type:contracts", "type:outbox", "type:util"] },
  { "sourceTag": "type:contracts", "onlyDependOnLibsWithTags": ["type:kernel", "type:util"] },
  { "sourceTag": "type:slice",     "onlyDependOnLibsWithTags": ["type:contracts", "type:kernel", "type:outbox", "type:util"] },
  { "sourceTag": "type:ui",        "onlyDependOnLibsWithTags": ["type:ui", "type:util"] }
]
```

An edge app importing a slice lib is a CI failure, not a review comment. Layer
discipline inside a slice (application → domain, never the reverse) is convention +
the review checklist below.

## Review Checklist (additions for this architecture)

- [ ] No business logic, ORM access, or authorization decisions in edge apps
- [ ] Every handler (command AND query) authorizes before doing anything else
- [ ] Command handlers: one transaction covering state + outbox; no external calls in-tx
- [ ] Query handlers cause no observable side effects
- [ ] Domain logic is a decider (`decide`/`evolve`/`initialState`) - same style regardless of persistence
- [ ] `domain/` is pure - no `async`, no ORM types, no clock/randomness
- [ ] Event-sourced aggregate? Has an ADR; one stream per command; reads via projections, never stream-folds at query time
- [ ] No Emmett types past the slice boundary
- [ ] No DI container - ports + per-slice wiring functions (`wire<Slice>Slice(deps)`), environment selection in plain code in the composition root
- [ ] Tests use fakes (in-memory implementations from `libs/shared/testing`), not mocking-library stubs of ports
- [ ] Repositories only retrieve/store - no policy, no events, no self-opened transactions
- [ ] Domain events never serialized outward - integration events are translations
- [ ] Same-transaction reaction? Has a one-line justification; aggregate boundary was considered first
- [ ] Edges render from the command result - no immediate re-query after a command
- [ ] Errors follow the taxonomy: scoped miss = 404 (never 403); business refusals = typed DomainError thrown by the decider, translated once by the handler factory; no unhandled exception crosses the API
- [ ] Event consumers are idempotent and dispatch through the application layer
- [ ] New cross-slice dependency? Must be via the other slice's contracts (events)
