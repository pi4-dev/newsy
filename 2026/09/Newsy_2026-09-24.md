# Newsy — 2026-09-24

**Data podsumowania:** 2026-09-24  
**Okno przyrostowe:** od raportu 2026-09-23 07:30 CEST do 2026-09-24 07:30 CEST; dla pominiętych wcześniej zdarzeń maks. 7 dni.

## Raport technologiczny

### Technologia

#### Management plane pozostaje głównym celem ataków na infrastrukturę

Eclypsium InfraTrust za okres 25.08–17.09 zebrał 158 nowych advisories u 17 vendorów, obejmujących 1699 CVE; 42 advisories były critical, 71 umożliwiało zdalny atak bez uwierzytelnienia, a pięć trafiło do CISA KEV. Materialny wzorzec to koncentracja podatności w systemach zarządzających — FMC/ISE, Fabric Composer, SD-WAN Orchestrator, UFM, SmartFabric Manager czy NSM — czyli komponentach posiadających credentiale i ścieżkę zmian do całej infrastruktury.

**Znaczenie:** management plane trzeba traktować jak Tier-0: osobna strefa/OOB, allowlist management sources, MFA/PAM, brak ekspozycji Internet, telemetry/audit poza zarządzanym systemem i możliwość szybkiego odtworzenia. Kompromitacja kontrolera ma blast radius większy niż pojedynczego switcha/firewalla; patch SLA dla management plane powinno być krótsze niż dla dataplane.

**Źródło:** https://www.bleepingcomputer.com/news/security/infratrust-report-warns-network-management-systems-under-attack/

## Raport AI-ML

### Technologia

#### ClusterMAX 3.0 przesuwa baseline GPU cloud na Blackwell/MI355X i 800G/XDR

SemiAnalysis rozszerzył ClusterMAX z 209 do 323 obserwowanych GPU cloudów, z 77 dostawcami objętymi pogłębioną oceną. Test referencyjny wymaga 32 GPU, preferuje B200/B300/GB200/GB300 lub MI355X, 800G RoCE/XDR InfiniBand, co najmniej 10 TB wydajnego POSIX/RWX storage i 10 TB S3; osobno testowane są audit/configuration, microbenchmarks, workload performance, lifecycle, reliability i fault tolerance.

**Znaczenie architektoniczne:** procurement GPU cloud powinien przejść z „GPU SKU + cena/h” na powtarzalny acceptance test całego klastra. Dla 32+ GPU bottleneckiem może być fabric/storage/scheduler, a nie accelerator; wymagane są testy goodput, collective tail latency, storage contention, node replacement i recovery pod Slurm/K8s. Vendor deklarujący Blackwell bez 800G/XDR i operacyjnego fault-domain modelu nie jest równoważny produkcyjnemu AI cluster.

**Źródło:** https://newsletter.semianalysis.com/p/clustermax-30-the-industry-standard

### Implikacje praktyczne

1. W RFP dla GPU cloud dodać własny pre-production acceptance suite: NCCL/RCCL, storage RWX/POSIX, failure injection, node replacement i scheduler recovery.
2. Traktować 800G RoCE/XDR jako nowy punkt odniesienia dla dużych Blackwell/MI355X deploymentów, ale mierzyć application goodput zamiast nominalnego bandwidth.
3. Wymagać firmware/software inventory oraz spójności wersji jako części odbioru; heterogeniczne firmware jest częstym źródłem tail latency i niestabilności collectives.
4. Oddzielać dostępność GPU od dostępności kompletnego klastra: accelerator inventory bez fabric/storage/operations nie jest realną capacity.

### Trend tygodnia

Rynek GPU cloud dojrzewa z prostego wynajmu acceleratorów do oceny całego systemu. Baseline przesuwa się z H100 na Blackwell/MI355X oraz z 400G w stronę 800G/XDR. Coraz większa część różnicy między dostawcami wynika z operacji, niezawodności, security, storage i network goodput, a nie z samego GPU SKU.

### To obserwować

- 800G RoCE vs XDR InfiniBand: application goodput i p99 collective latency;
- czas naprawy/reprovisioningu uszkodzonego node'a w Slurm/K8s;
- liczba dostawców z produkcyjnym B300/GB300/MI355X;
- security isolation management plane i tenant fabric;
- realna dostępność RWX/POSIX storage przy równoległym treningu.

## euro neocloud

### Nebius — wejście do najwyższej klasy ClusterMAX

**Fakty:** ClusterMAX 3.0 wskazuje materialną poprawę Nebius w ocenie kompletnego GPU cloudu; test obejmuje nie tylko GPU, ale networking, storage, orchestration, monitoring, reliability i security. Jest to istotniejsze niż sama deklarowana liczba GPU, ponieważ metodologia używa rzeczywistych 32-GPU klastrów i testów Slurm/K8s.

**Ocena analityczna:** sygnał operacyjny jest pozytywny, ale nie eliminuje ryzyka szybkiej rozbudowy capacity i zależności od dostaw Blackwell/power. **Ocena ryzyka: Średnie** — poprawa jakości platformy zmniejsza execution risk, lecz pozostaje ryzyko capacity ramp, CAPEX i koncentracji technologicznej NVIDIA.

**Źródła:** https://newsletter.semianalysis.com/p/clustermax-30-the-industry-standard ; https://clustermax.semianalysis.com/

## Newsletters summary

### ClusterMAX 3.0: GPU cloud należy odbierać jak system, nie SKU

- **Technologia / Zdarzenie:** SemiAnalysis ClusterMAX 3.0 — https://newsletter.semianalysis.com/p/clustermax-30-the-industry-standard
- **Mechanizm działania:** audyt klastra poprzedza obciążenie i sprawdza hardware inventory, firmware/software, GPU access, containers, scheduler, networking, storage, monitoring i security; następnie wykonywane są testy wydajności, reliability i fault tolerance.
- **Wpływ na architekturę:** daje wzorzec acceptance testing dla neocloudu/on-prem AI factory. Pozwala wykryć klastry, które nominalnie mają ten sam GPU SKU, lecz różnią się goodputem, stabilnością collectives, storage i operacyjnością.
- **Failure modes i edge cases:** krótki benchmark może nie ujawnić sporadycznych fabric faults, thermal throttling, noisy-neighbor ani problemów występujących dopiero przy setkach/tysiącach GPU; własne soak/failure-injection tests nadal są konieczne.
