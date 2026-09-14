---
name: four-layer-test-architecture
description: The four-layer test model (Test Cases / DSL / Protocol Driver / System Under Test) and the raise/lower mechanic that lets you swap an implementation idiom while keeping the same behavior tests green. Use whenever a restyle needs to decouple scenarios from an implementation so the idiom can change underneath them.
---

# Four-Layer Test Architecture

A restyle changes *how* a unit is implemented without changing *what* it does.
The only way to prove "what it does" stayed the same is to run the **same
behavior tests** against the new implementation. The four-layer model is what
makes that possible: it separates the part of a test that describes behavior
(stable) from the part that talks to the implementation (swappable).

```
   Test Case   Test Case   Test Case   Test Case      <- behavior, STABLE
   \______________  ____________________/
                  \/
                 DSL                                   <- domain vocabulary, STABLE
                  |
        Protocol Driver(s)                             <- binds DSL -> SUT, SWAPPABLE
                  |
         System Under Test                             <- the implementation idiom, SWAPPABLE
```

## The four layers

| Layer | What it is | Changes during a restyle? |
|---|---|---|
| **Test Case** | One scenario: arrange → act → assert, written purely in domain terms. | **No.** If a test case must change, the restyle is not behavior-preserving — stop. |
| **DSL** | The vocabulary the test cases speak (domain verbs like `<command>(...)` plus assertable outcomes like `expect<Result>(...)`). Domain verbs, no implementation types. | **No.** The DSL is the contract surface. |
| **Protocol Driver** | The adapter that turns each DSL call into real calls against the SUT, at a chosen *binding level*. | **Yes.** This is the seam you swap. |
| **System Under Test** | The actual code: the source idiom, then the target idiom that replaces it. | **Yes.** This is the idiom you swap. |

The whole method rests on one inversion: **most test suites fuse the Test Case
and the Protocol Driver together** — the test method calls `new SomeImplementationType()`
and `instance.doThing()` directly. Splitting them apart is what unlocks the
restyle. Once the scenarios speak only DSL, you can re-point the driver at any
idiom and the scenarios do not notice.

## Binding levels

A Protocol Driver attaches the DSL to the SUT at one of three levels:

| Level | Binds to | Speed | Coupling to idiom | Use as |
|---|---|---|---|---|
| `domain` | the domain object or pure function directly (the source idiom's methods; the target idiom's core) | fastest | **high** — knows the idiom | fast inner-loop feedback on one idiom |
| `application` | the application facade / use-case service | fast | **none** — idiom-agnostic | the **contract level**; portable across idioms |
| `system` | the transport edge (HTTP / CLI / message) | slow | none | full end-to-end confidence (optional) |

The `application` level is special: the facade is the public contract, so a
driver bound there does not know or care which idiom lives below it. **Scenarios
become portable the moment their driver binds at `application`.**

## The two moves: raise and lower

- **Raise** = move a scenario's binding level *up* (e.g. `domain → application`).
  You raise **before** swapping an idiom, to detach the scenarios from the
  current implementation and turn them into an idiom-agnostic contract.
- **Lower** = add a binding *down* (e.g. `application → domain`) against the
  **new** idiom. You lower **after** swapping, to get fast white-box feedback on
  the target idiom's core.

A full restyle is therefore: **raise → swap → lower**, and the same Test Cases
ride through all three steps unchanged.

```
Start:   TestCase --(driver: domain)--> source idiom
Raise:   TestCase --(driver: application)--> facade -> source idiom    (green)
Swap:    TestCase --(driver: application)--> facade -> target idiom    (green)  <- behavior proven preserved
Lower:   TestCase --(driver: domain)----> target idiom core    (green)  <- fast feedback on new core
```

## Designing the DSL

The DSL is a small set of domain verbs plus an outcome you can assert on. Keep it
honest:

- **Verbs name domain actions**, not implementation calls: name each one after a
  command the unit performs — never `callSomeMethod(...)` or `insertRow(...)`.
- **Outcomes are observable facts**, not internal state: a domain result the test
  can compare ("the operation happened and this observable result holds"), not a
  peek at internal fields. Whether that value comes from a returned event, a read
  projection, or a query facade is the *driver's* problem, not the test's.
- **One DSL, many drivers.** The DSL type/interface is written once. Each binding
  level gets its own driver implementing the same DSL.

### Sketch (language-neutral; names are placeholders for your unit's vocabulary)

```
interface UnitDsl {
    CommandOutcome doCommand(args...);   // one verb per command the unit performs
    QueryOutcome   observeResult();      // an observable fact the test asserts on
    ...
}

class DomainDriver      implements UnitDsl { /* calls the source idiom or the target idiom's core */ }
class ApplicationDriver implements UnitDsl { /* calls the facade, observes via query facade */ }
```

A Test Case is written against `UnitDsl`. Which driver it gets is wiring,
chosen once in a scenario base class — never inside a test body.

## Rules

1. **Test Cases and DSL never change during a restyle.** They are the frozen
   contract. A required change to either means the swap altered behavior — stop
   and surface it.
2. **Only the Protocol Driver and the SUT change.** Everything else is held.
3. **Every step ends green.** Raise → green. Swap → green. Lower → green. There
   is no "temporarily red while restyling" state that survives a gate.
4. **Drivers contain no assertions.** Drivers translate DSL → SUT and return
   observable outcomes. Assertions live only in Test Cases.
5. **The contract-level driver (`application`) is the source of truth for
   behavior preservation.** `domain`-level drivers are for speed, not for the
   preservation proof — a `domain` driver is idiom-specific and gets rewritten
   per idiom, so it cannot, by itself, prove two idioms agree.

## Do not

- Do not let a Test Case construct an implementation type (`new SomeImplementationType()`)
  or call an implementation method directly. That fuses Test Case and Driver and
  defeats the model.
- Do not put behavior logic in the DSL or the driver — they only translate and
  observe.
- Do not assert on storage rows, internal ids, or framework artifacts from a Test
  Case. Assert on domain outcomes the DSL exposes.
- Do not change a Test Case to make a new idiom pass. Change the driver; if the
  driver cannot make it pass without altering the assertion, the behavior changed.

## Checklist before a restyle proceeds

- [ ] Scenarios are expressed in DSL verbs, not implementation calls.
- [ ] At least one driver binds at the `application` (contract) level and is green.
- [ ] The DSL exposes every outcome the scenarios assert on (no peeking below it).
- [ ] Wiring (which driver a scenario uses) lives in one place, not in test bodies.
- [ ] You can name, in one sentence, what each layer would have to change for the
      restyle — and only "Protocol Driver" and "SUT" are on that list.
