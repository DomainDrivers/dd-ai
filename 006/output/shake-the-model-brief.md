# Business brief — SmartSchedule

## What the business is

A company that runs many client projects at the same time and makes its money by
putting a limited pool of people and equipment to work on those projects as
profitably as possible. Every project needs particular capabilities — skills and
equipment — during particular time windows; the people and machines that can supply
those capabilities are finite and can each only be in one place at a time. The
business's daily work is the continuous matching of scarce capabilities to competing
project needs, the pricing of that matching for profit, and the early detection of
projects that are slipping toward a loss.

## The work, by activity

### Taking on and shaping projects
The business accepts projects and breaks each one into stages. Stages depend on one
another — some must finish before others can start, others can run side by side — and
part of shaping a project is working out a valid order and how much can run in
parallel, which fixes how long the project will take. Some stages are singled out as
critical: they carry the project and are planned first, sometimes pinned to a named
resource in a named window. For each project the business records what it must
deliver: the capabilities it will need and the time windows it needs them in — its
demands — sometimes broken down stage by stage. It records a possible start date and
derives a schedule from it, and can rework that schedule so the stages fit the actual
free time of the resources it means to use.

### Keeping the pool of people and equipment
The business maintains a catalog of the resources it can draw on: employees, known by
their seniority, skills and permissions, and devices, known by the capabilities their
assets provide. Both people and machines are described purely by the capabilities they
offer. Resources can be grouped — one resource can sit under a larger parent. A
resource is available only within limits, and the business tracks, at fine time
granularity, when each resource is free and when it is already taken. It also learns
of new employees from an existing external personnel record it does not itself
control, and folds them into its own picture of the pool. From time to time the
business invests in the pool — buying a new machine or training a new skill — and it
wants to know whether that investment will pay back.

### Committing resources to the work
The core operational act is committing a specific capability to a specific project
demand for a specific time window. Because the pool is shared, the same resource is
wanted by more than one project, so a commitment first secures that resource's time
against all rivals; releasing it later hands the time back. When a project's chosen
resources are named, the business checks up front that they are genuinely free in the
windows required before it counts on them. A commitment can be undone and the resource
handed to a more deserving project, in which case the project that lost it must cope
with the loss. At any moment a project's demands may be only partly met, and the
business keeps a running view of which demands are still unsatisfied. The business
tries to make these commitments the most profitable ones possible given everything
competing for the same finite capacity.

### Steering the portfolio for profit
The business treats its live projects as a portfolio and constantly weighs money
decisions across it. It records what each project earns and what it costs, and derives
each project's standing profit. It runs what-if questions before committing real money
or people: whether buying a particular priced capability would earn back more than it
costs, what the most profitable selection of work would be for a given amount of
capacity, and how much profit would change if a capability were moved off one project
and onto another. These what-if answers drive real choices — which projects to
favour, whether to relocate a resource mid-flight, and whether an investment in the
pool is worth making.

### Watching every project for risk and reacting
The business watches each project for signs it is heading for trouble: demands that
stay unmet, a resource that has been taken away by a competing project, earnings that
have been revised downward, or a critical stage coming up without the pieces in place.
It re-sweeps the whole portfolio on a regular weekly cadence as well as reacting to
these signals as they happen. When a project looks at risk the business acts: it flags
that demands are or are not satisfied, hunts for a free replacement resource, weighs
relocating a resource from elsewhere when that relocation is itself profitable, or
raises a risk alert for human attention. The sharpest reaction available to it is to
abandon a project that no longer promises a profit — ideally before it starts, so the
loss is avoided rather than incurred.

## Who is involved
- **The scheduling business itself** — the operator deciding, day to day, which
  resource serves which project when, and steering the portfolio for profit.
- **Clients and their projects** — the work the business takes on; each wants its
  capabilities delivered in its windows, and a late or failed delivery can carry a
  contractual penalty.
- **Employees and devices** — the people and machines whose finite capabilities and
  finite time are the thing being sold; they can be trained, bought, grouped, and
  moved between projects.
- **An external personnel record** — an existing source outside the business's control
  that announces when a new employee exists, which the business incorporates into its
  own pool.

## Where money and risk move
- **The business earns** a margin on projects delivered profitably; the headline
  health measure is the profitability of the whole portfolio — its margin and the
  share of projects delivered in profit.
- **The business spends and invests** in enlarging or upskilling the pool — buying
  equipment, training skills — and wants a positive return on each such investment,
  judged forecast against outcome.
- **The business saves** by spotting unpromising projects and killing them before they
  consume resources, counting the loss it avoided.
- **The business is exposed** to contractual penalties when work is delivered late or
  not at all, and treats penalties avoided or reduced as money made.
- **The scarce, perishable asset** is resource-time: an hour of a person or machine
  not sold is lost for good, and every commitment ties up capacity some other project
  wanted.

## Rules of the game
- A resource's time in a given window can be held by only one project at a time;
  competition for it is resolved by taking it, and the loser is left to react.
- A resource must be confirmed free in the required windows before the business relies
  on it.
- A project is only as orderable as its stage dependencies allow; a valid ordering
  must respect them.
- A commitment can be reversed and the freed time reassigned to a more profitable use.
- A project may run with some of its demands unmet, and that gap is itself watched as
  a risk signal.
- Killing a project is a legitimate and valued outcome, most valuable before the
  project starts.

## Vocabulary
- **Project** — a unit of client work the business takes on, broken into stages.
- **Stage** — a piece of a project; stages have dependencies and some are *critical*.
- **Demand** — a capability a project needs in a specific time window.
- **Capability** — a skill or piece of equipment a resource offers and a demand
  requires.
- **Resource** — an employee or device supplying capabilities within limited
  availability; may be grouped under a parent.
- **Availability** — when a resource is free versus already taken, tracked at fine
  time granularity.
- **Commitment** — assigning a specific capability to a specific demand for a specific
  window, securing the resource's time against rivals.
- **Portfolio** — the whole set of live projects, steered together for profit.
- **Earnings / cost / margin** — what a project brings in, what it consumes, and the
  profit between them.
- **Relocation / transfer** — moving a capability off one project and onto another.
- **Optimal setup** — the most profitable selection of work for a given amount of
  capacity.
- **Risk** — the standing chance a project will fail to deliver or fail to profit,
  watched continuously.

## Scale and cadence facts
- Many projects run in parallel, and the pool is shared across all of them.
- Resource availability is tracked at fine time-segment granularity.
- The whole portfolio's risk is re-evaluated on a weekly cadence, on top of
  event-by-event reactions.
