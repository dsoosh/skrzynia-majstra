# Design: identyfikacja narzędzi, MVP

Ten dokument to plan z kroku 0 briefu (`docs/brief.md`): architektura, struktura repo, biblioteki,
embeddingi, koszty, źródła i otwarte pytania. Kod powstaje dopiero po akceptacji.

## 1. Architektura

```
 przeglądarka (localhost)
        │  HTMX
        ▼
 ┌──────────────── FastAPI (lokalnie, 127.0.0.1) ────────────────┐
 │ intake ──► grouping ──► extraction ──► embeddings ──► search ─► report
 │   │                        │  ▲            │             │
 │   ▼                        ▼  │            ▼             ▼
 │ data/originals/       cost guard       SQLite (data/skrzynia.db): zdjęcia, grupy, cechy, koszty
 │ data/derived/         (limit $)        ChromaDB (data/chroma/): wektory, lokalnie
 │                                        embeddingi: OpenAI (raz, przy zapisie cech)
 │                                             ▲
 │ references (CLI) ── fetcher (httpx, robots.txt, limit, cache) ── zaakceptowane źródła
 └───────────────────────────────────────────────────────────────┘
```

- **Jeden proces** FastAPI z zadaniami w tle (kolejka w SQLite, jeden worker w tym samym procesie).
  Bez Celery/Redis: przy jednym użytkowniku to zbędne.
- **Wszystkie dane lokalnie**: SQLite na dane relacyjne, ChromaDB (`PersistentClient`) na wektory.
  Sieci wymagają tylko świadome kroki: analiza zdjęć (Claude API + embedding OpenAI dla wyniku)
  i pobieranie referencji. Wyszukiwanie i raporty działają offline.
- **Pobieranie referencji** uruchamiane świadomie z CLI (`skrzynia refs fetch <źródło>`), nie z UI,
  żeby nic nie szło do sieci przez przypadek. W UI tylko podgląd i ręczne dodanie referencji.
- **UI po polsku**, kod po angielsku.

## 2. Przepływ pracy

1. **Wgranie paczki** (drag & drop wielu plików, także HEIC z telefonu). Każdy plik:
   SHA-256 → jeśli już jest, pomijamy (idempotencja) → oryginał zapisany bez zmian w
   `data/originals/<rrrr>/<mm>/<sha256>.<ext>` → kopia robocza `data/derived/<sha256>_1000.jpg`
   (dłuższy bok 1000 px, JPEG q85, z poprawioną orientacją EXIF).
2. **Grupowanie**: sortowanie po `DateTimeOriginal` z EXIF (brak EXIF → czas pliku, oznaczony w UI),
   nowa grupa, gdy przerwa między kolejnymi zdjęciami > progu (domyślnie 120 s; suwak w UI
   z podglądem przed zatwierdzeniem).
3. **Korekta grup w UI**: przeciąganie miniatur między grupami (SortableJS), scal / podziel / nowa grupa /
   odrzuć zdjęcie. Grupa dostaje automatycznie **kolejny wolny numer** `SM-NNN`.
   Jedna grupa = jedno narzędzie = jeden numer (bez podnumerów a/b/c); kilka narzędzi
   z jednej sesji zdjęć rozdzielasz na osobne grupy.
4. **Analiza** (przycisk na grupie albo „analizuj wszystkie gotowe”): jedno zapytanie do Haiku ze
   wszystkimi zdjęciami grupy. Model zwraca:
   - dla każdego zdjęcia: `view` (słownik ujęć dla danego typu), krótki opis, czytelność znaków,
     czy potrzebne zbliżenie w wyższej rozdzielczości;
   - cechy narzędzia wg schematu (rdzeń + blok typu).
   Jeśli model zgłosi nieczytelny znak na konkretnym zdjęciu, aplikacja wysyła **tylko to zdjęcie**
   w 1600 px w drugim, małym zapytaniu (odczyt znaku), i dopisuje wynik.
5. **Weryfikacja**: cechy widoczne i edytowalne w UI. Moje poprawki zapisane osobno
   (`user_overrides`), nie nadpisują wyniku modelu; do embeddingu idzie wersja po poprawkach.
6. **Wyszukiwanie** top 5 i **raport** (strona w aplikacji + eksport do pliku HTML/Markdown).

## 3. Schemat cech (uogólniony na dowolne narzędzie)

