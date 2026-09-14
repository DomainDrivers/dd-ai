# Algorithm scan — per area

**Input**: `areas.md` (6 candidate areas, frozen 2026-07-07) · **Source of truth**: `corpus.md` (57 quotes, 4 sources)
**Catalog**: `algorithm-catalog` skill (42 canonical entries) · **Scanned**: 2026-07-08

This report surfaces canonical algorithms hidden behind the construction-project
domain vocabulary. Both matching mechanisms were run per area — a **bridge-word**
lexical pass over the verbatim (Polish / mixed-language) quotes, and an
independent **structural** reading of each area's shape. A candidate is listed if
*either* fired; agreement (`match_basis: both`) only raises confidence. Quotes are
preserved verbatim in their source language and never translated.

The single loudest finding: a **stage-precedence DAG with durations** is the
substrate under four of the six areas. Recognising it as **Job scheduling /
Critical path** (a polynomial, textbook problem) directly answers the CEO's
explicit ask for an arrangement algorithm that is *"tańszy i działający dobrze …
działa szybciej … modyfikowalny"* — the expensive proprietary schedulers he
complains about are solving a problem with a known O(V+E) core.

---

## Area: `project-phases` — *fazy projektu*

```yaml
area_id: "project-phases"
name: "fazy projektu"
algorithm_candidates:
  - algorithm_id: "job-scheduling"
    algorithm_name: "Job scheduling / Critical path"
    family: "Graph"
    match_basis: "both"
    fit_question: "Do tasks carry durations, deadlines, or resource needs on top of a precedence order, so the question is 'what schedule finishes soonest / by when?'"
    bridge_words:
      - quote: "zaplanowano etapy projektu (długość, zależności między etami, ewentualne zasoby)"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor stormingu: dyrektor (director)"
        word: "długość"          # duration
      - quote: "zaplanowano etapy projektu (długość, zależności między etami, ewentualne zasoby)"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor stormingu: dyrektor (director)"
        word: "zależności między etami"   # dependencies between stages
      - quote: "start date saved (event) → stage dates (schedule) calculated (event) → project end date calculated (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Scheduling flow — automatic mode"
        word: "schedule / end date"
      - quote: "edit stage duration (command) → Stage duration set (event) → later stages datess calculated (event) → project end date calculated (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Reschedule / reallocation flow"
        word: "duration / later stages dates calculated"
    domain_to_algorithm_mapping:
      "Etap (stage)": "Node"
      "zależności między etapami / 'zależą od etapów w fazie pierwszej'": "Directed edge (predecessor stage → successor stage)"
      "długość / stage duration": "Node (or edge) weight"
      "project end date calculated": "Longest weighted path = critical path length"
      "etapy … niezależne … wykonywane sekwencyjnie (stages parallized)": "Nodes with no edge between them (schedulable in parallel)"
      "faza (phase)": "A precedence layer / antichain of independent stages"
      "opóźnienie / later stages dates recalculated": "Forward-pass date shift; slack on non-critical stages"
    near_misses:
      - "precedence only, no timing → topological-sort"
      - "stages compete for limited resources (RCPSP) → job-scheduling NP-hard variant (list-scheduling / LPT heuristics)"
    confidence: "high"
    rationale: >
      Two mechanisms agree. Lexically, the storming board attaches an explicit
      duration (`długość`) and dependencies to each stage and the process board
      computes `stage dates (schedule)` and a `project end date` from a start
      date — the exact inputs and output of the Critical Path Method. Structurally
      the phase/stage model IS a precedence DAG with weights: independent stages
      inside a phase are unconnected nodes (parallel), each next phase depends on
      the prior, and editing one stage's duration re-computes all `later stages
      datess` and the end date — a CPM forward pass. This is the algorithm the CEO
      says he wants cheaper and faster; the good news is its core is polynomial.
    skiena_pointer: "job-scheduling"

  - algorithm_id: "topological-sort"
    algorithm_name: "Topological Sort"
    family: "Graph"
    match_basis: "both"
    fit_question: "Must items be flattened into one order that honours every 'must-come-before'?"
    bridge_words:
      - quote: "Faza projektu, czyli to jest taka część projektu, która zawiera etapy, które od siebie są niezależne. Mogą być wykonywane sekwencyjnie, a w kolejnych fazach etapy są od siebie zależne."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L81–83"
        word: "zależne"          # dependent
      - quote: "Czyli jak w faze drugą coś wpadło, tam mamy kilka etapów, to wiadomo, że one zależą od etapów w fazie pierwszej."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L84–85"
        word: "zależą od etapów"  # depend on stages
      - quote: "what is it the order of stages?"
        source: "intake/process level.es.md"
        loc: "Hot zones — Stage ordering"
        word: "order of stages"
      - quote: "Stages (tree) → Define stages (tree) → stages parallized → stages defined"
        source: "intake/process level.es.md"
        loc: "Lane: Scheduling flow — automatic mode"
        word: "stages parallized"
    domain_to_algorithm_mapping:
      "Etap (stage)": "Node"
      "'zależą od etapów w fazie pierwszej' (depends on)": "Directed edge (predecessor → successor)"
      "valid execution sequence / order of stages": "Topological order"
      "contradictory dependencies": "Cycle (would make the plan impossible)"
    near_misses:
      - "durations and a computed end date also matter → job-scheduling"
      - "if cyclic/contradictory stage dependencies must be kept and minimally broken → feedback-arc-set"
    confidence: "medium"
    rationale: >
      The bare ordering question ('what is it the order of stages?', stages that
      `zależą od` earlier stages) is a textbook topological sort. It is listed at
      medium rather than high because the evidence carries durations and an end
      date on top of the ordering, which redirects the deep fit to job-scheduling
      — topological sort is the un-timed skeleton of the same DAG. A cycle in the
      dependency web would signal contradictory constraints and falls out of the
      same traversal.
    skiena_pointer: "topological-sorting"
top_pick: "job-scheduling"
```

