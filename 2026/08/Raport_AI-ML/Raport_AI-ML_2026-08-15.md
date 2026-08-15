# Raport AI-ML — 2026-08-15

**Okres analizy:** 2026-08-09 – 2026-08-15 (Europe/Warsaw)

## Biznes

### NVIDIA i Wall Street budują platformy finansowania compute o skali >500 mld USD
NVIDIA współpracuje z dużymi instytucjami finansowymi, w tym Apollo, BlackRock, Blackstone, Brookfield, Goldman Sachs i KKR, nad strukturami finansowania infrastruktury AI, które mogą uruchomić ponad 500 mld USD kapitału. To przesuwa koszt infrastruktury AI z klasycznego CAPEX IT w stronę project finance / infrastructure finance, ale jednocześnie zwiększa ryzyko niedopasowania okresu finansowania do ekonomicznego cyklu życia generacji GPU.

**Znaczenie architektoniczne:** większa dostępność finansowania ułatwi budowę wielogigawatowych klastrów, lecz może utrwalać vendor lock-in wokół NVIDIA oraz wymuszać wieloletnie zobowiązania capacity/offtake. Przy planowaniu 3–5 lat należy modelować spadek wartości sprzętu, refresh GPU, koszty energii i ryzyko niższego wykorzystania.

**Źródła:**
- https://www.ft.com/content/4c93c894-04b8-49dc-be41-98ae79f540f8
- https://www.reuters.com/business/nvidia-scales-back-250-billion-openai-data-center-guarantee-wsj-reports-2026-08-14/

### CoreWeave potwierdza dalszy wzrost backlogu i presję na skalowanie infrastruktury
Po wynikach za Q2 2026 rynek zwrócił uwagę na backlog przekraczający 100 mld USD i dalsze podnoszenie planów inwestycyjnych. Skala kontraktów poprawia widoczność popytu, ale jednocześnie wzmacnia zależność modelu biznesowego od szybkiego oddawania nowych MW, dostaw GPU, długu i wykorzystania zakontraktowanej mocy.

**Znaczenie architektoniczne:** capacity planning powinien uwzględniać nie tylko liczbę GPU, ale także dostępność energii, chłodzenia, sieci backend i czasu integracji nowych generacji. Przy korzystaniu z neoclouda trzeba monitorować jego leverage, terminy uruchamiania regionów oraz koncentrację klientów, bo mogą bezpośrednio wpływać na SLO i dostępność capacity.

**Źródła:**
- https://www.reuters.com/podcasts/reuters-morning-bid/banking-coreweave-2026-08-12/
- https://investors.coreweave.com/

## Technologia

### IBM i Together AI budują klaster inferencyjny HGX B300 + Spectrum-X
IBM i Together AI podpisały wieloletnią umowę o wartości 240 mln USD na klaster inferencyjny w IBM Cloud. Początkowa konfiguracja ma obejmować około 2 tys. GPU Blackwell 300 w systemach HGX B300 oraz sieć NVIDIA Spectrum-X Ethernet, z naciskiem na inference modeli otwartych.

**Analiza:** dla inference kluczowe stają się efektywność energetyczna, memory bandwidth, skalowanie poziome i koszt tokena, a nie wyłącznie peak FLOPS. Wybór Spectrum-X pokazuje dalsze przesuwanie dużych wdrożeń AI w stronę Ethernetu z mechanizmami congestion control/RDMA; organizacje powinny porównywać tę ścieżkę z InfiniBand pod kątem operacyjności, observability i przenośności.

**Źródło:** https://www.reuters.com/business/ibm-together-ai-ink-240-million-deal-nvidia-powered-ai-inference-cluster-2026-08-11/

### Cisco: skok zamówień na sieci AI zwiększa znaczenie Ethernetu dla klastrów GPU
Wyniki Cisco za Q4 FY2026 wskazują około 4 mld USD zamówień AI w kwartale i około 9,3 mld USD w całym roku fiskalnym. Wzrost jest silnie związany z data-center switchingiem i rozbudową sieci pod obciążenia AI.

**Analiza:** rynek backend/fabric nie jest już jednoznacznie domeną InfiniBand. Przy projektowaniu nowych klastrów trzeba utrzymywać równoległe kompetencje w RoCEv2, ECN/PFC, telemetryce kolejek, buffer management oraz automatyzacji fabric; same port-speed i oversubscription ratio nie wystarczają jako wskaźniki jakości sieci AI.

**Źródła:**
- https://investor.cisco.com/financials/quarterly-results/default.aspx
- https://www.wsj.com/business/earnings/cisco-reports-higher-fourth-quarter-profit-as-ai-orders-roll-in-e82db3b4

## Implikacje praktyczne

1. **Nie wiązać 5-letniego modelu finansowego z jedną generacją GPU.** Umowy capacity, leasing i project finance powinny mieć mechanizmy refresh/remarketing i scenariusze spadku wartości B300/GB300 po wejściu kolejnych generacji.
2. **Projektować fabric jako osobną platformę operacyjną.** Dla Ethernet/RoCE wymagane są SLO dla congestion, packet loss, ECN marking, PFC pause i tail latency oraz telemetryka per-port/per-queue.
3. **Rozdzielać kontrakt na compute od gwarancji dostępności energii i DC.** MW zakontraktowane nie oznaczają MW aktywnych; capacity planning powinien śledzić energization date, cooling readiness i harmonogram dostaw GPU.
4. **Utrzymywać portability na poziomie orkiestracji i runtime.** Kubernetes/Ray/KServe/vLLM oraz otwarte warstwy storage/observability zmniejszają koszt migracji między hyperscalerem, neocloudem i on-prem.
5. **Wprowadzić FinOps dla inference na poziomie tokenów i GPU-sekund.** Monitorować utilization, batch size, KV-cache efficiency, queueing latency, energy/token oraz koszt rezerwacji niewykorzystanej capacity.

## Trend tygodnia

Kapitał i infrastruktura AI coraz silniej łączą się w jeden rynek: dostawca GPU, operator data center, neocloud i instytucja finansowa zaczynają współdzielić ryzyko budowy capacity. Jednocześnie inference staje się wystarczająco duży, aby uzasadniać dedykowane klastry B300 i rozbudowane sieci Ethernet/RoCE. Dla architektów oznacza to przejście od wyboru pojedynczej technologii do zarządzania pełnym łańcuchem zależności: finansowanie → energia → chłodzenie → GPU → fabric → runtime → SLO. Największym ryzykiem 3–5 lat nie jest już samo niedostarczenie GPU, lecz niezgodność czasu życia infrastruktury, kontraktów energetycznych i ekonomiki obciążeń.

## To obserwować

- uruchamianie produkcyjnej dostępności HGX B300/GB300 i realne wyniki inference per watt;
- tempo adopcji Spectrum-X i innych Ethernet fabrics względem InfiniBand;
- ceny długoterminowych rezerwacji B300/GB300 w hyperscalerach i neocloudach;
- koszt finansowania infrastruktury AI i warunki zabezpieczeń na GPU/data center;
- aktywne MW vs zakontraktowane MW u największych neocloudów;
- wykorzystanie GPU oraz udział inference w przychodach operatorów;
- rozwój vLLM, Triton i innych runtime'ów pod multi-node inference;
- standardy telemetryki RoCE/ECN/PFC i automatyzacji capacity/SLO;
- różnicę kosztu tokena między NVIDIA, AMD i custom ASIC w dużej skali.