Wspólny rdzeń dla każdego narzędzia + blok `type_specific` zależny od `tool_type`.
Schematy w `schemas/` jako JSON Schema; dodanie nowego typu = nowy plik + słownik ujęć.

```jsonc
{
  "schema_version": "1.0",
  "tool_type": "strug | dłuto | piła | wiertarka | świder | ściskacz | miara | inny",
  "tool_type_other": null,
  "subtype": "np. gładzik",
  "is_part_only": false,               // np. SM-007: sam nóż bez korpusu
  "photos": [
    {"photo_id": "…", "view": "czoło", "description": "stempel na czole, …",
     "marks_visible": true, "needs_closeup": false}
  ],
  "materials": [{"part": "korpus", "material": "buk", "basis": "usłojenie, kolor"}],
  "dimensions": {"length_mm": null, "width_mm": null, "blade_width_mm": null,
                 "source": "z miarki na zdjęciu | z notatek | szacunek"},
  "marks": [{"location": "nóż", "text": "dokładnie to, co widać", "symbol": "oko z brwią",
             "technique": "bity | wypalony | naklejka | odlew | grawer",
             "legibility": "czytelny | częściowo | nieczytelny", "photo_id": "…"}],
  "components": [                      // części, które mogą mieć różne pochodzenie
    {"part": "nóż", "maker_guess": null, "basis": "", "era_guess": null}
  ],
  "consistency": {"all_parts_consistent": null, "notes": "nóż i korpus: ten sam okres/producent?"},
  "maker_guess": {"name": null, "basis": "na podstawie którego znaku/cechy"},
  "era_guess": {"from": null, "to": null, "basis": "uzasadnienie"},
  "country_guess": {"value": null, "basis": ""},
  "condition": ["pęknięcie od gardzieli wzdłuż spodu"],
  "distinctive_features": [],
  "missing_views": [{"view": "tył korpusu", "why": "możliwy drugi stempel"}],
  "confidence": "niska | średnia | wysoka",
  "type_specific": {
    // strug:
    "construction": "drewniany | metalowy | przejściowy",
    "has_horn": true, "closed_tote": false,
    "double_iron": null, "wedge": true, "sole_profile": "płaska | profil | …"
    // dłuto: "bevel": "…", "handle": "…", "socket_or_tang": "…"
    // piła: "teeth_per_inch": null, "handle": "…", "back": "…"
    // wiertarka: "drive": "korbowa | zębata", "chuck": "…"
  },
  "search_text": "zwięzły opis do wyszukiwania (generowany przez aplikację, nie przez model)"
}
```

Poprawki względem szkicu z briefu:
- `photos[]` — opis i rodzaj ujęcia każdego zdjęcia ustala model (zamiast nazw plików).
- `components[]` zamiast `blade`/`body` — działa dla każdego narzędzia (nóż/korpus, brzeszczot/rączka…).
- Każdy domysł (`maker_guess`, `era_guess`, `country_guess`) ma pole `basis`; `null` gdy brak podstaw.
- `marks[].photo_id` i `technique` — znak da się wskazać na zdjęciu i odróżnić znak producenta
  od wypalonego znaku właściciela.
- `missing_views` z uzasadnieniem; ujęcia zgłaszane wielokrotnie trafiają do słownika ujęć danego typu.
- `is_part_only` dla samych części (SM-007).

Słownik ujęć startowo dla strugu: bok z miarką, góra, stopa i szczelina, czoło ze stemplem, nóż A/B,
ostrze z bliska, wady, tył korpusu, klin, inne. Dla pozostałych typów podobne krótkie listy
(rozszerzane, gdy model zgłosi brak).

## 4. Wywołanie Claude (ekstrakcja)

- SDK `anthropic` (Python), model `claude-haiku-5-5`. Obsługuje obrazy (do 600 na zapytanie, JPEG/PNG/WebP/GIF).
- **Kolejność treści**: system prompt (instrukcje + schematy + słowniki ujęć, stały, z `cache_control`)
  → zdjęcia z etykietami `Zdjęcie <photo_id>:` → notatki z UI → polecenie.
  System prompt ma > 512 tokenów, więc łapie się na cache (min. dla Haiku 5.5: 512).
- **Format wyjścia**: structured outputs (`output_config.format` z JSON Schema), a niezależnie
  walidacja `jsonschema` po stronie aplikacji. Przy błędzie walidacji jedna ponowna próba z treścią
  błędu; potem zapis błędu w `extractions.error`. W kroku 1 sprawdzę na żywym API, czy Haiku 5.5
  przyjmuje nasz schemat w structured outputs; jeśli nie, fallback: narzędzie z `strict: true`.
