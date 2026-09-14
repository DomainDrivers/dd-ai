---
name: algorithm-catalog
description: "Catalog of 42 canonical algorithms/data structures for reducing a domain problem to a known, already-solved problem. Each entry carries a core fit question, bridge words, a 'maps when' situation, an explicit domain → algorithm element mapping, anti-signals with near-miss redirects, pitfall, and a Skiena cross-reference. Language-agnostic matching: bridge-words name semantic intents and match domain vocabulary in any language."
---

# algorithm-catalog

Catalog of canonical algorithmic structures grouped by family. Used for
reduction — *reduce the domain problem to a known, already-solved
problem* — so the modeler inherits known complexity, known pitfalls, and
existing libraries instead of reinventing. Recognising the algorithm is a
*fail-fast*: if the area really is a Topological Sort or a Bipartite Matching,
the model almost writes itself; if it is *not* a recognised problem, the
mismatch surfaces immediately as genuinely novel territory.

Each entry is built for **deep recognition and deep mapping**, not just a
keyword hit:

- **Fit question** — the one-line test that decides whether the domain *is* an
  instance of this problem.
- **Bridge words** — illustrative, semantic, any-language vocabulary that points
  here (recall, not an exhaustive keyword list).
- **Maps when** — the plain-English business situation this problem fits.
- **Domain → algorithm** — the explicit element-by-element mapping shape (the
  thing the modeler actually inherits: *what plays the role of node / edge /
  item / constraint / weight*). This is the payload — a candidate without a
  concrete mapping is a guess.
- **Not this if** — anti-signals that rule it out, each with a **near-miss
  redirect** to the problem it is really an instance of. This is what keeps
  matching honest (a "dependent" web is not automatically a Topological Sort).
- **Pitfall / Skiena** — the inherited engineering facts.

## The catalog at a glance

| Algorithm | Essence | Family |
|---|---|---|
| Topological sort (`topological-sort`) | linearize under dependencies | Graph |
| Job scheduling / Critical path (`job-scheduling`) | precedence + durations + deadlines | Graph |
| Transitive closure & reachability (`transitive-closure`) | who eventually affects whom | Graph |
| Connected Components / Union-Find (`union-find`) | connectivity groups | Graph |
| Shortest path — Dijkstra/A* (`shortest-path`) | cheapest route through a weighted network | Graph |
| Minimum spanning tree (`minimum-spanning-tree`) | cheapest connect-everything | Graph |
| Maximum flow / Min cut (`max-flow`) | throughput through a capacitated network | Graph |
| Bipartite matching (`bipartite-matching`) | pair two groups, no sharing | Graph |
| Assignment / GAP — Hungarian (`assignment`) | cheapest one-to-one assignment | Graph |
| Stable matching — Gale–Shapley (`stable-matching`) | two-sided preferences, no blocking pair | Graph |
| Graph coloring (`graph-coloring`) | categorize so conflicting items differ | Graph |
| Traveling salesman / Vehicle routing (`traveling-salesman`) | visit all, shortest tour | Graph |
| Feedback Arc Set (`feedback-arc-set`) | break cycles to recover a DAG | Graph |
| Graph partition (`graph-partition`) | split a network into balanced chunks with few crossings | Graph |
| Edge & vertex connectivity (`edge-vertex-connectivity`) | network fragility / single points of failure | Graph |
| Vertex cover (`vertex-cover`) | fewest nodes that touch every connection | Graph |
| Clique (`clique`) | the largest fully-connected group | Graph |
| Priority queue (`priority-queue`) | always process the most urgent next | Tree & data-structure |
| Median & selection (`median-and-selection`) | the k-th / percentile without a full sort | Tree & data-structure |
| Trie — prefix tree (`trie`) | prefix matching, autocomplete | Tree & data-structure |
| B-tree / B+tree (`b-tree`) | sorted, disk-friendly index | Tree & data-structure |
| Segment tree / Fenwick — BIT (`segment-tree`) | range aggregates with updates | Tree & data-structure |
| Interval tree (`interval-tree`) | overlapping time/range intervals | Tree & data-structure |
| Kd-tree / spatial index (`kd-tree`) | nearest & region queries in space | Tree & data-structure |
| Point location (`point-location`) | which region/zone contains this point | Tree & data-structure |
| Knapsack 0/1 (`knapsack`) | most value into one budget | Optimization |
| Knapsack multidimensional (`knapsack-multidim`) | subset under several limits | Optimization |
| Bin packing (`bin-packing`) | fewest fixed-capacity containers | Optimization |
| Set cover (`set-cover`) | fewest sets to cover every requirement | Optimization |
| Interval scheduling / Independent set (`interval-scheduling`) | largest non-conflicting selection | Optimization |
| Linear / Integer programming — LP/IP (`linear-programming`) | linear objective under linear constraints | Optimization |
| Constraint Programming — CSP (`csp`) | variables + rich constraints | Optimization |
| SAT (`sat`) | boolean satisfiability | Optimization |
| MaxSAT / MinSAT (`maxsat`) | best approximate satisfaction | Optimization |
| Inverted index (`inverted-index`) | full-text search | Search & retrieval |
| Suffix trees & arrays (`suffix-array`) | index one big text, query it many ways | Search & retrieval |
| Approximate string matching (`approximate-string-match`) | fuzzy / typo-tolerant matching | Search & retrieval |
| Longest common subsequence (`lcs`) | largest shared ordered content (diff) | Search & retrieval |
| LSH — Locality-Sensitive Hashing (`lsh`) | near-duplicate / set similarity at scale | Search & retrieval |
| HNSW / IVF — vector similarity (`vector-similarity`) | semantic nearest neighbor | Search & retrieval |
| EDF / Rate Monotonic (`edf`) | real-time deadline scheduling | Time & scheduling |
| FPP — Fixed Priority Preemptive (`fpp`) | static-priority preemptive scheduling | Time & scheduling |

> Full per-entry detail (fit question, bridge words, domain → algorithm
> mapping, anti-signals, pitfall, Skiena pointer) is in *Catalog by family*
> below — this table is only the fast shortlist for browsing.

## Language-agnostic matching (English index, any-language corpus)

