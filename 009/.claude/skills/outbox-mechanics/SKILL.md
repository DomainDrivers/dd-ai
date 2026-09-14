---
name: outbox-mechanics
description: The universal anatomy of a transactional outbox — write in the producer's transaction, relay, sink, serializer, at-least-once delivery, inbound adapter with inbox dedup — plus the menu of implementation choices at each sub-decision, with trade-offs. Use when designing how to route a cross-module message through an outbox.
---

# Outbox mechanics

A transactional outbox makes "the producer changed its state" and "the message
went out" succeed or fail **together**, without a distributed transaction. The
producer writes the message into an outbox record **in its own transaction**; a
relay reads pending records and forwards them; the consumer handles them. This
skill names the parts and, for each, the **menu** you must put to the user — it
does not pick for you. Discover what the project already runs, propose the option
consistent with it, and let the user instruct the choice.

## The fixed anatomy

```
producer tx: { change state ; write message to outbox }   ── atomic together
relay:        read pending outbox records → push to sink → mark sent
sink:         where the relay pushes (in-process bus now, broker later)
consumer:     inbound adapter handles the message → calls consumer's public API
```

Two invariants hold whatever you choose:

- the outbox write is **in the producer's transaction on the producer's resource**
  (see **transactional-atomicity** — this is non-negotiable for the guarantee);
- delivery is **at-least-once**, so the consumer must be **idempotent** — and that
  idempotency must itself be **atomic with the effect** and **race-safe** (see
  *Consumer side* below); a redeliver must be a no-op.

**Order is not one of them.** A polling relay, a retried push, and a second relay
worker each deliver messages in an order the producer did not choose. Ask whether
the consumer depends on order. If it does, carry an **ordering key** on the record
and keep one message per key in flight — a single relay worker, or a per-key claim —
and say what that costs in throughput.

## The decisions — put each as a menu

### 1. Adopt or hand-roll

Before designing anything, look for an outbox the project can already run: a
framework's event-publication registry, an outbox module, a queue client that enqueues
inside the producer's own transaction.

- **Adopt what is there** — the relay, the retries and the operational view come with
  it, and this is the usual production answer. You inherit its record shape, its
  delivery semantics and its dependency, and the guarantee is no longer something a
  reader can follow end to end in your own code.
- **Hand-roll it** — a table, a poller, a status column. Small enough to read in one
  sitting, no dependency, and every guarantee is visible in the code, which is what
  makes it a good walking skeleton. You own the relay and its failure handling.

Recommend whichever the project is already set up for; put both to the user either way.

### 2. Interception mechanism (how invasive is the swap?)

- **Existing port or bus** → swap the implementation behind it (opt-in). **Zero
  producer edits** — every producer that depends on the abstraction is rerouted at
  once. Best case; look for it first.
- **Direct synchronous call** → **introduce a sending port**: a one-method
  interface the producer calls instead of the consumer's public API. Default
  implementation = today's direct call (behavior unchanged); opt-in implementation =
  park the message in the outbox. Producer changes by one line and now depends on a
  message, not on the consumer.

### 3. The store

The outbox record lives in the producer's **transactional resource**, because the
write must join the producer's transaction. Whatever that resource is (a relational
DB, a document store, an event store), the outbox is a collection/table/stream in
**that same** resource. Choosing a store on a *different* resource breaks atomicity
— route that decision through **transactional-atomicity**.

### 4. The record shape

One record per message, carrying at least: a **message id** (for dedup), a **type
tag** (so the consumer knows what it is), the **serialized payload**, and a
**status/timestamp** (pending → sent). Keep it generic — the same record type
carries events and commands alike; the type tag and name distinguish them.

The message id is minted **when the record is written**, inside the producer's
transaction — never by the relay as it pushes. A relay-minted id is a new id on every
retry, so the key the consumer dedups on changes and a redelivery stops looking like
one: at-least-once quietly becomes apply-twice.

### 5. The relay mechanism

