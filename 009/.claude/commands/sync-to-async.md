---
description: Convert a synchronous in-process call between two modules into asynchronous messaging through a transactional outbox, eliciting every implementation choice at a gate — mock vs fake, how to build the outbox, which contract-test mechanism. Project- and stack-agnostic.
argument-hint: "[a public API method or call site that synchronously invokes another module — e.g. ModuleA#doX calling moduleB.act(...), or a plain description of the call]"
---

# /sync-to-async

Take a fire-and-forget synchronous call that crosses a module boundary and make
it asynchronous through an outbox — without the application service noticing, and
with the test safety net moved to the new boundary. This command **carries no
project facts**: at every decision it discovers what the codebase already does,
proposes an option grounded in that, and **stops for you to instruct the choice**.

The target is `$ARGUMENTS` — a concrete public API method and the call it makes, or a
description; if it is a description, locate the call site first. If it is empty, ask
for the call site before Step 0a.

The HOW lives in four skills — load them as the steps say: **integration-classification**
(what kind of message is this?), **outbox-mechanics** (the outbox menu),
**transactional-atomicity** (can the message commit atomically?), **message-contracts-and-tests**
(the doubles and the contract menu).

## Gate discipline (applies to every step)

Each step ends at a **gate**. At a gate: run the loop **discover → propose →
stop**.

- **Discover** — read the codebase; do not assume. Surface what is actually there.
- **Propose** — state the decision, list the realistic options with a one-line
  trade-off each, and recommend the one *consistent with what you discovered*
  (never a hardcoded default). If discovery found nothing, say so and ask outright.
- **Stop** — present the finding and the proposal as a short menu and wait for the
  user's decision. Do not advance, and do not build, until the user instructs the
  choice. Carry the confirmed choice forward; do not re-ask it later.

Never silently pick an implementation. The user instructs the choice at every gate.

## Commit cadence

Every substep that changes code commits on its own, once its gate is confirmed —
**1a, 1b, 2, 3, 4a, 4b**. After the user approves the diff at a build gate, make a
single focused commit for that substep before starting the next one, with a message
naming what the substep did (the rewritten producer test, the producer publish
change, the contract test, the outbox, the failing consumer test, the inbound
adapter). Commit a **RED** substep even though its test fails — each substep is its
own reviewable, taggable point. The discovery and design steps (**0a–0d**) change no
code and are **not** committed. Never squash substeps together, and never `git push`
unless asked.

## Step 0a — Discovery brief

Map the call without changing anything. First locate and **show
the call site** — the statement where the producer module invokes the consumer
module in-process (the method and the crossing call). The entry point is whatever
the project uses as a module's public API — a facade class, an application service,
an exported function, a handler. Then read the project for the
facts every later step is grounded in.

**Always present the discovery brief as this exact table** — one row per fact below,
each filled from what you actually found in the codebase, never assumed. If a row
has no answer in the code, write `none found`; do not invent it.

| Discovered fact | Finding |
|---|---|
| **Producer module** | the calling module (and the public API method that makes the call) |
| **Consumer module** | the called module (and the public API method invoked) |
| **Communication style** | direct synchronous call / existing event on a bus / existing sending port — and whether the return value is used (fire-and-forget vs request/response) |
| **Who reads the effect next** | who observes the consumer's effect right after the call — a later step in the same request, the next screen, another module's decision, a test — and whether they need it to be there already |
| **Producer storage** | where the producer's state lives and the persistence mechanism |
| **Transaction span today** | what the current transaction covers — one module, or both across the call |
| **Outbox today** | an existing outbox/relay to reuse, or none (greenfield) |
| **Ordering need** | whether the consumer depends on the order of these messages, and per what key |
| **Test double convention** | how the codebase doubles collaborators (mock / fake / spy, and the library) |
| **Integration infra** | what tests use to spin up the store/runtime (and the test framework) |
| **Test runner command** | the command that runs the project's tests, and how to run a single test |
| **Test that asserts the cross-module effect** | the existing test + assertion that checks the effect in the other module across this call — the one that cannot survive async |

If producer and consumer share the **same transactional resource**, flag early that
an atomic outbox is reachable (confirmed formally at the atomicity step).

**Gate.** Show the call site and this table, and ask the user to confirm or correct
it. Everything downstream is grounded in this map.

