## ADDED Requirements

### Requirement: Embeddingi w Supabase
System SHALL liczyć embeddingi opisu cech narzędzi i referencji u wybranego dostawcy i zapisywać je
w Supabase z pgvector, bez ponownego liczenia dla niezmienionego tekstu.

#### Scenario: Niezmieniony opis
- **WHEN** `search_text` ma ten sam hash co zapisany embedding
- **THEN** system nie wywołuje API dostawcy

### Requirement: Wyszukiwanie hybrydowe top 5
System SHALL zwracać 5 najlepszych referencji, łącząc filtr typu narzędzia, podobieństwo tekstu znaków
i podobieństwo wektorowe, z wagami z konfiguracji.

#### Scenario: Zgodny znak
- **WHEN** moje narzędzie i referencja mają ten sam tekst znaku
- **THEN** referencja jest wysoko w wynikach, a uzasadnienie wskazuje zgodny znak
