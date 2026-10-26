# Brief: identyfikacja starych narzędzi (Skrzynia Majstra)

Oryginalny brief etapu 1 (strugi) plus zmiany zakresu wprowadzone przy starcie projektu.

## Zmiany względem oryginalnego briefu (2026-10-26)

- Całość opiera się o Claude API; zmiany śledzone w **OpenSpec** (`openspec/`).
- Zamiast samego pipeline'u CLI: **aplikacja webowa serwowana lokalnie**.
- Oryginały zdjęć zapisywane na dysku w pełnej rozdzielczości; do Claude idą kopie przeskalowane w dół.
- Aplikacja **grupuje zdjęcia po znaczniku czasu** i pozwala **przenosić zdjęcia między grupami** (wrzucenie dużej paczki naraz).
- Opis każdego zdjęcia (co przedstawia, jakie to ujęcie) ustala Haiku podczas analizy, nie nazwa pliku.
- Zakres od razu rozszerzony: **katalogowanie i wyszukiwanie dowolnego narzędzia**, nie tylko strugów. Strugi pozostają pilotem.
- Aplikacja webowa do katalogowania, wcześniej „poza zakresem”, wchodzi do zakresu.
- Zamiast Supabase z pgvector: **lokalny ChromaDB**. Dane relacyjne w lokalnym SQLite.
- Embeddingi: **OpenAI `text-embedding-3-small`** (Anthropic nie ma własnego modelu embeddingów).
- Weryfikacja źródeł i dalsza praca **lokalnie u właściciela** (środowisko chmurowe blokuje domeny źródeł).
- Pilot referencji: **museum-digital.de + holzwerken.de** (po sprawdzeniu regulaminów i robots.txt).
- Lista „Moje strugi do pilota” **na razie odłożona**. Każde narzędzie ma własny numer, bez podnumerów a/b/c;
  aplikacja nadaje **kolejny wolny numer**.
- Kontakt w User-Agent nieobowiązkowy (domyślnie bez e-maila).

---

## Oryginalny brief

### Kontekst

Prowadzę Skrzynię Majstra: skupuję stare narzędzia ręczne (głównie niemieckie i francuskie strugi drewniane, dłuta, piły, wiertarki ręczne), odnawiam je i sprzedaję, głównie na eBay.de i Etsy. Przy każdym narzędziu muszę ustalić producenta, okres i typ, a także czy części pasują do siebie (np. nóż Goldenberga w korpusie innego producenta).

Cel pipeline'u: robię zdjęcia narzędzia według stałego zestawu ujęć, model wyciąga z nich ustrukturyzowane cechy, a wyszukiwanie po podobieństwie zwraca najbliższe udokumentowane egzemplarze z bazy referencyjnej, z linkami do źródeł. Zaczynamy od strugów, potem dłuta i reszta.

### Zakres etapu 1

1. **Ekstrakcja cech z moich zdjęć.** Moje strugi, sfotografowane według zestawu ujęć, idą do modelu, który zwraca JSON zgodny ze schematem.
2. **Baza referencyjna.** Zebranie udokumentowanych strugów (zdjęcia + opis: producent, okres, typ) z dozwolonych źródeł. Pilot: ok. 20 egzemplarzy z 1–2 źródeł.
3. **Embeddingi i wyszukiwanie.** Cechy referencji i moich narzędzi zamienione na embeddingi, zapisane w Supabase z pgvector (zmienione: lokalny ChromaDB). Zapytanie zwraca top 5 dopasowań.
4. **Raport.** Dla każdego mojego strugu: wyciągnięte cechy, top 5 dopasowań ze źródłami i krótkie uzasadnienie, plus lista dodatkowych zdjęć, które pomogłyby w identyfikacji.

Poza zakresem (później): inne typy narzędzi*, automatyczne uzupełnianie arkusza inwentarza w Google Sheets, szkice wiadomości do producentów, aplikacja webowa do katalogowania*.

\* przeniesione do zakresu, patrz „Zmiany” wyżej.

### Zasady, których nie łamiemy