This catalog is English-only **by design**, and that does not restrict it
to English corpora. Bridge-words name **semantic intents**, not literal
strings: the matcher is the reading model, not a grep, so a corpus word in
*any* language matches a row when it carries the same meaning — a phrase
meaning "depends on / is blocked by" yields Topological sort. The matched word
is recorded **verbatim in its source language** (quote + source + location),
never translated. The structural mechanism is language-free entirely: use the
**Fit question** and the **Domain → algorithm** shape even when no bridge-word
ever appears.

## Skiena cross-reference

This catalog is the curated modeling shortlist: the entries carry the deep
per-entry analysis below, scoped to canonical problems in *The Algorithm
Design Manual*. It is the authoritative source for `algorithm_id` values
(the `id:` slugs below).

Every entry carries a `Skiena:` cross-reference into a problem in *The
Algorithm Design Manual* (a few as a `(related)` variant). The `Skiena:`
lines are the source of `skiena_pointer` and serve as literature pointers
into the book's catalog.

## Bridge-word → algorithm shortlist

| Bridge-word intent | First candidates |
|---|---|
| dependent / preceding / blocks / build order | Topological sort (`topological-sort`), Job scheduling (`job-scheduling`) |
| precedence + duration / deadline / makespan | Job scheduling / Critical path (`job-scheduling`) |
| reachability / propagation / "eventually affects" | Transitive closure (`transitive-closure`), Union-Find (`union-find`) |
| conflicting / collision / break a cycle | Feedback Arc Set (`feedback-arc-set`), CSP (`csp`) |
| matched / paired / assigned | Bipartite matching (`bipartite-matching`), Stable matching (`stable-matching`), Assignment (`assignment`) |
| satisfying / excluding / forcing | SAT (`sat`), CSP (`csp`), MaxSAT (`maxsat`) |
| constrained / capacity / budget / subset selection | Knapsack (`knapsack`), LP/IP (`linear-programming`), CSP (`csp`) |
| several limits bind at once / constrained on X and Y | Knapsack multidimensional (`knapsack-multidim`) |
| fit into fewest containers | Bin packing (`bin-packing`) |
| cover everything with fewest sets | Set cover (`set-cover`) |
| cover every connection with fewest nodes | Vertex cover (`vertex-cover`) |
| largest non-overlapping selection | Interval scheduling / Independent set (`interval-scheduling`) |
| categorize so no two conflict | Graph coloring (`graph-coloring`) |
| balanced split / shard / decompose network | Graph partition (`graph-partition`) |
| single point of failure / fragility / resilience | Edge & vertex connectivity (`edge-vertex-connectivity`) |
| fully-connected group / coalition / dense cluster | Clique (`clique`) |
| cheapest / fastest route between points | Shortest path (`shortest-path`) |
| optimal / best under constraints | LP/IP (`linear-programming`), CSP (`csp`), Knapsack (`knapsack`) |
| visit-all / shortest tour / routing | Traveling salesman / VRP (`traveling-salesman`) |
| cheapest connect-all / linkage cluster | Minimum spanning tree (`minimum-spanning-tree`) |
| sorted / range / aggregation | Segment tree / Fenwick (`segment-tree`), B-tree (`b-tree`) |
| k-th / median / percentile / top-k | Median & selection (`median-and-selection`) |
| process by priority / score / severity | Priority queue (`priority-queue`) |
| hard deadline / real-time / periodic task / must-not-miss | EDF / Rate Monotonic (`edf`), FPP (`fpp`) |
| which zone / region contains a point | Point location (`point-location`) |
| nearest facility / within radius / coordinates / geo-fence | Kd-tree / spatial index (`kd-tree`) |
| interval / overlaps / time collision | Interval tree (`interval-tree`) |
| similar / near / semantic | LSH (`lsh`), Vector similarity (`vector-similarity`) |
| fuzzy / typo / record linkage / dedup | Approximate string matching (`approximate-string-match`) |
| diff / what changed / shared ordered chunk | Longest common subsequence (`lcs`) |
| prefix / starts-with / autocomplete / longest-prefix | Trie (`trie`) |
| search / full-text / find containing / keyword query | Inverted index (`inverted-index`) |
| indexed substring / repeated fragment (one big text) | Suffix trees & arrays (`suffix-array`) |
| throughput / flow / bottleneck | Max flow / Min cut (`max-flow`) |
| connectivity / cluster / island | Union-Find / Connected Components (`union-find`) |

Vocabulary often hits several rows — that is expected. Overlap is data, not
noise: it surfaces near-misses. Resolve overlap with the **Fit question** and
the entry's **Not this if** anti-signals, never by guessing.

## Catalog by family

### Graph family

**Topological sort** (`id: topological-sort`) — linearize under dependencies
- **Fit**: must items be flattened into one order that honours every "must-come-before"?
- **Bridge words**: dependent, preceding, blocks, requires, build order, must-come-before, pipeline stages
- **Maps when**: a "do A before B" web must be turned into a valid execution order, or its very existence checked.
- **Domain → algorithm**: task/stage → node · "depends on / blocked by" → directed edge (predecessor → successor) · valid execution sequence → topological order · the impossible case → a cycle.
- **Not this if**: cycles are legitimate and must be kept/minimized → `feedback-arc-set`; durations, deadlines or resource limits matter → `job-scheduling`; you only need "can A reach B?" not an order → `transitive-closure`; each node has exactly one parent → it is a tree, not a sort.
- **Pitfall**: a cycle means contradictory constraints — detect it (it falls out of the same traversal).
- **Skiena**: `topological-sorting`

**Job scheduling / Critical path** (`id: job-scheduling`) — precedence + durations + deadlines
- **Fit**: do tasks carry durations, deadlines, or resource needs on top of a precedence order, so the question is "what schedule finishes soonest / by when?"
- **Bridge words**: schedule, makespan, critical path, deadline, duration, lead time, end date, slack, plan, sequence with timing
- **Maps when**: a multi-stage plan must be timed — earliest finish, the bottleneck chain, what slips the end date.
- **Domain → algorithm**: task/stage → node · "must finish before" → edge · effort/length → node or edge weight · schedule → start/finish times · longest weighted path → critical path · the late tasks → deadline violations.
- **Not this if**: precedence only, no timing → `topological-sort`; pure subset-under-budget with no ordering → `knapsack`; assigning tasks to interchangeable agents to minimize count → `assignment`/`bin-packing`.
- **Pitfall**: pure precedence is O(V+E) (critical-path method), but adding resource limits or makespan minimization makes most variants NP-hard → use list-scheduling / LPT heuristics.
- **Skiena**: `job-scheduling`

