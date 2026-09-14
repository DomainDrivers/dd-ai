# Candidate Areas (frozen)

**Frozen at**: 2026-07-07T19:06:30Z
**Turns**: 3

## Candidate areas

### fazy projektu

**ID**: `project-phases`
**User-asserted**: false

Structuring a project into phases and stages — their dependencies,
parallelization, and the schedule (start / stage / end dates) — including the
cheaper, faster, modifiable phase-arrangement algorithm the CEO explicitly wants.
Strategic link: the optimization target (q5); seeing the phases is what reveals
project risk.

**Evidence** (6 quotes, 3 sources):
- `intake/wywiad-z-prezesem.md` L81–83:
  > Faza projektu, czyli to jest taka część projektu, która zawiera etapy, które od siebie są niezależne. Mogą być wykonywane sekwencyjnie, a w kolejnych fazach etapy są od siebie zależne.
- `intake/wywiad-z-prezesem.md` L84–85:
  > Czyli jak w faze drugą coś wpadło, tam mamy kilka etapów, to wiadomo, że one zależą od etapów w fazie pierwszej.
- `intake/wywiad-z-prezesem.md` L86–89:
  > Chciałbym mieć taki algorytm, który jest tańszy i działający dobrze, niż te wszystkie gotowe rozwiązania, które mi układa w projektach te fazy… No i przede wszystkim jest modyfikowanym…
- `intake/big picture.es.md` Lane: aktor stormingu: dyrektor (director):
  > zaplanowano etapy projektu (długość, zależności między etami, ewentualne zasoby)
- `intake/process level.es.md` Lane: Scheduling flow — automatic mode:
  > Stages (tree) → Define stages (tree) → stages parallized → stages defined
- `intake/process level.es.md` Hot zones:
  > what is it the order of stages?

---

### ryzyka projektowe

**ID**: `project-risk`
**User-asserted**: false

Detecting, estimating and monitoring project risk — including nested-stage risk
propagation (a delay in foundations cascading to ceilings & electrical) — and
suggesting mitigation strategies for a human to decide and execute. Strategic
link: the chief risk (q2); mitigation is suggest-only per the C1 resolution.

**Evidence** (7 quotes, 4 sources):
- `intake/wywiad-z-prezesem.md` L55–58:
  > Trudnym ogólnie zagnieżdżeniem jest ocena wpływu ryzyka na kolejne zagnieżdżone etapy w fazach projektu. Czyli np. jak ryzyko opóźnienia w fundamentach, wpływa na stropy i elektrykę na takie dwa kolejne etapy.
- `intake/wywiad-z-prezesem.md` L53–54:
  > Czy takie ryzyka projektowe, tutaj dużo dużo dużo ich jest. I fajnie, jakby liderzy sami wykrywali takie ryzyka i proponowali strategię zarządzania nimi.
- `intake/wywiad-z-prezesem.md` L16–18:
  > Jak ryzyko w jednym etapie wpłynie na kolejne fazy? Potrzebujemy mechanizmu, który pomoże nam wykrywać takie zagnieżdżone ryzyka i reagować na nie odpowiednio wcześnie.
- `intake/big picture.es.md` Lane: aktor stormingu: dyrektor (director):
  > monitoring ryzyka … oszacowano ryzyko
- `intake/process level.es.md` Lane: Project intake & risk detection:
  > risk detected (event) → manager informed (event) → Risk dashboard (riskPushNotification) (read model)
- `intake/user-answers.md` C1 inline answer:
  > only suggest

---

### opłacalność

**ID**: `profitability`
**User-asserted**: false

Profitability and what-if simulation — the value engine. Does it pay off to move
a resource / hire a contractor / terminate a project / remove someone; ROI of
adding a real or hypothetical excavator or worker; earnings, margin, revenue;
portfolio-level optimization and the "czy się jeszcze opłaca" dashboard.
Strategic link: the revenue/savings driver (q1, q5) — revenue from executing
projects, optimized via saving and margin.

