# Project Context

## Purpose
Skrzynia Majstra: lokalna aplikacja webowa do katalogowania i identyfikacji starych narzędzi ręcznych
(strugi, dłuta, piły, wiertarki ręczne…). Zdjęcia narzędzia → ustrukturyzowane cechy (Claude) →
wyszukiwanie podobnych, udokumentowanych egzemplarzy w prywatnej bazie referencyjnej, z linkami do źródeł.

## Tech Stack
- Python 3.12+, zarządzanie zależnościami: `uv`
- Web: FastAPI + Jinja2 + HTMX (+ SortableJS do przeciągania zdjęć), serwowane lokalnie (`127.0.0.1`)
- Model: Claude API, oficjalny SDK `anthropic` (ekstrakcja: `claude-haiku-5-5`)
- Dane lokalnie: SQLite (`sqlite3`, bez ORM-a) + ChromaDB (`PersistentClient`) na wektory
- Embeddingi: OpenAI `text-embedding-3-small` (SDK `openai`), wektory w lokalnym ChromaDB; dopasowanie tekstu znaków: `rapidfuzz`
- Obrazy: Pillow (+ `pillow-heif` dla zdjęć z telefonu), EXIF do grupowania
- Pobieranie źródeł: `httpx` + `selectolax`; Playwright tylko gdy konieczny
- Walidacja: JSON Schema (`jsonschema`) + modele Pydantic generowane ze schematów
- Testy: `pytest`; lint/format: `ruff`

## Project Conventions

### Code Style
- `ruff format` + `ruff check`; typowanie wszędzie, gdzie to proste.
- Nazwy w kodzie po angielsku; UI, raporty i treść promptów dla modelu po polsku.

### Architecture Patterns
- Moduły domenowe w `src/skrzynia/` (intake, extraction, references, search, costs, reports, web).
- Każdy krok idempotentny: klucz = hash treści (zdjęcia) albo hash wejścia (ekstrakcja, embedding).
- Sekrety tylko w `.env` (w `.gitignore`); konfiguracja w `config.toml`.

### Testing Strategy
- Testy jednostkowe bez sieci i bez API (nagrane odpowiedzi w `tests/fixtures/`).
- Wywołania płatnego API tylko w komendach uruchamianych świadomie, zawsze za bramką kosztów.

### Git Workflow
- Każda zmiana zakresu = propozycja w `openspec/changes/<id>/`, archiwizowana po wdrożeniu.

## Domain Context
- Narzędzia głównie niemieckie i francuskie (ECE, Ulmia, Steiner, Goldenberg, RICHA…).
- Części bywają niezgodne (nóż innego producenta niż korpus) — trzeba to zgłaszać wprost.
- Numer inwentarzowy w formacie `SM-NNN`.

## Important Constraints
- Zakaz scrapowania marketplace'ów (eBay, Allegro, OLX, Kleinanzeigen, Etsy…).
- Każde źródło sprawdzone (regulamin + robots.txt) przed pobieraniem; status w `sources.md`.
- Baza referencyjna prywatna; cudzych zdjęć nie publikujemy.
- Twardy limit wydatków; każdy koszt logowany.

## External Dependencies
- Anthropic Claude API
- OpenAI Embeddings API (tylko przy zapisie nowych/zmienionych cech)
