---
name: transactional-atomicity
description: Whether a producer can write the outbox record in the SAME transaction on the SAME resource as its state change, so the message commits with that change — the rule, how to judge any producer, the verdict table, and the options when the guarantee does not hold. Use whenever routing a producer through an outbox.
---

# Transactional atomicity

The outbox is only as good as one fact: the outbox write must run in the **same
transaction** as the producer's state change, on the **same transactional
resource**. Otherwise you have at-least-once delivery with no atomicity — a
silently lossy or duplicating handoff (the state commits but the message is lost,
or the message goes out but the state rolls back). Check this every time; never claim
atomicity you do not have.

## How to judge any producer

Two questions, in order:

1. **Is the state change transactional?** Does the producing operation run inside a
   transaction (or unit of work) on its store, such that a failure rolls the change
   back?
2. **Does the outbox write land on that same transactional resource?** The outbox
   store and the producer's state must be the **same** resource, enrolled in the
   **same** transaction — not two stores you hope commit together.

Atomicity holds only when both answers are yes.

## Generalized verdict table

| Producer shape | Verdict |
|---|---|
| State change is transactional **and** the outbox write is on that same resource, in that same transaction | ✅ **atomic** — they commit or roll back together |
| Same resource, but the operation is **not** wrapped in a transaction | ⚠️ **atomic after a fix** — make the operation transactional so both writes enroll; then order of save-vs-publish no longer matters |
| Producer state lives in a **different resource** than the outbox store | ❌ **not atomic** — the two resources share no local transaction (short of a distributed 2PC, which the outbox exists to avoid); the guarantee does not hold |

## When it is not atomic — the options

If the producer's state and a shared outbox cannot live on one transactional
resource, do not paper over it with a bean swap. Surface it as an **architectural
decision** and offer:

- **Outbox in the producer's own store** — put the outbox where the state already
  is, so the two share that store's transaction. Best when the store supports it.
- **Make the operation transactional** — if the only gap is a missing transaction on
  an otherwise-shared resource, add it (a small, knowing change).
- **A different reliability mechanism** — e.g. derive the message from the
  producer's own committed state (listen-to-yourself / change-data-capture on the
  producer's store) instead of a separate outbox write.
- **Documented best-effort** — accept a lossy publish, explicitly, with the risk
  written down. Only if the consequence is genuinely tolerable.

## Report

For the call at hand, state the verdict (atomic / atomic-after-a-fix /
not-atomic-because-different-resource) and, if not atomic, the option chosen and its
consequence. A distribution that quietly drops the atomicity guarantee is worse than
the synchronous call it replaced — it fails silently instead of loudly.