---

## Area: `project-risk` — *ryzyka projektowe*

```yaml
area_id: "project-risk"
name: "ryzyka projektowe"
algorithm_candidates:
  - algorithm_id: "transitive-closure"
    algorithm_name: "Transitive closure & reachability"
    family: "Graph"
    match_basis: "both"
    fit_question: "Must you answer 'can A eventually reach / influence / impact B?' across a chain of relationships, not just direct links?"
    bridge_words:
      - quote: "Trudnym ogólnie zagnieżdżeniem jest ocena wpływu ryzyka na kolejne zagnieżdżone etapy w fazach projektu. Czyli np. jak ryzyko opóźnienia w fundamentach, wpływa na stropy i elektrykę na takie dwa kolejne etapy."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L55–58"
        word: "wpływu ryzyka na kolejne zagnieżdżone etapy"   # impact of risk on subsequent nested stages
      - quote: "Jak ryzyko w jednym etapie wpłynie na kolejne fazy? Potrzebujemy mechanizmu, który pomoże nam wykrywać takie zagnieżdżone ryzyka i reagować na nie odpowiednio wcześnie."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L16–18"
        word: "wpłynie na kolejne fazy"   # will affect subsequent phases
      - quote: "Oczywiście, że wpływa, bo jak jedno się opóźni, to naturalnie drugie się opóźniają. Ale takich zagnieżdżeni dużo więcej, chciałbym wykrywać takie zagnieżdżenia."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L59–60"
        word: "wykrywać takie zagnieżdżenia"   # detect such nestings
    domain_to_algorithm_mapping:
      "Etap (stage)": "Node"
      "'wpływa na kolejne etapy' (directly affects the next stage)": "Directed edge"
      "'jak ryzyko … wpłynie na kolejne fazy' (eventually affects downstream)": "Closure edge (reachability)"
      "the full set of stages a risk touches ('takie zagnieżdżenia')": "Reachable set = blast radius"
      "fundamenty → stropy → elektryka (foundations → ceilings → electrical)": "A concrete reachability chain"
      "minimal non-redundant cascade web": "Transitive reduction"
    near_misses:
      - "if you need the chain as an *order*, not the reachable set → topological-sort"
      - "if you need *how much* the end date slips, not *which* stages are hit → job-scheduling"
    confidence: "high"
    rationale: >
      A near-perfect structural fit. The CEO's hardest problem is stated as
      reachability: a delay risk in `fundamenty` propagates to `stropy` and
      `elektryka` — 'jak jedno się opóźni, to naturalnie drugie się opóźniają' —
      and he wants to 'wykrywać takie zagnieżdżenia', i.e. compute the full
      downstream set an originating risk touches. That is exactly the reachable
      set over the stage-dependency DAG. Both mechanisms fire: the propagation
      vocabulary is explicit and the structure is a closure over the same graph
      the phases area builds.
    skiena_pointer: "transitive-closure-and-reduction"

  - algorithm_id: "job-scheduling"
    algorithm_name: "Job scheduling / Critical path"
    family: "Graph"
    match_basis: "both"
    fit_question: "Do tasks carry durations/deadlines on top of precedence, so the question is 'what schedule finishes soonest / by when?'"
    bridge_words:
      - quote: "Oczywiście, że wpływa, bo jak jedno się opóźni, to naturalnie drugie się opóźniają."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L59–60"
        word: "opóźni … drugie się opóźniają"   # one delays → the next delays
      - quote: "edit stage duration (command) → Stage duration set (event) → later stages datess calculated (event) → project end date calculated (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Reschedule / reallocation flow"
        word: "later stages datess calculated"
      - quote: "monitoring postępu (event) → wykryto opóźnienie (risk)"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor: maneger projektu"
        word: "wykryto opóźnienie"   # delay detected
    domain_to_algorithm_mapping:
      "Etap (stage)": "Node"
      "precedence between stages": "Directed edge"
      "opóźnienie (a delay added to a stage)": "Increased node/edge weight"
      "'drugie się opóźniają' (downstream date shift)": "Recomputed critical path (forward pass)"
      "the slipped project end date": "Deadline violation / lost slack"
    near_misses:
      - "if only *which* stages are affected matters, not the magnitude → transitive-closure"
    confidence: "medium"
    rationale: >
      The *qualitative* cascade is transitive closure; the *quantitative* version —
      'how many days does foundations-delay push the end date' — is a critical-path
      forward pass, and the process board literally models it as `later stages
      datess calculated → project end date calculated`. Delay propagation and
      schedule propagation are the same recomputation, so job-scheduling is the
      companion algorithm to transitive-closure here, keyed to the same DAG.
    skiena_pointer: "job-scheduling"
top_pick: "transitive-closure"
```

