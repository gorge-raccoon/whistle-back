# Whistle Back
Urządzenie nasłuchujące melodię z otoczenia i odwzorowujące ją za pomocą 
brzęczyka.



## Potrzebne elementy
W projekcie wykorzystuję:
* [klon Arduino Micro][arduino-ali] wyposażony w mikrokontroler [ATmega32U4][atmega] i 2,5&nbsp;kB pamięci SRAM,
* [mikrofon elektretowy ze wzmacniaczem MAX4466][mic-ali] pracujący z napięciem 2.4&nbsp;V &ndash; 5.5&nbsp;V,
* [brzęczyk piezoelektryczny][buzzer-ali] bez wbudowanego generatora sygnału,
* płytkę stykową z 400 otworami,
* przycisk monostabilny,
* przewody połączeniowe.

<a href="https://www.aliexpress.com/item/Micro-ATmega32U4-5V-16MHz-Pins-soldered-Compatible-with-Arduino-Micro-and-Leonardo/32676998690.html">![obrazek klonu Arduino Micro][arduino-img]</a><a href="https://www.aliexpress.com/item/New-Electret-Microphone-Amplifier-MAX4466-With-Adjustable-Gain-For-Arduino/32717091448.html">![obrazek mikrofonu][mic-img]</a><a href="http://www.robitshop.com/passive-buzzer-module">![obrazek brzęczyka][buzzer-img]</a><a href="https://www.aliexpress.com/item/V1NF-Hot-Sale-400-Points-Solderless-Bread-Board-Breadboard-PCB-Test-Board-Free-Shipping/32633859572.html">![obrazek płytki stykowej][breadboard-img]</a><a href="http://minielektro.dk/dip-tryk-knap.html">![obrazek przycisku][button-img]</a><a href="https://kamami.pl/13022-przewody-i-zlacza-do-arduino">![obrazek przewodów][wires-img]</a>

Zdecydowałem się wykorzystać platformę Arduino Micro między innymi dlatego, że pozwala na zasilanie urządzeń peryferyjnych napięciem 3,3&nbsp;V. Mikrofon zasilany takim napięciem charakteryzuje się większą czułością, niż w przypadku zastosowania napięcia 5&nbsp;V.



## Funkcjonalność
Urządzenie pracuje według następującego schematu.

1. Oczekiwanie na naciśnięcie przycisku.
2. Nagrywanie melodii z otoczenia.
3. Dla każdej z próbek czasowych:
  1. Przeprowadzenie transformaty Fouriera.
  2. Zapisanie odpowiedniego sygnału wyjściowego.
4. Odtworzenie sygnału wyjściowego.
5. Powrót do punktu 1.



## Lista zadań
Aby ułatwić sobie pracę, przygotowałem listę czynności, które należy wykonać.
- [x] Zakup potrzebnych elementów.
- [x] Implementacja rekurencyjnego algorytmu FFT w języku Python.
- [x] Implementacja algorytmu Cooleya-Tukeya w języku C.
- [x] Test współpracy klonu Arduino z Arduino IDE.
- [x] Przylutowanie pinów modułu mikrofonu.
- [x] Złożenie układu na płytce stykowej.
- [x] Podstawowa obsługa przycisku.
- [x] Podstawowa obsługa brzęczyka.
- [x] Podstawowa obsługa mikrofonu.
- [x] Obsługa rozpoczęcia i zakończenia nagrywania.
- [x] Wykonywanie transformat Fouriera.
- [x] Generowanie adekwatnego sygnału wyjściowego.



## Opis użytego algorytmu
Wykorzystałem najbardziej powszechną formę algorytmu Cooleya–Turkeya, Radix-2. Polega ona na obliczeniu transformat indeksów parzystych i nieparzystych osobno, a następnie połączeniu danych. Przyspieszanie obliczeń osiąga się przy wykorzystaniu okresowości transformaty Fouriera.

W trakcie działania algorytmu zamienianie kolejności wyliczonych wartości nie jest wykonywane po każdym etapie transformaty. W wyniku tego otrzymuje się tablicę z indeksami w kolejności **odwróconych bitów**. W mojej implementacji dokonuję zmiany kolejności indeksów na końcu.

![diagram wykonywania FFT][bit-reversal]



## Złożenie układu na płytce stykowej
Złożyłem układ tak, jak przedstawione jest to na poniższym diagramie.

![diagram płytki stykowej][breadboard-diagram]

![zdjęcie płytki stykowej][breadboard-photo]

