# Newsy — 2026-09-22

**Data podsumowania:** 2026-09-22  
**Okno przyrostowe:** od raportu 2026-09-21 07:30 CEST do 2026-09-22 07:30 CEST; dla pominiętych wcześniej zdarzeń maks. 7 dni.

## Raport technologiczny

### Technologia

#### Cloudflare: >100 TB RAM odzyskane przez zmianę modelu consistent hashing

Cloudflare opisał optymalizację implementacji consistent hashing w usłudze opartej o Pingora/Rust. Redukcja liczby reprezentacji serwerów w strukturach routingu i zastąpienie kosztownego modelu mapowania bardziej zwartą konstrukcją dało ponad 100 TB oszczędności RAM w skali globalnego fleet.

**Znaczenie:** przy hyperscale koszt algorytmiczny struktur sterujących staje się kosztem infrastruktury. Własne systemy LB/proxy warto profilować nie tylko CPU/request, lecz również bytes/backend i bytes/route; failure mode to pogorszenie równomierności rozkładu lub większy churn przy zmianach membership, więc oszczędność pamięci musi być weryfikowana razem z remap ratio i tail latency.

**Źródło:** https://blog.cloudflare.com/saving-100-tb-of-ram-with-math/

## Raport AI-ML

### Biznes

#### Nscale ujawnia skalę ryzyka finansowania podczas IPO

Brytyjski Nscale złożył dokumentację do wejścia na NYSE. Za H1 2026 wykazał 140,6 mln USD przychodu i 1,02 mld USD straty netto; równocześnie deklaruje ponad 100 mld USD zakontraktowanej wartości i wielogigawatowy pipeline, podczas gdy aktywna/leased capacity jest nadal niewielka względem planów.

**Znaczenie architektoniczne:** kontrakty na GPU nie są równoważne dostępnej mocy obliczeniowej — execution risk leży w power, finansowaniu, budowie DC i terminowej dostawie acceleratorów. Przy wyborze neocloudu trzeba oddzielać contracted backlog od commissioned MW/GPU i wymagać SLA powiązanego z konkretnym site/cluster.

**Źródła:** https://www.investing.com/news/stock-market-news/nscale-files-for-ipo-seeks-nyse-listing-under-ticker-nscl-432SI-4907816 ; https://www.ft.com/content/93ba41c5-d777-4733-8738-93a06e8fead1

### Technologia

#### SemiAnalysis: inference trzeba modelować jako przepływ prefill → midfill → decode

Nowa analiza SemiAnalysis rozdziela inference na prefill, midfill, decode-attention i decode-experts. Kluczowy element to KV state jako przenośne, immutable blobs w warstwie współdzielonej pamięci/storage, co pozwala schedulerowi dobierać worker do fazy i SLO zamiast utrzymywać session affinity do konkretnego GPU.

**Znaczenie architektoniczne:** dla agentic workloads midfill staje się osobnym profilem capacity — długi istniejący KV cache plus relatywnie mały przyrost wejścia. Disaggregated serving może zwiększyć utilization, ale przenosi bottleneck na KV transport, shared DRAM/SSD, fabric i scheduler; p99 zależy wtedy od locality, cache admission i przepustowości sieci równie mocno jak od FLOPS GPU.

**Źródło:** https://newsletter.semianalysis.com/p/computation-and-data-movement-for

### Implikacje praktyczne

1. Capacity planning inference rozdzielać co najmniej na prefill, midfill i decode; jedna średnia tokens/s ukrywa różne bottlenecki.
2. Projektować KV cache jako współdzielony resource domain z telemetryką hit-rate, bytes/request, p99 fetch i kosztu migracji między workerami.
3. Przy kontraktach neocloud wymagać danych commissioned MW/GPU per site, a nie opierać sizingu na backlogu lub planowanej mocy.
4. Przy disaggregated serving testować awarie storage/fabric: utrata KV tier może degradować cały inference mimo zdrowych GPU.

### Trend tygodnia

Inference przesuwa się z modelu „GPU server” do wielowarstwowej fabryki tokenów. Scheduler, KV cache, DRAM/SSD i fabric zaczynają determinować utilization oraz p99 równie silnie jak sam accelerator. Jednocześnie szybka ekspansja neocloudów zwiększa różnicę między zakontraktowaną a fizycznie uruchomioną capacity.

### To obserwować

- p95/p99 midfill oraz KV-cache transfer bandwidth;
- commissioned vs contracted MW u europejskich neocloudów;
- koszt storage/DRAM na aktywną sesję agentic;
- disaggregated prefill/decode w Dynamo, Mooncake, vLLM i SGLang;
- realny time-to-service nowych klastrów B200/B300/Rubin.