- **Scheduled poller** — periodically read pending records, push, mark sent.
  Simplest; at-least-once; latency bounded by the interval. The common default.
- **Log-tailing / change-data-capture** — a process tails the store's commit log
  and emits new records. Lower latency, no polling load; more infrastructure.
- **Store-native stream/trigger** — if the store can push on insert. Least code
  where available; ties you to that store.

### 6. The sink

- **In-process bus** — the relay pushes back onto the in-process event
  mechanism, so existing in-process subscribers keep working unchanged. This is what
  keeps the conversion a *walking skeleton*: nothing downstream notices yet.
- **Broker** — swapping the sink for a real message broker producer is the
  **only** edit needed to actually run producer and consumer as separate processes.
  Call this out: the crossing is now one line away from cross-process.

### 7. The serializer

The relay/store serializes the message and the consumer deserializes it. Pick one
serializer and use it **everywhere** — and the contract test (see
**message-contracts-and-tests**) must pin **that same** serializer, or the snapshot
guards bytes you never actually send. Mind visibility/config so plain value objects
round-trip.

## Consumer side — inbound adapter + inbox dedup

The handler on the receiving side is a **thin inbound adapter**: it takes the
delivered message and calls the consumer's public API — **no domain logic**
(hexagonal style). A class holding that public API, one `handle(<Message>)` method
that maps message fields to a call on it, registered so the sink delivers to it.

Because delivery is at-least-once, the adapter must be **idempotent**: a redelivery
of the same message id is a no-op. Two properties of that dedup are **invariants,
not menu choices** — get either wrong and at-least-once silently turns lossy or
double-applying:

- **The dedup record commits with the effect.** Recording "processed" and calling
  the consumer's public API must be **one atomic unit of work on one resource** — the
  consumer-side mirror of the producer's write-in-one-transaction (see
  **transactional-atomicity**). Commit them separately and a failure between them
  either loses the effect (marked processed, never applied) or double-applies it
  (applied, never marked → reapplied on the next delivery). This forces the handler
  to run **inside** that unit of work, so the delivery must be **synchronous** with
  it: handing the message to a *separate* thread or transaction (fire-and-forget
  async dispatch, or an after-commit hook) breaks the guarantee. If the sink cannot
  deliver synchronously into a shared transaction, you must make the effect
  idempotent some other way — and say so explicitly.
- **The dedup is an atomic claim, not a read-then-write.** "Have I seen this id? no
  → apply → remember" has a check-then-act race: two concurrent deliveries of the
  same id both pass the check and both apply. Claim the id with a **single atomic
  operation keyed by the message id** — e.g. an insert that no-ops on a unique-key
  conflict, an insert-if-absent / conditional put, or a compare-and-set — and apply
  the effect only if *this* caller won the claim. The unique key serializes
  concurrent redeliveries down to exactly one effect.

*Where* the ids are remembered **is** a gate decision: an **in-memory stub** is
acceptable for a walking skeleton (say it is a stub — it forgets on restart, so
at-least-once does not survive a restart); a **durable store** keyed by message id
gives real cross-restart dedup. Either way it must still satisfy both invariants
above — even the in-memory stub needs an atomic claim and the same unit of work as
the effect to be correct under concurrency.

**One more race, on the source side.** If more than one relay worker can read the
pending records, two workers may pick the *same* record (a plain read does not
reserve it). The atomic dedup above keeps this **correct** (one effect wins, the
other no-ops) but **wasteful** (duplicated work). To remove the waste, have the
relay claim records atomically too — a locking or skip-the-locked read, a status
compare-and-set, or a lease — so workers take disjoint batches. A single-worker
relay has no such race; note the limitation rather than over-building.

## Discover before you propose

Read the project before offering the menu: what is the producer's store and
transaction model, is there already a port/bus to swap behind, what scheduling or
CDC infrastructure exists, which serializer the codebase already uses. Recommend the
option consistent with that. When nothing relevant exists, say so and ask outright —
do not invent a project fact.