**Transitive closure & reachability** (`id: transitive-closure`) — who eventually affects whom
- **Fit**: must you answer "can A *eventually* reach / influence / impact B?" across a chain of relationships, not just direct links?
- **Bridge words**: reachable, propagate, eventually affects, downstream impact, indirect dependency, ripple, cascade, ancestors/descendants
- **Maps when**: an effect (risk, cost, delay, contamination, blame) flows along edges and you need the full set of things touched, or to compress a tangled web to its minimal equivalent.
- **Domain → algorithm**: entity → node · "directly affects" → edge · "eventually affects" → closure edge · the full blast radius → reachable set · the minimal equivalent web → transitive reduction.
- **Not this if**: you need an *ordering* of the chain → `topological-sort`; you only need undirected "are these grouped?" → `union-find`; weighted cheapest route, not mere reachability → `shortest-path`.
- **Pitfall**: closure can be dense (O(V²) edges) — store reachable-sets, do not materialize every edge.
- **Skiena**: `transitive-closure-and-reduction`

**Connected Components / Union-Find** (`id: union-find`) — connectivity groups
- **Fit**: must you group items into "islands" by shared/connected attributes, and merge groups fast as links arrive?
- **Bridge words**: cluster, group, connected, island, isolated, related, merge, same network
- **Maps when**: items collapse into equivalence classes (same household, same fraud ring, same supply island) under a "linked to" relation.
- **Domain → algorithm**: item → node · "linked to / same as" → edge · a group → a connected component · group membership query → find(root) · merging two groups → union.
- **Not this if**: direction and reachability matter → `transitive-closure`; you need *one tightly-knit* fully-connected group → `clique`; cheapest tree linking groups → `minimum-spanning-tree`; you need a *balanced* split rather than the natural islands → `graph-partition`; you want the choke points that break connectivity → `edge-vertex-connectivity`.
- **Pitfall**: use weighted union + path compression or it degrades to O(n) per op.
- **Skiena**: `connected-components`, `set-data-structures`

**Shortest path (Dijkstra/A\*)** (`id: shortest-path`) — cheapest route through a weighted network
- **Fit**: is there a network with costs on its links where you need the cheapest/fastest way between points?
- **Bridge words**: optimal, shortest, cheapest, fastest, route, distance, navigation, least cost
- **Maps when**: a journey/transformation/cost accrues along edges and you want the minimum-total path.
- **Domain → algorithm**: location/state → node · transition → edge · cost/time/distance → edge weight · cheapest journey → shortest path · the heuristic estimate → A\* potential.
- **Not this if**: you must visit *all* nodes, not go A→B → `traveling-salesman`; connect everything cheaply, not route → `minimum-spanning-tree`; capacities throttle volume → `max-flow`.
- **Pitfall**: negative edges break Dijkstra → Bellman-Ford; negative cycles have no answer.
- **Skiena**: `shortest-path`

**Minimum spanning tree** (`id: minimum-spanning-tree`) — cheapest connect-everything
- **Fit**: must every node be connected into one network at minimum total link cost (no routing, just connectivity)?
- **Bridge words**: connect all, backbone, cheapest network, spanning, linkage cluster, wiring, hierarchy by closeness
- **Maps when**: lay the cheapest cabling/road/pipeline that reaches all sites, or cluster items by single-linkage distance.
- **Domain → algorithm**: site/item → node · possible link → edge · build/distance cost → edge weight · the chosen network → spanning tree · a cluster cut → the heaviest tree edge removed.
- **Not this if**: only a *subset* must be connected (helper nodes allowed) → `steiner-tree` (Skiena, hard); you need point-to-point routes → `shortest-path`.
- **Pitfall**: MST minimizes total edge cost, *not* path length between any two nodes.
- **Skiena**: `minimum-spanning-tree`

**Maximum flow / Min cut** (`id: max-flow`) — throughput through a capacitated network
- **Fit**: is something fungible pushed through a network with per-link capacity limits, and you want max throughput (or the cheapest bottleneck to cut)?
- **Bridge words**: throughput, capacity, flow, bottleneck, supply, distribute, route demand, min cut
- **Maps when**: supply→demand routing under capacities, or finding the limiting bottleneck set (min cut = max flow).
- **Domain → algorithm**: junction → node · channel → edge · channel limit → capacity · total moved → flow value · the limiting links → min cut. (Bipartite matching and many assignment problems reduce to this.)
- **Not this if**: one-to-one pairing with no capacities → `bipartite-matching`; weighted optimal pairing → `assignment`; linear objective with general constraints → `linear-programming`.
- **Pitfall**: algorithm choice depends on the graph (dense vs sparse, integer vs real capacities).
- **Skiena**: `network-flow`

**Bipartite matching** (`id: bipartite-matching`) — pair two groups, no sharing
- **Fit**: must items from group A pair with items from group B under a compatibility, each used at most once?
- **Bridge words**: matched, paired, assigned, one-to-one, compatible, partner, allocate
- **Maps when**: workers↔tasks, donors↔recipients, slots↔bookings — maximize valid pairings.
- **Domain → algorithm**: group-A item → left node · group-B item → right node · "compatible with" → edge · the pairing → matching · the unpaired → exposed vertices.
- **Not this if**: pairs carry costs and you want the *cheapest* full pairing → `assignment`; both sides rank preferences and stability matters → `stable-matching`; capacities > 1 per node → `max-flow`.
- **Pitfall**: maximum matching ≠ stable matching ≠ minimum-cost matching — name which you mean.
- **Skiena**: `matching`