- **Myślenie**: Haiku 5.5 ma adaptacyjne myślenie domyślnie, `effort` domyślnie `medium`.
  Zaczniemy od `low` i porównamy z `medium` na 2–3 strugach (myślenie to tokeny wyjściowe = koszt).
- **Brak `temperature`** (od API 2026-10-21 każdy parametr próbkowania daje 400).
- **Odmowa** (`stop_reason == "refusal"`) i `max_tokens` obsłużone jako błąd zapisany przy narzędziu.
- **Idempotencja**: `input_hash = sha256(model, wersja promptu, wersja schematu, effort, hashe zdjęć, notatki)`.
  Ten sam hash → wynik z bazy, bez zapytania.
- Batch API (−50%) niepotrzebne przy tej skali; można dodać później dla masowych przebiegów.

## 5. Baza referencyjna

- Rekord referencyjny = jeden udokumentowany egzemplarz (albo pozycja katalogu): zdjęcia + opis ze
  źródła + `source_url`, `author`, `fetched_at`, `license`/`terms_note`, `source_id`.
- Pliki: `data/reference/<źródło>/<id>/images/…` + `meta.json` (jak w briefie) i to samo w bazie.
- Cechy referencji wyciąga ten sam ekstraktor (ten sam schemat), z opisem ze źródła jako dodatkowym
  tekstem; pola z opisu (producent, okres) mają `basis: "opis źródła"`.
- Fetcher: `httpx`, User-Agent `SkrzyniaMajstraBot/0.1` (kontakt opcjonalny w `config.toml`, domyślnie pusty), sprawdzenie `robots.txt`
  (`urllib.robotparser`) przy każdym URL, min. 5 s między zapytaniami do jednej domeny, cache odpowiedzi
  na dysku (`data/cache/http/`, z ETag/Last-Modified). Źródło bez statusu `zaakceptowane` w `sources.md`
  (odczytywanym przez program z tabeli) → fetcher odmawia.
- Osobny adapter na źródło (`references/sources/<nazwa>.py`): lista egzemplarzy + parser strony.
- **Ręczne dodanie referencji** w UI: wklejam URL i opis, dodaję własne zdjęcia/notatki. Działa też dla
  źródeł, których nie wolno pobierać automatycznie (czytam sam, zapisuję tylko link i swoje notatki).
- **Propozycja (później)**: tabela znaków towarowych z okresami używania (np. znaki Ulmii z datami
  z holzwerken.de, znaki jakości Goldenberga). Dopasowanie po tekście znaku daje datowanie wprost.

## 6. Embeddingi i wyszukiwanie

Wyszukiwanie hybrydowe, bo przy starych narzędziach decyduje tekst znaku, a nie ogólne podobieństwo:

1. **Filtr**: `tool_type` (+ opcjonalnie kraj) jako filtr `where` po metadanych w ChromaDB.
2. **Wektor**: ChromaDB zwraca ~20 kandydatów po embeddingu `search_text` (deterministycznie złożony
   z cech: typ, podtyp, materiały, znaki, symbole, cechy wyróżniające, wymiary w przedziałach).
3. **Znaki**: dla kandydatów oraz dla wszystkich referencji z podobnym znakiem liczone jest podobieństwo
   tekstu znaków (`rapidfuzz`, po normalizacji: wielkie litery, bez interpunkcji). Przy kilku tysiącach
   rekordów to milisekundy w Pythonie, bez osobnego indeksu.
4. **Wynik** = ważona suma (wagi w `config.toml`), top 5. Uzasadnienie w raporcie generowane
   z danych (które znaki pasują, które cechy się zgadzają/różnią) — bez dodatkowego zapytania do modelu.
   Opcjonalnie później: krótkie uzasadnienie od Haiku na podstawie obu JSON-ów (grosze).

Embedding obrazów (podobieństwo wizualne stempli) — opcja na później, jeśli tekst nie wystarczy.

### Przechowywanie

- **ChromaDB** w trybie lokalnym (`chromadb.PersistentClient(path="data/chroma")`), bez serwera.
  Kolekcje: `my_tools` i `references`, rozdzielone per model embeddingów (nazwa kolekcji zawiera
  nazwę modelu, więc zmiana modelu = nowa kolekcja i przeliczenie, stare dane nienaruszone).
  Id dokumentu = id narzędzia/referencji; metadane: `tool_type`, `country`, `search_text_hash`.
