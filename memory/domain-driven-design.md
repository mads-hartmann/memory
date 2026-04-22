---
title: Domain Driven Design
tags:
- architecture
- design-patterns
---

Domain Driven Design (DDD) is a software development approach introduced by Eric Evans in his 2003 book *Domain-Driven Design: Tackling Complexity in the Heart of Software*. The core idea is that the structure and language of your code should match the business domain it models.

## Core concepts

**Domain** — The subject area your software addresses (e.g. banking, logistics, healthcare). Everything in DDD revolves around deeply understanding this domain.

**Ubiquitous Language** — A shared vocabulary between developers and domain experts. The same terms are used in conversation, documentation, and code. If the business calls it an "Order", the code has an `Order`, not a `PurchaseRecord` or `CartSubmission`.

**Bounded Context** — An explicit boundary within which a particular domain model applies. The same concept can mean different things in different contexts (e.g. a "Customer" in the Sales context vs. the Support context). Each bounded context has its own model and ubiquitous language.

**Context Map** — A diagram showing how bounded contexts relate to and integrate with each other.

## Building blocks

**Entity** — An object with a distinct identity that persists over time. Two entities are the same if they share the same identity, regardless of their attributes. Example: a `User` identified by a `userId`.

**Value Object** — An object defined entirely by its attributes, with no identity. Two value objects are equal if all their attributes are equal. They are immutable. Example: a `Money` object with `amount` and `currency`.

**Aggregate** — A cluster of entities and value objects treated as a single unit for data changes. One entity acts as the **Aggregate Root** and is the only entry point for modifications. Enforces invariants (business rules) within its boundary.

```ts
// Order is the aggregate root — you never modify OrderLine directly
class Order {
  private lines: OrderLine[] = [];

  addLine(product: Product, quantity: number) {
    if (this.lines.length >= 50) throw new Error("Order cannot exceed 50 lines");
    this.lines.push(new OrderLine(product, quantity));
  }
}
```

**Repository** — An abstraction for retrieving and persisting aggregates. It gives the illusion of an in-memory collection while hiding database details.

**Domain Service** — Logic that doesn't naturally belong to any single entity or value object. Example: a `PaymentService` that coordinates between a `Wallet` and an `Order`.

**Domain Event** — A record of something that happened in the domain. Named in the past tense. Example: `OrderPlaced`, `PaymentFailed`. Used to decouple parts of the system.

**Factory** — Encapsulates the creation logic for complex aggregates or entities when construction is too involved to live in a constructor.

## Strategic vs. tactical DDD

**Strategic DDD** deals with the big picture — identifying bounded contexts, subdomains, and how they relate. This is where you decide how to carve up a large system.

- **Core domain** — The part that differentiates your product. Invest the most here.
- **Supporting subdomain** — Necessary but not differentiating. Build it simply or buy it.
- **Generic subdomain** — Completely commodity. Use off-the-shelf solutions (e.g. email delivery, auth).

**Tactical DDD** deals with implementation patterns within a single bounded context — entities, aggregates, repositories, domain events, etc.

## Integration patterns between bounded contexts

| Pattern | Description |
|---|---|
| **Shared Kernel** | Two contexts share a subset of the model. Requires close coordination. |
| **Customer/Supplier** | Upstream context provides an API; downstream consumes it. |
| **Anti-Corruption Layer (ACL)** | A translation layer that prevents a foreign model from polluting your own. |
| **Published Language** | A well-documented shared format (e.g. JSON schema, Protobuf) for cross-context communication. |
| **Separate Ways** | Contexts have no integration — they solve problems independently. |

## When DDD is worth it

DDD adds real complexity. It pays off when:

- The domain logic itself is complex (not just the data volume or infrastructure).
- You have access to knowledgeable domain experts to collaborate with.
- The system is expected to evolve significantly over time.

It's likely overkill for simple CRUD applications or short-lived projects.