---
description: Imiennie zaproś jedną konkretną personę do wypowiedzi. Dobre dla cichego uczestnika (kasia-junior-dev) lub gdy chcesz pociągnąć temat (np. piotr-mecenas o uprawnieniach).
argument-hint: "<persona-name> [pytanie/temat]"
---

# /es-invite

> **Zanim zaczniesz**: wczytaj kontrakt orkiestratora — `.claude/skills/event-storming-orchestration/SKILL.md` — i trzymaj się go (jesteś skrybą/orkiestratorem, nigdy nie wcielasz się w personę).

Wywołaj **jedną** personę imiennie.

## Wejście
- `$1` — nazwa persony (slug), jedna z:
  - `prezes-japinski`
  - `ewa-pm`
  - `marek-brygadzista`
  - `kasia-junior-dev`
  - `tomek-architekt`
  - `agnieszka-ksiegowa`
  - `piotr-mecenas`
  - `gerome-don-kichot`
- `$ARGUMENTS` bez `$1` — pytanie / temat o który facylitator pyta, np. "a co o uprawnieniach na wysokości?"

Jeśli `$1` nie pasuje do żadnej, zwróć listę dostępnych person i zakończ.

## Przebieg

1. **Załaduj stan** sesji (`./sessions/<latest>/state.json`); odczytaj `state.mode` (`light`/`full`, brak pola = `full`) oraz `state.today[<$1>]` — numer wariantu dnia tej persony (sesja bez tego pola: dolicz wzorem z `/es-start` i dopisz). Skróty jej wcześniejszych wypowiedzi weź z `state.memory[<$1>]` (pięć ostatnich), a echo sali — z wpisów **innych** person z ostatniej rundy (trzy, z imieniem mówiącego). Sesja bez pola `memory`: zbuduj je raz z `transcript.md` i dopisz do stanu.
2. **Inkrementuj round** (tak samo jak `/es-round` — invite to też runda)
3. **Wywołaj wybraną personę** jako subagent przez `Agent` (jeden agent, nie wielu)
   Prompt:
   ```
   Facylitator zwraca się bezpośrednio do Ciebie:
   "<$ARGUMENTS bez $1>"
   
   Aktualny stan: runda <n>, poziom <level>.
   (tryb full — eventy na ścianie, id → tekst: <ev_001 → …>)
   
   Twój dzień w tej sesji: wariant <state.today[persona]> z sekcji „Twój dzień"
   w Twoim pliku. Nie zapowiadaj go wprost — ma barwić to, co Cię dziś uwiera
   i jak mówisz.
   
   Twoje wcześniejsze wypowiedzi w tej sesji (nie powtarzaj ich ani anegdot,
   których już użyłeś): <skróty, do pięciu; puste jeśli jeszcze nie mówiłeś>

   Co padło na sali ostatnio — możesz się do tego odnieść: <Imię: jedno
   zdanie; do trzech; puste na początku sesji>
   
   Odpowiedz w roli. W trybie full: jeśli sytuacja rodzi kartkę (event/hot_spot),
   daj ją w YAML jak w swoim agent file. W trybie light: sam dialog, bez kartek.
   Jeśli to pytanie do którego masz odpowiedź — odpowiedz. Jeśli nie wiesz / nie
   Twoja rola — odeślij (w roli) do innej persony.
   ```

4. **Zbierz odpowiedź**. W trybie full dodaj stickies do stanu (jak w `/es-round`); w trybie light nie ma kartek — masz samą wypowiedź. W obu trybach dopisz wpis do `state.memory[<$1>]` — jedno zdanie własnymi słowami, z numerem rundy.
5. **Zaktualizuj transkrypt**:
   ```markdown
   ## Runda <n> — invite → <persona> — <timestamp>
   
   **Facylitator → <persona display name>**: <pytanie>
   
   **<persona display name>**:
   > <dialogue>
   > 
   > Stickies: ...
   ```
6. **Wyjście**: krótko, pokazujesz **pełne dialogue** tej persony (bo to skoncentrowana wymiana) + jeden-dwie sticky w jakie weszły.

## Reguły
- **Tylko jedna persona** — to różnica vs `/es-round`.
- **Jeśli persona odeśle gdzie indziej** ("to do Mecenasa"), to **nie wywołuj automatycznie kolejnego** — zostaw user'owi możliwość zadecydowania (kolejny `/es-invite`).
- **Jeśli persona nie pasuje do tematu** ([np. user zaprosił prezesa do dyskusji o JPA]), persona ma to powiedzieć w roli (*"Tomku, ja tego nie rozumiem, Ty się tym zajmij"*).

## Przykłady wywołania
- `/es-invite kasia-junior-dev` — bez tematu, "Kasia, a Ty co?"
- `/es-invite piotr-mecenas a co z uprawnieniami operatora koparki?`
- `/es-invite agnieszka-ksiegowa a jak liczy się ten cashflow przy przeniesieniu?`

## Nie rób
- Nie wzywaj > 1 persony.
- Nie zmyślaj odpowiedzi — wywołaj realny subagent.
