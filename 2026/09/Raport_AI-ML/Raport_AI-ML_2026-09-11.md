# Raport AI-ML — 2026-09-11

**Data podsumowania:** 2026-09-11  
**Okno przyrostowe:** od edycji z 2026-09-10 do 2026-09-11 (Europe/Warsaw)

## Biznes

### 1. Zasilanie „behind the meter” staje się główną ścieżką skracania time-to-power

SemiAnalysis podał 10 września, że śledzi już **75 GW wiążących zamówień** na urządzenia dla zasilania centrów AI poza licznikiem sieciowym, z czego około **20 GW zamówiono w II kwartale 2026 r.**; do końca roku około 3 GW mocy IT w USA ma działać w takim modelu. Skala kontraktów przesuwa ryzyko realizacji z samego przyłącza sieciowego na pozwolenia emisyjne, dostawy gazu, produkcję turbin i silników, dostępność serwisu oraz zdolność wyspowej pracy przy skokowym profilu obciążenia GPU.

**Znaczenie:** organizacje planujące klastry wielkoskalowe powinny modelować koszt energii razem z wartością wcześniejszego uruchomienia, ale nie traktować podpisanego zamówienia na generację jako dostępnej mocy. Capacity plan musi osobno śledzić FID, pozwolenia, gazociąg, terminy OEM, N+1/N+2, black-start i jakość energii; w przeciwnym razie zakontraktowane GPU mogą czekać na niedojrzały power plane.

**Źródło:** [SemiAnalysis — What is So Hard About Behind-The-Meter Power For Datacenters? Part 1 (10.09.2026)](https://newsletter.semianalysis.com/p/what-is-so-hard-about-behind-the)

## Technologia

### 1. d-Matrix dołącza Raptor do domeny NVLink Fusion

d-Matrix ogłosił 10 września użycie **NVLink Fusion** do włączenia akceleratorów inference Raptor do systemów NVIDIA; finalizacja projektu układu jest planowana do końca 2026 r., a systemy na 2027 r. Firma równolegle współpracuje z Astera Labs nad szybkimi ścieżkami danych. To materialna aktualizacja wcześniej opisywanego Raptora: zmienia go z samodzielnego akceleratora w element rack-scale architektury z dostępem do domeny pamięci i połączeń NVIDIA.

**Znaczenie:** integracja może obniżyć koszt i opóźnienie inference dla wybranych modeli, ale zwiększa zależność od topologii, firmware'u i cyklu kwalifikacji NVIDIA. Przed adopcją potrzebne będą pomiary p95/p99 token latency, przepustowości na wat, zachowania przy utracie linku oraz narzutów przenoszenia modeli między CUDA GPU a Raptor; sam interfejs NVLink nie gwarantuje przenośności stosu wykonawczego.

**Źródło:** [Reuters — d-Matrix to use NVIDIA chip-linking technology in AI servers (10.09.2026)](https://www.reuters.com/business/media-telecom/chip-startup-d-matrix-use-nvidia-chip-linking-tech-ai-servers-2026-09-10/)

## Implikacje praktyczne

1. Rozdzielić rezerwację GPU od bram energetycznych: zamówienie akceleratorów uruchamiać dopiero po osiągnięciu mierzalnych kamieni milowych dla paliwa, pozwoleń, urządzeń i black-start.
2. Dla źródeł behind-the-meter wymagać modelu niezawodności obejmującego awarię wspólnej infrastruktury gazowej, ograniczenia emisyjne, zapas części i degradację mocy przy wysokiej temperaturze.
3. Utrzymywać warstwę servingową zdolną kierować modele między GPU i ASIC; kontrakty SLA powinny rozdzielać TTFT, inter-token latency, throughput i zużycie energii.
4. Traktować NVLink Fusion jako zależność platformową: przygotować testy zgodności wersji firmware, telemetrii linków, obsługi błędów i odtwarzania po awarii przed podpisaniem zobowiązań wolumenowych.
5. W horyzoncie 3–5 lat planować power observability równie szczegółowo jak GPU observability — od jakości energii i rezerwy wirującej po korelację z throttlingiem i błędami fabric.

## Trend tygodnia

Wąskim gardłem infrastruktury AI przestaje być wyłącznie dostępność akceleratorów; stają się nim kompletne domeny rack-scale oraz energia możliwa do uruchomienia bez wieloletniego oczekiwania na sieć. Jednocześnie dostawcy ASIC próbują wejść do ekosystemu NVIDIA przez NVLink Fusion, zamiast budować cały rack i software stack od zera. W efekcie przewaga przesuwa się w stronę operatorów zdolnych wspólnie projektować compute, fabric, serving i zasilanie, lecz rośnie ryzyko koncentracji na jednym control plane i łańcuchu dostaw.

## To obserwować

- pierwsze niezależne benchmarki Raptora w systemach NVLink Fusion: p99, tokens/s/W i zachowanie przy awarii linku;
- dotrzymanie przez d-Matrix terminu tape-out/final design do końca 2026 r. i dostępności systemów w 2027 r.;
- przejście 75 GW zamówień BTM przez FID, pozwolenia, dostawy paliwa i odbiór techniczny;
- udział projektów BTM wymagających późniejszego przejścia na fuel cells lub połączenie z siecią;
- metryki niezawodności mikrosieci dla obciążeń GPU: black-start, ride-through, harmoniczne i utrata wspólnego źródła paliwa.
