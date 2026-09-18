# Newsy — 2026-09-18

**Data podsumowania:** 2026-09-18  
**Okno przyrostowe:** od raportu 2026-09-17 do 2026-09-18 07:30 CEST

## Raport technologiczny

### Technologia

#### Cisco ISE: aktywnie wykorzystywany auth bypass CVSS 10.0

Cisco opublikowało 16 września CVE-2026-76460 w ISE/ISE-PIC: nieuwierzytelniony zdalny atakujący może ominąć uwierzytelnienie API; CVSS 10.0, brak workaroundu, Cisco potwierdza aktywne wykorzystanie. Równoległy wrześniowy hardening ISE obejmuje dodatkowe klasy błędów REST/RCE/SQLi/XXE, więc traktowanie poprawki jako pojedynczego hotfixu jest niewystarczające.

**Znaczenie:** ISE jest elementem NAC/AAA i policy control plane; kompromitacja może podważyć zaufanie do tożsamości i segmentacji w całej domenie. Priorytet: upgrade do fixed release, ograniczenie reachability API/GUI do management plane, rotacja sekretów po podejrzeniu kompromitacji i weryfikacja zmian policy/config.

**Źródła:** [Cisco — CVE-2026-76460](https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-ISE-ABP-VNSW7Tn5.html), [Cisco — ISE hardening September 2026](https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-hardening-ise-XU5EwX5T.html)

## Raport AI-ML

### Technologia

#### Google Agent Substrate: osobny data plane dla masowych sandboxów agentów

Google udostępnił Agent Substrate na GKE: runtime rozdziela lifecycle sandboxów od standardowego Kubernetes Pod lifecycle, używa pre-warmed workers, snapshot/resume oraz microVM Cloud Hypervisor lub gVisor. Deklarowane parametry to <500 ms resume, >500 aktywacji suspend/resume/s i >1000 uśpionych agentów/host, przy snapshotach lokalnych i w Cloud Storage.

**Znaczenie architektoniczne:** Przy dużej liczbie agentów bottleneck przesuwa się z inference na churn środowisk wykonawczych, storage snapshotów i credential/egress isolation. Rozdzielenie K8s jako machine control plane od lokalnego high-frequency sandbox data plane ogranicza presję na API server/scheduler; failure modes obejmują storm resume, hotspot lokalnego dysku/GCS, stale snapshots oraz błędną politykę egress/credential injection.