---

## Area: `profitability` — *opłacalność*

```yaml
area_id: "profitability"
name: "opłacalność"
algorithm_candidates:
  - algorithm_id: "linear-programming"
    algorithm_name: "Linear / Integer programming (LP/IP)"
    family: "Optimization"
    match_basis: "both"
    fit_question: "Can the goal be written as maximize/minimize a linear score subject to linear constraints?"
    bridge_words:
      - quote: "optymalizcja zasobów przez przenoszenie pomiędzy projektami"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor stormingu: dyrektor (director)"
        word: "optymalizcja zasobów"   # resource optimization
      - quote: "optymalizacja rozkładu zasobów"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor stormingu: dyrektor (director)"
        word: "optymalizacja rozkładu zasobów"   # optimization of resource distribution
      - quote: "firm gets paid for executing project, we want to optimize saving and margin"
        source: "intake/user-answers.md"
        loc: "Q1 inline answer"
        word: "optimize saving and margin"
      - quote: "revenue"
        source: "intake/user-answers.md"
        loc: "Q5 inline answer"
        word: "revenue"
    domain_to_algorithm_mapping:
      "decision: move resource / add machine / hire contractor / upskill": "Decision variable"
      "saving, margin, revenue": "Linear objective function (maximize)"
      "resource availability, budget, permits": "Linear constraints"
      "whole workers / whole machines": "Integer variables (IP)"
      "portfolio scope ('wszystkich projektów w moim systemie')": "Joint variables across all projects"
    near_misses:
      - "objective becomes a nonlinear / risk-coupled black box → csp / constrained optimization"
      - "configurable objective ('inna funkcja porównująca opłacalność np prestiż') → multi-objective optimization"
    confidence: "medium"
    rationale: >
      The director's dashboard is a portfolio-wide optimization: 'optymalizacja
      rozkładu zasobów' across projects to 'optimize saving and margin', top metric
      'revenue'. Written as maximize(margin) subject to resource and permit limits
      with integral hire/buy/move decisions, this is LP/IP. It is medium not high
      because the objective visibly couples with risk (a nonlinear term) and the
      board even flags a swappable objective ('inna funkcja … np prestiż'), which
      pushes the true problem toward constrained / multi-objective optimization.
    skiena_pointer: "linear-programming"

  - algorithm_id: "knapsack"
    algorithm_name: "Knapsack 0/1"
    family: "Optimization"
    match_basis: "structural"
    fit_question: "Must you pick a subset of items, each with a cost and a value, to maximize value within one budget?"
    bridge_words:
      - quote: "Albo nawet taką nieistniejącą koparkę, gdybym miał koparkę, jak bym ją dodał albo gdybym miał pracownika, jak bym go dodał, to czy to się opłaci bardziej, bo ja przecież znam rate leasingu za koparkę."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L48–50"
        word: null   # structural — 'rate leasingu' (cost) vs 'opłaci bardziej' (value)
      - quote: "Chciałbym zobaczyć, czy bardziej opłaca mi się skilować jakiegoś pracownika, czy może opłaca mi się bardziej coś kupić do tego projektu."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L61–64"
        word: null   # structural — pick the higher-value option under budget
    domain_to_algorithm_mapping:
      "investment option: excavator / new hire / upskill / purchase": "Item"
      "rate leasingu / salary / purchase cost": "Item weight (cost)"
      "margin gained / risk reduced": "Item value"
      "investment budget": "Capacity C"
      "the set of investments to make": "Optimal subset"
    near_misses:
      - "several capacities bind at once (cash AND headcount AND time) → knapsack-multidim"
      - "continuous portfolio-wide allocation, not a discrete subset → linear-programming"
    confidence: "low"
    rationale: >
      The equipment/hire what-ifs read as a discrete pick-the-best-under-budget
      choice: each option has a known cost (he stresses 'znam rate leasingu') and a
      benefit (more margin, less risk), and he asks which to fund. That is 0/1
      knapsack. Confidence is low because the corpus frames these as *pairwise*
      'does it pay off' comparisons and hypothetical simulations rather than a
      stated single-budget subset selection — the honest structural cousin of the
      LP above, not an independently evidenced budget model.
    skiena_pointer: "knapsack"
top_pick: "linear-programming"
honest_misses:
  - "prognozy / prognozować ('aby nasz system mógł również prognozować potencjalne skutki') — forecasting/prediction has no catalog entry (time-series / ML territory)."
  - "symulacja / what-if ('symulacja stanu innych projektów gdyby zamknięto ten') — scenario / Monte-Carlo simulation is a technique, not a canonical catalog algorithm."
  - "oszacować / szacować ('jakoś to szacować chciałbym', 'oszacowano ryzyko') — risk estimation / scoring is statistical, no catalog entry."
```

