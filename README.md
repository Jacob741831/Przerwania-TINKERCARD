# 🚦 Sygnalizacja świetlna – 4 kierunki

## 1. Opis projektu
Projekt przedstawia system sygnalizacji świetlnej dla czterech kierunków, wykorzystujący Arduino Uno oraz cztery czujniki ruchu PIR.

Każdy kierunek posiada czerwone i zielone światło. Gdy czujnik wykryje ruch, odpowiedni kierunek zostaje dodany do kolejki oczekujących. Jeśli żaden inny kierunek nie ma zielonego światła, program uruchamia zielone dla pierwszego oczekującego kierunku.

Zielone światło świeci przez 5 sekund, po czym wszystkie światła zmieniają się na czerwone. Następnie program może obsłużyć kolejny kierunek.

Projekt został przygotowany do symulacji w Tinkercad Circuits.

## 2. Wykorzystane elementy
- Arduino Uno
- 4 czerwone diody LED
- 4 zielone diody LED
- 4 czujniki ruchu PIR
- Rezystory ograniczające prąd diod LED
- Przewody połączeniowe
- Tinkercad Circuits

## 3. Podłączenie pinów

| Kierunek | Czerwone światło | Zielone światło | Czujnik PIR |
| :--- | :--- | :--- | :--- |
| 1 – prawo | D2 | D6 | D10 |
| 2 – góra | D3 | D7 | D11 |
| 3 – lewo | D4 | D8 | D12 |
| 4 – dół | D5 | D9 | D13 |

*Uwaga: Diody LED należy podłączyć przez odpowiednie rezystory ograniczające prąd.*

## 4. Obsługa przerwań
Projekt wykorzystuje mechanizm Pin Change Interrupt (PCINT) mikrokontrolera ATmega328P. Pozwala on wykrywać zmiany stanu na pinach D10–D13.

Standardowa funkcja `attachInterrupt()` nie obsługuje tych pinów w taki sam sposób jak piny D2 i D3 na Arduino Uno. Dlatego projekt wykorzystuje rejestry sprzętowe mikrokontrolera.

### 4.1. Inicjalizacja przerwań
- `initProgram()` – około linii 128–158. Konfiguruje piny, ustawia początkowy stan świateł i wywołuje funkcję inicjalizującą przerwania.
- `initPrzerwania()` – około linii 162–186. Konfiguruje rejestry i włącza przerwania dla pinów D10–D13.

Wywołanie funkcji inicjalizującej przerwania znajduje się w `initProgram()`:
```cpp
initPrzerwania();
```

### 4.2. Główna funkcja przerwania
`ISR(PCINT0_vect)` – około linii 194–224.  
Funkcja jest wywoływana, gdy zmieni się stan jednego z obsługiwanych pinów D10–D13. Sprawdza, który pin uległ zmianie, a następnie wywołuje odpowiednią funkcję obsługi czujnika.

### 4.3. Funkcje obsługujące czujniki

| Funkcja | Opis | Przybliżone linie |
| :--- | :--- | :--- |
| `ISR_czujnik1()` | Obsługa czujnika D10 | 233–235 |
| `ISR_czujnik2()` | Obsługa czujnika D11 | 238–240 |
| `ISR_czujnik3()` | Obsługa czujnika D12 | 243–245 |
| `ISR_czujnik4()` | Obsługa czujnika D13 | 248–250 |
| `obsluzCzujnik(byte i)` | Zapis stanu czujnika, rozpoznanie zbocza i aktualizacja kolejki | 258–283 |

*Numery linii są orientacyjne i mogą się różnić w zależności od edytora oraz aktualnej wersji kodu.*

## 5. Rejestry odpowiedzialne za przerwania
W funkcji `initPrzerwania()` wykorzystywane są trzy rejestry:
- **PCICR** – włącza obsługę przerwań Pin Change dla wybranej grupy pinów.
- **PCMSK0** – wybiera piny, które mogą wywołać przerwanie.
- **PCIFR** – umożliwia wyczyszczenie oczekującej flagi przerwania.

