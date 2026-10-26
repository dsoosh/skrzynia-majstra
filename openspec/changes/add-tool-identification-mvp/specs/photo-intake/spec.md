## ADDED Requirements

### Requirement: Przechowywanie oryginałów
System SHALL zapisywać każde wgrane zdjęcie bez zmian, w pełnej rozdzielczości, adresowane hashem SHA-256,
i SHALL tworzyć kopię roboczą przeskalowaną do 1000 px na dłuższym boku (JPEG) do wysyłki do modelu.

#### Scenario: Ponowne wgranie tego samego pliku
- **WHEN** użytkownik wgrywa plik, którego hash już istnieje
- **THEN** system nie zapisuje go drugi raz i pokazuje istniejące zdjęcie z jego grupą

#### Scenario: Zdjęcie z telefonu w HEIC
- **WHEN** użytkownik wgrywa plik HEIC
- **THEN** oryginał zostaje zapisany bez zmian, a kopia robocza powstaje jako JPEG z poprawną orientacją

### Requirement: Grupowanie po czasie wykonania
System SHALL grupować wgrane zdjęcia według czasu wykonania z EXIF, zaczynając nową grupę, gdy przerwa
między kolejnymi zdjęciami przekracza konfigurowalny próg.

#### Scenario: Paczka zdjęć trzech narzędzi
- **WHEN** użytkownik wgrywa 30 zdjęć zrobionych w trzech seriach rozdzielonych kilkuminutowymi przerwami
- **THEN** system proponuje trzy grupy

#### Scenario: Brak EXIF
- **WHEN** zdjęcie nie ma czasu wykonania w EXIF
- **THEN** system używa czasu pliku i oznacza zdjęcie w UI jako niepewnie zgrupowane

### Requirement: Ręczna korekta grup
System SHALL pozwalać przenosić zdjęcia między grupami, scalać i dzielić grupy, odrzucać zdjęcia
oraz SHALL nadawać każdej grupie kolejny wolny numer inwentarzowy `SM-NNN` (jedno narzędzie = jeden numer, bez podnumerów).

#### Scenario: Przeniesienie zdjęcia
- **WHEN** użytkownik przeciąga zdjęcie do innej grupy
- **THEN** zdjęcie należy do nowej grupy, a wcześniejszy wynik analizy obu grup oznaczony jest jako nieaktualny
