# Domain Drivers z AI

Repozytorium zawiera narzędzia prezentowane w edycji Domain Drivers + AI. Każdy numerowany folder
odpowiada jednemu odcinkowi. W subfolderze `.claude/` znajdziesz narzędzia Claude Code (komendy,
agenty i skille) wykorzystywane w danym odcinku. Subfolder `input/` zawiera artefakty wejściowe, a
 `output/` pliki wyjściowe wygenerowane przez AI. Wszelkie zmiany w kodzie są obecne w repozytorium [dd-java] na dedykowanych branchach. W celu użycia narzędzia skopiuj `.claude/` wybranego odcinka do katalogu głównego swojego
projektu i uruchom w nim Claude Code.

> Narzędzia w repozytorium obrazują zamysł, jak można wspomóc się AI w modelowaniu systemów.
> Mogą produkować niestabilne wyjście (wywołanie z takim samym promptem może skutkować
>  wynikiem różniącym się od prezentowanego w nagraniach). Narzędzia nie są przygotowane do wykorzystania w rozwiązaniach produkcyjnych.

## Zawartość folderów

| Odcinek | `input/` | `output/` |
|---|---|---|
| 002 | transkrypcja wywiadu z prezesem organizacji tworzącej system SmartSchedule, zdjęcia tablic Big Picture i Process Level | tekstowy opis event stormingu (`*.es.md`), bank dowodów `corpus.md`, raport sprzeczności, odpowiedzi użytkownika, obszary biznesowe `areas.md` i klastry poniżej progu dowodowego `potential-areas.md` |
| 003 | — | — |
| 004 | obszary `areas.md` i bank dowodów `corpus.md` z odcinka 002 | `algorithms/scan.md` — raport z dopasowania algorytmów |
| 005 | — (narzędzie czyta kod projektu [dd-java]) | — (raport w terminalu) |
| 006 | mapa kontekstów, metryki sukcesu, katalog modułów | brief biznesowy i raport z wygenerowanymi wymaganiami oraz klasyfikacją zmian  |
| 007 | obszary `areas.md` i bank dowodów `corpus.md` | — (raport w terminalu) |
| 008 | — (narzędzie pracuje na kodzie projektu [dd-java]) | — (zmiany w kodzie, widoczne w repo [dd-java]) |
| 009 | — (narzędzie pracuje na kodzie projektu [dd-java]) | — (zmiany w kodzie, widoczne w repo [dd-java]) |

[dd-java]: https://github.com/DomainDrivers/dd-java