---

## Area: `resource-allocation` — *przeniesienie zasobów do innego projektu*

```yaml
area_id: "resource-allocation"
name: "przeniesienie zasobów do innego projektu"
algorithm_candidates:
  - algorithm_id: "assignment"
    algorithm_name: "Assignment / GAP (Hungarian)"
    family: "Optimization"
    match_basis: "both"
    fit_question: "Must each agent take a task (one-to-one) so total cost is minimized / total value maximized?"
    bridge_words:
      - quote: "W ogóle jest taki przycisk w tym systemie, obecem tym, który mam, on słabo działa. T aki, który pozwala przydzielić pracownika do innego projektu. Ja chciałbym, żeby on dobrze działał ten przycisk."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L40–42"
        word: "przydzielić pracownika do innego projektu"   # assign a worker to another project
      - quote: "Czyli, że jak przesuwam do innego projektu, to wtedy pojawia mi się, czy zarobię wtedy więcej na tym przesunięciu."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L43–45"
        word: "czy zarobię wtedy więcej na tym przesunięciu"   # will I earn more from this move
      - quote: "allocate resources (command) → is resource available? (policy)"
        source: "intake/process level.es.md"
        loc: "Lane: Scheduling flow — automatic mode"
        word: "allocate resources"
    domain_to_algorithm_mapping:
      "zasób / pracownik (resource / worker)": "Agent (row)"
      "project-stage demand ('Add resource to stage')": "Task (column)"
      "'czy zarobię więcej na tym przesunięciu' (margin/cost of that move)": "Matrix cell value/cost"
      "the set of moves the system suggests": "Optimal assignment (cost-min / value-max permutation)"
      "a resource works one place at a time": "One-to-one capacity"
    near_misses:
      - "pure feasibility, no cost attached → bipartite-matching"
      - "a resource serves several demands under capacity → max-flow"
      - "both sides (resource ↔ project) rank preferences and stability matters → stable-matching"
    confidence: "medium"
    rationale: >
      The 'przycisk … który słabo działa' the CEO most wants fixed is a resource
      move whose worth is a per-pairing value: 'czy zarobię wtedy więcej na tym
      przesunięciu'. Assign resources to project-stage demands so total margin is
      maximized = the assignment problem (Hungarian for the clean one-to-one case,
      GAP once a resource can cover multiple demands). Both mechanisms fire; medium
      because the value per pairing is risk-coupled, nudging toward the LP/max-flow
      generalizations noted as near-misses.
    skiena_pointer: "matching"

  - algorithm_id: "bipartite-matching"
    algorithm_name: "Bipartite matching"
    family: "Graph"
    match_basis: "both"
    fit_question: "Must items from group A pair with items from group B under a compatibility, each used at most once?"
    bridge_words:
      - quote: "is resource available? — flows in from allocate resources; flows out to resources allocated (yes) vs equivalnet found / resources not aquired → demand queued (no)"
        source: "intake/process level.es.md"
        loc: "Pivotal events"
        word: "resources allocated / resources not aquired"
      - quote: "wykryto braki (risk) → wyszukano zamiennik zasobu (event)"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor: maneger projektu"
        word: "zamiennik zasobu"   # resource substitute
    domain_to_algorithm_mapping:
      "zasób (resource)": "Left node"
      "demand (stage's resource need)": "Right node"
      "'is resource available? / compatible': ": "Edge"
      "resources allocated": "Matched pair"
      "resources not aquired → demand queued": "Exposed (unmatched) vertex"
      "equivalnet found / zamiennik": "Alternative edge to a substitute resource"
    near_misses:
      - "the move carries a margin to optimize → assignment"
      - "a resource can cover many demands (capacity > 1) → max-flow"
    confidence: "medium"
    rationale: >
      Stripped of the economics, allocation is a feasibility matching:
      demand ↔ available resource, each resource used once, unmatched demand
      'queued', and a 'zamiennik'/'equivalnet' as an alternative edge when the
      first choice is unavailable. This is the un-weighted skeleton of the
      assignment candidate above.
    skiena_pointer: "matching"

  - algorithm_id: "max-flow"
    algorithm_name: "Maximum flow / Min cut"
    family: "Graph"
    match_basis: "structural"
    fit_question: "Is something fungible pushed through a network with per-link capacity limits, and you want max throughput (or the cheapest bottleneck to cut)?"
    bridge_words:
      - quote: "allocate resources (command) → is resource available? (policy) → project allocations scheduled (event) → resources allocated (event) … resources not aquired (event) → demand queued (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Scheduling flow — automatic mode"
        word: null   # structural — supply→demand under capacity
    domain_to_algorithm_mapping:
      "pool of available resources": "Source-side capacity"
      "project/stage demands": "Sink-side"
      "resource ↔ demand feasibility": "Edge"
      "a resource's availability / a project's need": "Node / edge capacity"
      "total demand satisfied": "Flow value"
      "the binding shortage across the portfolio": "Min cut"
    near_misses:
      - "one-to-one with no capacities → bipartite-matching"
      - "cheapest full pairing wanted → assignment"
    confidence: "low"
    rationale: >
      When one resource type can serve several projects and demands compete for a
      shared pool with 'demand queued' overflow, allocation across the whole
      portfolio ('optymalizcja zasobów przez przenoszenie pomiędzy projektami')
      generalizes matching into a capacitated supply→demand flow, whose min cut is
      the limiting shortage. Low confidence — capacities are implied, not stated.
    skiena_pointer: "network-flow"

  - algorithm_id: "interval-tree"
    algorithm_name: "Interval tree"
    family: "Tree & data-structure"
    match_basis: "both"
    fit_question: "Do you store intervals and ask 'which overlap this point/interval?' (collision, double-booking)?"
    bridge_words:
      - quote: "is resource available? (question)"
        source: "intake/process level.es.md"
        loc: "Lane: Floating questions"
        word: "is resource available?"
      - quote: "reallocate resources (command) → availability check (policy)"
        source: "intake/process level.es.md"
        loc: "Lane: Reschedule / reallocation flow"
        word: "availability check"
    domain_to_algorithm_mapping:
      "the window a new allocation needs": "Query interval"
      "existing allocations + vacation/sick (from resource-capability)": "Stored intervals"
      "'is resource available?' at allocation time (C3: check at allocation)": "Stabbing / overlap query"
      "a clash": "Overlapping interval = double-booking"
    near_misses:
      - "maximum set of non-overlapping allocations for one resource → interval-scheduling"
      - "categorize clashing allocations into fewest slots → graph-coloring"
    confidence: "medium"
    rationale: >
      C3 fixes the ordering as 'no upfront check' — availability is verified at
      allocation time. 'is resource available?' / 'availability check' against a
      resource's booked windows and its vacation/sick periods is an interval-overlap
      (stabbing) query, the classic interval-tree use. Shares its stored intervals
      with resource-capability (see cross-area obs_4).
    skiena_pointer: "intersection-detection"

  - algorithm_id: "priority-queue"
    algorithm_name: "Priority queue"
    family: "Tree & data-structure"
    match_basis: "bridge-word"
    fit_question: "Must work items be served in an order driven by a score/deadline/severity that can change over time?"
    bridge_words:
      - quote: "resources not aquired (event) → demand queued (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Scheduling flow — automatic mode"
        word: "demand queued"
      - quote: "wykryto opóźnienie (risk) → dodano zasób do kolejki projektu (event)"
        source: "intake/big picture.es.md"
        loc: "Lane: aktor: maneger projektu"
        word: "kolejki projektu"   # project queue
    domain_to_algorithm_mapping:
      "unmet demand ('demand queued')": "Queued element"
      "project urgency / risk / penalty exposure": "Key (inferred)"
      "serve next when a resource frees": "extract-max"
      "re-rank when the schedule shifts": "decrease-key"
    near_misses:
      - "a fixed precomputed schedule, not live dispatch → job-scheduling"
      - "hard, must-not-miss deadlines → edf"
    confidence: "low"
    rationale: >
      'demand queued' and 'kolejki projektu' are explicit queue vocabulary: unmet
      demands wait and are served as resources free. Low confidence because the
      corpus does not state the *ordering key* — whether the queue is FIFO or
      priority-ranked (by risk / penalty) is inferred, not evidenced.
    skiena_pointer: "priority-queues"
top_pick: "assignment"
honest_misses:
  - "resource training/moving from other project simulated ('resource training/moving from other project simulated') — the pre-move simulation is scenario evaluation, no catalog algorithm."
```

