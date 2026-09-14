---
description: Przeprowadź jedną rundę sesji — facylitator coś mówi, Ty odpalasz całą salę (wszystkie persony) równolegle, one same decydują kto się odzywa, potem zbierasz odpowiedzi i aktualizujesz stan.
argument-hint: "<co facylitator mówi/robi w tej rundzie>"
---

# /es-round

> **Zanim zaczniesz**: wczytaj kontrakt orkiestratora — `.claude/skills/event-storming-orchestration/SKILL.md` — i trzymaj się go (jesteś skrybą/orkiestratorem, nigdy nie wcielasz się w personę).

Jedna runda sesji Event Storming.

## Wejście
`$ARGUMENTS` — to co facylitator (user) mówi lub robi w tej rundzie.
Przykład: *"OK zespole, witam wszystkich. Cel: zrozumieć główne wydarzenia w naszej firmie. Każdy pisze samodzielnie 3-5 zdarzeń, w ciszy, 5 minut."*

## Twój flow

### 1. Załaduj stan
- Odczytaj `./sessions/<latest>/state.json`. Jeśli brak — odeślij user'a do `/es-start`.
- Inkrementuj `round`.
- `level` == `big_picture`.
- Odczytaj `state.mode` (`light` albo `full`; brak pola = `full`).
- Odczytaj `state.today` — numer wariantu dnia dla każdej persony. Sesja sprzed
  tego mechanizmu nie ma tego pola: dolicz je wtedy raz, wzorem z `/es-start`,
  i dopisz do stanu.
- Odczytaj `state.memory` — gotowe skróty wcześniejszych wypowiedzi, po jednym
  zdaniu, dopisywane po każdej rundzie. Do promptu persony bierzesz jej pięć
  ostatnich wpisów; bez tego persony recytują te same anegdoty co drugą rundę.
  Sesja sprzed tego mechanizmu nie ma tego pola: zbuduj je raz z `transcript.md`
  i dopisz do stanu. Transkryptu nie streszczasz drugi raz.
- Echo sali bierzesz z tej samej pamięci: wpisy z rundy `n-1` **innych** person,
  trzy ostatnie, z imieniem mówiącego. Bez tego persona odpowiada wyłącznie
  sobie: Prezes nie wejdzie Mecenasowi w słowo, Tomek nie zmentoruje Kasi.

> **Rozgałęzienie trybu.** W trybie **light** persony tylko mówią: użyj routingu z
> kroku 2 (cała sala, samodzielny wybór), ale wyślij personom **prompt light** z
> kroku 3 (bez kartek, sam dialog), zbierz wypowiedzi, dopisz je do
> `transcript.md` i pokaż inline z linią „Odezwali się / Cisi". **Pomiń kroki 4–8**
> (kartki, stan stickies, Miro). W trybie **full** wykonaj wszystkie kroki poniżej.

### 2. Kto reaguje w tej rundzie

Odpalasz całą salę — wszystkie persony równolegle — a każda sama decyduje, czy
się odezwać. Kto ma coś do dodania, daje event albo hot_spot; kto nie ma —
milczy (`stickies: []`, pusty albo jednozdaniowy `dialogue`). Część mówi, część
słucha, runda po rundzie, jak w prawdziwym pokoju.

Charaktery niosą się same: Piotr wchodzi przy uprawnieniach, Agnieszka przy
karach i cashflow, Marek przy realiach placu, Gerome przy dokumentach; Tomek
wraca z przedwczesnym projektowaniem, Gerome drąży i eskaluje.

Gdy facylitator woła kogoś imiennie ("Kasiu, a Ty...?") — to `/es-invite`.

### 3. Wywołaj persony **równolegle**

Zanim wyślesz prompty, wypisz jedną linię — inaczej facylitator patrzy w pustą
konsolę i uznaje narzędzie za zepsute:

```
Runda <n> — czekam na 8 uczestników (1-3 min).
```

Użyj jednej wiadomości z wieloma `Agent` toolami (multi-tool-use).

**Tryb full — prompt z kartkami:**