### Mapowanie pinów Arduino Uno

| Pin Arduino | Bit portu B | Numer przerwania |
| :--- | :--- | :--- |
| D10 | PB2 | PCINT2 |
| D11 | PB3 | PCINT3 |
| D12 | PB4 | PCINT4 |
| D13 | PB5 | PCINT5 |

## 6. Najważniejsze funkcje programu

| Funkcja | Opis |
| :--- | :--- |
| `setup()` | Uruchamia komunikację szeregową i inicjalizację programu. |
| `initProgram()` | Konfiguruje piny, światła i przerwania. |
| `initPrzerwania()` | Włącza przerwania PCINT dla czujników. |
| `ISR(PCINT0_vect)` | Wykrywa zmianę stanu na pinach D10–D13. |
| `obsluzCzujnik(byte i)` | Zapisuje stan czujnika i aktualizuje kolejkę. |
| `obsluzZdarzenia()` | Wyświetla informacje diagnostyczne w Serial Monitorze. |
| `obsluzKolejke()` | Wybiera kierunek, który otrzyma zielone światło. |
| `sprawdzCzasZielonego()` | Sprawdza, czy minęło 5 sekund zielonego światła. |
| `wlaczZielone(byte i)` | Włącza zielone światło dla wskazanego kierunku. |
| `wszystkieCzerwone()` | Włącza czerwone światła na wszystkich kierunkach. |

## 7. Zasada działania
1. Po uruchomieniu Arduino wszystkie światła ustawiane są na czerwone.
2. Program inicjalizuje przerwania dla czterech czujników PIR.
3. Zmiana stanu czujnika wywołuje przerwanie.
4. Wykrycie zbocza narastającego (`LOW -> HIGH`) dodaje kierunek do kolejki.
5. Jeżeli żaden kierunek nie ma zielonego światła, program wybiera pierwszy oczekujący kierunek.
6. Zielone światło świeci przez 5 sekund.
7. Po upływie tego czasu wszystkie światła wracają do czerwonego.
8. Program przechodzi do obsługi kolejnego oczekującego kierunku.

## 8. Zmienne wykorzystywane w obsłudze przerwań
- `kolejka[]` – przechowuje informację, które kierunki oczekują na zielone światło.
- `stanyCzujnikow[]` – przechowuje ostatni odczytany stan każdego czujnika.
- `typZmiany[]` – przechowuje typ ostatniego zbocza.
- `zdarzenia` – przechowuje flagi zdarzeń oczekujących na obsługę w głównej pętli.
- `poprzedniPortB` – umożliwia wykrycie, które bity portu B zmieniły stan.

*Zmienne współdzielone między funkcjami przerwań a główną pętlą programu zostały oznaczone jako `volatile`.*

## 9. Ograniczenia projektu
- Kolejka jest obsługiwana według stałej kolejności indeksów, a nie czasu zgłoszenia.
- Kilka zdarzeń z tego samego kierunku może zostać połączonych w jedną flagę kolejki.
- Program nie uwzględnia dodatkowego czasu bezpieczeństwa między zmianami świateł.
- Czujniki PIR mogą wymagać czasu na stabilizację po włączeniu zasilania.
- Projekt ma charakter edukacyjny i nie powinien być stosowany jako rzeczywisty system sterowania ruchem drogowym.

## 10. Uruchomienie w Tinkercad
1. Utwórz nowy układ w Tinkercad Circuits.
2. Dodaj Arduino Uno, diody LED, rezystory i czujniki PIR.
3. Połącz elementy zgodnie z tabelą pinów.
4. Wklej kod programu do edytora Arduino.
5. Uruchom symulację.
6. Obserwuj działanie świateł oraz komunikaty w Serial Monitorze ustawionym na 9600 baud.

**Cel projektu:** demonstracja obsługi czujników za pomocą przerwań sprzętowych, współdzielenia danych między `ISR` a `loop()` oraz sterowania czterema kierunkami sygnalizacji świetlnej.
