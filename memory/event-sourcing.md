---
title: Event Sourcing
tags:
- architecture
- design-patterns
- persistence
---

Event sourcing is a persistence pattern where state is derived from an append-only log of immutable events rather than storing the current state directly.

Instead of `UPDATE users SET balance = 90`, you record `MoneyWithdrawn { amount: 10 }`. The current state is rebuilt by replaying all events.

## Key ideas

- **Events are the source of truth.** The database row / document is just a projection.
- **Append-only log.** Events are never mutated or deleted; only new events are appended.
- **Projection / Read model.** Current state is computed by folding events. Multiple projections can model the same events differently for different use cases.
- **Time travel.** Because the full history is retained you can replay events to any point in time — useful for debugging and auditing.

## Relationship to DDD

Event sourcing pairs naturally with [Domain Driven Design](domain-driven-design.md). [Domain Events](domain-driven-design.md) (`OrderPlaced`, `PaymentFailed`) become the events stored in the log, and [Aggregates](domain-driven-design.md) expose an `apply(event)` method that advances their state.

## Trade-offs

| Benefit | Cost |
|---|---|
| Full audit trail | Extra complexity (CQRS often required) |
| Easy temporal queries | Schema evolution of old events is hard |
| Decoupled consumers via event stream | Rebuilding projections can be slow at scale |

## Common pairing: CQRS

Commands mutate state by appending events; queries read from a pre-built projection. Keeping the two paths separate is called **CQRS** (Command Query Responsibility Segregation). See [CQRS](cqrs.md) for a full breakdown.