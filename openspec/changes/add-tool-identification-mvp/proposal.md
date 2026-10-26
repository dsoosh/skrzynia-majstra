# Change: Lokalna aplikacja do katalogowania i identyfikacji narzędzi (MVP, pilot: strugi)

## Why
Przy każdym skupionym narzędziu trzeba ustalić producenta, okres, typ i zgodność części. Dziś to
ręczne przeszukiwanie forów i katalogów. Aplikacja ma z serii zdjęć wyciągnąć cechy (Claude Haiku)
i podać najbliższe udokumentowane egzemplarze z prywatnej bazy referencyjnej, z linkami do źródeł.

## What Changes
- **photo-intake**: wgrywanie paczki zdjęć do lokalnej aplikacji webowej; oryginały na dysku w pełnej
  rozdzielczości; automatyczne grupowanie po czasie wykonania (EXIF); ręczne przenoszenie zdjęć
  między grupami, scalanie i dzielenie grup; automatyczne nadanie grupie kolejnego wolnego numeru `SM-NNN`.
- **feature-extraction**: jedno zapytanie do `claude-haiku-5-5` na narzędzie ze wszystkimi zdjęciami;
  model sam rozpoznaje typ narzędzia, opisuje każde zdjęcie (rodzaj ujęcia, co widać) i zwraca JSON
  zgodny ze schematem: wspólny rdzeń + blok specyficzny dla typu (strug, dłuto, piła, wiertarka, inny).
- **reference-collection**: pobieranie udokumentowanych egzemplarzy tylko z zaakceptowanych źródeł
  (`sources.md`), grzecznie (limit, User-Agent, cache), z metadanymi źródła i licencji przy każdym rekordzie.
- **similarity-search**: embeddingi cech (moich narzędzi i referencji) z OpenAI `text-embedding-3-small`,
  liczone raz przy zapisie i trzymane w lokalnym ChromaDB; wyszukiwanie hybrydowe offline: wektor +
  dopasowanie tekstu znaków + filtry (typ, kraj); top 5. Dane relacyjne w lokalnym SQLite.
- **cost-control**: cennik w konfiguracji, log tokenów i kosztu na każde wywołanie i narzędzie, twardy
  limit na uruchomienie i miesięczny; zatrzymanie przed przekroczeniem.
- **identification-report**: raport HTML (w aplikacji i jako plik) na narzędzie: cechy, top 5 z linkami
  i uzasadnieniem, brakujące ujęcia.

## Impact
- Nowe capabilities: `photo-intake`, `feature-extraction`, `reference-collection`,
  `similarity-search`, `cost-control`, `identification-report`.
- Nowy kod: całe repo (projekt startuje od zera).
- Zewnętrzne usługi: Claude API (analiza), OpenAI Embeddings (wektor przy zapisie cech)
  i zaakceptowane źródła (pobieranie referencji). Baza (SQLite + ChromaDB) i wyszukiwanie działają lokalnie.
- Poza zakresem: Google Sheets, szkice wiadomości do producentów, publikacja czegokolwiek.