```
Facylitator powiedział: "<$ARGUMENTS>"

Aktualny stan sesji:
- Runda: <n>
- Poziom: big_picture
- Liczba dotychczasowych stickies: <m>
- Eventy na ścianie (id → tekst, wszystkie): <ev_001 → …, ev_002 → …>
- Twoje dotychczasowe stickies w tej sesji: <Twoje>

Twój dzień w tej sesji: wariant <state.today[persona]> z sekcji „Twój dzień"
w Twoim pliku. Nie zapowiadaj go wprost — ma barwić to, co Cię dziś uwiera
i jak mówisz.

Twoje wcześniejsze wypowiedzi w tej sesji (nie powtarzaj ich ani anegdot,
których już użyłeś): <skróty, do pięciu; puste w rundzie 1>

Co padło na sali w poprzedniej rundzie — możesz się do tego odnieść, przyznać
rację albo zaprzeczyć: <Imię: jedno zdanie; do trzech; puste w rundzie 1>

Zareaguj w roli. Big Picture pozwala TYLKO na dwa typy sticky:

- **`event`** (pomarańczowa kartka) — coś się WYDARZYŁO w domenie. Czasownik w
  czasie przeszłym ("Umowa podpisana", "Klient zapłacił zaliczkę"). To paliwo
  Event Stormingu — dawaj swobodnie, gdy faktycznie nazywasz coś nowego. Może
  być więcej niż jeden event w jednej rundzie.
- **`hot_spot`** (różowa kartka) — **niepewność, spór albo ból**. Gdy nie masz
  pewności, gdy Twoja wersja różni się od cudzej, gdy coś w tej historii boli
  albo grozi — stawiasz różową kartkę zamiast udawać jasność. Nie każde pytanie
  nią jest: "ciekawe", "muszę zajrzeć w papiery" to zwykła wypowiedź.
  Jeśli wstawiasz hot_spot, dorzuć opcjonalnie `relates_to: ev_NNN` — id z
  listy eventów powyżej. Nie zmyślaj id spoza tej listy. Bez tej informacji
  hot_spot ląduje na końcu wału.

Inne typy sticky (command, actor, aggregate, read_model, policy,
external_system, question) **nie są dostępne** na poziomie Big Picture.
Jeśli kusi Cię command lub aggregate — to znak że za szybko schodzisz na
projektowanie. Trzymaj się eventów i hot spotów.

Reagujesz tylko gdy masz coś swojego — nowy event, realny hot_spot albo
konkretną reakcję na to co padło. Jak nie masz nic — nie odzywasz się:
`stickies: []` i pusty albo jednozdaniowy `dialogue`.

Zwróć YAML jak w Twoim agent file (z typami event/hot_spot tylko).
```

**Tryb light — prompt bez kartek** (gdy `state.mode == "light"`):

```
Facylitator powiedział: "<$ARGUMENTS>"

Aktualny stan: runda <n>, sesja Event Storming (tryb light).

Co padło na sali w poprzedniej rundzie — możesz się do tego odnieść, przyznać
rację albo zaprzeczyć: <Imię: jedno zdanie; do trzech; puste w rundzie 1>

Twój dzień w tej sesji: wariant <state.today[persona]> z sekcji „Twój dzień"
w Twoim pliku. Nie zapowiadaj go wprost — ma barwić to, co Cię dziś uwiera
i jak mówisz.

Twoje wcześniejsze wypowiedzi w tej sesji (nie powtarzaj ich ani anegdot,
których już użyłeś): <skróty, do pięciu; puste w rundzie 1>

Zareaguj w roli — zwykłą wypowiedzią, jak przy prawdziwej tablicy. Bez kartek,
bez YAML: sam dialog po polsku, w charakterze. Jeśli nie masz nic do dodania w
tej rundzie — możesz zamilczeć (jedno zdanie albo nic).
```

**Zasada dla orkiestratora — promptowanie person**:

- W promptach **nie wymuszaj sticky liczbowo** ("daj 1-3 sticky"). Pozwól
  personom decydować.
- **Nie podpowiadaj treści hot_spotu.** Ma wychodzić z tego, czego persona
  naprawdę nie wie albo z czym się nie zgadza — nie z Twojej zachęty. Tłumić go
  też nie wolno: persona niepewna swojej wersji stawia różową kartkę.
- Możesz zachęcać do `event` karty gdy facylitator wyraźnie prosi
  ("naklejcie", "dorzućcie") albo gdy persona ma świeży konkret z kontekstu.

### 4. Zbierz odpowiedzi
- Wyparsuj YAML z każdej persony. **Brak bloku YAML w odpowiedzi = `stickies:
  []`** — persona powiedziała coś, ale nic nie nakleiła. Bierzesz sam dialog i
  idziesz dalej; nie układasz kartki za nią i nie zgadujesz, co "chciała" napisać
- **Odfiltruj** sticky o typach innych niż `event` / `hot_spot` (jeśli persona
  wyprodukowała command/aggregate/itp. — pomiń, zaloguj do
  `./sessions/<id>/sync.log` z notą "rejected: <typ> by <persona>")
- Nadaj `id` każdej sticky: `ev_NNN` lub `hs_NNN` — kolejny numer w sesji
- Dodaj `author_persona` i `round`
- Dla hot_spot: jeśli persona podała `relates_to: ev_NNN` — zachowaj
- `synced: false` (flaga sync — patrz `miro-card-spec`)

