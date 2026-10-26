## ADDED Requirements

### Requirement: Twardy limit wydatków
System SHALL przed każdym płatnym zapytaniem porównać sumę wydatków i górny szacunek kosztu zapytania
z limitem na uruchomienie i limitem miesięcznym, i SHALL zatrzymać przetwarzanie, gdy limit zostałby przekroczony.

#### Scenario: Limit osiągnięty
- **WHEN** kolejne zapytanie przekroczyłoby limit
- **THEN** system nie wysyła zapytania i pokazuje komunikat z wydaną kwotą i limitem

### Requirement: Log zużycia
System SHALL zapisywać dla każdego zapytania model, tokeny wejściowe, zapis i odczyt cache, tokeny
wyjściowe i koszt, z sumami na narzędzie, uruchomienie i miesiąc.

#### Scenario: Podsumowanie uruchomienia
- **WHEN** przebieg analizy się kończy
- **THEN** UI i CLI pokazują koszt łączny i na każde narzędzie