---

## Area: `resource-capability` — *Resources and capability management*

```yaml
area_id: "resource-capability"
name: "Resources and capability management"
algorithm_candidates:
  - algorithm_id: "maxsat"
    algorithm_name: "MaxSAT / MinSAT"
    family: "Optimization"
    match_basis: "both"
    fit_question: "When hard rules collide, must you satisfy the most (weighted) soft rules?"
    bridge_words:
      - quote: "Czyli mogę wysłać pracownika bez umiejętności na jakiś projekt. Akceptuję to, to się może wydarzyć, ale nie mogę wysłać do projektu kogoś, kto nie ma odpowiedniego zezwolenia do tego projektu, bo ktoś się kończy krymiałem."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L65–69"
        word: "bez umiejętności … Akceptuję to … nie mogę … bez … zezwolenia"   # skill-less accepted (soft) vs permit-less forbidden (hard)
      - quote: "any permit is hardblock"
        source: "intake/user-answers.md"
        loc: "Q4 inline answer"
        word: "hardblock"
      - quote: "hard block for permit"
        source: "intake/user-answers.md"
        loc: "C2 inline answer"
        word: "hard block for permit"
    domain_to_algorithm_mapping:
      "assign this resource to this project (yes/no)": "Boolean variable"
      "'nie mogę … bez … zezwolenia' / permit required": "Hard clause (must hold)"
      "'bez umiejętności … Akceptuję to' / skill required": "Soft, weighted clause (violation = accepted risk with a risk cost)"
      "a legal staffing choice": "Assignment satisfying all hard clauses, maximizing soft-clause weight"
    near_misses:
      - "if every rule were mandatory (no accepted-risk soft rules) → sat"
      - "if rules involve quantities / rich relations, not just booleans → csp"
    confidence: "medium"
    rationale: >
      The permit-vs-skill split is the textbook hard-clause/soft-clause structure
      of MaxSAT: a missing permit is a 'hardblock' (hard clause that must hold, on
      pain of legal shutdown — 'zamykają mnie'), while a missing skill is an
      'accepted' violation carrying a risk weight. Eligibility of a staffing choice
      is 'satisfy all hard permit clauses, maximize satisfied soft skill clauses'.
      Medium: as stated the per-resource check is a simple boolean evaluation, but
      its canonical reduction is squarely (Max)SAT once it is folded into choosing
      an assignment.
    skiena_pointer: "satisfiability"

  - algorithm_id: "sat"
    algorithm_name: "SAT (boolean satisfiability)"
    family: "Optimization"
    match_basis: "both"
    fit_question: "Can the rules be encoded as a boolean formula, and you need any true/false assignment that satisfies it (or proof none exists)?"
    bridge_words:
      - quote: "nie możemy pozwolić na to, byśmy wysłali kogoś do projektu, kto nie ma odpowiednich kwalifikacji czy zezwoleń."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L20–22"
        word: "nie może … bez … zezwoleń"   # must not, without permits (forcing/excluding)
      - quote: "any permit is hardblock"
        source: "intake/user-answers.md"
        loc: "Q4 inline answer"
        word: "hardblock"
    domain_to_algorithm_mapping:
      "'resource has permit X' (yes/no)": "Boolean variable"
      "'assignment requires permit for the project'": "Clause (must hold)"
      "is this staffing feasible at all?": "Satisfiability"
      "the eligible configuration": "Satisfying assignment"
    near_misses:
      - "least-violating outcome wanted (soft skills) → maxsat"
      - "quantities, not just booleans → csp / linear-programming"
    confidence: "medium"
    rationale: >
      The permit gate is pure boolean feasibility: an assignment is admissible iff
      it satisfies every hard permit clause. This is the SAT skeleton of the MaxSAT
      candidate — same variables, minus the soft skill clauses.
    skiena_pointer: "satisfiability"

  - algorithm_id: "csp"
    algorithm_name: "Constraint Programming (CSP)"
    family: "Optimization"
    match_basis: "structural"
    fit_question: "Is it 'assign values to variables so all these (possibly non-linear/logical) constraints hold'?"
    bridge_words:
      - quote: "Change skills (command) → Skills changed (event) → capability unlocked (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Employee / device / material capability lane"
        word: null   # structural — capability unlocking gates which assignments are feasible
    domain_to_algorithm_mapping:
      "which resource fills which demand": "Variable"
      "eligible resources for a demand": "Domain"
      "permit (hard) + skill (soft) + capability-unlocked rules": "Constraints"
      "a legal, fully-staffed plan": "Satisfying assignment"
    near_misses:
      - "purely boolean rules → sat"
      - "explicit hard/soft split, maximize soft → maxsat"
    confidence: "low"
    rationale: >
      Read as one staffing problem, the permit/skill/capability rules are
      heterogeneous constraints over 'who fills what' — the CSP framing. Listed low
      and structural: it is the modelling umbrella over the sat/maxsat candidates
      rather than an independently evidenced constraint network.
    skiena_pointer: "constrained-unconstrained-optimization"

  - algorithm_id: "interval-tree"
    algorithm_name: "Interval tree"
    family: "Tree & data-structure"
    match_basis: "both"
    fit_question: "Do you store intervals and ask 'which overlap this point/interval?' (double-booking)?"
    bridge_words:
      - quote: "Add vacation (command) → Vacation added (event) → Add sick leave (command) → Sick leave added (event)"
        source: "intake/process level.es.md"
        loc: "Lane: Employee / device / material capability lane"
        word: "vacation / sick leave"
    domain_to_algorithm_mapping:
      "vacation period / sick-leave period": "Busy interval"
      "employee / device / material": "Owner of an interval set"
      "'is this resource free during the assignment window?'": "Stabbing / overlap query"
      "overlap with vacation or sick leave": "Resource unavailable in that window"
    near_misses:
      - "maximum non-overlapping usage of one resource → interval-scheduling"
    confidence: "medium"
    rationale: >
      Vacation and sick leave define per-resource busy windows. The availability
      question posed in resource-allocation ('is resource available?') is answered
      by testing whether the assignment window overlaps any of these intervals — an
      interval-tree stabbing query. Same structure and (partly) the same data as the
      interval-tree candidate in resource-allocation (cross-area obs_4).
    skiena_pointer: "intersection-detection"
top_pick: "maxsat"
```