**Źródło:** [Google Cloud, 15.09.2026](https://cloud.google.com/blog/products/containers-kubernetes/agent-substrate-available-on-gke)

### Implikacje praktyczne

1. Dla agent platforms mierzyć osobno inference latency i sandbox activation latency; standardowy Pod-per-turn nie skaluje się ekonomicznie do masowych, głównie idle agentów.
2. Credential injection przenieść poza filesystem/ENV widoczny dla agenta i wymuszać egress policy na gatewayu.
3. Capacity plan obejmować IOPS i przepustowość snapshot store oraz storm resume po awarii workerów, nie tylko CPU/RAM/GPU.
4. Przy wyborze runtime porównać microVM vs gVisor pod kątem kompatybilności syscalli, density i blast radiusu.

### Trend tygodnia

Warstwa wykonawcza agentów zaczyna być osobnym typem infrastruktury, a nie wariantem klasycznego stateless Kubernetes. Kluczowe stają się szybkie suspend/resume, aktywne-only compute, izolacja kernela i kontrolowany egress. To przesuwa optymalizację z liczby podów na koszt aktywnego czasu oraz przepustowość lifecycle sandboxów.

### To obserwować

- GA i ograniczenia produkcyjne Agent Substrate poza GKE;
- realny p95/p99 resume przy stormie i cold storage;
- IOPS/GCS cost per 1M agent-turns;
- gVisor vs microVM density i syscall compatibility;
- awarie credential-injection/egress gateway jako wspólnego control point.

## euro neocloud

### Nebius

#### Druga podwyżka cen GPU on-demand w trzy miesiące

Nebius potwierdził 17 września podwyżki pay-as-you-go od 1 października: wybrane GPU NVIDIA drożeją o 17–21%, CPU-only do 25%, a memory offerings około 41%. B300 ma kosztować około 9,50 USD/GPU-h; źródła rynkowe wskazują około 56% wzrostu tej stawki w mniej niż pół roku. Duże wielomiesięczne klastry nadal mogą otrzymywać commitment discounts.

**Ocena analityczna:** Sygnał ostrzegawczy nie dotyczy płynności operatora, lecz dostępności capacity i przewidywalności kosztu spot/on-demand. **Ocena ryzyka: Średnie** — dla kontraktów committed ryzyko jest niższe, ale workloady burstowe i DR oparte na on-demand wymagają ponownego modelowania TCO i limitów budżetowych; kolejna podwyżka zwiększa wartość multi-provider portability.

**Źródła:** [Reuters, 17.09.2026](https://www.aol.com/articles/nebius-hikes-ai-cloud-prices-161227000.html), [Nebius pricing coverage](https://www.thenew.money/article/nebius-is-raising-on-demand-gpu-prices)

## Newsletters summary

### Marimo: osiem sekund od RCE do bastionu

- **Technologia / Zdarzenie:** [Sysdig — machine-speed exploitation CVE-2026-39987](https://www.sysdig.com/blog/machine-speed-hold-the-ai-hand-rolled-marimo-cve-2026-39987-exploit)
- **Mechanizm działania:** Pre-auth WebSocket `/terminal/ws` dawał PTY shell; operator pobrał cloud credentials, odczytał SSH key z AWS Secrets Manager i wykonał pivot na bastion w osiem sekund.
- **Wpływ na architekturę:** Notebook/AI-dev hosts należy traktować jak privileged cloud entry point. Sekrety dostępne z runtime i szeroki east-west reachability redukują czas obrony praktycznie do zera.
- **Failure modes i edge cases:** Detekcja oparta na fingerprintach narzędzi lub prompt-injection traps nie wystarcza; potrzebne są short-lived credentials, workload identity, egress controls i detekcja sekwencji API→Secrets Manager→SSH.

### Parallels Desktop: lokalny proces do root przez appliance installer

- **Technologia / Zdarzenie:** [JFrog — CVE-2026-90894](https://jfrog.com/blog/parallels-desktop-turns-appliance-install-into-root-shell/)
- **Mechanizm działania:** `prl_disp_service` działa jako root i udostępnia world-writable socket; słabe uwierzytelnienie klienta plus argument injection do `tar --use-compress-program` pozwala uruchomić kod jako root.
- **Wpływ na architekturę:** Mac workstation używany do administracji/CI powinien mieć Parallels >=27.0.1 i ograniczony lokalny software supply chain; hypervisor desktopowy jest częścią privileged attack surface.
- **Failure modes i edge cases:** EDR nie usuwa podatnego trust boundary; kompromitowany package lub job CI działający jako zwykły user może eskalować bez interakcji administratora.

### Workspace MCP: third-party connectors jako domyślny kanał danych

- **Technologia / Zdarzenie:** Gemini w Workspace otrzymuje MCP integrations z Salesforce, HubSpot, Asana, Monday, QuickBooks, Mailchimp i Atlassian Rovo; według newslettera third-party connectors są domyślnie włączone dla użytkowników z dostępem Gemini.
- **Mechanizm działania:** Agent uzyskuje tool access do zewnętrznych SaaS bezpośrednio z Gmail/Docs/Drive/Sheets/Chat; kontrola administracyjna może być stosowana per domain/OU/group.
- **Wpływ na architekturę:** MCP staje się nową warstwą egress/data-access policy. Inventory connectorów, scopes, OAuth grants i audyt tool calls powinny wejść do standardowego IAM/DLP governance.
- **Failure modes i edge cases:** Default-on zwiększa ryzyko shadow integrations, excessive scopes i cross-SaaS data propagation; prompt injection w dokumencie/mailu może wywołać narzędzie mające szersze uprawnienia niż sam kontekst użytkownika.
