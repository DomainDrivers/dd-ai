# SmartSchedule — Modules & Capabilities

A module-by-module capability catalog, grounded in the facades and public
entry points under `src/main/java/domaindrivers/smartschedule/`.

---

## `availability` — `AvailabilityFacade`

The write-side contention core. Owns *who holds which resource in which time
segment* and enforces the blocking / releasing / taking-over of time slots at
segment granularity. A separate read model serves calendars for querying.

**operations →**
- `createResourceSlots(resourceId, timeslot)` — register a new resource's availability across a time slot (creates the segmented records).
- `createResourceSlots(resourceId, parentId, timeslot)` — same, but nests the resource under a parent resource.
- `block(resourceId, timeSlot, requester)` — try to lock a resource's slot for an owner; `true` if it succeeded.
- `release(resourceId, timeSlot, requester)` — give back a slot the owner previously held.
- `disable(resourceId, timeSlot, requester)` — forcibly take over a slot from its current owners; publishes `ResourceTakenOver`.
- `blockRandomAvailable(resourceIds, within, owner)` — block any one currently-free resource out of a set; returns the chosen id.
- `findGrouped(resourceId, within)` — load the grouped-availability aggregate for a resource in a slot.
- `loadCalendar(resourceId, within)` — read model: one resource's calendar (owner per segment).
- `loadCalendars(resources, within)` — read model: calendars for many resources at once.

---

## `allocation` — `AllocationFacade`

The hub tying projects to resources. Holds each project's *demands vs. actual
allocations* over a time window, drives availability blocking when allocating,
and emits allocation events consumed by risk. Note the deliberate cross-module
transaction in `allocateToProject`.

**operations →**
- `createAllocation(timeSlot, scheduledDemands)` — create a project-allocations aggregate with its demands and window; publishes `ProjectAllocationScheduled`, returns the id.
- `findAllProjectsAllocations(projectIds)` — allocation summary for the given projects.
- `findAllProjectsAllocations()` — allocation summary for all projects.
- `allocateToProject(projectId, allocatableCapabilityId, timeSlot)` — allocate a specific scheduled capability to a project, blocking its availability first; returns the allocated-capability id.
- `releaseFromProject(projectId, allocatableCapabilityId, timeSlot)` — release a capability from a project and free its availability.
- `editProjectDates(projectId, fromTo)` — change the project's allocation time window.
- `scheduleProjectAllocationDemands(projectId, demands)` — set/extend the demands a project needs.

*(package-private `allocateCapabilityToProjectForPeriod` picks any capability matching a `Capability` for a slot — used internally, not exposed.)*

### `allocation.cashflow` — `CashFlowFacade`

Project financials: records income and cost and derives earnings per project.
Publishes `EarningsRecalculated`, which risk listens to.

**operations →**
- `addIncomeAndCost(projectId, income, cost)` — record/update income & cost, recompute earnings, publish the change.
- `find(projectId)` — earnings for one project.
- `findAllEarnings()` — earnings for every project.

### `allocation.capabilityscheduling` — `CapabilityScheduler` + `CapabilityFinder`

The bridge between raw resources and allocatable work. The scheduler turns a
resource's capabilities into time-boxed *allocatable capabilities* (and
registers their availability); the finder queries them.

**`CapabilityScheduler` operations →**
- `scheduleResourceCapabilitiesForPeriod(resourceId, capabilities, timeSlot)` — materialize a resource's capabilities as allocatable capabilities for a period and register availability slots.
- `scheduleMultipleResourcesForPeriod(resources, capability, timeSlot)` — schedule one shared capability across many resources for a period.
- `findResourceCapabilities(resourceId, capability, period)` — resolve the allocatable-capability id for a resource + capability + period.

**`CapabilityFinder` operations →**
- `findAvailableCapabilities(capability, timeSlot)` — matching capabilities that are *actually free* in the slot (cross-checks availability calendars).
- `findCapabilities(capability, timeSlot)` — matching capabilities regardless of current availability.
- `findById(id)` / `findById(ids)` — load allocatable-capability summaries by id.

### `allocation` — `PotentialTransfersService`

A service (not a facade) that computes the profit change of moving a capability
to a project, by running before/after scenarios through the optimizer. Risk uses
it to decide whether a relocation is worth suggesting.

**operations →**
- `profitAfterMovingCapabilities(projectId, capabilityToMove, timeSlot)` — profit delta of transferring a capability into a project for a slot.

---

## `planning` — `PlanningFacade`

The project-scheduling brain. Creates projects, parallelizes their stages,
captures demands, chooses resources, and produces schedules (critical-stage,
manual, or availability-adjusted). Publishes planning events
(`CapabilitiesDemanded`, `CriticalStagePlanned`).

