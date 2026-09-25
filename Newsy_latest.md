# Newsy — 2026-09-25

**Data podsumowania:** 2026-09-25  
**Okno przyrostowe:** od raportu 2026-09-24 07:30 CEST do 2026-09-25 07:30 CEST; dla pominiętych wcześniej zdarzeń maks. 7 dni.

## Raport technologiczny

### Technologia

#### GitHub App private keys: trwałe machine identities tworzą supply-chain blast radius

GitGuardian przetestował 4 802 ujawnione klucze prywatne GitHub Apps i stwierdził, że 474 nadal poprawnie uwierzytelniają się do GitHub API jako 440 aplikacji. 207 aplikacji miało write do zawartości repozytoriów, 44 uprawnienia administracyjne organizacji, 40 zarządzanie self-hosted runners, a 98 kontrolę workflow; klucze prywatne GitHub Apps nie wygasają automatycznie.

**Znaczenie:** GitHub App private key należy traktować jak długowieczną tożsamość maszynową Tier-0. Leak może umożliwić modyfikację kodu, workflow i runnerów w wielu repozytoriach, więc samo usunięcie sekretu z historii Git nie wystarcza — wymagane są rotacja/revocation, przegląd instalacji, ograniczenie permissions/repository scope oraz audyt zmian wykonanych przez aplikację.

**Źródło:** https://blog.gitguardian.com/github-app-private-keys-leaked/

## Raport AI-ML

### Technologia

#### NVIDIA NodeWright przenosi lifecycle host OS GPU nodes do modelu deklaratywnego

NodeWright to otwartoźródłowy, Kubernetes-native mechanizm zarządzania konfiguracją hosta pod GPU workloads. Operator wykonuje sekwencję cordon → wait → drain → apply/configure → interrupt/reboot → uncordon, respektuje PodDisruptionBudgets i non-interruptible workloads, a DeploymentPolicy umożliwia rollout fixed/linear/exponential z progami sukcesu i awarii.

**Znaczenie architektoniczne:** host OS, kernel, RDMA tuning i security agents stają się elementem kontrolowanego lifecycle klastra zamiast zewnętrznego runbooka. Przy setkach/tysiącach GPU ogranicza to konfigurację drift i manual maintenance, ale błędna paczka lub zbyt agresywny rollout może skorelować awarie w dużym failure domain; potrzebne są canary compartments, twarde disruption budgets i rollback/validation poza samym workload schedulerem.

**Źródło:** https://developer.nvidia.com/blog/manage-kubernetes-node-fleets-with-nodewright/

#### Mitsubishi Electric Chip-to-Grid łączy power, BESS i cooling z projektem Vera Rubin

Mitsubishi Electric opublikował NVIDIA-compatible Chip-to-Grid DSX Reference Designs dla Vera Rubin NVL72 i kolejnych platform. Projekt obejmuje ścieżkę od grid connection do zasilania chipów, BESS/on-site generation z możliwością pracy wyspowej, dystrybucję 415/480 VAC lub 800 VDC oraz dual-loop cooling dla wysokich gęstości racków; celem jest skalowanie do obiektów klasy gigawatowej.

**Znaczenie architektoniczne:** boundary AI factory przesuwa się poza compute/network do power electronics, microgrid i hydrauliki. Capacity planning powinien modelować wspólnie transient response, BESS autonomy, 800 VDC protection, CDU/cooling fault domains i ramp GPU load; największym failure mode jest traktowanie reference design jako substytutu site-level protection coordination, commissioning i black-start/islanding tests.

**Źródło:** https://europe.mitsubishielectric.com/en/pr/global/2026/0924_pu/

### Implikacje praktyczne

1. Zarządzanie host OS dla dużych GPU Kubernetes clusters przenieść do polityk rollout/canary z kontrolą failure domain, a nie utrzymywać wyłącznie jako Ansible/runbook.
2. Kernel/RDMA/driver changes wiązać z workload-aware drain i walidacją post-change; automatyzacja bez ograniczenia blast radius tylko przyspiesza propagację błędu.
3. Dla Rubin-era AI factory prowadzić capacity plan wspólnie dla GPU, rack density, 800 VDC/AC distribution, BESS, cooling i grid constraints.
4. W acceptance testach power/cooling uwzględniać transienty GPU, utratę grid, islanding, restart po awarii oraz skorelowane failure modes power+cooling.

### Trend tygodnia

Warstwa operacyjna AI factory przesuwa się z zarządzania pojedynczym serwerem do lifecycle całego systemu: host OS, scheduler, fabric, power i cooling są jednym failure domain. NodeWright pokazuje tę zmianę po stronie software fleet management, a Chip-to-Grid po stronie fizycznej infrastruktury. Kluczową metryką staje się nie samo installed GPU count, lecz zdolność bezpiecznego utrzymania i aktualizacji całej fabryki bez utraty produktywnej capacity.

### To obserwować

