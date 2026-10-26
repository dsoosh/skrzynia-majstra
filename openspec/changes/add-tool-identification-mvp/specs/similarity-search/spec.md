## ADDED Requirements

### Requirement: Lokalna baza wektorowa
System SHALL liczyć embeddingi opisu cech narzędzi i referencji modelem OpenAI `text-embedding-3-small`
w chwili zapisu cech, zapisywać je w lokalnym ChromaDB (`data/chroma/`) i nie liczyć ich ponownie
dla niezmienionego tekstu. Koszt embeddingów SHALL być logowany i liczony do limitu wydatków.

#### Scenario: Niezmieniony opis
- **WHEN** `search_text` ma ten sam hash co zapisany embedding
- **THEN** system nie liczy embeddingu ponownie

#### Scenario: Praca bez internetu
- **WHEN** komputer nie ma połączenia z siecią, a wektory są już policzone
- **THEN** wyszukiwanie i raporty działają normalnie

#### Scenario: Odbudowa indeksu
- **WHEN** użytkownik uruchamia `skrzynia embed --rebuild` albo zmienia model embeddingów
- **THEN** system odtwarza kolekcje ChromaDB w całości z danych w SQLite

### Requirement: Wyszukiwanie hybrydowe top 5
System SHALL zwracać 5 najlepszych referencji, łącząc filtr typu narzędzia, podobieństwo tekstu znaków
i podobieństwo wektorowe, z wagami z konfiguracji.

#### Scenario: Zgodny znak
- **WHEN** moje narzędzie i referencja mają ten sam tekst znaku
- **THEN** referencja jest wysoko w wynikach, a uzasadnienie wskazuje zgodny znak
