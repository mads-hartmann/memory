---
title: CQRS (Command Query Responsibility Segregation)
tags:
- architecture
- design-patterns
---

CQRS separates the write side (commands) from the read side (queries) of a system into distinct models, each optimised for its purpose.

- **Command** — an intent to change state (`PlaceOrder`, `WithdrawMoney`). Returns nothing meaningful, only success/failure.
- **Query** — a request for data. Has no side-effects; never mutates state.

Keeping these paths separate lets you scale, optimise, and evolve them independently.

## How it works

```
Client
  │
  ├─ Command ──► Command Handler ──► Domain Model ──► Event Store / DB
  │
  └─ Query  ──► Read Model (projection) ◄── denormalised view built from events/writes
```

The read model is a pre-built, denormalised projection tuned for fast reads. It is rebuilt (or incrementally updated) from the write side's output.

## Relationship to event sourcing

CQRS and [event sourcing](event-sourcing.md) are independent patterns but pair together naturally:

- The command side appends domain events to the event log (see [event sourcing](event-sourcing.md)).
- The query side subscribes to those events and maintains one or more read-optimised projections.
- Multiple projections can serve different query needs from the same event stream — no schema compromise required.

Neither pattern requires the other, but event sourcing almost always surfaces the need for CQRS because replaying events to answer every query would be too slow.

## Trade-offs

| Benefit | Cost |
|---|---|
| Read and write models each optimised independently | More moving parts; higher initial complexity |
| Easy to add new read projections without touching writes | Eventual consistency between write and read sides |
| Scales read and write paths separately | Developers must reason about two separate models |

## When to use

- High read/write asymmetry (many more reads than writes, or vice versa).
- Multiple consumers need the same data shaped differently.
- Already adopting [event sourcing](event-sourcing.md).

Avoid for simple CRUD apps — the overhead rarely pays off.