**Assignment / GAP (Hungarian)** (`id: assignment`) — cheapest one-to-one assignment
- **Fit**: must each agent take a task (one-to-one) so total cost is minimized / total value maximized?
- **Bridge words**: assignment, assign at cost, who does what, cheapest allocation, rostering
- **Maps when**: assign people to jobs / orders to trucks / shifts to staff with a cost or score per pairing.
- **Domain → algorithm**: agent → row · task → column · pairing cost/value → matrix cell · the optimal assignment → permutation minimizing total.
- **Not this if**: only feasibility of pairing, no cost → `bipartite-matching`; agents take multiple tasks under capacity → `max-flow`/`linear-programming`; ranked-preference stability → `stable-matching`.
- **Pitfall**: generalized assignment (capacities, multi-task) is NP-hard; the clean one-to-one case is polynomial (Hungarian).
- **Skiena**: `matching` / `linear-programming` (related)

**Stable matching (Gale–Shapley)** (`id: stable-matching`) — two-sided preferences, no blocking pair
- **Fit**: do *both* sides rank each other, and you need a matching with no pair that would both rather defect?
- **Bridge words**: preference, ranking, stable, mutually acceptable, no one wants to swap
- **Maps when**: residents↔hospitals, students↔schools, riders↔drivers — preference-respecting allocation.
- **Domain → algorithm**: side-A actor → proposer · side-B actor → acceptor · ranked list → preference order · the result → stable matching · a defecting pair → blocking pair (must be zero).
- **Not this if**: only one side has preferences / it is a cost → `assignment`; no preferences, just feasibility → `bipartite-matching`.
- **Pitfall**: which side proposes biases the outcome (proposers get their best stable partner) — it is asymmetric.
- **Skiena**: `matching` (variant)

**Graph coloring** (`id: graph-coloring`) — categorize so conflicting items differ
- **Fit**: must items be assigned to a few categories/slots such that no two *conflicting* items share one?
- **Bridge words**: assign category, no two adjacent, conflict-free slot, frequency, shift, exam slot, register allocation, round
- **Maps when**: schedule exams/shifts so clashing parties never collide; assign channels/rooms; minimize the number of slots.
- **Domain → algorithm**: item → node · "cannot share a slot with" → edge · slot/category → color · the schedule → a proper coloring · fewest slots → chromatic number.
- **Not this if**: it is the *connections* that must be partitioned (each node one per round) → `edge-coloring` (Skiena); largest single conflict-free set → `interval-scheduling`; capacity-fit, not conflict → `bin-packing`.
- **Pitfall**: NP-hard; greedy (Welsh–Powell / DSATUR) gives good practical colorings.
- **Skiena**: `vertex-coloring` (and `edge-coloring`)

**Traveling salesman / Vehicle routing** (`id: traveling-salesman`) — visit all, shortest tour
- **Fit**: must an agent visit a set of locations (and return) with minimum total distance/time?
- **Bridge words**: route, tour, visit all, delivery sequence, round trip, drilling order, last-mile, sequencing of stops
- **Maps when**: order the stops on a delivery/inspection/maintenance run; sequence operations to minimize travel/changeover.
- **Domain → algorithm**: stop → node · travel cost between stops → edge weight · the run → a tour · shortest tour → TSP optimum · multiple vehicles + capacities → VRP.
- **Not this if**: go A→B only → `shortest-path`; visit every *edge* not node → `eulerian-cycle` (Skiena); just need *an* order honouring precedence → `topological-sort`.
- **Pitfall**: NP-hard; use OR-tools / Christofides / 2-opt — never brute force beyond ~10 stops.
- **Skiena**: `traveling-salesman`

**Feedback Arc Set** (`id: feedback-arc-set`) — break cycles to recover a DAG
- **Fit**: does a "should-come-before" graph contain cycles you must break with the fewest removals to recover an order?
- **Bridge words**: conflicting, mutually exclusive, deadlock, break cycle, untangle, inconsistency, contradiction
- **Maps when**: a dependency/ranking graph is contradictory and you must drop the fewest edges to make it acyclic.
- **Domain → algorithm**: rule/ordering claim → edge · contradiction loop → cycle · the edges to relax → feedback arc set · the recovered order → topological sort of the remainder.
- **Not this if**: the graph is already acyclic → `topological-sort`; you just need to *detect* a cycle, not minimize removals → cycle detection (part of topo sort).
- **Pitfall**: NP-hard → use approximation (Eades heuristic).
- **Skiena**: `feedback-edge-vertex-set`

**Graph partition** (`id: graph-partition`) — split a network into balanced chunks with few crossings
- **Fit**: must a network be cut into balanced groups while minimizing the connections that cross between them?
- **Bridge words**: decompose, shard, balance load, community, minimize crossings, split into teams/modules, partition the network
- **Maps when**: split work/data/modules so each chunk is similar-sized and inter-chunk chatter is low — sharding, module/team decomposition, community detection.
- **Domain → algorithm**: unit → node · interaction/coupling → weighted edge · a chunk → a partition block · cross-chunk coupling → cut edges (minimize) · fairness → equal block sizes.
- **Not this if**: groups fall out of "is it connected at all?" → `union-find`; you assign categories to avoid conflict, not balance size → `graph-coloring`; one tightly-knit group, not a balanced split → `clique`.
- **Pitfall**: NP-hard; multilevel heuristics (METIS, Kernighan–Lin) are the practical tools.
- **Skiena**: `graph-partition`

**Edge & vertex connectivity** (`id: edge-vertex-connectivity`) — network fragility / single points of failure
- **Fit**: must you know how many links or nodes must fail before the network splits — or which single link/node is a point of failure?
- **Bridge words**: single point of failure, fragility, resilience, choke point, articulation, bridge, k-connected, redundancy, what breaks it
- **Maps when**: resilience/risk analysis of supply chains, dependency graphs, infrastructure — find the choke points whose loss disconnects things.
- **Domain → algorithm**: component → node · dependency/link → edge · a critical link → a bridge / min-cut edge · a critical node → an articulation vertex · robustness → minimum removals to disconnect.
- **Not this if**: you want the *reachable set*, not the fragile points → `transitive-closure`; max throughput (though min-cut shares the math) → `max-flow`; just "are these grouped?" → `union-find`.
- **Pitfall**: edge-connectivity ≠ vertex-connectivity — name which failure you fear (a link vs a whole node).
- **Skiena**: `edge-and-vertex-connectivity`

