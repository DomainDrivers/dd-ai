# Candidate Areas (frozen)

**Frozen at**: 2026-06-11T13:19:09Z
**Turns**: 4

Seven candidate business areas, each backed by cited corpus quotes. Names are
verbatim domain vocabulary (Polish from the interview / boards, English where the
process board is English). This artifact only *names* areas and records evidence;
it does not classify or assign boundaries.

## Candidate areas

### Fazy i etapy projektu

**ID**: `fazy_i_etapy_projektu`
**User-asserted**: false
**Surfaced via**: clustering
**Evidence quality**: 1 fact, 2 opinion, 1 declarative (+3 unassessed board quotes)

Planning a project's phases/stages — a stage tree where intra-phase stages are
independent and inter-phase stages are dependent — plus an ordering algorithm and
schedule/end-date computation. **Resolved (C1):** the schedule is auto-calculated
from the stage tree by default; "manual mode" is an explicit, locked override a
recalculation will not overwrite.

**Evidence** (7 quotes, 3 sources + user):
- `intake/wywiad-z-prezesem.md` L81–L83 [declarative]:
  > Faza projektu … zawiera etapy, które od siebie są niezależne … a w kolejnych fazach etapy są od siebie zależne.
- `intake/wywiad-z-prezesem.md` L86–L88 [opinion]:
  > Chciałbym mieć taki algorytm … układa mi te fazy, patrzy na te etapy.
- `intake/wywiad-z-prezesem.md` L78–L80 [opinion]:
  > Ciężko jest ułożyć te fazy projektu … chciałbym zobaczyć proponowane fazy.
- `big picture.es.md` Director #2:
  > zaplanowano etapy projektu (długość, zależności między etami, ewentualne zasoby)
- `process level.es.md` Stages #1–#3 / #11–#12:
  > Define stages (tree) … stages defined / stage dates (schedule) calculated … project end date calculated
- `user-answers.md` C1 inline answer [fact]:
  > Auto-calculated by default; manual mode is a locked override.

---

### Ryzyko projektowe

**ID**: `ryzyko_projektowe`
**User-asserted**: false
**Surfaced via**: clustering
**Evidence quality**: 2 fact, 2 opinion, 1 declarative (+4 unassessed board quotes)

Scoring and monitoring project risk and propagating it across dependent phases
(nested risk), so risky projects are caught early. **Resolved (G1):** risk is
measured in money (expected loss / cost-at-risk). **Resolved (C3):** the system
detects + suggests; a human approves any change.

**Evidence** (8 quotes, 3 sources + user):
- `intake/wywiad-z-prezesem.md` L16–L18 [opinion]:
  > Jak ryzyko w jednym etapie wpłynie na kolejne fazy? Potrzebujemy mechanizmu … wykrywać takie zagnieżdżone ryzyka.
- `intake/wywiad-z-prezesem.md` L57–L59 [opinion]:
  > jak ryzyko opóźnienia w fundamentach, wpływa na stropy i elektrykę …
- `intake/wywiad-z-prezesem.md` L43–L45 [declarative]:
  > Czy ryzyko po prostu wszystkich projektów w moim systemie jest mniejsze.
- `big picture.es.md` Director #7/#8:
  > monitoring ryzyka / symulacja stanu innych projektów gdyby zamknięto ten
- `process level.es.md` Risk policy #1 / Project intake #6:
  > risk policy (weekly) / risk detected
- `user-answers.md` G1 inline answer [fact]:
  > Risk = money (expected loss / cost-at-risk).
- `user-answers.md` C3 inline answer [fact]:
  > System suggests; human approves any change.

---

### Alokacja i dostępność zasobów

**ID**: `alokacja_i_dostepnosc_zasobow`
**User-asserted**: false
**Surfaced via**: clustering
**Evidence quality**: 1 opinion, 1 declarative (escalated) (+ unassessed board quotes)

The resource-allocation *mechanism* — check availability, allocate or queue the
demand, find a substitute, reallocate, release. Includes the "is there even
capacity/potential?" gate before planning.

**Evidence** (6 quotes, 3 sources):
- `process level.es.md` Allocation #5 / #8–#9:
  > is resource available? / resources not aquired … demand queued
- `process level.es.md` Allocation #17–#18:
  > release resources … resources released
- `big picture.es.md` Project-manager #16–#17 / #3:
  > wykryto braki / wyszukano zamiennik zasobu / czy w ogóle są zasoby? (czy jest potencjał?) jeszcze nie warto planować etapów
- `intake/wywiad-z-prezesem.md` L40–L42 [opinion]:
  > przycisk … pozwala przydzielić pracownika do innego projektu … żeby on dobrze działał.
- `intake/wywiad-z-prezesem.md` L7–L9 [declarative; escalated]:
  > przycisk do przenoszenia pracowników pomiędzy projektami, nie działa tak efektywnie.

---

### Zarządzanie zasobami, kompetencjami i zezwoleniami