- **Nie scrapujemy eBay, Allegro, OLX, Kleinanzeigen ani innych marketplace'ów.** Ich regulaminy tego zabraniają.
- **Każde źródło sprawdzamy przed pobieraniem:** regulamin (Terms of Use) i robots.txt. Jeśli zasady są niejasne albo zabraniają automatycznego dostępu, nie pobieramy i pytamy. Status każdego źródła w `sources.md`.
- **Grzecznie:** limit zapytań (jedno na kilka sekund), identyfikujący się User-Agent, cache.
- **Przy każdym rekordzie:** URL źródła, autor (jeśli podany), data pobrania, licencja lub zasady użycia.
- **Baza jest prywatna,** do wewnętrznej identyfikacji. Nie publikujemy cudzych zdjęć i nie używamy ich w ofertach.
- **Sekrety** tylko w `.env`, który trafia do `.gitignore`.

### Koszty

- Przed uruchomieniem na większej liczbie zdjęć: aktualny cennik i szacunek kosztu.
- Najpierw kilka zdjęć, potem pilot (~20 egzemplarzy), dopiero potem więcej.
- Twardy limit wydatków w konfiguracji; pipeline zatrzymuje się po przekroczeniu.
- Log zużycia tokenów i kosztu na każde narzędzie.

### Technikalia (z briefu)

- Python. Pobieranie stron: zwykły HTTP + parsowanie HTML; Playwright tylko gdy konieczny.
- Ekstrakcja: Claude Haiku (`claude-haiku-5-5`).
- Zdjęcia skalowane do ok. 1000 px na dłuższym boku, JPEG; większa rozdzielczość tylko dla zbliżeń, jeśli znaki są nieczytelne.
- Jedno narzędzie = jedno zapytanie ze wszystkimi zdjęciami. Wspólne instrukcje (prompt caching).
- JSON walidowany schematem; jedna ponowna próba, potem zapis błędu.
- Embeddingi: zewnętrzny dostawca (wybór należy do właściciela).
- Supabase z pgvector (zmienione: lokalny ChromaDB + SQLite).
- Idempotencja: ponowne uruchomienie nie pobiera i nie analizuje drugi raz.

### Zestaw ujęć dla strugu

Przed renowacją, jasny blat lub szara płyta, dzienne światło z boku, bez flesza.

| Nr | Ujęcie |
|---|---|
| 01 | Całość z boku, z miarką |
| 02 | Całość z góry |
| 03 | Stopa i szczelina |
| 04 | Czoło ze stemplem (z bliska) |
| 05 | Nóż z obu stron |
| 06 | Ostrze z bliska |
| 07 | Wady: pęknięcia, ubytki |
| 08+ | Dodatkowe (tył korpusu, klin…) |

Brakujące ujęcia model wpisuje w `missing_views`; trafiają na stałe do zestawu.

### Strugi do pilota

- SM-019 strug długi drewniany (spust, ok. 60 cm), bez widocznej cechy
- SM-020 gładzik z rogiem, naklejka „RICHA Qualität”
- SM-021 gładzik bez rogu, nóż z cechą Goldenberg (Francja)
- SM-022 wręgownik 30 mm, sygnatura „30 m/m” na czole
- SM-027 wręgownik drewniany, do opisania
- SM-032 trzy wręgowniki drewniane z naklejkami
- SM-033 mały strug z rogiem, naklejka do odczytania
- SM-036 spust z zamkniętą rączką, nóż Goldenberg (oko z brwią nad napisem)
- SM-007 sam nóż strugu „Original Steiner 1859” (bez korpusu)

### Zasady dla modelu przy ekstrakcji

Opisuje tylko to, co widać; nie zgaduje producenta bez podstaw; każdy szacunek okresu ma uzasadnienie; niezgodność noża i korpusu zgłasza wprost.

### Kryteria odbioru

- Każdy rekord referencyjny ma URL źródła, datę pobrania i informację o zasadach użycia.
- Każdy wynik ekstrakcji przechodzi walidację schematu albo ma zapisany błąd.
- Raport pokazuje dla każdego narzędzia top 5 dopasowań z linkami i uzasadnieniem oraz listę brakujących ujęć.
- Koszt każdego uruchomienia jest zalogowany i mieści się w limicie.
- `README.md` opisuje, jak uruchomić każdy krok i jak dodać nowe narzędzie.
