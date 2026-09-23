# Newsy — 2026-09-23

**Data podsumowania:** 2026-09-23  
**Okno przyrostowe:** od raportu 2026-09-22 07:30 CEST do 2026-09-23 07:30 CEST; dla pominiętych wcześniej zdarzeń maks. 7 dni.

## Raport technologiczny

### Technologia

#### VMware wycofuje distributed firewall z offloadu SmartNIC

VMware/Broadcom przestał sprzedawać wariant Distributed Firewall działający na SmartNIC; Distributed Services Engine pozostaje elementem VCF. Powodem była niska adopcja oraz koszt integracji/microcode dla heterogenicznych DPU/NIC, przy jednoczesnym wzroście możliwości klasycznych NIC.

**Znaczenie:** dla private cloud DPU nie powinno być traktowane jako domyślna warstwa enforcementu bez potwierdzonego lifecycle konkretnego vendor+NIC. Failure mode to zależność polityk bezpieczeństwa od funkcji sprzętowej, którą producent może zdeprecjonować szybciej niż cykl życia klastra; projektować fallback do software dataplane i mierzyć CPU tax.

**Źródło:** https://www.theregister.com/virtualization/2026/09/21/vmware-has-quietly-walked-back-its-smartnic-ambitions/5297654

## Raport AI-ML

### Biznes

#### Accelevation wycenia IPO do 5,37 mld USD

Dostawca infrastruktury zasilania, chłodzenia i modułowych systemów DC chce pozyskać do 720 mln USD przy wycenie do 5,37 mld USD. To kolejny sygnał, że kapitał AI przesuwa się z samych acceleratorów w stronę fizycznego supply chain power/cooling.

**Znaczenie architektoniczne:** capacity planning trzeba rozszerzyć o dostępność CDU, switchgear, busway i prefabrykowanych modułów; GPU delivery bez równoległego facility BOM nie oznacza time-to-service. Wąskie gardła infrastruktury elektrycznej i cieplnej mogą determinować harmonogram bardziej niż lead time serwerów.

**Źródło:** https://www.reuters.com/technology/accelevation-backers-aim-raise-720-million-us-ipo-2026-09-22/

### Technologia

#### NVIDIA DSX Ready formalizuje kwalifikację BESS i CDU

NVIDIA uruchomiła DSX Ready: program kwalifikacji komponentów infrastruktury AI factory względem wymagań DSX. Pierwsze kategorie to BESS oraz CDU; kwalifikowane są m.in. rozwiązania Hitachi Energy, LG Energy Solution, Tesla, LG Electronics, LiquidStack i Vertiv. NVIDIA wyraźnie zaznacza, że kwalifikacja komponentu nie zastępuje site-level engineering ani nie gwarantuje stabilności całego obiektu.

**Znaczenie architektoniczne:** power i cooling stają się częścią referencyjnej architektury accelerator platform, co może skrócić integrację, ale zwiększa ecosystem coupling. W procurement należy rozdzielać „qualified component” od walidacji hydraulic loop, transient response, redundancy, controls integration i site fault domains.

**Źródło:** https://blogs.nvidia.com/blog/dsx-ready-ai-factories-power-cooling/

#### Ekonomia 1 GW: koszt kapitału dominuje nad energią

Analiza z newslettera The Gigawatt Economy, oparta m.in. na modelu Epoch AI, przyjmuje dla hipotetycznego 1 GW IT opartego o GB200 NVL72 około 38 mld USD CAPEX: ok. 21,2 mld USD serwery, 4,9 mld USD networking i 11,8 mld USD facility/land/utility works. Roczny ownership cost modelowany jest na ok. 8,5 mld USD, podczas gdy energia to ok. 0,6 mld USD; najtrudniejszy do pozyskania zasób nie musi więc być największą pozycją TCO.

**Znaczenie architektoniczne:** podstawową metryką ekonomiczną staje się produktywne wykorzystanie drogiego IT w ramach dostępnego MW. Oversizing power bez wysokiego GPU utilization nie poprawia ekonomiki; z kolei opóźnione przyłącze zamraża kapitał w sprzęcie o krótkim cyklu amortyzacji.

**Źródło:** newsletter The Business Engineer / The AI Supercycle, 2026-09-23; model bazowy: https://epoch.ai/

### Implikacje praktyczne