- **SQLite** (`data/skrzynia.db`) na resztę: zdjęcia, grupy, cechy, poprawki, referencje, koszty, kolejka.
  ChromaDB nie jest bazą relacyjną, więc źródłem prawdy jest SQLite; ChromaDB da się w całości
  odbudować z SQLite komendą `skrzynia embed --rebuild`.

### Model embeddingów: OpenAI `text-embedding-3-small` (decyzja)

Anthropic nie udostępnia własnego modelu embeddingów (w dokumentacji odsyła do Voyage AI).
Wybór: **OpenAI `text-embedding-3-small`** (1536 wymiarów, $0.02 / 1M tokenów, wielojęzyczny).

- Embedding liczony jest **raz**, w chwili zapisu cech (analiza narzędzia albo import referencji),
  czyli w krokach, które i tak łączą się z siecią. Wektor trafia do lokalnego ChromaDB.
- Wyszukiwanie porównuje już zapisane wektory, więc **samo wyszukiwanie i raporty działają offline**.
  Sieć potrzebna jest tylko do policzenia wektora dla nowego lub zmienionego opisu (np. po Twojej poprawce).
- Koszt logowany w `api_usage` tak samo jak wywołania Claude i liczony do limitu wydatków.
- Embeddingi są za adapterem (`search/embeddings.py`); model to jedna linia w `config.toml`.
  Nazwa kolekcji ChromaDB zawiera nazwę modelu, więc zmiana modelu = nowa kolekcja + `--rebuild`.

## 7. Koszty (cennik Anthropic sprawdzony 2026-10-26)

`claude-haiku-5-5` (zapytania ≤ 100 tys. tokenów): wejście $0.10 / MTok, wyjście $0.50 / MTok,
zapis do cache (5 min) $0.125 / MTok, odczyt z cache $0.01 / MTok. Batch: połowa ceny.
Obraz kosztuje `⌈szer/28⌉ × ⌈wys/28⌉` tokenów: 1000×750 px → 972 tokeny; 1600×1200 → 2494.

Szacunek na jedno narzędzie (8–10 zdjęć):

| Pozycja | Tokeny | Koszt |
|---|---|---|
| Zdjęcia 9 × ~1000 tok. | ~9 000 wej. | $0.0009 |
| Instrukcje + schematy (cache) | ~4 000 | $0.0005 przy zapisie, potem $0.00004 |
| Notatki + polecenie | ~300 | ~$0 |
| Wyjście: JSON + myślenie | ~3 000–5 000 | $0.0015–0.0025 |
| Zbliżenie znaku (gdy potrzebne) | ~3 000 wej. + 500 wyj. | $0.0006 |
| **Razem** | | **ok. $0.003–0.005** (z ponowną próbą maks. ~$0.01) |

| Etap | Liczba | Szacunek |
|---|---|---|
| Próba ekstrakcji | 2–3 egzemplarze referencyjne | < $0.02 |
| Pilot referencji | ~20 | ~$0.05 |
| Embeddingi pilota (OpenAI) | ~20 rekordów × ~300 tok. | < $0.001 |
| Docelowo | 1000 narzędzi | ~$3–5 |

Limity domyślne w `config.toml` (do zmiany): **$0.50 na uruchomienie, $5 na miesiąc**. Przed każdym
zapytaniem bramka liczy górny szacunek (tokeny obrazów z wymiarów + tekst + `max_tokens` × cena
wyjścia); jeśli `wydane + szacunek > limit` → stop i komunikat w UI. Po zapytaniu zapis faktycznego
`usage` (wejście, cache zapis/odczyt, wyjście) i kosztu w tabeli `api_usage`, z sumami na narzędzie,
uruchomienie i miesiąc. Ceny w `config.toml`, bo się zmieniają.

## 8. Struktura repo