**ID**: `zarzadzanie_zasobami_kompetencjami_zezwolenia`
**User-asserted**: false
**Surfaced via**: clustering (absorbed candidate "Kwalifikacje i zezwolenia" at the review gate)
**Evidence quality**: 2 fact, 1 opinion, 1 declarative (escalated)

Master data for resources and capabilities (resource definitions and instances,
employee/device/material summaries, vacation/sick-leave, skills→capability) **and**
the legal/competence gating of worker→project assignment. **Resolved (C2):**
missing skills = warn (assignment allowed with accepted risk); missing permit =
hard block (criminal exposure).

**Evidence** (6 quotes, 2 sources + user):
- `intake/wywiad-z-prezesem.md` L66–L69 [fact]:
  > mogę wysłać pracownika bez umiejętności … Akceptuję to … ale nie mogę wysłać … kto nie ma odpowiedniego zezwolenia … No i zamykają mnie.
- `intake/wywiad-z-prezesem.md` L20–L22 [declarative; escalated]:
  > nie możemy pozwolić … kto nie ma odpowiednich kwalifikacji czy zezwoleń.
- `intake/wywiad-z-prezesem.md` L61–L62 [opinion]:
  > czy bardziej opłaca mi się skilować jakiegoś pracownika, czy może opłaca mi się bardziej coś kupić.
- `process level.es.md` Resources & capability #1–#3 / #14–#16 / #10–#13:
  > resource definitions … / Change skills … capability unlocked / Add vacation … Add sick leave
- `big picture.es.md` External #2:
  > systemy do zarządzania zasobami (inventory, trainig, vacation)
- `user-answers.md` C2 inline answer [fact]:
  > Skills = warn, permits = hard block.

---

### Opłacalność decyzji i prognozy finansowe

**ID**: `oplacalnosc_decyzji_i_prognozy_finansowe`
**User-asserted**: false
**Surfaced via**: clustering (absorbed candidates "Autonomia / własność decyzji" and "Alternatywna miara opłacalności (prestiż)" at the review gate)
**Evidence quality**: 2 fact, 3 opinion, 2 declarative (1 escalated) (+ unassessed board quotes)

The recurring "czy się opłaca?" decisions — move a worker, buy/lease equipment,
hire vs upskill, terminate a project, pay vs avoid a penalty — with ROI/earnings
forecasts and what-if simulation of resources not yet owned. Includes two folded
facets: **decision autonomy/ownership** (resolved C3 — system suggests, human
approves) and an **alternative non-financial metric** (e.g. prestige) seen only
on the board.

**Evidence** (10 quotes, 3 sources + user):
- `intake/wywiad-z-prezesem.md` L9–L11 [declarative; escalated]:
  > czy przenoszenie pracownika przyniesie nam korzyści finansowe?
- `intake/wywiad-z-prezesem.md` L48–L50 [opinion]:
  > nawet taką nieistniejącą koparkę … czy to się opłaci bardziej, bo … znam rate leasingu.
- `intake/wywiad-z-prezesem.md` L51–L52 [declarative]:
  > czy jak mi się ryzyko zmniejszy, to … się opłaca taki manew.
- `intake/wywiad-z-prezesem.md` L54 [opinion]:
  > fajnie, jakby liderzy sami wykrywali takie ryzyka i proponowali strategię zarządzania nimi.
- `intake/wywiad-z-prezesem.md` L77 [opinion]:
  > Najpierw żeby system sam mi podpowiadał takie ruchy.
- `big picture.es.md` value #6 / #7:
  > ZMIANA: inna funkcja porównująca opłacalność (np prestiż) / dashboard czy się jeszcze opłaca?
- `process level.es.md` Hot zones / Earnings #1–#2:
  > does it pay off to move resource from another project? / add income and cost … earnings recalculated
- `user-answers.md` C3 inline answer [fact]:
  > System suggests; human approves any change.
- `user-answers.md` G1 inline answer [fact]:
  > Risk = money, nets against earnings/penalties.

---

### Kary umowne i terminy

**ID**: `kary_umowne_i_terminy`
**User-asserted**: false
**Surfaced via**: clustering
**Evidence quality**: 2 opinion, 1 declarative (+ unassessed board quotes)

Contractual-penalty deadlines for delays, with an early warning that feeds a
pay-the-penalty-vs-move-resources comparison.

**Evidence** (4 quotes, 2 sources):
- `intake/wywiad-z-prezesem.md` L70–L71 [opinion]:
  > często tydzień przed dniem, w którym miałam zapłacić jakąś karę za spóźnienie, chciałbym dostać informacje o tym.
- `intake/wywiad-z-prezesem.md` L72–L75 [opinion]:
  > mogę sobie … szacować, że kogoś chce przenieść … żeby zmieniać tę karę. A może kara nie jest tak duża wcale …
- `intake/wywiad-z-prezesem.md` L23–L25 [declarative]:
  > informacje na temat potencjalnych kar finansowych za opóźnienia … odpowiednio wcześnie.
- `big picture.es.md` Director #12 / value #5:
  > zapłacenie kary umownej / zapłaciłem karę umowną za zerwanie umowy i zatrzymanie projektu