> Note: most of this area is master-data CRUD (`Add resource definition`,
> `Add resource instantion`, `Employee/Device/Material summaries`) with no
> algorithmic content. The algorithmic kernels are the eligibility constraint
> (permit/skill) and the availability windows (vacation/sick) above.

---

## Area: `contractual-penalties` — *kary umowne za opóźnienia*

```yaml
area_id: "contractual-penalties"
name: "kary umowne za opóźnienia"
algorithm_candidates:
  - algorithm_id: "job-scheduling"
    algorithm_name: "Job scheduling / Critical path"
    family: "Graph"
    match_basis: "both"
    fit_question: "Do tasks carry durations/deadlines on top of precedence, so the late tasks are deadline violations?"
    bridge_words:
      - quote: "Ostatnią rzeczą jest to, że często tydzień przed dniem, w którym miałam zapłacić jakąś karę za spóźnienie, chciałbym dostać informacje o tym."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L70–71"
        word: "karę za spóźnienie"   # penalty for lateness (deadline)
      - quote: "informacje na temat potencjalnych kar finansowych za opóźnienia w projektach powinny być dostarczane odpowiednio wcześnie."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L23–25"
        word: "kar finansowych za opóźnienia"   # financial penalties for delays
    domain_to_algorithm_mapping:
      "projekt / etap (project / stage)": "Node"
      "precedence between stages": "Directed edge"
      "stage durations": "Weights"
      "contractual due date": "Deadline"
      "computed project end date > due date": "Deadline violation → penalty triggers"
      "days of slack to the due date": "Early-warning margin ('tydzień przed')"
    near_misses:
      - "ordering only, no dates → topological-sort"
      - "alerting ranked by soonest due date → priority-queue"
    confidence: "medium"
    rationale: >
      A contractual penalty is a deadline violation on the same scheduling DAG:
      compare the CPM-computed project end date to the contractual due date; if the
      end date slips past it, a 'kara za spóźnienie' falls due, and the slack to
      that date is what makes a 'tydzień przed' warning computable. This binds the
      penalties area to the same critical-path engine as project-phases and
      project-risk (cross-area obs_1).
    skiena_pointer: "job-scheduling"

  - algorithm_id: "priority-queue"
    algorithm_name: "Priority queue"
    family: "Tree & data-structure"
    match_basis: "both"
    fit_question: "Must work items be served in an order driven by a deadline that can change over time?"
    bridge_words:
      - quote: "często tydzień przed dniem, w którym miałam zapłacić jakąś karę za spóźnienie, chciałbym dostać informacje o tym."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L70–71"
        word: "tydzień przed dniem … chciałbym dostać informacje"   # a week before the due day, alert me
    domain_to_algorithm_mapping:
      "an upcoming penalty due date": "Element"
      "days until it falls due": "Key"
      "warn about the soonest-due penalty first": "extract-min"
      "recompute when the schedule shifts the due date": "decrease-key"
    near_misses:
      - "hard, must-not-miss real-time deadline → edf"
      - "the penalty itself as a schedule slip → job-scheduling"
    confidence: "medium"
    rationale: >
      The 'warn me a week before the nearest penalty is due' requirement is a
      deadline-ordered alert queue: keep upcoming penalty dates keyed by proximity
      and surface the soonest. Medium — the ordering key (soonest due date) is
      explicit here, unlike the resource-allocation queue.
    skiena_pointer: "priority-queues"

  - algorithm_id: "linear-programming"
    algorithm_name: "Linear / Integer programming (LP/IP)"
    family: "Optimization"
    match_basis: "structural"
    fit_question: "Can the goal be written as maximize/minimize a linear score subject to linear constraints?"
    bridge_words:
      - quote: "I np. wtedy mogę sobie znowu szacować, że kogoś chce przenieść do innego projektu, żeby zmieniać tę karę. A może kara nie jest tak duża wcale, w stosunku do tego, co stracę przesuwając tego pracownika z tego projektu."
        source: "intake/wywiad-z-prezesem.md"
        loc: "L72–75"
        word: null   # structural — penalty avoided vs cost of the move
    domain_to_algorithm_mapping:
      "decision: move worker to avoid the penalty (yes/no)": "Decision variable"
      "penalty avoided − margin lost on the source project": "Objective (net benefit)"
      "resource availability / permit limits": "Constraints"
    near_misses:
      - "a single two-number comparison, not an optimization → plain arithmetic (folds into profitability)"
      - "portfolio-wide trade-off across many penalties/moves → linear-programming"
    confidence: "low"
    rationale: >
      Weighing 'kara' against 'co stracę przesuwając tego pracownika' is the same
      value engine as opłacalność, specialized to penalty avoidance: net benefit =
      penalty avoided minus margin lost. Low and structural — at the scale stated
      it is one comparison, and it belongs to the profitability optimizer
      (cross-area obs_3).
    skiena_pointer: "linear-programming"
top_pick: "job-scheduling"
```