Mikrofon zasilany jest napięciem 3,3&nbsp;V. Pin 3V3 połączony jest również z pinem REF dlatego, że czytamy wartości analogowe z mikrofonu właśnie o takim napięciu. Pin przycisku (D5) został skonfigurowany jako INPUT_PULLUP.



## Odczytywanie danych z mikrofonu
Tak skonfigurowany układ jest w stanie poprawnie odczytywać sygnał z mikrofonu, co widać na poniższych zrzutach ekranu.

![zrzut ekranu projektu stream_mic][stream-mic-code]

![wykres wartości zwracanych przez układ][stream-mic-chart]



## Ograniczenia Arduino
Arduino Micro posiada 2,5&nbsp;kB pamięci RAM, a jedna liczba zespolona zajmuje 8&nbsp;B. Maksymalnie przechowywać w pamięci możemy więc około 300 próbek. Dodając ograniczenie implementacji algorytmu FFT na tablice próbek długości 2<sup>N</sup> i potrzebę stworzenia drugiej tablicy tej samej długości, maksymalnie możemy przechowywać w pamięci 128 próbek.



## Projekty Arduino
W projekcie *fft_buzzer* zaimplementowałem pełną funkcjonalność.

Projekty *fft* i *fft_continuous* zawierają implementację algorytmu FFT. Projekt *stream_mic* zawiera obsługę mikrofonu.



## Napotkane trudności
Podczas rozwoju projektu musiałem rozwiązać wiele problemów. Były to między innymi nieprawidłowe wyniki obliczeń wynikające z przekroczenia zakresu i propagacji błędów reprezentacji zmiennoprzecinkowej. Musiałem również nauczyć się poprawnie korzystać z danych przechowywanych w pamięci programu za pomocą słowa kluczowego **PROGMEM**.

Kluczowym okazało podłączenie styku GND mikrofonu do osobnego pinu mikrokontrolera. W w przeciwnym wypadku mikrofon odczytywał sygnał przesunięty o pewną stałą wartość.

Precyzyjne odczytanie (i odtworzenie) częstotliwości sprawia problemy.


## Źródła
Odnośniki na obrazkach prowadzą do ich źródła.

[Diagram](https://commons.wikimedia.org/wiki/File:DIT-FFT-butterfly.png) przedstawiający wykonywanie FFT został stworzony przez użytkownika Virens.

Diagram układu przygotowałem w programie [Fritzing](http://fritzing.org/home/).



## Bibliografia
* [Fast Fourier Transforms](http://www.katjaas.nl/FFT/FFT.html) [dostęp: 2016-11-8]  
  obrazowe przedstawienie sposobu działania i implementacji algorytmu Colleya-Turkeya
* [Cooley&ndash;Tukey FFT algorithm](https://en.wikipedia.org/wiki/Cooley–Tukey_FFT_algorithm) [dostęp: 2016-11-8]  
  artykuł w Wikipedii
* [Szeregi i transformaty Fouriera](http://www.icsr.agh.edu.pl/~mownit/output/pdf/fourier.pdf) [dostęp: 2016-11-8]  
  notatki do wykładu z MOwNiT-u dra Bubaka
* R. Burden, J. Faires. Numerical Analysis. Edycja 9. Strony 547&ndash;557.  
  rozdział o szybkiej transformacie Fouriera
* [Arduino - Reference](https://www.arduino.cc/en/Reference/HomePage) [dostęp: 2016-12-06]  
  dokumentacja języka programowania Arduino

***

Paweł Szopiński
    
[arduino-ali]: https://www.aliexpress.com/item/Micro-ATmega32U4-5V-16MHz-Pins-soldered-Compatible-with-Arduino-Micro-and-Leonardo/32676998690.html
[atmega]: http://www.atmel.com/devices/atmega32u4.aspx
[mic-ali]: https://www.aliexpress.com/item/New-Electret-Microphone-Amplifier-MAX4466-With-Adjustable-Gain-For-Arduino/32717091448.html
[buzzer-ali]: https://www.aliexpress.com/item/Passive-Buzzer-Module-for-Arduino-AVR-PIC-Good-New-KY-006/32273623799.html

[arduino-img]: img/arduino.png
[mic-img]: img/mic.png
[buzzer-img]: img/buzzer.png
[breadboard-img]: img/breadboard.png
[button-img]: img/button.png
[wires-img]: img/wires.png

[bit-reversal]: img/bit-reversal.png

[breadboard-diagram]: img/breadboard-diagram.png
[breadboard-photo]: img/breadboard-photo.png

[stream-mic-code]: img/stream-mic-code.png
[stream-mic-chart]: img/stream-mic-chart.png
