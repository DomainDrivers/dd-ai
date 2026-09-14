---
name: integration-classification
description: 'Classify a cross-module in-process call before distributing it: is the message an event (a fact, past tense, 0..N subscribers) or a command (an imperative Verb-Noun to one addressee), and is the call disqualified because the caller depends on the return value? Use at the start of a sync-to-async conversion.'
---

# Integration classification

Before you distribute a call, decide what kind of message crosses it. Get this
wrong and you distribute something that should not be distributed, or you name a
command like an event and the consumer reads intent that is not there.

## Find the call site first

If you were handed a description rather than a concrete call, locate the call site:
the place where one module invokes the other module's public API in-process. You
need the actual statement to judge fire-and-forget, because the judgement is about
what the **caller** does with the result.

## Fire-and-forget vs request/response (the disqualifier)

Read what the caller does with the return value of the cross-module call.

- **Fire-and-forget** — the caller ignores the result (void, or a value it discards
  and does not branch on). The effect in the other module is a *consequence*, not an
  *answer*. **Convertible.**
- **Request/response** — the caller cannot proceed without the result: it branches
  on it, returns it, or feeds it into the same unit of work. Distributing this
  changes **correctness**, not just topology — the caller would now race ahead of an
  answer that has not arrived. **Disqualified.** Stop and say so.

"Low cost to distribute" is a property of fire-and-forget calls only.

## The second disqualifier — who reads the effect next

Fire-and-forget says what the *caller* does with the result. It says nothing about the
next **reader** of the effect. Ask who observes the effect in the other module right
after the call — a later step in the same request, the next screen, another module's
decision, a test — and whether they need it to be there already.

- **Nobody needs it immediately** — the effect is genuinely a consequence.
  **Convertible.**
- **Somebody reads it in the same breath** — the code has read-after-write, and
  distributing the call replaces it with eventual consistency: that reader will
  sometimes see the world before the effect lands. This is a **business decision, not
  a refactor**. Put it to the user with the options — accept the delay, keep the call
  synchronous, or give the reader its own projection of the effect — and do not settle
  it yourself.

A call can pass the fire-and-forget test and still fail this one.

## Event or command

A convertible call carries one of two messages. The difference is intent, and the
name must carry it.

- **Event** — a *fact* one module publishes: something that already happened, past
  tense, with **0..N** subscribers who may or may not exist. If an event already
  flows on a bus and the consumer already reacts to it, it **stays an event** — you
  are only changing the transport, not the meaning.
- **Command** — an *instruction* one module sends another: imperative
  `<Verb><Noun>`, **one** logical addressee that is expected to act. Converting a
  **direct synchronous call** reads naturally as a command — there was no message
  before, so a sending port is introduced and the producer now depends on a message
  rather than on the consumer's public API.

Rule of thumb: if the producer is announcing and does not care who listens, it is
an event; if the producer is delegating a specific action to a specific module, it
is a command.

## A converted direct call — put both readings to the user

An existing event settles its own kind. A direct call does not: it carries no message
today, so the same crossing can be **sent** as a command or **announced** as an event,
and the choice moves where the knowledge lives. Offer both with the trade-off.

- **Command** — the producer keeps knowing *what the consumer must do* and addresses
  it directly. Closest to the call being replaced, easiest to follow, behaviour
  unchanged. The price: the producer's assumption about the consumer's job travels
  inside the message, so the logical coupling stays — only the transport changed.
- **Event** — the producer states *what happened on its own side* and stops there;
  reacting is the consumer's policy. Looser, and the usual reading when the consumer
  is a separate bounded context that should own its own reaction. The price: the
  reaction is no longer visible from the producer, the fact needs a subscriber before
  it means anything, and adding the next reaction no longer touches the producer.

Recommend from what you discovered — an event when the consumer owns a policy of its
own across a context boundary, a command when the action is the producer's own intent
and the consumer is simply the module that can carry it out — then stop and let the
user instruct the choice.

## Name by intent

- Event → **past tense**, describing the fact (`<Noun><Verbed>`).
- Command → **imperative** `<Verb><Noun>`, describing the instruction.

The name is the contract's headline. It tells the consumer whether it is free to
react however it likes (event) or is being asked to do a specific thing (command).

## What also changes — name the test that breaks

The existing test that drives the producer and asserts the *effect in the other
module* across the call cannot survive the change: once async, that effect has not
happened yet at call time. Identify it now; it is the test you will re-cut at the
new boundary (see **message-contracts-and-tests**).