---

## Cross-area observations

```yaml
cross_area_observations:
  - id: "obs_1"
    note: >
      Job-scheduling / Critical Path scores ≥medium in THREE areas — project-phases
      (high), project-risk (medium, delay propagation as a forward-pass), and
      contractual-penalties (medium, penalty = end-date-vs-deadline violation). One
      CPM engine over the stage-precedence DAG produces the phase schedule, the
      delay cascade magnitude, and the penalty early-warning. This is the single
      strongest consolidation signal and is exactly the 'tańszy, szybszy,
      modyfikowalny' algorithm the CEO asked for — recognising it as CPM inherits a
      polynomial O(V+E) core instead of the expensive proprietary schedulers he
      rejects.
    areas: ["project-phases", "project-risk", "contractual-penalties"]
    seam_type: "consolidation"

  - id: "obs_2"
    note: >
      The stage-precedence DAG is a shared kernel: topological-sort (order of
      stages), job-scheduling (dates), and transitive-closure (risk reachability)
      all read the SAME nodes (etapy) and edges (zależności). Build the graph once;
      each area is a different query over it (linearize / weight-and-time /
      reach-set).
    areas: ["project-phases", "project-risk"]
    seam_type: "shared_kernel"

  - id: "obs_3"
    note: >
      An optimization / value engine spans profitability (portfolio LP/IP over
      saving+margin), resource-allocation (assignment — a special case of LP), and
      contractual-penalties (penalty-vs-move net-benefit). The move-resource
      objective 'czy zarobię więcej na tym przesunięciu' IS the assignment cost
      matrix, so resource-allocation and profitability meet at an algorithmic seam:
      assignment ⊂ linear-programming.
    areas: ["profitability", "resource-allocation", "contractual-penalties"]
    seam_type: "consolidation"

  - id: "obs_4"
    note: >
      Interval-tree scores medium in TWO areas — resource-allocation ('is resource
      available? / availability check') and resource-capability (vacation / sick
      windows). One availability index of per-resource busy intervals answers the
      allocation-time check (C3: 'no upfront check'). Consolidate the interval
      store.
    areas: ["resource-allocation", "resource-capability"]
    seam_type: "consolidation"

  - id: "obs_5"
    note: >
      Algorithmic seam between resource-capability and resource-allocation: the
      permit/skill eligibility (SAT/MaxSAT) is the feasibility filter on the
      allocation matching/assignment — only permit-valid resources form candidate
      edges, and skill gaps enter as soft-weighted (accepted-risk) edge penalties.
      Capability constrains which matches even exist.
    areas: ["resource-capability", "resource-allocation"]
    seam_type: "seam"
```