**Vertex cover** (`id: vertex-cover`) — fewest nodes that touch every connection
- **Fit**: must you pick the *minimum set of nodes* so that every edge has at least one chosen endpoint?
- **Bridge words**: cover every connection, watch every link, fewest guards, witnesses, monitor all relationships, hit every edge
- **Maps when**: place fewest guards/monitors so every street/edge is watched; pick fewest tests/witnesses touching every requirement-link.
- **Domain → algorithm**: candidate (guard/test/monitor) → node · relationship to watch → edge · the chosen watchers → vertex cover · goal → fewest nodes covering all edges.
- **Not this if**: cover *elements* with chosen *sets* → `set-cover`; you want the largest *conflict-free* group instead → `interval-scheduling` (its complement is an independent set); partition connections per round → `graph-coloring`.
- **Pitfall**: NP-hard, but a simple 2-approximation exists (take both endpoints of any uncovered edge); a vertex cover's complement is an independent set.
- **Skiena**: `vertex-cover`

**Clique** (`id: clique`) — the largest fully-connected group
- **Fit**: must you find a group where *every* member is directly connected to every other (mutual, all-pairs)?
- **Bridge words**: fully connected, all-pairs compatible, dense group, coalition, every-to-every, mutual, tight-knit ring
- **Maps when**: detect a cohesive sub-community, a fully-compatible item set, a conflict-free coalition, or a ring where all parties interact.
- **Domain → algorithm**: member → node · "compatible/interacts with" → edge · the group → a clique · largest such group → maximum clique.
- **Not this if**: mere reachability/grouping (not all-pairs) → `union-find`; the largest *non*-conflicting set → `interval-scheduling` (independent set is the complement); a balanced split → `graph-partition`.
- **Pitfall**: NP-hard; dense-subgraph heuristics for large graphs — exact maximum clique only for small ones.
- **Skiena**: `clique`

### Tree & data-structure family

**Priority queue** (`id: priority-queue`) — always process the most urgent next
- **Fit**: must work items be served in an order driven by a score/deadline/severity that can change over time?
- **Bridge words**: priority, urgency, severity, deadline, top-k, next to process, queue by score, dispatch
- **Maps when**: dispatch the most urgent ticket/job/alert next; maintain a live "what's next" under changing priorities.
- **Domain → algorithm**: work item → element · urgency field → key · "serve next" → extract-min/max · re-prioritize → decrease-key.
- **Not this if**: a fixed pre-computed schedule, not live dispatch → `job-scheduling`; strict real-time deadlines with preemption → `edf`/`fpp`; order by dependency not score → `topological-sort`.
- **Pitfall**: changing a key needs a decrease-key-capable heap, or stale entries with lazy deletion.
- **Skiena**: `priority-queues`

**Median & selection** (`id: median-and-selection`) — the k-th / percentile without a full sort
- **Fit**: do you only need the k-th smallest, the median, or a percentile — not the whole sorted list?
- **Bridge words**: median, percentile, p95, top-k, k-th, quantile, threshold, rank cutoff
- **Maps when**: SLA percentiles, median price/income, "the top N by score", outlier cutoffs.
- **Domain → algorithm**: record → element · ranking field → key · the cutoff → the k-th order statistic · "top N" → a partial selection.
- **Not this if**: you need the *full* order → `sorting` (Skiena); live top-k over a changing stream → `priority-queue`; range aggregates → `segment-tree`.
- **Pitfall**: sorting the whole set to grab one percentile is O(n log n) waste — Quickselect is O(n) average.
- **Skiena**: `median-and-selection`

**Trie (prefix tree)** (`id: trie`) — prefix matching, autocomplete
- **Fit**: do you look things up by a shared *prefix* (autocomplete, longest-prefix routing)?
- **Bridge words**: prefix, starts-with, autocomplete, longest-prefix, code lookup
- **Maps when**: suggest completions, route by prefix (phone/IP/SKU), share storage of common prefixes.
- **Domain → algorithm**: key string → root-to-node path · character → edge · "all keys starting with P" → subtree under P.
- **Not this if**: exact key lookup only → a hash index (`dictionaries`, Skiena); fuzzy/typo matching → `approximate-string-match`; substring (not prefix) search → `inverted-index`/suffix structures.
- **Pitfall**: memory-heavy for sparse alphabets → use radix/compressed tries.
- **Skiena**: `string-matching` (related)

**B-tree / B+tree** (`id: b-tree`) — sorted, disk-friendly index
- **Fit**: must records be stored sorted and queried by exact key *and* range, at scale on disk?
- **Bridge words**: sorted index, range query, ordered keys, secondary index
- **Maps when**: the database access pattern is key + range scans over large, persisted, ordered data.
- **Domain → algorithm**: record key → index key · range scan → ordered leaf traversal · lookup → root-to-leaf descent.
- **Not this if**: pure point lookup, no ranges → hash index; multi-dimensional/spatial → `kd-tree`/`interval-tree`; in-memory range aggregates → `segment-tree`.
- **Pitfall**: usually you do not build this — let the database own it; choosing the right index column is the real decision.
- **Skiena**: `dictionaries` (related)

**Segment tree / Fenwick (BIT)** (`id: segment-tree`) — range aggregates with updates
- **Fit**: do you repeatedly ask sum/min/max over a *range* while values keep changing?
- **Bridge words**: range sum, prefix sum, max-in-interval, running aggregate, cumulative
- **Maps when**: live dashboards / leaderboards / running totals over a mutable array.
- **Domain → algorithm**: position → leaf · range aggregate → internal node · point/range update → tree update.
- **Not this if**: values are static → precomputed prefix sums; on-disk ordered ranges → `b-tree`; time-interval overlaps → `interval-tree`.
- **Pitfall**: range *updates* need lazy propagation.
- **Skiena**: `range-search` (related)

**Interval tree** (`id: interval-tree`) — overlapping time/range intervals
- **Fit**: do you store intervals and ask "which overlap this point/interval?" (collision, double-booking)?
- **Bridge words**: overlaps, interval, time collision, double-booking, conflicting reservation, busy window
- **Maps when**: detect schedule clashes, find all bookings active at time T.
- **Domain → algorithm**: booking/window → interval · "is it free?" → stabbing query · clash → overlapping intervals returned.
- **Not this if**: you need the *maximum non-overlapping set* → `interval-scheduling`; categorize clashing items into slots → `graph-coloring`; numeric range sums → `segment-tree`.
- **Pitfall**: half-open vs closed intervals — be explicit or you get off-by-one clashes.
- **Skiena**: `intersection-detection` / `range-search` (related)

