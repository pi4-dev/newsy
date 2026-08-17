# Raport AI-ML — 2026-08-17

**Okres analizy:** 2026-08-16 – 2026-08-17 (Europe/Warsaw)

## Biznes

### Microsoft: deklarowana capacity nie przekłada się wprost na aktywny compute

Śledztwo Guardiana z 17 sierpnia wskazuje na możliwą dużą rozbieżność między komunikowaną przez Microsoft mocą centrów danych a liczbą faktycznie zainstalowanych akceleratorów. Według dokumentów wewnętrznych opisanych przez redakcję Microsoft miał około 2,2 mln chipów AI w połowie 2026 r.; Microsoft zakwestionował obliczenia i wnioski, ale nie podał alternatywnych liczb. Materiał wskazuje, że głównym ograniczeniem może być tempo energizacji, budowy „warm shells” i uruchamiania kompletnej infrastruktury, a nie sama podaż GPU.

**Znaczenie architektoniczne:** deklarowane lub zakontraktowane MW nie powinny być używane jako równoważnik dostępnego compute. Przy ocenie hyperscalera albo neoclouda trzeba oddzielać land/power secured, energized capacity, commissioned IT load, zainstalowane GPU oraz capacity faktycznie dostępne dla klientów.

**Źródło:** https://www.theguardian.com/technology/2026/aug/17/are-microsofts-ai-plans-being-held-back-by-a-shortage-of-chips

## Technologia

### Epoch AI aktualizuje publiczny zbiór danych o dużych centrach AI

Aktualizacja z 16 sierpnia obejmuje 82 obiekty, szacowane 12,7 GW mocy IT i 13,7 mln ekwiwalentów H100. Dane powstają z pozwoleń, obrazów satelitarnych i modeli technicznych; Epoch deklaruje, że 80% estymacji mocy IT mieści się w przedziale około 1,4× wartości rzeczywistej, a daty uruchomienia w granicy sześciu miesięcy. Zbiór jest użyteczny do niezależnej kontroli harmonogramów, ale nie zastępuje telemetryki operatora ani potwierdzenia commissioned capacity.

**Analiza:** łączna moc obiektowa jest wyższa od mocy IT o narzut chłodzenia i infrastruktury pomocniczej, dlatego porównywanie publicznych GW bez definicji prowadzi do błędów w capacity planningu. Dla latency, throughput i reliability ważniejsza jest liczba aktywnych, sieciowo połączonych akceleratorów oraz osiągana efektywność niż teoretyczny peak; sam zbiór zaznacza, że praktyczna wydajność może wynosić 20–50% peak. Dane mogą stanowić zewnętrzny sygnał observability dla ryzyka opóźnień, lecz wymagają walidacji kontraktowej.

**Źródło:** https://epoch.ai/data/ai-data-centers

## Implikacje praktyczne

1. **Wymagać pięciu osobnych metryk capacity:** secured power, energized MW, commissioned IT MW, installed accelerators i customer-available accelerators.
2. **Powiązać płatności i rezerwacje z kamieniami odbiorowymi.** Umowy capacity powinny zawierać testy burn-in, fabric/storage throughput, dostępność runtime oraz kary za opóźnienie.
3. **Modelować ramp-up zamiast skoku 0→100%.** Uwzględniać dostawy transformatorów, switchgear, chłodzenia, serwerów, optyki i czas strojenia fabric.
4. **Utrzymywać niezależne źródła w capacity observability.** Łączyć raporty operatora z danymi o energizacji, pozwoleniami i obserwacją postępu budowy.
5. **Nie przeliczać MW bezpośrednio na GPU.** Stosować przedziały uwzględniające PUE, generację GPU, pobór całego noda, storage/network overhead i zakładane utilization.

## Trend tygodnia

Najważniejszym ograniczeniem skali AI staje się konwersja zapowiedzianej mocy i zakupionego sprzętu na uruchomiony, stabilny compute. Rynek coraz częściej raportuje GW i CAPEX, ale te wskaźniki nie opisują stopnia commissioning ani dostępności dla użytkownika. Niezależne dane o budowie obiektów zaczynają pełnić rolę zewnętrznego sygnału capacity risk. Dla architektów oznacza to konieczność traktowania energization i commissioning jako elementów SLO oraz ryzyka dostawcy.

## To obserwować

- uruchamianie Microsoft Fairwater Wisconsin i Fairwater Atlanta;
- różnicę między announced, energized i commissioned MW u hyperscalerów;
- liczbę zainstalowanych B200/B300/GB300 względem zamówień;
- terminy przyłączeń energetycznych i dostaw transformatorów;
- rzeczywistą dostępność nowych instancji GPU w regionach chmurowych;
- wartości PUE i udział mocy przeznaczony na IT;
- osiągany throughput jako procent teoretycznego peak;
- kompletność backend fabric, storage i optics przy nowych uruchomieniach;
- opóźnienia między dostawą GPU a udostępnieniem capacity klientom;
- kolejne rewizje zbioru Epoch AI i metodologia niepewności.