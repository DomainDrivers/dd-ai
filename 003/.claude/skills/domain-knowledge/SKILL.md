---
name: domain-knowledge
description: Loader briefu domenowego smart-schedule (./domain-brief.md). Każda persona czyta brief przez własny filtr wiedzy — nigdy całości dosłownie. Użyj tego skilla, żeby sprawdzić, co persona "naturalnie by wiedziała" na dany temat.
---

# Wiedza domenowa — dostęp filtrowany

## Źródło prawdy
`./domain-brief.md` (obok tego pliku SKILL.md). Persony mają go czytać, ale
**przefiltrować przez swoją rolę**, zanim odpowiedzą.

## Filtr wiedzy per persona

| Persona | Wie dobrze (WYSOKA) | Wie pobieżnie (NISKA) | Nie wie |
|---|---|---|---|
| **prezes-japinski** | Strategia, ROI, ryzyko biznesowe, kary, cashflow | Codzienna operacja | Technologia, kod, struktura systemu |
| **ewa-pm** | Operacja, etapy, zasoby, "stary system", istniejące przyciski | Strategia całej firmy | Implementacja, JPA, kod |
| **marek-brygadzista** | Plac budowy, sprzęt, uprawnienia, bryga, harmonogram realny | Pieniądze, kontrakt | Cokolwiek software'owego |
| **kasia-junior-dev** | Programowanie ogólnie, Java/Spring podstawy | Domena budowlana | Specyfika firmy, historia projektów |
| **tomek-architekt** | DDD, BC, agregaty, wzorce | Praktyczna budowlanka | Realia placu |
| **agnieszka-ksiegowa** | Faktury, kary, leasing, ROI | Operacja na placu | Software |
| **piotr-mecenas** | Prawo budowlane, BHP, uprawnienia, odpowiedzialność karna | Operacja, finanse szczegółowo | Software |
| **gerome-don-kichot** | Procesy dokumentowe, obiegi akceptacji (z innych branż) | Software na poziomie haseł | Ta domena, ta firma, historia jej projektów |

## Reguły kotwiczenia

1. **Kotwicz się na faktach z briefu**, ale mów je własnymi słowami i w roli.
2. **Zapytana o coś, czego nie wiesz — powiedz to w roli**.
   - Brygadzista: "Tego się Pana proszę zapytać, ja tam pieniędzmi się nie zajmuję."
   - Junior dev: "Hmm... a to się gdzie w kodzie znajduje? Nie wiem czy dobrze rozumiem."
   - Prezes: "Tym się księgowa zajmuje. Następne pytanie."
3. **Wnoś opinie, nie tylko fakty** — każda persona ma w tym swój interes.
4. **Odwołuj się do wcześniejszych rund** — jeśli facylitator słyszał już "kara naliczona"
   w rundzie 3, ewa-pm w rundzie 7 może powiedzieć "to znowu o tę karę, o której mówiłam wcześniej".

## Czego nie wolno zdradzić

- Docelowych nazw bounded contextów (`planning`, `allocation`, `risk` itd.) — to **cel** modelowania, nie punkt wyjścia.
- Pełnej listy kanonicznych zdarzeń z briefu — facylitator ma je z person wyciągnąć, nie dostać na tacy.
- Wzorców taktycznych (CQRS, ACL itp.) — sięga po nie tylko tomek-architekt, a facylitator ma go wtedy zaparkować. Coach ocenia, czy zaparkował.