**Kd-tree / spatial index** (`id: kd-tree`) — nearest & region queries in space
- **Fit**: do items live in a multi-dimensional space (locations, coordinates, feature tuples) and queries ask "what is nearest to / inside this region?"
- **Bridge words**: location, coordinates, nearest facility, geo-fence, within radius, region query, spatial, point-in-region
- **Maps when**: nearest-store/depot assignment, geo-fencing, "which records fall in this box", proximity alerts.
- **Domain → algorithm**: located item → point in k-space · query location → query point · "nearest" → nearest-neighbor search · "inside this area" → range/window query.
- **Not this if**: similarity is high-dimensional/semantic (embeddings) → `vector-similarity`; you need the *containing* region, not the nearest item → `point-location`; overlaps of *time* windows → `interval-tree`; ordered single-key ranges on disk → `b-tree`; partition space into nearest-site regions → `voronoi-diagrams` (Skiena).
- **Pitfall**: kd-trees degrade above ~10–20 dimensions (curse of dimensionality) → switch to `vector-similarity` (HNSW) for high-dim.
- **Skiena**: `kd-trees` (and `nearest-neighbor-search` / `range-search`)

**Point location** (`id: point-location`) — which region/zone contains this point
- **Fit**: is space divided into regions (zones, districts, polygons) and queries ask "which one contains this point?"
- **Bridge words**: which zone, which district, point-in-polygon, geofence, region containing, hit-test, tax zone, map tile
- **Maps when**: tax/jurisdiction zones, delivery-area lookup, geofencing alerts, "which territory is this address in?"
- **Domain → algorithm**: location → query point · zone → region (polygon) · the answer → the containing region · repeated queries → a planar subdivision / R-tree.
- **Not this if**: "nearest item", not "containing region" → `kd-tree`; assign each point to the nearest *site* → `voronoi-diagrams` (Skiena); time-window membership → `interval-tree`.
- **Pitfall**: point-in-polygon edge cases (boundaries, holes) — use a tested geometry library, never hand-roll.
- **Skiena**: `point-location`

### Optimization family

**Knapsack 0/1** (`id: knapsack`) — most value into one budget
- **Fit**: must you pick a *subset* of items, each with a cost and a value, to maximize value within one budget?
- **Bridge words**: budget, capacity, select subset, value vs cost, fits, what to buy, pick the best set
- **Maps when**: choose what to fund/buy/load given one limiting resource; subset-sum / integer-partition are special cases.
- **Domain → algorithm**: candidate → item · resource consumed → size/weight · benefit → value · the limit → capacity C · the chosen set → the optimal subset.
- **Not this if**: items must spread across *many* containers minimizing count → `bin-packing`; several simultaneous limits → `knapsack-multidim`; cover requirements not maximize value → `set-cover`; a linear objective with general constraints → `linear-programming`.
- **Pitfall**: NP-complete in general, but an "easy" hard problem — DP in O(nC) works when C is small (≤~1000s).
- **Skiena**: `knapsack`

**Knapsack multidimensional** (`id: knapsack-multidim`) — subset under several limits
- **Fit**: like knapsack, but each item consumes *several* limited resources at once (weight *and* volume *and* budget)?
- **Bridge words**: multi-constraint, constrained on X and Y, several capacities
- **Maps when**: selection where more than one capacity binds simultaneously.
- **Domain → algorithm**: item → element · each resource → a separate capacity dimension · the pick → subset feasible in all dimensions maximizing value.
- **Not this if**: one limit only → `knapsack`; continuous/linear → `linear-programming`.
- **Pitfall**: complexity grows exponentially with dimensions → solver, not DP.
- **Skiena**: `knapsack` (variant)

**Bin packing** (`id: bin-packing`) — fewest fixed-capacity containers
- **Fit**: must many sized items be distributed into the *minimum number* of fixed-capacity containers?
- **Bridge words**: pack into containers, fit items, minimize bins, cargo loading, VM placement, cutting stock, batch into trucks
- **Maps when**: load trucks/VMs/shipments/batches so as few containers as possible are used.
- **Domain → algorithm**: item → object with a size · container → a bin of fixed capacity · the loading → an assignment · goal → minimize bin count.
- **Not this if**: one container, maximize value of what fits → `knapsack`; conflict-based slotting (not capacity) → `graph-coloring`; non-overlap in time → `interval-scheduling`.
- **Pitfall**: NP-hard; First-Fit-Decreasing is within ~11/9 of optimal and is the practical default.
- **Skiena**: `bin-packing`

**Set cover** (`id: set-cover`) — fewest sets to cover every requirement
- **Fit**: must you choose the *minimum number* of available sets that together cover every required element?
- **Bridge words**: cover all, minimum selection, hit every requirement, skill coverage, sensor placement, fewest to satisfy all
- **Maps when**: hire the fewest people whose skills cover all needs; pick fewest tests covering all requirements; place fewest sensors covering all zones.
- **Domain → algorithm**: required thing → universe element · option (person/test/sensor) → a set of elements it covers · the chosen options → the cover · goal → fewest sets covering the universe.
- **Not this if**: pick the *largest disjoint* selection → `interval-scheduling`; cover graph *edges* by picking nodes → `vertex-cover`; maximize value under one budget → `knapsack`.
- **Pitfall**: NP-hard; the greedy "take the set covering the most uncovered" is the classic ln(n)-approximation.
- **Skiena**: `set-cover`

**Interval scheduling / Independent set** (`id: interval-scheduling`) — largest non-conflicting selection
- **Fit**: from items that pairwise conflict, must you select the *largest* mutually non-conflicting subset?
- **Bridge words**: non-overlapping, conflict-free, fit the most bookings, no two at once, disjoint selection
- **Maps when**: accept the most reservations that do not clash; pick the most compatible tasks for one resource.
- **Domain → algorithm**: candidate → interval/node · "conflicts with" → overlap/edge · the accepted set → an independent set · for intervals, earliest-finish-first → the optimum.
- **Not this if**: you must *cover* everything, not avoid conflict → `set-cover`; categorize all into fewest slots → `graph-coloring`; weighted/multi-resource → `linear-programming`.
- **Pitfall**: weighted-interval scheduling is solvable by DP; general independent set is NP-hard — the *interval* special case is easy (greedy earliest-finish).
- **Skiena**: `independent-set` (and `set-packing`)

