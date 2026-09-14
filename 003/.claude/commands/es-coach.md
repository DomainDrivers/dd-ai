---
description: Prośba o feedback od facilitator-coacha. Domyślnie krótki check ("jak idzie?") bez zamykania sesji; z --full pełny debrief w Markdown.
argument-hint: "[--full] [pytanie do coacha, np. 'czy parkuję Tomka dobrze?']"
---

# /es-coach

> **Zanim zaczniesz**: wczytaj kontrakt orkiestratora — `.claude/skills/event-storming-orchestration/SKILL.md` — i trzymaj się go (jesteś skrybą/orkiestratorem, nigdy nie wcielasz się w personę).

Checkup u coacha. **Nie zamyka sesji.** Zaprasza `facilitator-coach`, żeby
spojrzał na to, co do tej pory się wydarzyło, i dał informację zwrotną.

## Wejście

`$ARGUMENTS` — opcjonalna flaga `--full` oraz opcjonalne pytanie/obszar od
facylitatora.

- **bez `--full`** — tryb `midsession`: 5-7 zdań inline, nic nie ląduje w pliku.
- **`--full`** — tryb `debrief`: pełny raport w formacie z pliku coacha
  (checklista 10 anty-wzorców, metaprogramy, Top 3 momenty, Top 3 do pracy,
  cytat sesji). Zapisujesz go do `./sessions/<id>/debrief.md` i pokazujesz
  inline. Sesji **nadal nie zamykasz** — facylitator może grać dalej.

Przykłady:
- `/es-coach` — ogólny check
- `/es-coach czy za bardzo prowadzę?`
- `/es-coach Tomek mi wjeżdża z agregatami, jak go parkować?`
- `/es-coach --full` — debrief na koniec sesji
- `/es-coach --full jak mi poszło z Gerome'em?` — debrief z akcentem na jedno pytanie

## Przebieg

1. **Załaduj stan** bieżącej sesji (`./sessions/<latest>/state.json`).
   - Jeśli sesja ma 0 rund — odeślij: *"Sesja jeszcze nie ma rund. Coach
     potrzebuje materiału. Najpierw `/es-round`."* i STOP.

2. **Wywołaj `facilitator-coach`** jako subagent. Przekaż:
   - Ścieżkę do `transcript.md`
   - Ścieżkę do `state.json`
   - **Tryb**: `midsession` (krótki feedback, max 5-7 zdań + 1-2 konkretne
     sugestie) albo `debrief`, jeśli w `$ARGUMENTS` było `--full` (pełny
     raport w formacie z pliku coacha).
   - Pytanie facylitatora (`$ARGUMENTS` bez flagi) jeśli niepuste — coach
     **adresuje je wprost**, zanim doda inne obserwacje.

3. **Pokaż feedback userowi inline** — bez parafrazowania, verbatim z output
   subagenta. Po polsku, w tonie coacha (życzliwy + konkretny).

4. **Zapis**: w trybie `midsession` nie zapisuj raportu do osobnego pliku — to
   krótkotrwałe wejście. W trybie `debrief` zapisz raport verbatim do
   `./sessions/<id>/debrief.md` (nadpisując poprzedni). W obu trybach coach
   loguje wejście do `transcript.md` jako sekcja:

   ```markdown
   ## Coach (<mid-session | debrief>, runda <n>) — <ISO timestamp>

   **Pytanie facylitatora**: <args lub "brak — ogólny check">

   **Coach**:
   > <feedback verbatim>
   ```

5. **Nie zmieniaj** `state.round`, `level`, ani innych pól stanu. Coach jest
   obserwatorem.

## Reguły dla coacha (przekaż w prompcie)

- **Krótko** (`midsession`). Max 5-7 zdań. To szybki check, nie pełny raport.
  W `debrief` trzymaj format raportu z pliku coacha w całości.
- **Konkretnie**. Cytuj rundę albo personę po imieniu. Nie pisz ogólników.
- **Adresuj pytanie facylitatora jeśli było** — najpierw to, potem ewentualne
  dodatki.
- **Nie zaczynaj od pochwał** ani "wszystko świetnie". Najpierw obserwacja
  (neutralna albo krytyczna), potem 1-2 sugestie.
- **Nie wkraczaj jako persona** — coach to coach, nie stakeholder.
- **Dopasuj do trybu** (`state.mode`): w **light** nie ma kartek — oceniaj samą rozmowę (dominacja, cisza, dygresje, jakość pytań facylitatora). Anty-wzorce zależne od kartek (brak hot spotów, mieszanie poziomów, przedwczesne kartki agregatów) nie dotyczą.
- **Nie sugeruj przejścia poziomu** mid-session chyba że facylitator wprost
  pyta.

## Wyjście

```
═════════════ Coach (<mid-session | debrief>) ═════════════

<feedback verbatim z subagent — po polsku>

═══════════════════════════════════════════════════════════
```

W trybie `debrief` dopisz pod ramką jedną linię ze ścieżką zapisanego raportu.

Bez dodatkowej meta-linii od Ciebie. Coach skończył — Ty milczysz.

## Idempotencja / zapobieganie spamowi

- Jeśli `/es-coach` był wywołany w tej samej rundzie wcześniej (sprawdź
  ostatnie sekcje transcript.md od bieżącej rundy), ostrzeż: *"Coach już
  wchodził w tej rundzie. Uruchomić jeszcze raz?"*. Domyślnie: nie powtarzaj
  bez potwierdzenia. `--full` po wcześniejszym `midsession` w tej samej
  rundzie jest w porządku — to inne wejście.

## Nie rób

- Nie pisz raportu do pliku bez `--full` — bez flagi coach wchodzi tylko inline.
- Nie zamykaj sesji — również przy `--full`.
- Nie syntezuj dodatkowych obserwacji od siebie po wypowiedzi coacha.
- Nie wzywaj coacha proaktywnie z `/es-round` — coach wchodzi tylko gdy
  facylitator wprost wywoła `/es-coach`.
