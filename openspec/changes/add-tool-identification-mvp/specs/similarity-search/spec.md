## ADDED Requirements

### Requirement: Lokalna baza wektorowa
System SHALL liczyć embeddingi opisu cech narzędzi i referencji lokalnym modelem i zapisywać je
w lokalnym ChromaDB (`data/chroma/`), bez połączenia z siecią i bez ponownego liczenia dla niezmienionego tekstu.

#### Scenario: Niezmieniony opis
- **WHEN** `search_text` ma ten sam hash co zapisany embedding
- **THEN** system nie liczy embeddingu ponownie

#### Scenario: Praca bez internetu
- **WHEN** komputer nie ma połączenia z siecią
- **THEN** liczenie embeddingów, wyszukiwanie i raporty działają normalnie

#### Scenario: Odbudowa indeksu
- **WHEN** użytkownik uruchamia `skrzynia embed --rebuild` albo zmienia model embeddingów
- **THEN** system odtwarza kolekcje ChromaDB w całości z danych w SQLite

### Requirement: Wyszukiwanie hybrydowe top 5
System SHALL zwracać 5 najlepszych referencji, łącząc filtr typu narzędzia, podobieństwo tekstu znaków
i podobieństwo wektorowe, z wagami z konfiguracji.

#### Scenario: Zgodny znak
- **WHEN** moje narzędzie i referencja mają ten sam tekst znaku
- **THEN** referencja jest wysoko w wynikach, a uzasadnienie wskazuje zgodny znak
