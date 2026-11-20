# rhb-video-unlock
 
*[English version: README.en.md](README.en.md)*
 
Skrypt zmienia oprogramowanie radia **Uconnect 430N (RHB)** z pakietu **50.01.01**:
Radio ma obsługiwać 50.01.01. W nowszych wersjach zmienia się obsługa dysku i wersja 50.01.01 nie zawiera wymaganych zmian więcej: https://mygig-disk.com/index-rhb.php
 
- **wideo w czasie jazdy** na przednim ekranie;
- **rynek europejski**: FM **87,5–108,0 MHz co 100 kHz** zamiast
  87,7–107,9 co 200 kHz (radio z USA nie łapie stacji na 88,0, 92,6 itd.).
## ⚠️ Najpierw przeczytaj
 
- **Bez gwarancji**, licencja MIT. Projekt nie jest powiązany ze Stellantis,
  Chryslerem ani producentem radia. Zmianę wykonujesz **na własne ryzyko**.
- **Repozytorium nie zawiera oprogramowania radia.** Potrzebny jest
  oficjalny pakiet `RHB-RBZ.50.01.01.zip`. Można go pobrać z
  [mygig-disk.com](https://mygig-disk.com/download-rhb.php); tam jest też opis
  wgrywania oprogramowania.
## Które radia można zmienić
 
| Radio | Czy można? |
|---|---|
| RHB / RBZ, rocznik 2011–2015 | **tak** |
| RHB / RBZ, rocznik 2016, wersja systemu **0210 lub starsza** | **tak** |
| RHB / RBZ, rocznik 2016, wersja systemu **0211 lub nowsza** | ⛔ **NIE** |
| Nowsze radia (oprogramowanie 70.xx.xx i 05.xx.xx) | nie (inna obsługa dysku) |
 
> ⛔ **Wersja systemu 0211 lub nowsza:** próba instalacji 50.01.01 kończy
> się uszkodzeniem radia i utratą dostępu do dysku twardego. Naprawa jest
> możliwa tylko w warsztacie, po wyjęciu radia z auta.
 
Dodatkowo:
 
- **Sprawny dysk twardy.** Radio z uszkodzonym dyskiem nie przyjmie
  aktualizacji.
- **Oprogramowanie nowsze niż 10.00.11, ale starsze niż 33.01.23:** najpierw
  zaktualizuj radio do **33.01.23**, dopiero potem wgrywaj 50.01.01.
## Co się zmienia
 
| | USA (oryginał) | Po zmianie |
|---|---|---|
| Wideo w czasie jazdy | zablokowane | **odblokowane** |
| FM | 87,7–107,9 MHz co 200 kHz | **87,5–108,0 MHz co 100 kHz** |
| AM | 530–1710 kHz co 10 kHz | 531–1611 kHz co 9 kHz |
| Języki menu | 3 (EN-US, FR-CA, ES) | **6** (EN-UK, FR, ES, DE, IT, NL) |
| RDS | wersja amerykańska (RBDS) | europejska (RDS), w tym lista PTY |
| SiriusXM | jest | **znika** (w Europie i tak nie działa) |
| Presety radia | — | zerowane przy pierwszym starcie |
 
## Jak to działa
 
Radio ma dwa procesory:
 
1. **Główny SoC** (plik `MAIN_APL_CPU1.mbi`). Funkcja decydująca o blokadzie
   wideo zwraca swój wynik w rejestrze `r2`. Pod offsetem `0x0B06F8` zamiana
   `52 F3` → `E2 00` (`mov #0,r2`) sprawia, że zawsze zwraca „bez blokady”,
   czyli 0.
2. **MCU audio-CAN** (pliki `*.aud`). Przechowuje rynek radia jako
   indeks (0 = Europa, 1 = USA, 2 = Japonia, 3 = reszta świata) i w
   oryginale **przy każdym starcie wymusza USA**. Od tego indeksu zależy
   tuner (zakres i krok FM/AM), a SoC dostaje od MCU informację o rynku i
   sam przełącza menu na wersję europejską. Skrypt ustawia indeks na 0.
Pakiet ma 4 warianty `.aud`: `H_I` (CAN I-HS), `H_B` (CAN-B), `HSI` i `HSB`
(to samo z SiriusXM). Radio samo wybiera właściwy, więc skrypt zmienia
wszystkie cztery. Każdy z 5 plików jest sprawdzany sumą SHA-256 przed
zmianą i po zapisie; wynik jest bajt w bajt taki, jak przetestowany w aucie.
 
## Co jest potrzebne
 
- pendrive (patrz [Problemy](#problemy));
- [Rufus](https://rufus.ie) (Windows) do sformatowania pendrive'a;
- oficjalny pakiet `RHB-RBZ.50.01.01.zip` z
  [mygig-disk.com](https://mygig-disk.com/download-rhb.php);
- [Python](https://www.python.org/downloads/) 3.9 lub nowszy. Na Windows w
  instalatorze zaznacz **„Add python.exe to PATH”**, resztę zostaw domyślnie;
- skrypt `rhb_europe_unlock.py` z tego repozytorium.
## Instrukcja krok po kroku (sprawdzona)
 
**0. Zasilanie.** Przez całą aktualizację silnik pracuje. Nie wyłączaj
zapłonu i nie wyjmuj pendrive'a w trakcie.
 
**1. Pendrive: tablica MBR, jedna partycja FAT32.** Pendrive z tablicą GPT
radio czyta, ale nie pokazuje okna
aktualizacji. W Rufusie ustaw:
- Wybór bootowania: **niebootowalne**;
- Schemat partycjonowania: **MBR**;
- System plików: **FAT32**.
Kliknij START (**wszystkie dane z pendrive'a zostaną usunięte**). Jeśli po
formatowaniu na pendrivie są pliki `autorun`, usuń je.
 
**2. Najpierw oryginał.** Rozpakuj oryginalny `RHB-RBZ.50.01.01.zip` do
głównego katalogu pendrive'a i wgraj najpierw ten **niezmieniony** pakiet:
- „Update baseline data files on HDD?” → **YES** (dane na dysk radia);
- aktualizacja aplikacji → **YES**.
**3. Zmiana pendrive'a skryptem.** Na pendrive z oryginalnym pakietem
skopiuj `rhb_europe_unlock.py` do katalogu głównego, przejdź do tego
katalogu i uruchom skrypt (Python 3.9+):
 
```text
python rhb_europe_unlock.py
```
 
Na Windows najprościej: otwórz pendrive w Eksploratorze plików, w pasku
adresu wpisz `cmd` i naciśnij Enter. Wiersz poleceń otworzy się od razu w
katalogu pendrive'a.
 
Skrypt zmienia pliki na miejscu, bez kopii zapasowych, i sprawdza ich sumy
kontrolne. Jeśli któryś plik nie jest oryginałem 50.01.01, nic nie zmienia.
 
**4. Wgranie wersji zmienionej.** Radio ma już 50.01.01 i pakiet też —
to nie przeszkadza:
- „Update baseline data files on HDD?” → **NO** (dane są już na dysku z
  kroku 2 i nie są zmieniane);
- aktualizacja aplikacji 50.01.01 → 50.01.01 → **YES**. Radio wgrywa
  aplikację („flash Application software”) i oprogramowanie MCU („flash
  Audio CPU software”);
- gdy po zakończeniu radio **ponownie zaproponuje** tę samą aktualizację →
  **CANCEL**. Potem wyjmij pendrive; radio działa.
**Powrót do oryginału:** wgraj ponownie niezmieniony pakiet 50.01.01.
 
## Problemy
 
- **Radio nie pokazuje okna aktualizacji.** Sprawdź, czy pendrive ma tablicę
  MBR i partycję FAT32 (krok 1), a pliki pakietu leżą w głównym katalogu.
- **Radio restartuje się w okolicach 25% instalacji.** Użyj innego
  pendrive'a. Zgłoszony przypadek: Goodram 16 GB USB 2.0 nie działa, ta sama
  pojemność w wersji USB 3.0 działa.
- **`python` nie jest rozpoznawany** w wierszu poleceń. Użyj `py` albo
  zainstaluj Pythona ponownie z zaznaczonym „Add python.exe to PATH”.
 
