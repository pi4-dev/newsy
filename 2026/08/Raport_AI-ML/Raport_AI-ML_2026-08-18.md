# Raport AI-ML — 2026-08-18

**Okres analizy:** 2026-08-17 – 2026-08-18 (Europe/Warsaw)

## Biznes

### PORTS-Pike: NVIDIA zabezpiecza 4,25 GW infrastruktury dla 8-GW kampusu OpenAI

**Fakty:** OpenAI zakontraktował około 8 GW mocy IT w kampusie PORTS-Pike w Ohio na podstawie 20-letniej umowy najmu z SB Energy. NVIDIA ma zapewnić wsparcie kredytowe dla gruntu, energii i budowy shell dla początkowych 4,25 GW IT oraz zainwestować 1,5 mld USD w SB Energy; Reuters ocenia maksymalną skalę gwarancji na około 105 mld USD. Pierwsze 800 MW ma być dostępne w 2028 r., natomiast dalsze fazy zależą od nowych źródeł wytwarzania, linii przesyłowych, pozwoleń, finansowania i odbiorów technicznych.

**Analiza:** struktura przesuwa część ryzyka budowy z klienta na dostawcę GPU i dewelopera infrastruktury, ale równocześnie łączy land/power/shell, pełny stos DSX, compute i długoterminowy popyt w jeden ekosystem NVIDIA. Dla capacity planningu najważniejsze są kamienie commissioning i tempo ramp-up, nie końcowe 8 GW; OpenAI ma płacić dopiero za oddawaną capacity. Koncentracja na jednym dostawcy poprawia integrację i może skrócić time-to-token, lecz zwiększa vendor lock-in oraz ryzyko finansowe i operacyjne wspólne dla NVIDIA, SB Energy i OpenAI.

**Źródła:**
- https://openai.com/index/openai-joins-ports-pike-project/
- https://sbenergy.com/nvidia-ai-compute-ports-pike-ohio/
- https://www.reuters.com/business/media-telecom/nvidia-invest-15-billion-sb-energy-under-openai-data-center-deal-2026-08-17/

## Technologia

### DSX jako pełnostosowa architektura dla wielogigawatowego AI factory

PORTS-Pike ma wykorzystywać platformę NVIDIA DSX obejmującą GPU, CPU, sieć oraz wspólne projektowanie, testowanie i commissioning. OpenAI i NVIDIA zapowiedziały techniczny white paper o kwalifikacji komponentów, zarządzaniu workloadami, dostępności compute i wydłużaniu MTBI; chłodzenie ma być zamkniętym obiegiem z systemem air-cooled, a początkowe 800 MW ma wykorzystać w dużej części istniejącą infrastrukturę AEP.

**Analiza:** pełnostosowa kwalifikacja może poprawić reliability i ograniczyć overhead integracyjny, lecz mierzalna wartość pojawi się dopiero po publikacji danych o fabric efficiency, failure domains, repair time, utilization i energy/token. Skala wymaga hierarchicznego capacity managementu, izolacji awarii pomiędzy halami i fazami oraz SLO dla infrastruktury energetycznej, chłodzenia, backend fabric, storage i schedulerów. Portability będzie ograniczona, jeśli DSX i warstwa workload management staną się kontraktowo lub operacyjnie nierozłączne z hardware NVIDIA.

**Źródła:**
- https://openai.com/index/openai-joins-ports-pike-project/
- https://sbenergy.com/nvidia-ai-compute-ports-pike-ohio/

## Implikacje praktyczne

1. **Oddzielać końcową skalę kampusu od dostępnej capacity.** W modelach przyjmować fazy 800 MW → 4,25 GW → 8 GW, z osobnymi prawdopodobieństwami i datami energizacji.
2. **Wymagać płatności za commissioned capacity.** Wzorzec OpenAI — płatność dopiero po udostępnieniu ukończonej mocy — ogranicza koszt stranded reservations.
3. **Projektować vendor-exit na poziomie workloadów i danych.** Kontenery, scheduler, modele, checkpointy, telemetryka i dane treningowe powinny pozostać przenośne mimo pełnego stosu DSX.
4. **Traktować finansowanie jako zależność SLO.** Gwarancje, project finance, pozwolenia i dostawy energii powinny znaleźć się w rejestrze ryzyk tak samo jak GPU, optyka i chłodzenie.
5. **Weryfikować pełnostosową niezawodność przed skalowaniem.** Każdą fazę odbierać przez burn-in, job completion rate, fabric goodput, tail latency, MTTR/MTBI, energy/token i testy utraty domeny zasilania.

## Trend tygodnia

Dostawca akceleratorów coraz częściej przejmuje rolę współfinansującego i integratora całej infrastruktury od gruntu i energii po runtime. PORTS-Pike pokazuje przejście od sprzedaży GPU do zabezpieczania wieloletniego popytu poprzez gwarancje oraz pełnostosową architekturę. Zmniejsza to ryzyko integracji dla klienta, ale konsoliduje ryzyko technologiczne i finansowe w jednym ekosystemie. Najważniejszą miarą pozostaje tempo konwersji zapowiedzianych GW na odebraną i obciążalną capacity.

## To obserwować

- uruchomienie pierwszych 800 MW PORTS-Pike w 2028 r.;
- warunki i maksymalne wykorzystanie gwarancji NVIDIA;
- zamknięcie finansowania dla początkowych 4,25 GW IT;
- pozwolenia i harmonogram nowych źródeł energii oraz transmisji;
- publikację white paper NVIDIA–OpenAI o commissioning i reliability;
- rzeczywiste modele GPU, CPU i fabric w kolejnych fazach DSX;
- job completion rate, MTBI, MTTR i fabric goodput po uruchomieniu;
- PUE, zużycie wody i energy/token;
- tempo wzrostu capacity od 800 MW do 4,25 GW;
- wpływ długoterminowej wyłączności NVIDIA na ceny i portability.