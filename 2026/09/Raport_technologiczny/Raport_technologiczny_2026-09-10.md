# Raport technologiczny — 2026-09-10

**Data podsumowania:** 2026-09-10  
**Zakres przyrostowy:** od edycji 2026-09-09 do 2026-09-10 (Europe/Warsaw)

## Biznes

Brak nowych, odrębnych zdarzeń biznesowych spełniających próg istotności.

## Technologia

### WeWorm: zero-click RCE w stosie VoIP WeChat z propagacją między iOS i Androidem

Calif Research opublikował 8 września demonstrację robaka wykorzystującego błąd korupcji pamięci w stosie VoIP WeChat. Samo przychodzące połączenie wystarczało do przejęcia konta — ofiara nie musiała go odebrać — a skompromitowane konto mogło automatycznie zadzwonić do kolejnych kontaktów; demonstracja objęła propagację Pixel 10a → iPhone 17e → Pixel 10a. Badacze uzyskali odczyt i wysyłanie wiadomości oraz wykonywanie połączeń, a w połączeniu z innymi błędami możliwe było przejęcie urządzenia.

Tencent wydał 21 sierpnia WeChat 8.0.77 dla Androida i 8.0.76 dla iOS, a 28 sierpnia badacze potwierdzili dodatkowe ograniczenie exploita po stronie serwera dla wszystkich użytkowników. Calif podał również, że AI pomogła znaleźć błąd i przygotować pierwszy exploit RCE w około dwa dni, natomiast zbudowanie robaka zajęło kolejny tydzień.

**Znaczenie:** mobilne komunikatory i ich warstwa sygnalizacji/VoIP powinny być traktowane jak wystawione na Internet elementy infrastruktury z własnym budżetem ryzyka, a nie jak zwykłe aplikacje użytkownika. Dla środowisk zarządzanych oznacza to konieczność wymuszania minimalnych wersji przez MDM, monitorowania nietypowych serii połączeń oraz ograniczania zaufania wynikającego z książki kontaktów; mitigacja serwerowa zmniejsza ryzyko tego konkretnego exploita, ale nie eliminuje klasy ataków na multimedia przetwarzane przed interakcją użytkownika.

**Źródło:** [Calif Research — WeWorm, 8 września 2026](https://calif.io/research/weworm)
