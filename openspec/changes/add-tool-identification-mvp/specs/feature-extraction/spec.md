## ADDED Requirements

### Requirement: Jedno zapytanie na narzędzie
System SHALL wysyłać do `claude-haiku-5-5` jedno zapytanie ze wszystkimi zdjęciami grupy i wspólnymi,
cache'owanymi instrukcjami, i SHALL otrzymać JSON zgodny ze schematem rdzenia oraz bloku typu narzędzia.

#### Scenario: Analiza strugu
- **WHEN** użytkownik uruchamia analizę grupy ze zdjęciami strugu
- **THEN** system zapisuje JSON z typem `strug`, opisem i rodzajem ujęcia każdego zdjęcia, znakami i domysłami z uzasadnieniem

#### Scenario: Narzędzie innego typu
- **WHEN** grupa przedstawia dłuto
- **THEN** model ustawia `tool_type` na `dłuto` i wypełnia blok specyficzny dla dłuta

### Requirement: Opis zdjęć przez model
System SHALL ustalać rodzaj ujęcia i opis każdego zdjęcia na podstawie analizy modelu, nie nazwy pliku.

#### Scenario: Zdjęcie czoła ze stemplem
- **WHEN** w grupie jest zbliżenie czoła z wybitym znakiem
- **THEN** zdjęcie dostaje ujęcie `czoło` i opis wskazujący znak

### Requirement: Walidacja i ponowna próba
System SHALL walidować wynik względem JSON Schema; przy błędzie SHALL wykonać jedną ponowną próbę
z treścią błędu, a po drugim błędzie SHALL zapisać błąd przy narzędziu.

#### Scenario: Dwukrotnie niepoprawny wynik
- **WHEN** obie odpowiedzi nie przechodzą walidacji
- **THEN** narzędzie ma status błędu z zapisanym komunikatem i kosztem obu prób

### Requirement: Zasady ostrożności modelu
Instrukcje SHALL wymagać opisywania tylko tego, co widać, `null` zamiast zgadywania producenta,
uzasadnienia każdego szacunku okresu oraz wprost zgłaszanej niezgodności części.

#### Scenario: Nóż innego producenta niż korpus
- **WHEN** nóż ma cechę Goldenberga, a korpus znak innego producenta
- **THEN** `consistency.all_parts_consistent` ma wartość `false` z opisem niezgodności

### Requirement: Idempotencja ekstrakcji
System SHALL nie wysyłać zapytania, gdy istnieje wynik dla identycznego hasha wejścia
(model, wersja promptu, wersja schematu, zdjęcia, notatki).

#### Scenario: Ponowne uruchomienie
- **WHEN** użytkownik ponownie analizuje niezmienioną grupę
- **THEN** system pokazuje zapisany wynik i nie ponosi kosztu

### Requirement: Poprawki użytkownika
System SHALL przechowywać poprawki użytkownika osobno od wyniku modelu i używać wersji poprawionej
do wyszukiwania i raportu.

#### Scenario: Poprawka odczytu znaku
- **WHEN** użytkownik poprawia tekst znaku
- **THEN** wynik modelu pozostaje bez zmian, a wyszukiwanie używa poprawionego tekstu