---

## Domain-level honest misses

Bridge-words noticed across the corpus that map to **no** catalog entry — the
system's probabilistic / statistical surface, flagged as genuinely novel (not
classical-combinatorial) territory for deeper modeling:

- **prognozy / prognozować** — *"aby nasz system mógł również prognozować potencjalne skutki wprowadzania nowych środków"* (`wywiad-z-prezesem.md` L12–14). Forecasting / prediction: time-series / ML, no catalog entry.
- **symulacja / what-if** — *"symulacja stanu innych projektów gdyby zamknięto ten"* (`big picture.es.md`), *"resource training/moving from other project simulated"* (`process level.es.md`). Scenario / Monte-Carlo simulation is a technique, not a canonical algorithm.
- **oszacować / szacować / oszacowano ryzyko** — *"jakoś to szacować chciałbym"* (`wywiad-z-prezesem.md` L37–39), *"oszacowano ryzyko"* (`big picture.es.md`). Risk estimation / scoring: statistical, no catalog entry.
- **monitoring** — *"monitoring postępu (razem z kosztami)"*, *"monitoring ryzyka"*, *"monitoring sumarycznych kosztów"* (`big picture.es.md`). Observability / monitoring, not an algorithm.
- **negocjacje** — *"negocjacje z managarami"* (`big picture.es.md`). Human negotiation / arbitration (the unresolved q5 director-vs-manager rule); no algorithmic entry.

---

## Overall top pick

**`job-scheduling`** (Job scheduling / Critical Path) — highest single-area
confidence (high, `project-phases`), the widest cross-area reach (obs_1, three
areas), and the direct answer to the CEO's explicitly stated wish for a cheaper,
faster, modifiable phase-arrangement algorithm.