```
skrzynia-majstra/
  README.md                 jak uruchomić każdy krok, jak dodać narzędzie / typ / źródło
  sources.md                status źródeł (czytany też przez fetcher)
  config.toml               progi, limity kosztów, ceny, wagi wyszukiwania
  .env.example              ANTHROPIC_API_KEY, OPENAI_API_KEY
  pyproject.toml            uv; skrypt `skrzynia`
  openspec/                 project.md, changes/, specs/
  docs/brief.md
  schemas/
    tool_core.schema.json
    types/strug.schema.json, dluto.schema.json, pila.schema.json, wiertarka.schema.json, inny.schema.json
    views.toml              słowniki ujęć per typ
  prompts/extraction_v1.md  instrukcje dla modelu (wersjonowane)
  migrations/               SQL dla SQLite (tabele), numerowane
  src/skrzynia/
    config.py  db.py  costs.py
    intake/       upload, EXIF, skalowanie, grupowanie
    extraction/   budowa zapytania, wywołanie, walidacja, overrides
    references/   fetcher.py (robots, limit, cache), sources/<źródło>.py
    search/       embeddings.py (adapter: OpenAI), store.py (ChromaDB), hybrid.py
    reports/      render HTML/Markdown
    web/          app.py, routes/, templates/, static/
    cli.py        skrzynia serve | refs fetch | extract | embed | report | costs
  tests/
  data/                     (w .gitignore) originals/, derived/, reference/, cache/, reports/,
                            skrzynia.db (SQLite), chroma/ (ChromaDB)
```

Model danych (SQLite): `photos`, `photo_groups`, `tools`, `extractions`, `user_overrides`,
`reference_sources`, `reference_items`, `reference_images`, `api_usage`, `runs`, `jobs`.
Wektory w ChromaDB (kolekcje `my_tools__<model>`, `references__<model>`).

Kopia zapasowa = skopiowanie katalogu `data/` (wszystko jest w plikach).

## 9. Źródła — stan weryfikacji

**Ważne:** polityka sieci tego środowiska chmurowego blokuje wszystkie te domeny (połączenie odrzucane
przez proxy), więc nie mogłem jeszcze przeczytać ich regulaminów ani `robots.txt`. Poniższa lista
opiera się na wynikach wyszukiwarki. Każde źródło ma status **do weryfikacji** i fetcher go nie
ruszy, dopóki nie zmienię statusu po sprawdzeniu. Szczegóły w `sources.md`.

Priorytet (pod narzędzia niemieckie i francuskie):

| Źródło | Co zawiera | Wstępna ocena |
|---|---|---|
| holzwerken.de — „Kleines Werkzeugmuseum” | strony producentów (Ulmia: znaki z datami używania), datowanie, katalogi, egzemplarze strugów | najcenniejsze dla DE; prywatna strona, zdjęcia chronione — możliwe, że trzeba zapytać autora |
| lairdubois.fr | „pas à pas”: znaki jakości Goldenberga, katalogi Goldenberga z lat 70. | najcenniejsze dla FR; treści użytkowników, sprawdzić CGU |
| forum-outils-anciens.com | forum FR: wątki o Goldenbergu, identyfikacja | forum: raczej ręczne referencje (link + notatka) |
| woodworking.de (forum) | historia Ulmii, wątki identyfikacyjne | jw. |
| museum-digital.de | kolekcje muzeów niemieckich, strugi z opisem i wymiarami; licencja przy obiekcie, jest API | dobre do automatu, jeśli licencje otwarte |
| Wikimedia Commons | zdjęcia na wolnych licencjach, API | pewne prawnie, mało egzemplarzy z opisem producenta |
| Europeana | agregator kolekcji europejskich, API, prawa per rekord | do sprawdzenia, ile jest strugów |
| hyperkitten.com (Patrick Leach) | Stanley, głównie tekst | mało przydatne dla DE/FR |
| oldwoodworkingtools.com | nieznane (nie udało się otworzyć) | do sprawdzenia, czy to nie sklep |
| UKWorkshop, FineWoodworking (fora) | wątki identyfikacyjne, głównie UK/US | niski priorytet |
| disstonianinstitute.com | piły Disston | później (piły) |

Wykluczone z zasady: eBay, Allegro, OLX, Kleinanzeigen, Etsy, PicClick i inne marketplace'y/agregatory ofert.

Propozycja pilota (20 egzemplarzy): **museum-digital.de** (automat, jeśli licencje pozwolą) +
**holzwerken.de** (jeśli regulamin/autor pozwoli; inaczej ręczne referencje z linkiem).

## 10. Ryzyka

- Odczyt zatartych stempli w 1000 px — zbliżenia w 1600 px (cel: koszt nadal < $0.01/narzędzie).
- Mała baza referencyjna na starcie → dopasowania słabe; dlatego ręczne referencje i tabela znaków.
- Pojedynczy użytkownik, lokalnie: brak logowania w aplikacji; nasłuch tylko na `127.0.0.1`.
