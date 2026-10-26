# Tasks

## 0. Plan
- [x] 0.1 Architektura, struktura repo, biblioteki (`design.md`)
- [x] 0.2 Szacunek kosztów wg aktualnego cennika Anthropic
- [x] 0.3 Embeddingi: OpenAI `text-embedding-3-small`, wektory w lokalnym ChromaDB
- [x] 0.4 Lista kandydatów na źródła (`sources.md`), status: do weryfikacji
- [x] 0.5 Akceptacja planu i odpowiedzi na pytania otwarte

## 1. Szkielet + przyjmowanie zdjęć (bez sieci)
- [ ] 1.1 `pyproject.toml` (uv), `config.toml`, `.env.example`, `.gitignore`, ruff, pytest
- [ ] 1.2 Migracje SQLite (tabele) i inicjalizacja ChromaDB w `data/chroma/`
- [ ] 1.3 Upload paczki, SHA-256, zapis oryginałów, kopie 1000 px, HEIC
- [ ] 1.4 Grupowanie po EXIF z progiem; UI: przenoszenie, scalanie, dzielenie, numer SM
- [ ] 1.5 Testy grupowania i idempotencji

## 2. Ekstrakcja cech
- [ ] 2.1 JSON Schema: rdzeń + strug (+ szkielety dłuto/piła/wiertarka/inny), słownik ujęć
- [ ] 2.2 Prompt `extraction_v1` z cache; structured outputs + walidacja + 1 ponowna próba
- [ ] 2.3 Bramka kosztów i `api_usage`; limity z `config.toml`
- [ ] 2.4 Drugie zapytanie o zbliżenie znaku (1600 px) gdy model zgłosi potrzebę
- [ ] 2.5 UI: wynik, opisy zdjęć, poprawki użytkownika
- [ ] 2.6 Próba na 2–3 egzemplarzach referencyjnych (effort low vs medium), wyniki i koszt do akceptacji

## 3. Referencje
- [ ] 3.1 Weryfikacja (lokalnie) regulaminów i robots.txt: museum-digital, holzwerken.de; aktualizacja `sources.md`
- [ ] 3.2 Fetcher: robots.txt, limit, User-Agent, cache, odmowa dla niezaakceptowanych źródeł
- [ ] 3.3 Adaptery 1–2 źródeł; pilot ~20 egzemplarzy z `meta.json`
- [ ] 3.4 Ręczne dodawanie referencji w UI
- [ ] 3.5 Ekstrakcja cech referencji

## 4. Embeddingi i wyszukiwanie
- [ ] 4.1 Embeddingi OpenAI przy zapisie cech, zapis w ChromaDB, idempotencja po hashu `search_text`, `--rebuild`, koszt w `api_usage`
- [ ] 4.2 Wyszukiwanie hybrydowe (filtr + podobieństwo znaków `rapidfuzz` + wektor), top 5

## 5. Raport i dokumentacja
- [ ] 5.1 Raport HTML w aplikacji + eksport pliku
- [ ] 5.2 `README.md`: uruchomienie każdego kroku, dodawanie narzędzia / typu / źródła
- [ ] 5.3 Pilot na referencjach, podsumowanie kosztów (moje narzędzia: później)