### 5. Zaktualizuj stan
- Append wszystkie nowe stickies do `state.stickies`
- Jeśli typ == `hot_spot` → dodaj też do `state.hot_spots` (z `relates_to` jeśli jest)
- Dla **każdej persony, która zabrała głos**: dopisz do `state.memory[<persona>]`
  wpis `{ "round": <n>, "s": "<jedno zdanie własnymi słowami — co powiedziała i
  po jaką anegdotę sięgnęła>" }`. Trzymaj pięć ostatnich wpisów na personę,
  starsze usuwaj. Nie cytuj verbatim — persona przepisze cytat z powrotem.

### 6. Zaktualizuj transkrypt
W `transcript.md`:

```markdown
## Runda <n> — <ISO timestamp>

**Facylitator**: <co user powiedział>

**Ewa Nowak (PM)**:
> <dialogue>
> 
> Stickies: event "<text>" (ev_NNN); hot_spot "<text>" (hs_NNN, relates_to: ev_NNN)

**Marek Wiśniewski (brygadzista)**:
> ...
```

### 7. Miro
Runda kończy się na stanie i transkrypcie. Kartki czekają w stanie z
`synced: false`, aż facylitator wywoła `/es-sync-miro` — jeden push obejmuje
wtedy dorobek kilku rund.

Sync po każdej rundzie włącza `state.miro.auto_sync == true` (flaga
`/es-start --auto-sync`): wtedy i tylko wtedy zsynchronizuj nowe stickies
natychmiast, zgodnie z `/es-sync-miro`.

### 8. Wyjście do user'a — **pełne dialogi inline**

Pokaż w odpowiedzi **kompletny dialog każdej persony, która zabrała głos w tej
rundzie** — to jest podstawowa zawartość rundy dla facylitatora. Format:

```markdown
**<Imię Nazwisko> (<rola>)**:
> <dialogue verbatim z subagent output>
>
> Stickies: event "<text>" (<id>); hot_spot "<text>" (<id>); ...
```

Po wszystkich dialogach — **krótka linia meta**:

```
Runda <n> zamknięta. Dorzucono <k> stickies (E:<x> HS:<y>).
Cisi w tej rundzie: <lista person, które nie zabrały głosu>.
Odrzucono: <lista — jeśli ktoś próbował command/aggregate/itp.>.
```

Gdy `state.miro.enabled` i sync nie poszedł w kroku 7, dopisz do linii meta
licznik zaległości: *"Na Miro czeka <p> kartek — `/es-sync-miro`."*

Plik `./sessions/<id>/transcript.md` to **backup tej samej zawartości** — tam
żyje pełna historia sesji, łatwo do odczytania później. Inline w czacie:
żeby facylitator czytał reakcje od razu, bez kliknięcia w plik.

> **Nie skracaj wypowiedzi persony**. Jeśli persona napisała 6 zdań, pokaż 6
> zdań. Skrót robi sens tylko gdy odpowiedź była ekstremalnie długa (>15 zdań)
> — wtedy zaznacz "(skrót)" w prefiksie cytatu.

## Reguły
- **Persony równolegle, nie sekwencyjnie** — to ma być szybkie.
- **Nie roleplayuj nikogo Ty (orkiestrator)** — tylko cytuj wyniki subagentów.
- **Nie zgaduj** — jeśli persona nie odpowiedziała sensownie (puste `stickies: []` i brak `dialogue`), pokaż to: *"Marek nie zabrał głosu w tej rundzie."*
- **Brak YAML to brak kartek**, nie zaproszenie do napisania ich za personę. Odpowiedź bez bloku YAML wchodzi do transkryptu jako sam dialog.
- **Tylko event i hot_spot** — jakikolwiek inny typ sticky jest odrzucony cicho i zalogowany. Nie pokazuj go w outpucie facylitatorowi.
- **Hot_spot niesie niepewność i spór** — nie limitujesz ich liczby ani do nich nie zachęcasz. Ile ich padnie, wynika z tego, co dzieje się na sali.
- **Nie dawaj porad facylitacyjnych** po rundzie. Nie pisz "(coachowo: …)", nie sugeruj zmiany poziomu, nie podpowiadaj kolejnego pytania. Linia meta z licznikami sticky + cisi to wszystko. Coach jest osobno — przez `/es-coach`.

## Nie rób
- Nie syntezuj odpowiedzi person — daj im odpowiedzieć.
- Nie kreuj stickies sam — tylko persony.
- Nie doklejaj coach-tipów ani podsumowań strategicznych. Tylko fakty: dialogi + meta line.