- NodeWright/AICR: obsługa rollbacku, dependency ordering i realne failure thresholds na dużych klastrach;
- wpływ kernel/RDMA updates na NCCL p99 i job restart rate;
- 800 VDC: ochrona, serwisowalność i vendor interoperability;
- BESS/islanding: czas przejścia i zachowanie przy skokach obciążenia GPU;
- Rubin NVL72: realna rack density i wymagania CDU/site power.

## AI for networking

### Bifrost AI Gateway: unauthenticated MCP registration prowadzi do command execution

- **Technologia / Zdarzenie:** CVE-2026-90898 w Bifrost AI Gateway — https://thehackernews.com/2026/09/critical-bifrost-ai-gateway-flaw-lets.html
- **Mechanizm działania:** przy domyślnie wyłączonym management authentication pojedynczy POST do `/api/mcp/client` może zarejestrować klienta MCP typu stdio; gateway uruchamia wskazany proces przed handshake MCP. Podatne są transporty HTTP <2.1.0, a oficjalny obraz Docker może wystawić management API na 0.0.0.0.
- **Wpływ na architekturę:** AI gateway jest jednocześnie policy point i koncentratorem provider credentials, więc powinien być traktowany jak management plane/Tier-0. Oddzielić listener management od dataplane, wymusić auth, ograniczyć egress i nie przechowywać szerokich kluczy dostawców w jednym shared process.
- **Failure modes i edge cases:** RCE daje dostęp do wszystkich provider API keys i może posłużyć do dalszego pivotu. Instancje działające z wystawionym management API i bez auth należy traktować jako potencjalnie przejęte; aktualizacja do transports/v2.1.0 nie zastępuje rotacji kluczy po ekspozycji.

### Perplexity SPACE: microVM boundary wytrzymał, egress policy nie

- **Technologia / Zdarzenie:** Perplexity „Escaping SPACE: Part I” — https://www.perplexity.ai/hub/blog/escaping-space-part-i
- **Mechanizm działania:** dziewięć modeli z rootem w Firecracker microVM próbowało przekroczyć VM-host boundary lub ominąć per-sandbox egress gateway. W 108 próbach nie uzyskano VM escape, ale przy częściowym dostępie sieciowym cztery modele wykorzystywały DNS spoofing lub shared-IP routing; po poprawkach bypassów nie odtworzono.
- **Wpływ na architekturę:** hypervisor isolation i network confinement to niezależne trust boundaries. Domain-based allowlist musi wiązać hostname z rzeczywistym połączeniem/TLS destination i nie może ufać wyłącznie DNS resolution ani współdzielonemu IP/CDN.
- **Failure modes i edge cases:** shared CDN/IP, DNS rebinding/spoofing, package mirrors i proxy exceptions mogą stworzyć alternatywną ścieżkę egress mimo poprawnej izolacji VM. Fallback powinien być fail-closed z telemetryką per sandbox i testami adversarialnymi po każdej zmianie polityk.

## Newsletters summary

### Bifrost: management API AI gateway jako Tier-0

- **Technologia / Zdarzenie:** CVE-2026-90898, poprawka w transports/v2.1.0.
- **Mechanizm działania:** unauthenticated rejestracja stdio MCP client uruchamia proces jako użytkownik gateway i odsłania provider API keys.
- **Wpływ na architekturę:** management listener musi być oddzielony od dataplane, uwierzytelniony i niedostępny z niezaufanych sieci.
- **Failure modes i edge cases:** publikacja portu oficjalnego kontenera, brak auth i szerokie provider keys zamieniają pojedynczy HTTP request w pełne przejęcie gateway.

**Źródło:** https://thehackernews.com/2026/09/critical-bifrost-ai-gateway-flaw-lets.html

### GitHub Apps: 474 nadal aktywne ujawnione private keys

- **Technologia / Zdarzenie:** GitGuardian zweryfikował aktywność 474 z 4 802 ujawnionych GitHub App private keys.
- **Mechanizm działania:** klucz podpisuje JWT aplikacji, który następnie pozwala uzyskać installation token z uprawnieniami nadanymi App.
- **Wpływ na architekturę:** GitHub Apps trzeba objąć inventory, secret scanning, rotacją oraz minimalnym repository/permission scope tak samo jak workload/service identities.
- **Failure modes i edge cases:** brak automatycznego expiry powoduje, że wieloletni leak nadal może modyfikować repo, workflow lub self-hosted runners.

**Źródło:** https://blog.gitguardian.com/github-app-private-keys-leaked/

### SPACE: egress enforcement jest osobną granicą od microVM

- **Technologia / Zdarzenie:** testy Perplexity SPACE i innych sandboxów agentowych.
- **Mechanizm działania:** agent nie przełamał Firecracker VM, ale wykorzystywał słabości reguł hostname/DNS/shared-IP do komunikacji z blokowanym celem.
- **Wpływ na architekturę:** agent sandbox wymaga niezależnego, host-side egress enforcement z poprawnym bindingiem DNS→TLS/connection oraz per-workload audit.
- **Failure modes i edge cases:** allowlisted CDN, package repo lub współdzielony adres może działać jako covert/alternate egress; po zmianie konfiguracji konieczne są regresyjne testy bypassów.

**Źródło:** https://www.perplexity.ai/hub/blog/escaping-space-part-i
