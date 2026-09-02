# Raport AI-ML — 2026-09-02

**Data podsumowania:** 2026-09-02  
**Okres analizy:** od edycji 2026-09-01 do 2026-09-02 (Europe/Warsaw)

## Biznes

### SB Energy składa prospekt IPO bez działających centrów danych

SB Energy złożył formularz S-1, wykazując około 439 mld USD backlogu, z czego około 430 mld USD przypada na centra danych. W pierwszej połowie 2026 r. przychody wyniosły 138,7 mln USD, a strata netto 3,21 mld USD; spółka nie ma jeszcze operacyjnych centrów danych, a zakontraktowane 8,8 GW jest skoncentrowane na SoftBanku i OpenAI. NVIDIA zadeklarowała 1,5 mld USD prywatnej inwestycji, a OpenAI otrzymało warranty.

**Analiza techniczna:** Backlog nie jest odpowiednikiem dostępnej mocy: realizacja zależy od pozwoleń, przyłączy energetycznych, chłodzenia, dostaw transformatorów oraz uruchomienia sieci i serwerów. Dla odbiorcy oznacza to wysokie ryzyko harmonogramu i koncentracji kontrahentów, a dla rynku — dalsze finansowanie capacity przed uzyskaniem danych o MTBF, PUE, wykorzystaniu i marży operacyjnej.

**Źródła:** [SEC — formularz S-1 SB Energy, 2026-09-01](https://www.sec.gov/Archives/edgar/data/2133037/000162828026059639/sbenergy-sx1.htm), [Reuters, 2026-09-01](https://www.reuters.com/business/energy/softbank-backed-sb-energy-moves-closer-public-markets-with-us-ipo-filing-2026-09-01/)

### Dell podnosi prognozę przychodów z serwerów AI do 74 mld USD

Dell podniósł prognozę rocznych przychodów do 192 mld USD, a oczekiwane przychody z serwerów zoptymalizowanych pod AI do 74 mld USD. W ciągu ostatnich 12 miesięcy firma przyjęła zamówienia na serwery AI przekraczające 130 mld USD; przychody Q2 wzrosły o 58% do około 47 mld USD, zaś tradycyjne serwery i networking wzrosły ponad dwukrotnie.

**Analiza techniczna:** Popyt przesuwa ograniczenie z samego GPU na integrację całego racka: zasilanie, chłodzenie cieczą, fabric, storage i obsługę serwisową. Duży udział systemów NVIDIA ogranicza przenośność stosu CUDA, dlatego capacity planning powinien obejmować alternatywne profile AMD/ASIC oraz koszt migracji kerneli i serving runtime.

**Źródła:** [Dell — Q2 FY27 Performance Review](https://investors.delltechnologies.com/static-files/5d15be71-1d9a-45ec-8308-8987fb41084d), [Reuters, 2026-09-01](https://www.reuters.com/business/dell-again-lifts-forecasts-ai-demand-powers-record-results-2026-09-01/)

## Technologia

### MLPerf Storage v3.0 obejmuje KV cache, bazy wektorowe i S3

MLCommons dodał do MLPerf Storage v3.0 testy operacji odczytu/zapisu KV cache dla inference oraz indeksowania i zapytań w bazach wektorowych, a także obsługę obiektowego interfejsu S3. Benchmark rozszerza pomiar poza utrzymywanie wysokiego wykorzystania GPU w treningu na ścieżki danych krytyczne dla RAG i autoregresyjnego serving.

**Analiza techniczna:** Wyniki należy mapować na P95/P99 TTFT, przepustowość tokenów, hit ratio cache i odporność na rekonstrukcję indeksu, a nie traktować jako jeden ranking storage. Separacja warstw checkpoint/object, vector DB i KV cache ułatwia niezależne skalowanie, ale zwiększa liczbę domen awarii i wymaga wspólnej telemetrii kolejek oraz backpressure.

**Źródło:** [MLCommons, 2026-09-01](https://mlcommons.org/2026/09/mlperf-storage-v3-0-results/)

### NVIDIA publikuje metodykę doboru GPU pod rzeczywisty profil inference

NVIDIA proponuje wymiarowanie inference od profilu użycia: długości wejścia i wyjścia, concurrency, TTFT, inter-token latency, cache hit rate oraz rozkładu obciążenia. Wysoki hit rate KV cache pomija część prefill, obniżając TTFT i zapotrzebowanie na GPU; zalecany jest model core-and-flex łączący bazową pojemność z elastycznym burstem.

**Analiza techniczna:** Metodyka jest użyteczna, ale kalkulacje dostawcy należy odtworzyć na własnych trace’ach, z P99 i scenariuszami awarii. Quantization, pruning i distillation poprawiają TCO tylko po walidacji jakości, a chmurowy burst wymaga kontroli transferu danych, warm-upu modeli i zgodności runtime między lokalną a zewnętrzną pulą.

**Źródło:** [NVIDIA Developer Blog, 2026-09-01](https://developer.nvidia.com/blog/how-to-size-gpus-for-ai-inference-and-tco-without-overspending/)

## Implikacje praktyczne

1. Kontraktować moc AI etapami, wiążąc płatności z energization, testami fabric/storage, osiągniętym PUE oraz SLO dostępności, nie z samym backlogiem dostawcy.
2. Budować model capacity na rozkładach ISL/OSL, concurrency, P99 TTFT/ITL i cache hit rate; średnia tokens/s nie wystarcza do wymiarowania.
3. Rozdzielić klasy storage dla checkpointów, vector DB i KV cache, lecz ujednolicić obserwowalność: queue depth, eviction, hit ratio, tail latency i koszt na żądanie.
4. Utrzymywać plan przenośności dla co najmniej dwóch klas akceleratorów i warstwę serving abstrahującą CUDA/ROCm/ASIC; regularnie mierzyć koszt rekompilacji i regresję jakości.
5. Dla elastycznego burstu wcześniej testować cold start, dostępność kwot, egress i degradację po utracie regionu.

## Trend tygodnia

Rynek przechodzi od narracji o niedoborze pojedynczych GPU do finansowania i pomiaru całych systemów AI. Kapitał jest angażowany przed uruchomieniem mocy, podczas gdy warstwa techniczna zaczyna mierzyć KV cache, vector DB, tail latency i efektywność konkretnego profilu żądań. Rosnące zamówienia OEM potwierdzają popyt, lecz zwiększają znaczenie weryfikacji backlogu, koncentracji klientów i gotowości energetycznej. Przewagę operacyjną daje dziś spójne capacity planning od tokenu przez storage i fabric aż po przyłącze energetyczne.

## To obserwować

- finalne warunki IPO SB Energy, tempo konwersji 8,8 GW backlogu na działającą moc i udział dwóch głównych klientów;
- marżę operacyjną Della na systemach AI oraz backlog, terminy dostaw i attach rate storage/networking;
- pierwsze porównywalne wyniki MLPerf Storage v3.0 dla KV cache, VDB i S3;
- relację P99 TTFT/ITL do kosztu przy różnych hit ratio KV cache i strategiach core-and-flex;
- udział alternatywnych akceleratorów oraz dojrzałość portability między CUDA, ROCm i dedykowanymi ASIC.
