## ADDED Requirements

### Requirement: Raport identyfikacji
System SHALL generować dla każdego narzędzia raport z wyciągniętymi cechami, top 5 dopasowaniami
z linkami do źródeł i uzasadnieniem oraz listą brakujących ujęć; raport dostępny w aplikacji i jako plik.

#### Scenario: Raport dla SM-036
- **WHEN** użytkownik otwiera raport SM-036
- **THEN** widzi cechy, 5 dopasowań z linkami, uzasadnienia i brakujące ujęcia

#### Scenario: Brak wyniku ekstrakcji
- **WHEN** ekstrakcja zakończyła się błędem
- **THEN** raport pokazuje zapisany błąd zamiast dopasowań