**Linear / Integer programming (LP/IP)** (`id: linear-programming`) — linear objective under linear constraints
- **Fit**: can the goal be written as maximize/minimize a linear score subject to linear constraints?
- **Bridge words**: maximize, minimize, optimize, allocate under limits, blend, production plan, max profit, min cost
- **Maps when**: resource allocation / transportation / blending / planning where everything is (roughly) linear.
- **Domain → algorithm**: decision → variable · objective → linear function · business limit → linear constraint · integral choices → integer variables.
- **Not this if**: the objective is a nonlinear black box → `csp` / constrained optimization (`constrained-unconstrained-optimization`, Skiena); purely logical (true/false) → `sat`; pure network throughput → `max-flow`.
- **Pitfall**: use a solver, never hand-roll; integrality (IP) is much harder than continuous (LP).
- **Skiena**: `linear-programming`

**Constraint Programming (CSP)** (`id: csp`) — variables + rich constraints
- **Fit**: is it "assign values to variables so all these (possibly non-linear/logical) constraints hold"?
- **Bridge words**: satisfies, compatible, feasible assignment, all-different, constrained configuration, rostering rules
- **Maps when**: rostering, timetabling, configuration — many heterogeneous constraints, feasibility first.
- **Domain → algorithm**: choice → variable · allowed value → domain · business rule → constraint · a valid plan → a satisfying assignment.
- **Not this if**: clean linear objective → `linear-programming`; pure boolean logic → `sat`; one budget subset → `knapsack`.
- **Pitfall**: modeling is the skill; good global constraints (`all_different`, `cumulative`) make or break performance.
- **Skiena**: `constrained-unconstrained-optimization` / `satisfiability` (related)

**SAT** (`id: sat`) — boolean satisfiability
- **Fit**: can the rules be encoded as a boolean formula, and you need *any* true/false assignment that satisfies it (or proof none exists)?
- **Bridge words**: excluding, forcing, requires/implies, mutually exclusive, must-hold, consistency, feasibility (binary)
- **Maps when**: product configurators, eligibility logic, conflict/consistency checks — everything is yes/no.
- **Domain → algorithm**: binary choice → boolean variable · rule → clause · "is any valid config possible?" → satisfiability · the config → a satisfying assignment.
- **Not this if**: variables are quantities, not booleans → `csp`/`linear-programming`; you want the *least-violating* assignment → `maxsat`.
- **Pitfall**: NP-complete, but modern solvers handle millions of clauses — the win is *encoding* it as SAT.
- **Skiena**: `satisfiability`

**MaxSAT / MinSAT** (`id: maxsat`) — best approximate satisfaction
- **Fit**: when hard rules collide, must you satisfy the *most* (weighted) soft rules?
- **Bridge words**: maximally satisfied, least violated, soft constraints, best-effort compliance
- **Maps when**: over-constrained rosters/policies where some rules must yield — minimize weighted violations.
- **Domain → algorithm**: must-hold rule → hard clause · preference → weighted soft clause · the outcome → assignment maximizing satisfied weight.
- **Not this if**: a satisfying assignment is guaranteed/required → `sat`; quantitative objective → `linear-programming`.
- **Pitfall**: weighting the soft clauses is a modeling decision with real fairness consequences.
- **Skiena**: `satisfiability` (variant)

### Search & retrieval

**Inverted index** (`id: inverted-index`) — full-text search
- **Fit**: must you find documents/records *containing* given terms, fast, over a large corpus?
- **Bridge words**: search, find containing, full-text, keyword query, relevance
- **Maps when**: a search bar / log search / catalog text search over many documents.
- **Domain → algorithm**: document → posting list entry · term → dictionary key · query → set intersection of posting lists · relevance → TF-IDF/BM25 score.
- **Not this if**: prefix-only → `trie`; it is one big text queried many ways → `suffix-array`; fuzzy/typo tolerance → `approximate-string-match`; semantic similarity → `vector-similarity`.
- **Pitfall**: tokenization/stemming/analyzer choice dominates result quality.
- **Skiena**: `string-matching` / `searching` (related)

**Suffix trees & arrays** (`id: suffix-array`) — index one big text, query it many ways
- **Fit**: is there one long, stable text searched many times for substrings, longest repeats, or shared fragments?
- **Bridge words**: indexed text, repeated fragment, longest repeat, substring search, shared chunk, full-text over one corpus
- **Maps when**: search one big document/genome/log archive repeatedly; find duplicated passages; longest-repeated-substring analytics.
- **Domain → algorithm**: text → indexed string · query → substring lookup over the index · repeated fragment → repeated suffix · text shared between two → a generalized suffix structure.
- **Not this if**: many short patterns over *changing* text → `inverted-index` (or `string-matching`, Skiena); prefix-only → `trie`; fuzzy → `approximate-string-match`.
- **Pitfall**: suffix *trees* are memory-heavy → prefer suffix *arrays* (+ LCP) in practice.
- **Skiena**: `suffix-trees-and-arrays`

**Approximate string matching** (`id: approximate-string-match`) — fuzzy / typo-tolerant matching
- **Fit**: must records/strings match *despite* typos, spelling variants, or formatting differences?
- **Bridge words**: fuzzy match, typo tolerant, edit distance, spell check, record linkage, dedupe, near-duplicate names
- **Maps when**: deduplicate customers, link records across systems, tolerant search, OCR/spelling correction.
- **Domain → algorithm**: string → sequence · "how different" → edit (Levenshtein) distance · a match → distance below a threshold · entity resolution → blocking + pairwise scoring.
- **Not this if**: exact match only → `inverted-index` (or `dictionaries`, Skiena); you need the shared ordered chunk (a diff) → `lcs`; meaning-level similarity → `vector-similarity`; structural near-duplicates of *sets* → `lsh`.
- **Pitfall**: all-pairs comparison is O(n²) — use blocking/indexing (e.g. trigram, phonetic) first.
- **Skiena**: `approximate-string-matching`