## euro neocloud

### Nscale — IPO ujawnia wysoką kapitałochłonność i concentration/execution risk

**Fakty:** Nscale podał 140,6 mln USD przychodu i 1,02 mld USD straty netto w H1 2026. Spółka rozwija około 1,3 GW projektów i deklaruje ponad 100 mld USD wartości kontraktów, ale obecna uruchomiona/leased capacity pozostaje niewielka względem pipeline.

**Ocena analityczna:** sygnały ostrzegawcze to duża luka między contracted value a rozpoznanym przychodem, szybkie zużycie kapitału, zależność od finansowania kolejnych DC oraz koncentracja dużych umów na kilku odbiorcach. **Ocena ryzyka: Wysokie** — nie z powodu popytu na GPU, lecz execution/financing risk pomiędzy kontraktem a commissioned capacity.

**Źródła:** https://www.investing.com/news/stock-market-news/nscale-files-for-ipo-seeks-nyse-listing-under-ticker-nscl-432SI-4907816 ; https://www.ft.com/content/93ba41c5-d777-4733-8738-93a06e8fead1

## Newsletters summary

### SemiAnalysis: KV state jako współdzielony obiekt infrastrukturalny

- **Technologia / Zdarzenie:** „Computation and Data Movement for Inference”.
- **Mechanizm działania:** prefill/midfill/decode są rozdzielane na worker pools, a KV state jest przenoszony jako immutable blobs przez shared DRAM/storage.
- **Wpływ na architekturę:** placement może być oparty o aktualną fazę i SLO zamiast session affinity; fabric i storage stają się elementem krytycznej ścieżki inference.
- **Failure modes i edge cases:** KV miss, przeciążenie shared tier, hotspoty, długi context i koszt migracji mogą zwiększać p99 mimo wolnych GPU.

### Codex: Heapjack/Overpatch pokazują błędną granicę sandboxu

- **Technologia / Zdarzenie:** ujawniono dwa naprawione sandbox escapes Codex; jeden działał również w trybie read-only.
- **Mechanizm działania:** Heapjack odzyskiwał trust token ze współdzielonego V8 heap, a Overpatch wykorzystywał logikę uprawnień apply_patch do zapisu poza workspace.
- **Wpływ na architekturę:** coding agent powinien działać w izolacji egzekwowanej poza procesem/agentycznym runtime — VM/microVM, osobny egress proxy, brak host credentials i krótkotrwałe workload identities.
- **Failure modes i edge cases:** repo jako hostile input, token leakage, symlink/path traversal i persistent host modification. Minimalne wskazane wersje poprawek: Codex CLI 0.149.0 i Desktop 26.818.21641.

### Plugin4Shell: SHA pinning nieskuteczne, jeśli agent sam rozwiązuje repo

- **Technologia / Zdarzenie:** Plugin4Shell dotyczy Claude Code, Codex, GitHub Copilot i Gemini CLI; repozytorium kontrolowane przez atakującego może podmienić kod mimo zatwierdzonego SHA.
- **Mechanizm działania:** walidacja pinning i pobranie pluginu nie tworzyły jednej zewnętrznie egzekwowanej granicy zaufania; zmiana default branch pozwalała agentowi pobrać inny kod niż oczekiwany.
- **Wpływ na architekturę:** plugin marketplace nie może być root of trust, jeśli runtime sam interpretuje referencję. Potrzebny immutable artifact digest, registry/proxy kontrolowany poza agentem oraz allowlista egress.
- **Failure modes i edge cases:** auto-update pluginów i przejęcie upstream repo zamieniają supply-chain compromise w zero-click execution. Według AIR poprawki są w Claude Code 2.1.179 i Codex 0.146.0; dla wskazanych wersji Copilot/Gemini CLI pełnej poprawki nie było w momencie publikacji.

### TLDR AI: AX — agent workload jako izolowany workload klastra

- **Technologia / Zdarzenie:** Google AX deklaruje uruchamianie bardzo dużej liczby autonomicznych zadań agentowych nad Agent Substrate.
- **Mechanizm działania:** task deklaruje workspace i gateway; runtime sandboxuje wykonanie, podłącza workspace i ogranicza sieć.
- **Wpływ na architekturę:** agent orchestration zaczyna przypominać Kubernetes, ale jednostką schedulingu jest długotrwały, stanowy i sieciowo aktywny agent; potrzebne quota, workload identity, egress policy i per-task audit.
- **Failure modes i edge cases:** agent retry storms, niekontrolowany fan-out, kosztowne tool loops, credential propagation i przeciążenie gateway/control plane.
