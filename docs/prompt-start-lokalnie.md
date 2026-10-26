# Prompt startowy dla Claude Code (lokalnie)

Przed wklejeniem:
1. `git clone https://github.com/dsoosh/skrzynia-majstra.git && cd skrzynia-majstra`
2. `git checkout claude/bold-mccarthy-1xpzpw`
3. Utwórz `.env` z `ANTHROPIC_API_KEY=...` i `OPENAI_API_KEY=...` (plik jest w `.gitignore`).
4. Uruchom `claude` w katalogu repo i wklej poniższy tekst.

---

```
Kontynuujemy projekt Skrzynia Majstra. Plan jest już zaakceptowany — nie projektuj go od nowa.

Przeczytaj najpierw w całości: CLAUDE.md, openspec/project.md, wszystkie pliki w
openspec/changes/add-tool-identification-mvp/ (proposal, design, tasks, specs), sources.md
i docs/brief.md. Jeśli coś w nich jest sprzeczne albo niejasne, zapytaj, zanim napiszesz kod.

Pracujesz na gałęzi claude/bold-mccarthy-1xpzpw. Commituj małymi krokami, odhaczaj zadania
w tasks.md w tym samym commicie co kod. Nie pushuj bez mojej zgody.

Zakres tej sesji — trzy etapy, po każdym zatrzymaj się, podsumuj (co powstało, koszt, co dalej)
i czekaj na moje OK:

ETAP A — weryfikacja źródeł pilota (zadanie 3.1), bez pobierania treści:
- Dla museum-digital (https://museum-digital.de/) i holzwerken.de (https://www.holzwerken.de/museum/)
  przeczytaj regulamin / impressum / informacje o prawach autorskich, robots.txt i, jeśli jest,
  dokumentację API. Pobierz tylko te strony, każdą raz, z User-Agent SkrzyniaMajstraBot/0.1.
- Uzupełnij kolumny ToS, robots.txt, Licencja zdjęć i Uwagi w sources.md (z linkami do tego,
  co przeczytałeś, i datą). Statusu na `zaakceptowane` NIE zmieniaj sam — zaproponuj status
  z uzasadnieniem, a ja zdecyduję. Jeśli zasady są niejasne, napisz, o co warto zapytać autora strony.
- Sprawdź też, czy museum-digital ma strugi niemieckich producentów (Ulmia, ECE, Steiner…)
  i ile mniej więcej rekordów z opisem producenta/okresu da się znaleźć.

ETAP B — szkielet i przyjmowanie zdjęć (zadania 1.1–1.5), bez sieci i bez płatnych API:
- pyproject (uv), config.toml, .env.example, ruff, pytest; migracje SQLite; inicjalizacja ChromaDB.
- Wgrywanie paczki zdjęć (także HEIC), SHA-256 i pomijanie duplikatów, zapis oryginałów bez zmian,
  kopie robocze 1000 px z poprawną orientacją EXIF.
- Grupowanie po EXIF z regulowanym progiem; UI: przeciąganie zdjęć między grupami, scal / podziel /
  odrzuć; automatyczny kolejny wolny numer SM-NNN.
- Testy grupowania i idempotencji. Na koniec instrukcja w README, jak uruchomić aplikację,
  i pokaż mi, jak ją odpalić.

ETAP C — ekstrakcja cech (zadania 2.1–2.6), dopiero po moim OK dla A i B:
- Schematy JSON (rdzeń + strug + szkielety pozostałych typów), słownik ujęć, prompt extraction_v1.
- Przed pierwszym wywołaniem sprawdź w aktualnej dokumentacji Anthropic: nazwę modelu
  claude-haiku-5-5, obsługę obrazów, structured outputs dla tego modelu, prompt caching i cennik.
  Jeśli coś różni się od design.md, zaktualizuj design.md i powiedz mi.
- Bramka kosztów i api_usage działają, ZANIM wyślesz pierwsze płatne zapytanie.
- Próba na 2–3 egzemplarzach: jeśli źródła z etapu A nie są jeszcze zaakceptowane, poproś mnie
  o kilka zdjęć do testu zamiast pobierać cokolwiek. Porównaj effort low vs medium.
  Pokaż mi wyniki JSON, opisy zdjęć i koszt każdego wywołania.

Moich narzędzi z listy w docs/brief.md na razie nie ruszamy.
```
