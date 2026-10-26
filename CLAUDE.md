# CLAUDE.md — Skrzynia Majstra

Lokalna aplikacja webowa do katalogowania i identyfikacji starych narzędzi ręcznych.
Właściciel pisze po polsku; odpowiadaj po polsku. Kod, nazwy i commity techniczne po angielsku
albo po polsku (bez mieszania w jednym pliku); UI, raporty i prompty dla modelu po polsku.

## Najpierw przeczytaj
- `openspec/project.md` — stos i konwencje
- `openspec/changes/add-tool-identification-mvp/` — `proposal.md`, `design.md` (plan), `tasks.md`, `specs/`
- `sources.md` — status źródeł; `docs/brief.md` — brief i decyzje właściciela

## Praca z OpenSpec
- Plan i wymagania są w propozycji `add-tool-identification-mvp`. Realizuj zadania z `tasks.md` po kolei
  i odhaczaj je (`- [x]`) w tym samym commicie, w którym powstaje kod.
- Zmiana zakresu lub decyzji = najpierw aktualizacja `design.md`/specyfikacji, potem kod.
- Nie archiwizuj propozycji bez zgody właściciela.

## Zasady, których nie łamiemy
- **Nie pobieramy niczego z marketplace'ów** (eBay, Allegro, OLX, Kleinanzeigen, Etsy, PicClick…).
- Fetcher pobiera tylko ze źródeł o statusie `zaakceptowane` w `sources.md`; przestrzega `robots.txt`,
  min. 5 s między zapytaniami do domeny, User-Agent `SkrzyniaMajstraBot/0.1`, cache na dysku.
  Regulamin niejasny albo zabrania automatów → nie pobierasz, pytasz właściciela.
- Każdy rekord referencyjny: URL źródła, autor (jeśli jest), data pobrania, licencja / zasady użycia.
- Baza referencyjna jest prywatna; cudzych zdjęć nie publikujemy.
- Sekrety tylko w `.env` (jest w `.gitignore`). Nigdy nie commituj kluczy ani katalogu `data/`.

## Koszty
- Każde płatne wywołanie (Claude, OpenAI embeddings) idzie przez bramkę kosztów i jest logowane
  w `api_usage`. Twarde limity w `config.toml` (domyślnie $0.50 / uruchomienie, $5 / miesiąc).
- Zanim uruchomisz cokolwiek płatnego na więcej niż kilku elementach: podaj szacunek i zapytaj.
- Testy nie wołają płatnych API (nagrane odpowiedzi w `tests/fixtures/`).

## Stałe decyzje techniczne
- Python 3.12+, `uv`, FastAPI + Jinja2 + HTMX, nasłuch tylko na `127.0.0.1`.
- Ekstrakcja: `claude-haiku-5-5` przez SDK `anthropic`; jedno zapytanie na narzędzie ze wszystkimi
  zdjęciami; wspólny system prompt z `cache_control`; structured outputs + walidacja `jsonschema`,
  jedna ponowna próba. Bez `temperature`/`top_p`/`top_k`. Obsłuż `stop_reason` `refusal` i `max_tokens`.
- Zdjęcia: oryginał bez zmian (adresowany SHA-256), do modelu kopia 1000 px (JPEG); zbliżenia 1600 px tylko na żądanie.
- Dane: SQLite (`data/skrzynia.db`) jako źródło prawdy, ChromaDB (`data/chroma/`) na wektory, odtwarzalne z SQLite.
- Embeddingi: OpenAI `text-embedding-3-small`, liczone raz przy zapisie cech; wyszukiwanie działa offline.
- Numer narzędzia: kolejny wolny `SM-NNN`, jeden na narzędzie, bez podnumerów.
- Wszystko idempotentne (hash treści / hash wejścia).

## Zanim zrobisz commit
`uv run ruff format && uv run ruff check && uv run pytest` — wszystko zielone.

## Na koniec każdego kroku
Krótko: co powstało, ile kosztowało (z `api_usage`), co proponujesz dalej. Czekaj na OK przed kolejnym krokiem.