---

### Monitoring postępu i kosztów

**ID**: `monitoring_postepu_i_kosztow`
**User-asserted**: false
**Surfaced via**: clustering
**Evidence quality**: 1 declarative (+ unassessed board quotes)

Operational monitoring of a running project — progress with costs, summary costs,
delay/budget-overrun detection, and reactions (escalation, scope cut).

**Evidence** (5 quotes, 2 sources):
- `big picture.es.md` Director #6 / #15:
  > monitoring postępu (razem z kosztami) / monitoring sumarycznych kosztów
- `big picture.es.md` Project-manager #15 / #18–#19:
  > wykryto przekroczenie budżetu / eskalacja / cięcie zakresu
- `process level.es.md` Project intake #7 / #9:
  > manager informed / Risk dashboard (riskPushNotification)
- `intake/wywiad-z-prezesem.md` L35–L36 [declarative] (oblique):
  > Dużo tych projektów … nie udają … wykrywać już na samym wstępie.

---

## Show-me-the-money gap report

| # | Question | Status | Notes |
|---|----------|--------|-------|
| q1 | Revenue & savings | answered ⚠ | Savings via penalty-avoidance, killing doomed projects early, resource-move ROI. Caveat: profitability metric/threshold still undefined (gap G2); quotes mostly opinion |
| q2 | Risks & opportunities | answered | Risk monitoring, nested cross-phase risk, legal risk; opportunities = optimization, prestige. Risk unit now defined (money, G1 closed) |
| q3 | Integrations | partial | Excel + resource-mgmt systems named, but direction/data-contract/source-of-truth undefined (gap G8) |
| q4 | Regulatory constraints | answered | Permits = hard legal block w/ criminal liability (L66–L69, the one fact quote); "legal rules" on board; two-tier rule confirmed (C2) |
| q5 | Optimization target & metrics | partial ⚠ | Intent clear (min risk / max opłacalność); risk unit now money (G1 closed), but profitability metric/threshold (G2) and director-portfolio-vs-PM-project conflict rule still undefined |

## Source critique — informational

Sources critiqued: `intake/wywiad-z-prezesem.md` (single-speaker; `big picture.es.md`
and `process level.es.md` read for cross-source requirement consistency only).
Findings: 16 (9 high, 5 medium, 2 low). Confidence: 29 quotes annotated — 28
low-confidence (opinion/declarative/hearsay), 1 fact (L66–L69), 3 escalations (all
in the polished letter). Marginalized topics surfaced and folded into areas: 3
(leaders auto-detecting risk → area Opłacalność; prestige metric → area
Opłacalność; system auto-suggesting moves → area Opłacalność). Hidden-dependency
seams pre-flagged: 3. Requirement contradictions: 3 (**all 3 resolved** — C1, C2,
C3); requirement gaps: 8 (1 resolved — G1; 7 open, carried as discovery to-dos).

**Resolved at the gate:**
- C1 — schedule auto-calc default; manual is a locked override.
- C2 — skills = warn, permits = hard block.
- C3 — system suggests, human approves any change.
- G1 — risk = money (expected loss / cost-at-risk).

**Open discovery to-dos** (deferred diagnostic questions):
- G2 — profitability metric + threshold for each "czy się opłaca?" decision — relates to areas Opłacalność, Ryzyko; gap q1/q5
- G3 — penalty-warning lead time ("a week" vs "appropriately early"; fixed or by size?) — relates to area Kary umowne
- G4 — nested-risk propagation rule beyond the foundations→ceilings→electrics example — relates to area Ryzyko
- G5 — phase-ordering objective function + non-adjacent phase dependencies — relates to area Fazy i etapy
- G6 — what-if (non-existent resource) simulation contract (inputs supplied vs held; persisted or transient) — relates to areas Opłacalność, Alokacja
- G7 — "liderzy" actor identity & authority (existing roles or a new one; advisory or can change the plan) — relates to areas Ryzyko, Opłacalność
- G8 — external-system contracts (read/write/own; source of truth for availability/skills/leave) — relates to area Zarządzanie zasobami; gap q3
- F1 — project-failure base rate + definition of "failed" (load-bearing premise, unmeasured)
- F3 — which "sam sobie oszacuję" capabilities the system should compute/recommend vs only display
- F5 — current off-the-shelf tooling cost/time baseline the new algorithm must beat

**Hidden-dependency seams** (pre-flagged, not acted on):
- Resource-move profitability ↔ portfolio risk score (a money-only move can raise portfolio risk).
- Penalty-warning lead time ↔ move-vs-pay comparison (the warning is only useful as input to the comparison).
- Phase/stage ordering ↔ nested-risk propagation (seeing the phases is what reveals the risk).

## Below-threshold clusters (informational)

- "Integracje zewnętrzne (Excel, systemy zasobowe)" — ~2 board quotes (really gap G8)
- "Środowisko korporacyjne (dzieje się poza systemem)" — 1 quote (out-of-system context note)
- "Negocjacje z managerami" — 1 quote (a resourcing-flow step)