**Evidence** (8 quotes, 4 sources):
- `intake/wywiad-z-prezesem.md` L48–50:
  > Albo nawet taką nieistniejącą koparkę, gdybym miał koparkę, jak bym ją dodał albo gdybym miał pracownika, jak bym go dodał, to czy to się opłaci bardziej, bo ja przecież znam rate leasingu za koparkę.
- `intake/wywiad-z-prezesem.md` L43–45:
  > Czyli, że jak przesuwam do innego projektu, to wtedy pojawia mi się, czy zarobię wtedy więcej na tym przesunięciu. Czy ryzyko po prostu wszystkich projektów w moim systemie jest mniejsze.
- `intake/wywiad-z-prezesem.md` L61–64:
  > Chciałbym zobaczyć, czy bardziej opłaca mi się skilować jakiegoś pracownika, czy może opłaca mi się bardziej coś kupić do tego projektu.
- `intake/big picture.es.md` Lane: aktor stormingu: dyrektor (director):
  > dashboard czy się jeszcze opłaca? przekroczono budżet/zwiększon o ryzyko/etc
- `intake/big picture.es.md` Lane: value / legend area (green notes):
  > na czym zarabiamy/oszczedzamy
- `intake/process level.es.md` Lane: Floating questions:
  > does it pay off to move resource from another project? … does it pay off to terminate a project? … does it pay to hire a contractor? … does it pay of to remove someone?
- `intake/user-answers.md` Q1 inline answer:
  > firm gets paid for executing project, we want to optimize saving and margin
- `intake/user-answers.md` Q5 inline answer:
  > revenue

---

### przeniesienie zasobów do innego projektu

**ID**: `resource-allocation`
**User-asserted**: false

Allocating and moving/reallocating resources across projects and stages — the
"move worker" button that must work well — with availability checked at
allocation time and substitution when short. Strategic link: the mechanical act
the profitability engine (opłacalność) evaluates; ordering per the C3 resolution
(check at allocation, not upfront).

**Evidence** (7 quotes, 4 sources):
- `intake/wywiad-z-prezesem.md` L40–42:
  > W ogóle jest taki przycisk w tym systemie, obecem tym, który mam, on słabo działa. T aki, który pozwala przydzielić pracownika do innego projektu. Ja chciałbym, żeby on dobrze działał ten przycisk.
- `intake/wywiad-z-prezesem.md` L7–11:
  > Nasza obecna infrastruktura systemowa, mimo iż ma pewne funkcjonalności, jak przycisk do przenoszenia pracowników pomiędzy projektami, nie działa tak efektywnie…
- `intake/big picture.es.md` Lane: aktor stormingu: dyrektor (director):
  > przeniesienie zasobów do innego projektu … optymalizcja zasobów przez przenoszenie pomiędzy projektami
- `intake/process level.es.md` Lane: Scheduling flow — automatic mode:
  > allocate resources (command) → is resource available? (policy) → … → resources allocated (event) … resources not aquired (event) → demand queued (event)
- `intake/process level.es.md` Lane: Reschedule / reallocation flow:
  > reallocate resources (command) → availability check (policy)
- `intake/user-answers.md` C3 inline answer:
  > no upfront check

---

### Resources and capability management

**ID**: `resource-capability`
**User-asserted**: false

Resource / worker / equipment / material master data; skills and permits
(permit missing = hard block, skill missing = accepted risk); vacation and sick
leave; capability unlocking. Strategic link: the permit/legal constraint (q4)
lives here as a hard rule; supplies the resources the opłacalność and
resource-allocation areas reason about.

**Evidence** (5 quotes, 3 sources):
- `intake/wywiad-z-prezesem.md` L65–69:
  > Czyli mogę wysłać pracownika bez umiejętności na jakiś projekt. Akceptuję to, to się może wydarzyć, ale nie mogę wysłać do projektu kogoś, kto nie ma odpowiedniego zezwolenia do tego projektu, bo ktoś się kończy krymiałem. No i zamykają mnie. Nie chciałbym tego.