1. Do BOM AI factory włączyć kwalifikowane CDU/BESS, ale utrzymać niezależny site acceptance test dla transientów, hydrauliki, sterowania i failover.
2. Capacity plan prowadzić równolegle w GPU, MW-IT, MW-facility, rack density oraz CDU/switchgear lead time; żadna pojedyncza wartość MW nie opisuje realnej capacity.
3. W modelu TCO śledzić GPU utilization i time-to-service jako ryzyko kapitałowe; koszt energii może być wtórny wobec amortyzacji acceleratorów.
4. Przy wielogigawatowych projektach oddzielać operating, under-construction, power-secured i announced capacity.

### Trend tygodnia

AI infrastructure przechodzi z optymalizacji serwera do optymalizacji całej fabryki: accelerator, fabric, KV/storage, zasilanie i chłodzenie są jednym systemem capacity. NVIDIA rozszerza własny reference envelope na BESS/CDU, a rynek kapitałowy wycenia firmy dostarczające fizyczne elementy power/cooling. Jednocześnie koszt niewykorzystanego accelerator CAPEX rośnie szybciej niż znaczenie samej ceny kWh.

### To obserwować

- DSX Ready: kolejne kategorie poza BESS/CDU;
- commissioned MW-IT vs announced MW u hyperscalerów i neocloudów;
- lead time CDU, switchgear, transformerów i BESS;
- GPU utilization / tokens-per-MW zamiast samego installed GPU count;
- time-to-power względem czasu amortyzacji B200/B300/Rubin.

## AI for networking

### Fastly: runtime control i firewall dla ruchu AI/agentów

- **Technologia / Zdarzenie:** Fastly AI Runtime Control i AI Firewall — https://investors.fastly.com/news-releases/news-release-details/fastly-launches-ai-firewall-and-ai-runtime-control-secure-and
- **Mechanizm działania:** enforcement jest przenoszony do edge/data plane: polityki mogą kontrolować dostęp do modeli, usage/cost oraz interakcje agentów z enterprise API, łącząc routing i security controls w jednej warstwie.
- **Wpływ na architekturę:** AI gateway staje się policy enforcement point podobnym do API gateway/SASE; pozwala centralizować model routing i governance bez implementacji kontroli w każdej aplikacji.
- **Failure modes i edge cases:** centralizacja tworzy shared failure domain i dodatkowy hop w krytycznej ścieżce inference. Należy testować fail-open/fail-closed, timeout budgets, streaming, tool-call chains, policy cache consistency oraz zachowanie przy awarii control plane.

## Newsletters summary

### Docker Sandboxes: virtio-fs i socket relay naruszały granicę workspace

- **Technologia / Zdarzenie:** CVE-2026-77179 i CVE-2026-79994, poprawione w Docker Sandboxes 0.42.0+.
- **Mechanizm działania:** guest mógł wykorzystać zmianę ścieżki/symlink pomiędzy walidacją a użyciem; na macOS virtio-fs pozwalał wyjść poza workspace, a analogiczny TOCTOU w relay mógł przekierować hosta do nieautoryzowanego AF_UNIX socket.
- **Wpływ na architekturę:** VM boundary nie wystarcza, jeżeli host-side file/socket broker interpretuje mutowalne pathname. Autoryzacja powinna wiązać się z uchwytem/obiektem, a nie ponownie rozwiązywaną ścieżką.
- **Failure modes i edge cases:** hostile repo lub przejęty coding agent może przejść z workspace do host credentials/code execution. Minimalizować RW mounts, stosować clone mode i aktualizować do >=0.42.0.

**Źródło:** https://thehackernews.com/2026/09/critical-docker-sandboxes-flaw-lets.html

### VMware SmartNIC: hardware offload bez adopcji nie utrzymał lifecycle

- **Technologia / Zdarzenie:** VMware wycofał sprzedaż distributed firewall dla SmartNIC.
- **Mechanizm działania:** funkcje DFW były offloadowane do DPU/SmartNIC, wymagając ścisłej integracji microcode z konkretnymi platformami sprzętowymi.
- **Wpływ na architekturę:** enterprise private cloud nie skopiował automatycznie hyperscale DPU modelu; hardware offload wymaga wystarczającej skali, stabilnego ecosystemu i wyraźnego CPU/TCO gain.
- **Failure modes i edge cases:** vendor deprecation, nierówna obsługa kart, upgrade coupling firmware-hypervisor-security policy oraz brak równoważnego fallbacku.

**Źródło:** https://www.theregister.com/virtualization/2026/09/21/vmware-has-quietly-walked-back-its-smartnic-ambitions/5297654
