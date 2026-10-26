## ADDED Requirements

### Requirement: Tylko zaakceptowane źródła
System SHALL pobierać dane wyłącznie ze źródeł ze statusem `zaakceptowane` w `sources.md`
i SHALL nigdy nie pobierać z marketplace'ów.

#### Scenario: Źródło do weryfikacji
- **WHEN** użytkownik uruchamia pobieranie ze źródła o statusie `do weryfikacji`
- **THEN** fetcher odmawia i podaje powód

### Requirement: Grzeczne pobieranie
Fetcher SHALL przestrzegać `robots.txt`, wysyłać identyfikujący User-Agent, robić najwyżej jedno
zapytanie na kilka sekund do jednej domeny i korzystać z cache.

#### Scenario: Ścieżka zabroniona w robots.txt
- **WHEN** URL jest zabroniony przez `robots.txt`
- **THEN** fetcher go pomija i zapisuje to w logu

#### Scenario: Ponowne uruchomienie
- **WHEN** strona była już pobrana
- **THEN** fetcher używa cache i nie wysyła ponownego zapytania

### Requirement: Metadane rekordu
Każdy rekord referencyjny SHALL mieć URL źródła, datę pobrania, informację o licencji lub zasadach
użycia oraz autora, jeśli źródło go podaje.

#### Scenario: Zapis rekordu
- **WHEN** rekord zostaje pobrany
- **THEN** powstaje `meta.json` i wiersz w bazie z tymi polami

### Requirement: Ręczne referencje
System SHALL pozwalać ręcznie dodać referencję (URL, opis, własne notatki i zdjęcia) bez automatycznego pobierania.

#### Scenario: Wątek z forum
- **WHEN** użytkownik dodaje link do wątku z forum z notatką
- **THEN** referencja trafia do bazy i wyszukiwania bez pobierania strony