- `intake/process level.es.md` Lane: Resources and capability management:
  > resource definitions (read model) → Add resource definition (command) → Resource definition added (event) → resource summaries (read model) → Add resource instantion of given definition (command) → Resource instantion added (event)
- `intake/process level.es.md` Lane: Employee / device / material capability lane:
  > Change skills (command) → Skills changed (event) → capability unlocked (event)
- `intake/user-answers.md` Q4 inline answer:
  > any permit is hardblock
- `intake/user-answers.md` C2 inline answer:
  > hard block for permit

---

### kary umowne za opóźnienia

**ID**: `contractual-penalties`
**User-asserted**: false

Contractual penalties for late projects: early warning (~a week before the
penalty falls due) and weighing the penalty against the cost of moving a resource
to avoid it. Strategic link: a concrete money-out event feeding the profitability
decisions (q1) and a legal/contractual obligation (q4).

**Evidence** (4 quotes, 2 sources):
- `intake/wywiad-z-prezesem.md` L70–71:
  > Ostatnią rzeczą jest to, że często tydzień przed dniem, w którym miałam zapłacić jakąś karę za spóźnienie, chciałbym dostać informacje o tym.
- `intake/wywiad-z-prezesem.md` L72–75:
  > I np. wtedy mogę sobie znowu szacować, że kogoś chce przenieść do innego projektu, żeby zmieniać tę karę. A może kara nie jest tak duża wcale, w stosunku do tego, co stracę przesuwając tego pracownika z tego projektu.
- `intake/big picture.es.md` Lane: aktor stormingu: dyrektor (director):
  > zapłacenie kary umownej
- `intake/big picture.es.md` Lane: value / legend area (green notes):
  > np. na tym, że zapłaciłem karę umowną za zerwanie umowy i zatrzymanie projektu

---

## Show-me-the-money gap report

| # | Question | Status | Notes |
|---|----------|--------|-------|
| q1 | Revenue and savings | answered | Revenue = executing construction projects; system optimizes cost saving and margin (`user-answers.md` Q1). |
| q2 | Risks and opportunities | answered | Business, nested-stage, HR/permit and delay-penalty risks all present; leaders-propose-strategy as the opportunity. |
| q3 | Integrations | answered | Four external systems on the big-picture board: Excel; resource-management (inventory/training/vacation); work-time/settlement monitoring; corporate environment. Data-flow direction and online-vs-batch unspecified. |
| q4 | Regulatory constraints | answered | Any missing required permit = hard block; missing skill = accepted risk (`user-answers.md` Q4, C2). No formal regime named; treated as: all permits equal, hard block. |
| q5 | Optimization target and business metrics | answered | Top-level metric = revenue, via saving/margin levers (`user-answers.md` Q5, Q1). Open discovery to-do: the arbitration rule when the director's portfolio optimum conflicts with a manager's single-project optimum was not specified. |

## Contradictions — informational

Sources scanned: `intake/wywiad-z-prezesem.md`, `intake/big picture.es.md`,
`intake/process level.es.md`. Contradictions found: 3 (high: 2, medium: 1);
resolved by the user: 3; unresolved: 0.

Resolved:
- **Risk-mitigation moves: suggest-only vs automatic** (high) → suggest-only; the
  system proposes moves, a human decides and executes (`user-answers.md` C1).
- **Worker lacking skills: forbidden vs accepted** (high) → permit missing = hard
  block, skill missing = accepted risk (`user-answers.md` C2).
- **Ordering: resource check gates stage planning vs stages first** (medium) → no
  upfront check; stages first, resource availability at allocation
  (`user-answers.md` C3).

**Open discovery to-dos**: none from contradictions (all resolved). One open
strategic to-do: q5 director-vs-manager arbitration rule (unspecified).

## Below-threshold clusters (informational)

- "Offer evaluation" — ~4 quotes, 1 source (`process level.es.md`); folded into `opłacalność`.
- "Integrations / external systems" — named only on `big picture.es.md`; cross-cutting integration context, not an area.
- "Nested-risk propagation" — strong in the interview but folded into `ryzyka projektowe`.