## Step 0b — Classify the call

Load **integration-classification**. Confirm the call is **fire-and-forget**: the caller
ignores the return value. If the caller branches on the result (request/response),
STOP — this call is expensive to distribute; say so and do not convert it. Then apply
the **second disqualifier**, on the reader rather than the caller: take "Who reads the
effect next" from the discovery brief and, if someone needs the effect to be there
already, present the options the skill lists and stop — that trade is the user's to
make, not yours. Decide
whether the message is an **event** or a **command**. A fact already flowing on a bus
stays an event; a converted direct call can be read either way, so put both readings
with their trade-off and let the user instruct it — a command introduces a sending
port, an event leaves the reaction to the consumer's own policy. Name it by intent.

**Gate.** Show the classification, the message name, and the test from the discovery
brief that will be re-cut. Stop for confirmation.

## Step 0c — Design the outbox (don't build yet) — optional if one already exists

**Skip this design when the discovery brief's "Outbox today" found an outbox/relay
already in place.** There is then nothing to design: reuse the existing outbox and
only confirm the two things still specific to *this* message — the **interception
mechanism** (how this producer hands the message in) and that the **serializer**
matches the contract test — then move on to the atomicity check. Design from
scratch only on a greenfield where no outbox was found.

Otherwise, load **outbox-mechanics**. Walk its menu, each item a sub-decision
grounded in the discovery brief: whether to **adopt an outbox** the project's
framework or a library already ships or to **hand-roll** one; the **interception
mechanism** (swap behind an
existing port/bus = zero producer edits; a direct call = introduce a sending port,
one-line producer change); the **store** (must be the producer's transactional
resource — see Step 0d); the **record shape**; the **relay mechanism** (scheduled
poller / log-tailing / store-native stream); the **sink** (in-process now, broker
later); the **serializer** (must match the contract test's serializer).

**Gate.** Present either the reused outbox plus the two confirmations above, or —
on a greenfield — the proposed outbox design as a menu of these choices, in both
cases with trade-offs and your discovery-grounded recommendation. Stop for the user
to instruct each choice. Do not write code yet.

## Step 0d — Atomicity check

Load **transactional-atomicity**. State plainly whether this producer can give the
outbox a write in the **same transaction on the same resource** as its state
change. If it cannot (the state lives in a different store, or the write is not
transactional), present the options — make the write transactional, put the outbox
in the producer's own store, or a documented best-effort — and their consequences.

**Gate.** State the verdict (atomic / atomic-after-a-fix / not-atomic-because-different-resource)
and, if not atomic, stop for the user to choose an option. Never claim atomicity
the producer does not have.

## Step 1a — Choose the producer test double, then rewrite the producer test (RED)

Load **message-contracts-and-tests**. The producer test must now assert it
**published the right message** and stop at the transaction boundary — at the
producer boundary the emitted message *is* the observable behavior. Ask which test
double to use for the publisher: **mock**, **fake**, or **spy**, recommending
whatever the project's existing collaborator-doubling convention is (from Step 0a).

If the discovery brief wrote `none found` for the test that asserts the cross-module
effect, there is nothing to re-cut. Say so, and put two options at this gate alongside
the double: write a **characterization test** on today's synchronous behaviour first,
watch it pass, and re-cut that one — or write the "message published" test straight
away and state that the old behaviour was never covered. Recommend pinning the effect
first when it is worth keeping.

**Gate.** Stop for the user's choice, then rewrite the cross-module assertion — or
write the test the `none found` branch chose — into the "message published" assertion
using that double. Run it with the project's test
command from Step 0a and show the output: the test **fails (RED)** — the producer
still calls the consumer synchronously, so no message is published yet. On
confirmation, **commit substep 1a** (the rewritten producer test) before Step 1b.

## Step 1b — Make the producer publish the message, greening the test (GREEN)

Now make the safety net green with the **trivial producer change**: replace the
synchronous cross-module call with a **publish/send through the message-publishing
abstraction** — an existing event publisher or bus the producer already owns, or a
**sending port introduced** for a converted command (Step 0b). This is the one-line
change the rewritten test was written to drive; the application service is otherwise
untouched and does not know the call became a message.

Publish **in-process for now** through whatever abstraction exists or was just
introduced — durability through the outbox is wired **behind that same abstraction**
later (Step 3), without touching the producer again.

**Gate.** Show the one-line diff, run the test again, and show the output: the Step
1a test now **passes (GREEN)**. Stop for confirmation, then **commit substep 1b** (the producer publish change) before Step 2.

## Step 2 — Choose the contract-test mechanism, then pin the contract

The serialized shape is now a wire contract. Ask which mechanism to pin it with: a
**snapshot/approval file**, a **consumer-driven contract**, a **message-contract
library**, a **schema-registry compatibility check**, or a **plain
serialize-and-compare** — recommending whatever fits the project's stack and test
conventions. Whichever is chosen, it must use the **same serializer the outbox
ships**, with fixed ids and timestamps in the fixture.

**Gate.** Stop for the user's choice, then add the contract test, naming the
message by intent (Step 0b). Run it and show the output. On confirmation, **commit
substep 2** (the contract test) before Step 3.

## Step 3 — Build the outbox and route the message through it

Now implement the outbox per the design confirmed in Step 0c, and route the message
through it **opt-in and non-invasively** by swapping the implementation **behind the
same publishing abstraction Step 1b introduced** — the producer code stays exactly as
1b left it, only the message now lands in the outbox instead of being delivered
in-process. Honor the atomicity decision from Step 0d.

**Gate.** Show the diff. Stop for confirmation, then **commit substep 3** (the outbox
and the routed message) before Step 4a.

## Step 4a — Choose the consumer test levels, then write the failing consumer test(s) first

Start the consumer side with a test, before the adapter exists. Load
**message-contracts-and-tests** (*Consumer-side*). Ask which test levels to add: a
**unit** test (construct the adapter with a doubled consumer public API, call
`handle(message)`, verify the call on it) and/or an **integration** test (on the project's discovered
integration infrastructure: enqueue/relay the message, let it be handled, assert
through the consumer's **real public API** that the effect happened). Recommend the levels
the project's test pyramid already supports.

Whichever levels are chosen, write them **failing first** — they define the
adapter's contract before a line of it exists. In the integration test, include the
**idempotency case**: deliver the **same message twice** and assert the effect
happened **exactly once** — the test that will prove the inbox dedup holds.

**Gate.** Stop for the user's choice of levels, then write the chosen test(s). Run
them and show the output: they fail (RED). On confirmation, **commit substep 4a** (the failing consumer test(s))
before Step 4b.

## Step 4b — Choose inbox dedup, build the inbound adapter to green, then report

Now build the thin inbound adapter that takes the delivered message and calls the
consumer's public API — no domain logic, hexagonal style — to make the Step 4a test(s)
pass. Delivery is at-least-once, so the handler must be idempotent. Per
**outbox-mechanics** (*Consumer side*), two things are **invariants, not choices**:
the dedup record must commit in the **same unit of work as the effect** (so delivery
into the handler is synchronous, not async), and the dedup must be an **atomic claim
keyed by the message id** (insert-if-absent / no-op-on-conflict / compare-and-set),
never a read-then-write. The genuine **gate choice** is only *where* the ids live:
ask whether to use an **in-memory stub** (a walking skeleton — say it forgets on
restart) or the project's own durable store, and say which it is. If the chosen sink
cannot deliver synchronously into the effect's transaction, surface that and make
the effect idempotent another way.

**Gate.** Stop for the user's choice of dedup store, then build the adapter so it
honors both invariants. Run the Step 4a test(s) and show them green. On
confirmation, **commit substep 4b** (the inbound adapter and inbox dedup).

Then report: what is atomic on the producer side (and whether you added a
transaction); that delivery is at-least-once, so the handler is idempotent via a
dedup that is an atomic claim committed in the **same unit of work as the effect**
(and that this depends on synchronous delivery); that async removed temporal but
not logical coupling; whether delivery order is preserved for this message, and how
if the consumer needs it; and that the single edit left to go cross-process is swapping
the relay's sink for a broker (which reintroduces the source-side claim race —
note whether the relay needs an atomic/locked read then).

**Final gate.** Summarize the diff and the verification. The per-substep commits
already cover every code change; do not add a squashed wrap-up commit, and do not
`git push`, unless asked.