**operations →**
- `addNewProject(name, stages…)` — create a project, auto-parallelizing the stages.
- `addNewProject(name, parallelizedStages)` — create a project from an already-parallelized stage list.
- `defineStartDate(projectId, possibleStartDate)` — set a possible start date and derive the schedule.
- `defineProjectStages(projectId, stages…)` — (re)define stages and their parallelization.
- `addDemands(projectId, demands)` — add capability demands; publishes `CapabilitiesDemanded`.
- `defineDemandsPerStage(projectId, demandsPerStage)` — define demands broken down per stage; publishes `CapabilitiesDemanded`.
- `defineResourcesWithinDates(projectId, chosenResources, timeBoundaries)` — pick resources for the project inside a date range.
- `adjustStagesToResourceAvailability(projectId, timeBoundaries, stages…)` — recompute the schedule so stages fit the chosen resources' availability.
- `planCriticalStageWithResource(projectId, criticalStage, resourceId, stageTimeSlot)` — pin a critical stage to a resource + slot; publishes `CriticalStagePlanned`.
- `planCriticalStage(projectId, criticalStage, stageTimeSlot)` — pin a critical stage to a slot without naming a resource.
- `defineManualSchedule(projectId, schedule)` — set a fully manual schedule.
- `durationOf(stages…)` — total duration of the given stages.
- `load(projectId)` — load one project card (summary).
- `loadAll(projectsIds)` / `loadAll()` — load project cards.

---

## `simulation` — `SimulationFacade`

Answers "what-if" business questions on top of the optimizer — is buying a
capability worth it, and what's the best project selection for a given capacity.
Pure computation, no persistence.

**operations →**
- `profitAfterBuyingNewCapability(projectsSimulations, capabilitiesWithoutNewOne, newPricedCapability)` — profit delta of adding one priced capability (optimal-with − its price − optimal-without).
- `whatIsTheOptimalSetup(projectsSimulations, totalCapability)` — the optimal project/capability assignment and its total profit for the available capacity.

---

## `optimization` — `OptimizationFacade`

The domain-agnostic engine underneath simulation: a knapsack-style optimizer.
Given value-weighted items and a total capacity, it picks the subset maximizing
value while matching each item's weight dimensions to capacity dimensions.

**operations →**
- `calculate(items, totalCapacity)` — optimize with the default value-descending ordering.
- `calculate(items, totalCapacity, comparator)` — optimize with a custom item ordering; returns `Result` (profit, chosen items, item→capacities map).

---

## `resource` — `ResourceFacade` (+ `EmployeeFacade`, `DeviceFacade`)

The catalog of what resources exist and what they can do. Employees and devices
are registered here with their capabilities, and each can be turned into
scheduled allocatable capabilities. `ResourceFacade` is a thin aggregator over
the two.

**`ResourceFacade` operations →**
- `findAllCapabilities()` — every capability across employees and devices.

**`EmployeeFacade` operations →**
- `addEmployee(name, lastName, seniority, skills, permissions)` — register an employee; skills + permissions become capabilities.
- `findEmployee(employeeId)` — load an employee summary.
- `findAllCapabilities()` — all employee capabilities.
- `scheduleCapabilities(employeeId, oneDay)` — schedule the employee's capabilities as allocatable capabilities for a day.

**`DeviceFacade` operations →**
- `createDevice(model, assets)` — register a device with its asset capabilities.
- `findDevice(deviceId)` — load a device summary.
- `findAllCapabilities()` — all device capabilities.
- `scheduleCapabilities(deviceId, oneDay)` — schedule the device's capabilities as allocatable capabilities for a day.

---

## `risk` — `RiskPeriodicCheckSagaDispatcher`

A saga / process manager, not a facade — it has no callable API of its own. It
watches allocation, cashflow, and availability events plus a weekly timer,
advances a per-project risk saga, and reacts by notifying or hunting for
resource fixes.

**operations → (event & schedule handlers)**
- on `ProjectAllocationScheduled` — start/advance the project's risk saga.
- on `NotSatisfiedDemands` — record missing demands against each affected saga.
- on `EarningsRecalculated` — feed earnings into the saga.
- on `ResourceTakenOver` — react for every project that lost the resource.
- weekly cron — re-evaluate all sagas.
- resulting actions: notify demands-satisfied, find available replacements, simulate a *profitable* relocation (via `PotentialTransfersService` → simulation), or push a possible-risk alert.

---

## Structural note

`capabilityscheduling`, `cashflow`, and `PotentialTransfersService` all live
*inside* the `allocation` package as sub-modules rather than as top-level peers
— so "allocation" is really a cluster of four cooperating pieces, not one flat
module.
