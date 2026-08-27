# Raport AI-ML — 2026-08-27

**Data podsumowania:** 2026-08-27  
**Analizowany okres:** od edycji z 2026-08-26 do 2026-08-27 (Europe/Warsaw)

## Biznes

### Wyniki NVIDIA potwierdzają dalszy wzrost popytu, ale zwiększają wagę ryzyka koncentracji

NVIDIA podała za Q2 FY27 przychód 96,2 mld USD (+106% r/r), z czego 89,0 mld USD przypadło na Data Center (+117% r/r); marża brutto wyniosła 75,0%. Prognoza na Q3 to 108 mld USD ±2%, bez założonego przychodu z chińskiego rynku compute, a Vera Rubin weszła w pełną produkcję u partnerów obejmujących CoreWeave, Google Cloud, Azure, OCI i Nebius.

**Analiza architektoniczna:** Tempo wzrostu i równoległy ramp Rubin potwierdzają, że terminy dostaw GPU, pamięci i infrastruktury rackowej nadal będą elementem capacity risk. Planowanie powinno oddzielać deklarowane zamówienia od mocy rzeczywiście uruchomionej i osiągającej wymagane SLO; obserwować należy również spadek prognozowanej marży do 74%, ponieważ presja kosztowa komponentów może przełożyć się na ceny instancji i warunki rezerwacji.

**Źródło:** [NVIDIA — wyniki Q2 FY27, 2026-08-26](https://nvidianews.nvidia.com/news/nvidia-announces-financial-results-for-second-quarter-fiscal-2027)

### Zgłoszone przejęcie Hugging Face przez NVIDIA za 12,9 mld USD

Reuters, powołując się na The Information i osobę znającą transakcję, podał 27 sierpnia, że NVIDIA uzgodniła zakup Hugging Face za 12,9 mld USD. NVIDIA i Hugging Face nie potwierdziły informacji Reutersowi, dlatego na tym etapie jest to materialne, ale nieoficjalne zdarzenie.

**Analiza architektoniczna:** Kontrola nad głównym repozytorium modeli, datasetów i bibliotek rozszerzałaby lock-in NVIDIA z warstwy sprzętu i CUDA na dystrybucję artefaktów oraz developer workflow. Organizacje powinny utrzymywać niezależny registry, kopie modeli i datasetów z pinowanymi hashami, SBOM/provenance oraz możliwość uruchomienia pipeline’u bez usług Hugging Face; istotne będą warunki dostępu, neutralność wobec AMD/TPU i zmiany licencji po ewentualnym zamknięciu transakcji.

**Źródło:** [Reuters — zgłoszone przejęcie Hugging Face, 2026-08-27](https://www.reuters.com/technology/nvidia-talks-acquire-hugging-face-13-billion-deal-business-insider-reports-2026-08-27/)

## Technologia

### Cisco włącza systemy Supermicro do Secure AI Factory z NVIDIA

Od października 2026 Cisco ma oferować chłodzone cieczą i powietrzem systemy Supermicro oparte na NVIDIA NVL72, HGX i MGX. Referencyjna architektura NCP łączy Cisco Silicon One w front-endzie, przełączniki Cisco oparte na NVIDIA Spectrum-X w back-endzie oraz wspólny model Nexus One z wyborem NX-OS lub SONiC; observability ma korelować stan jobów z compute, NIC, optyką i fabric.

**Analiza architektoniczna:** To upraszcza odpowiedzialność za walidację rack-to-fabric i może skrócić integrację chłodzenia, sieci oraz serwerów, ale zwiększa sprzężenie BOM-u, firmware i supportu między trzema dostawcami. Przed zakupem należy wymagać testów p99 collective latency, PFC/ECN, incast, telemetry gap, awarii pojedynczego racka i degradacji chłodzenia oraz jasno przypisać ownership incydentów przekraczających granice server–NIC–switch.

**Źródła:** [Cisco Newsroom — rozszerzenie Secure AI Factory, 2026-08-25](https://newsroom.cisco.com/c/r/newsroom/en/us/a/y2026/m08/cisco-secure-ai-factory-nvidia-rack-scale.html) · [Cisco — FAQ architektury rack-scale](https://www.cisco.com/c/en/us/solutions/collateral/artificial-intelligence/cisco-secure-ai-factory-nvidia-rackscale-faq.html)

## Implikacje praktyczne

1. Utrzymywać własny, wersjonowany registry modeli i datasetów z hashami, podpisami, SBOM oraz procedurą odtworzenia niezależną od Hugging Face.
2. W umowach GPU rozdzielać rezerwację, datę energization, acceptance test i gwarantowany throughput; nie liczyć zapowiedzianej mocy jako dostępnej capacity.
3. Dla zintegrowanych fabryk AI wymagać jednego RACI i ścieżki eskalacji dla serwera, NIC, optyki, przełącznika, NOS i chłodzenia.
4. Porównywać backendy przez koszt przy spełnionym SLO: TTFT, inter-token latency, job completion time, tokeny/J, retry i utracony czas GPU.
5. Zachować portability na poziomie artefaktów, API i schedulerów; walidować co najmniej jedną ścieżkę awaryjną poza CUDA/NVIDIA.

## Trend tygodnia

NVIDIA rozszerza kontrolę zarówno w pionie — od GPU, sieci i racka po potencjalną dystrybucję modeli — jak i w poziomie przez partnerów integrujących kompletne fabryki AI. Rosnące przychody Data Center potwierdzają popyt, lecz jednocześnie przenoszą ryzyko z samej dostępności GPU na supply chain, energię, registry artefaktów i wieloletnie zależności kontraktowe. W praktyce wartość architektury wielodostawczej będzie zależała bardziej od przetestowanej ścieżki wyjścia niż od deklarowanej zgodności API.

## To obserwować

- oficjalne potwierdzenie, warunki regulacyjne i model zarządzania Hugging Face po ewentualnym przejęciu;
- realne terminy i yield ramp Vera Rubin oraz dostępność pamięci i optyki;
- ceny i SLA pierwszych ofert Cisco/Supermicro Secure AI Factory od października 2026;
- niezależne wyniki job completion time i tail latency dla referencyjnej architektury Nexus One/Spectrum-X;
- marżę Data Center NVIDIA oraz wpływ kosztów komponentów na ceny chmurowe i rezerwacje.