**Longest common subsequence** (`id: lcs`) — largest shared ordered content (diff)
- **Fit**: must you measure or extract the largest shared chunk / ordered selection between two sequences?
- **Bridge words**: diff, common subsequence, shared chunk, sequence similarity, version compare, alignment, what changed
- **Maps when**: file/version diffing, plan-vs-actual sequence comparison, change detection, DNA/sequence alignment.
- **Domain → algorithm**: sequence → token list · shared contiguous run → longest common *substring* · shared ordered (gaps allowed) selection → LCS *subsequence* · the changes → the non-LCS remainder.
- **Not this if**: single-character typo distance → `approximate-string-match` (edit distance); set overlap with no order → set similarity (`lsh`); semantic similarity → `vector-similarity`.
- **Pitfall**: substring (contiguous) vs subsequence (gaps allowed) are different problems — name which.
- **Skiena**: `longest-common-substring`

**LSH (Locality-Sensitive Hashing)** (`id: lsh`) — near-duplicate / set similarity at scale
- **Fit**: must you find near-duplicates among huge numbers of items without comparing all pairs?
- **Bridge words**: near-duplicate, similar set, fingerprint, plagiarism, dedupe at scale
- **Maps when**: detect duplicate documents/listings/images via MinHash/SimHash buckets.
- **Domain → algorithm**: item → signature (MinHash/SimHash) · "probably similar" → same hash bucket · candidate pairs → within-bucket comparisons.
- **Not this if**: exact dense-vector nearest neighbor → `vector-similarity`; small data (all-pairs fine) → direct similarity; string-edit fuzz → `approximate-string-match`.
- **Pitfall**: tune bands/rows for the precision/recall tradeoff — it is probabilistic.
- **Skiena**: `nearest-neighbor-search` (related)

**HNSW / IVF — vector similarity** (`id: vector-similarity`) — semantic nearest neighbor
- **Fit**: must you retrieve the most *semantically* similar items via embedding vectors?
- **Bridge words**: semantic similarity, embedding, nearest neighbor, recommendation, RAG retrieval, k-NN
- **Maps when**: recommendation, semantic search, RAG, anomaly detection over learned embeddings.
- **Domain → algorithm**: item → embedding vector · "similar to" → small vector distance · top matches → approximate k-NN over an HNSW/IVF index.
- **Not this if**: exact tokens/strings → `inverted-index`/`approximate-string-match`; set overlap → `lsh`; low-dimensional spatial points → `kd-tree`.
- **Pitfall**: recall vs latency is a tuning knob; embedding quality bounds everything.
- **Skiena**: `nearest-neighbor-search` (related)

### Time & scheduling

**EDF (Earliest Deadline First) / Rate Monotonic** (`id: edf`) — real-time deadline scheduling
- **Fit**: must tasks with hard deadlines be scheduled so none is missed (preemptive, real-time)?
- **Bridge words**: deadline, real-time, periodic task, must-not-miss, hard timing
- **Maps when**: control loops, telemetry, hard-real-time pipelines with strict timing guarantees.
- **Domain → algorithm**: task → job with a deadline · priority → earliest deadline (EDF) or shortest period (RM) · feasibility → utilization bound.
- **Not this if**: soft priorities, no hard deadlines → `priority-queue`; offline plan with precedence → `job-scheduling`.
- **Pitfall**: EDF is optimal on one CPU but unstable past 100% utilization; multiprocessor changes the rules.
- **Skiena**: `job-scheduling` (related)

**FPP (Fixed Priority Preemptive)** (`id: fpp`) — static-priority preemptive scheduling
- **Fit**: are task priorities *fixed* and the highest-priority ready task always preempts?
- **Bridge words**: fixed priority, higher wins, preempt, priority ceiling
- **Maps when**: embedded/RTOS systems with statically assigned priorities.
- **Domain → algorithm**: task → fixed-priority job · scheduler → run highest ready · shared resource → priority-inheritance/ceiling.
- **Not this if**: priorities derive from deadlines dynamically → `edf`; non-real-time dispatch → `priority-queue`.
- **Pitfall**: priority inversion → use inheritance/ceiling protocols.
- **Skiena**: `job-scheduling` (related)

## Output contract

This catalog is a **lookup source, not an output owner** — consumers own
their own output schemas. For interoperability, a consumer resolving an
entry should emit candidates keyed by `algorithm_id` (the `id:` slugs
defined here), with verbatim bridge-word quotes, an explicit domain →
algorithm element mapping, a match basis, a confidence level, any near-miss
redirects, and `skiena_pointer` (the entry's `Skiena:` cross-reference slug).
The recommended candidate shape:

```yaml
- algorithm_id: "job-scheduling"        # `id:` slug from this catalog
  algorithm_name: "Job scheduling / Critical path"
  match_basis: "bridge-word | structural | both"
  fit_question: "<the entry's fit question, verbatim from this catalog>"
  bridge_words:
    - quote: "<verbatim>"
      source: "<file>"
      loc: "<line or section ref>"
      word: "<extracted word, verbatim — null for a structural match>"
  domain_to_algorithm_mapping:    # every element the entry's Domain → algorithm names
    stage: "node"
    "must finish before": "directed edge (predecessor → successor)"
    "stage length": "node or edge weight"
    "end date": "longest weighted path (critical path)"
  near_misses:                    # the entry's 'Not this if' redirects the evidence brushed against
    - to: "topological-sort"        # `id:` slug of the redirected entry
      because: "<verbatim quote that brushed against it>"
  confidence: "high | medium | low"
  rationale: "<one paragraph, source language>"
  skiena_pointer: "job-scheduling"
```

`bridge_words` always preserves the corpus words verbatim in their
source language — the English row that matched is *not* a substitute for
the evidence quote. `domain_to_algorithm_mapping` should carry **every
element** the entry's *Domain → algorithm* line names (node, edge, weight,
constraint, …), not a single line — the mapping is the payload, and a
one-element mapping is usually an under-read. `near_misses` carries the
entry's *Not this if* redirects that the evidence brushed against, each as
`{to, because}` — the redirected `algorithm_id` and the quote behind it.
