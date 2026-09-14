---
name: competitor-scan
description: How to distill a short, grounded digest of what the competition offers that
  this business may lack — customer-facing capabilities, each carrying a source-ref
  (a competitor page, a provided competitor document, or a name the user gave), stripped
  of the competitor's own solution structure and filtered to the ones that plausibly
  bear on the metric under test. The digest is a second business-level input the
  firewalled requirement generator may read alongside the brief, so it must stay as
  solution-free as the brief and never invent a competitor or a feature. Load with the
  business brief whenever the test is framed; it also defines when the digest stays empty.
---

# The competitor digest

The brief says what *this* business does. The competitor digest says what *rival*
businesses in the same market already offer that this one may not — a second well of
plausible future asks. The generator reads it alongside the brief and turns a **few** of
its entries into requirements, so the digest carries the same load-bearing discipline as
the brief: every entry grounded, and no solution structure in it.

The digest is business-level market intelligence, not architecture. That is exactly why
it may cross the firewall: it describes what customers can get elsewhere, never how any
system — this business's or the competitor's — is built.

## Identifying the competition

Find the competition from whatever the run makes available, in this order of trust:

- **Provided competitor material** — a competitor's page, brochure, feature list,
  review, or comparison the project or user hands in. Highest trust; quote from it.
- **Names the user gives** — the user points at specific rivals. Research those.
- **Web research on an identifiable market** — when the brief makes the market
  identifiable **and the user has said yes to a web pass**. Name the rivals you would
  search for and get that yes first: the search terms come from the business's own
  facts. Then read those players' public, customer-facing descriptions of what they
  offer.

Never invent a competitor, and never invent a feature to attribute to one. A rival you
cannot name and a capability you cannot point to do not go in the digest. If none can be
grounded, the digest is empty and the run proceeds on the brief alone — say so plainly
rather than pad it.

## What goes into an entry

Each entry is **one capability a competitor offers, described as a customer-facing
business ability** — what a customer gets, in the business's own vocabulary — plus:

- **who offers it** — the competitor(s) the capability was observed at,
- **source-ref** — where it was observed: `web:<url>`, `doc:<provided material>`, or
  `user:<name the user gave>`. Every entry carries one; an entry with no ref is dropped,
- **gap check** — whether this business appears to already do this (from the brief) or
  visibly lacks it. Prefer the gaps; a capability the business already has is weak fuel
  for a *future* ask,
- **metric bearing** — the metric or metrics under test the entry speaks to, and one
  line on why the capability plausibly bears on them. Let the generator do the full
  mechanism; the digest only needs the thread that makes the entry worth passing on. One
  digest serves the whole run: with several metrics in play, tag each entry with the
  ones it bears on, and leave out an entry that bears on none.

## Strip the competitor's solution too

A competitor's feature page is a solution artifact just like a context map — harvest the
business capability and discard the form. The strip list from the `business-brief` skill
applies unchanged: no module or service names, no integration mechanisms, no
pattern/paradigm labels, no diagram topology — neither the competitor's nor a guess at
this business's. "Rival X lets a customer reprice a booking themselves without calling
support" is a capability; "Rival X exposes a self-service pricing API" is not.

Run the same **invariance test** on every entry: would it read the same whether the
competitor ran on a monolith, twenty services, or a call centre with spreadsheets? If a
sentence only makes sense as technology, it is describing the competitor's solution —
rephrase to the customer-facing ability or drop it.

## A short research pass, not a market survey

This is a **quick** scan — a handful of the most telling capabilities, not an exhaustive
competitive study. A few well-chosen, well-grounded entries beat a long list; the
generator turns at most a few of them into requirements anyway. When you reach the web,
keep it to a short pass: a query or two, read the rivals' own customer-facing pages, and
stop once you have the handful that most clearly (a) bear on the metric under test and
(b) are a gap for this business. Depth on the metric beats breadth across the rival's
whole catalogue.

Leave out anything that cannot be observed publicly and grounded: a competitor's
internal pricing, its costs, its roadmap rumours, its tech stack. Report only what a
customer of the rival could actually see and get.

## Output shape

A short list, one entry per capability:

```
C<n>. Capability: <customer-facing ability, business terms>
      Offered by: <competitor(s)>
      Source: web:<url> | doc:<material> | user:<name>
      Gap: <this business appears to lack this | already does a version of this — how>
      Bears on: <the metric(s) under test this entry speaks to>
      Metric bearing: <one line: why it plausibly moves them>
```

Then, in one line, state the coverage: which markets/rivals were reached, and whether
web research ran or only provided material was used — so a reader can judge how complete
the scan is. If nothing could be grounded, say that, and the digest is empty.
