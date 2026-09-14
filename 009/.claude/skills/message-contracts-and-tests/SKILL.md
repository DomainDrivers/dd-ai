---
name: message-contracts-and-tests
description: 'How to re-cut the tests when a cross-module call becomes async: a producer-side "message published" test with a chosen double, a contract test pinning the serialized shape with a chosen mechanism, and a consumer-side inbound-adapter test at chosen levels, each with its menu of options. Use when rewriting tests around a newly distributed cross-module call.'
---

# Message contracts and tests

When the call goes async, the old test that drove the producer and asserted the
consumer's effect across the call no longer holds — once async, that effect has not
happened at call time. Replace it with three tests. Each carries one **choice** you
put to the user at a gate; recommend the option consistent with the project's
existing test conventions (discover them first), never a hardcoded default.

## 1. Producer-side — "the message was published"

Drive the producer up to publication; assert it published the **right message**,
then stop at the transaction boundary. Do not reach into the consumer. At the
producer boundary the emitted message **is** the observable behavior, so doubling
the publisher here is correct, not a smell.

**Choice — which test double for the publisher?**

- **Mock** — assert the publisher was called with the expected message (interaction
  verification). Fits codebases that already verify collaborator interactions; most
  direct way to say "this message, with these fields, was published".
- **Fake** — a lightweight working publisher that records what it received; the test
  then inspects the recording (state verification). Reads less coupled to the call
  shape; reusable across tests.
- **Spy** — wrap the real publisher and observe calls. Use when you also need the
  real side effect; otherwise heavier than needed.

Recommend whatever the project already uses for collaborators. Whichever is chosen,
the assertion is on the **message**, not on downstream effects.

## 2. Contract — pin the serialized shape

The serialized shape is now a **wire contract**: the consumer deserializes exactly
these bytes. Pin it so an accidental field rename or type change fails a test
instead of a consumer.

**Choice — which contract mechanism?**

- **Snapshot / approval file** — serialize a fixed fixture, compare to a committed
  golden file; first run writes it, later runs diff against it. Simplest; lives with
  the producer; you own the file.
- **Consumer-driven contract** — the consumer declares what it needs; the producer
  verifies it still satisfies it (e.g. a Pact-style broker). Strongest cross-team
  story; most infrastructure.
- **Message-contract library** — a library that snapshots and checks
  backward/forward compatibility of a message shape. Adds compatibility checks over
  a plain snapshot; ties you to the library and its runtime.
- **Schema-registry compatibility** — register the schema and let the registry
  enforce compatibility on change. Natural when a broker + registry already exist.
- **Plain serialize-and-compare** — hand-rolled: serialize the fixture, assert on
  the string/bytes. Zero dependencies; the graceful-degradation fallback when none
  of the above is present.

Two rules whatever you pick: use the **same serializer the outbox ships** (config
and visibility identical), or the contract guards bytes you never send; and **fix
the ids and timestamps** in the fixture, or random values change the bytes every
run. As the message evolves, prefer a mechanism that can assert
backward/forward-compatibility.

## 3. Consumer-side — the inbound adapter

Drive the inbound adapter with the message; assert through the consumer's **public API**
that the effect happened. The adapter holds no domain logic, so the test is about
wiring: message in → correct public API call → effect.

**Choice — which test levels?**

- **Unit** — construct the adapter with a doubled public API, call `handle(message)`,
  verify the call on it. Fast; proves the mapping.
- **Integration** — on the project's real integration infrastructure (whatever it
  already uses to spin up the store/runtime in tests), enqueue/relay the message and
  assert via the real public API. Proves the round-trip: serialize → outbox → relay →
  adapter → effect. Add an **idempotency case** here — deliver the same message
  twice, assert the effect happened **exactly once**. This is the test that proves
  the inbox dedup holds: it is the only place the dedup runs against the real store,
  so it is where you confirm the dedup is an atomic claim committed in the same unit
  of work as the effect (see *Consumer side* in **outbox-mechanics**), not a
  read-then-write that a redelivery slips past.

Recommend the levels the project's test pyramid already supports; build the ones the
user instructs.

## Surface environment constraints — do not hardcode them

Test-double and serialization tooling can carry language/runtime constraints
(mocking concrete classes, module/visibility access, runtime flags, a library that
must be fetched before first compile). When you hit one, **surface it at the gate**
as a consequence of the chosen mechanism — do not assume a fixed environment, and do
not bake a specific runtime quirk into this skill.
