# Newsy — 2026-09-17

**Data podsumowania:** 2026-09-17  
**Okno przyrostowe:** od raportu 2026-09-16 do 2026-09-17 07:30 CEST

## Raport technologiczny

### Technologia

#### Cisco FMC: dwa krytyczne RCE bez obejścia

Cisco opublikowało 16 września dwa krytyczne problemy FMC. CVE-2026-20242 (CVSS 9.8) umożliwia nieuwierzytelnionemu atakującemu zdalne wykonanie poleceń jako root przez External Database Access; CVE-2026-20324 (CVSS 9.9) pozwala zarejestrowanemu lub przejętemu peerowi `sftunnel` zapisać arbitralny plik i osiągnąć root RCE. Cisco nie podaje workaroundów.

**Znaczenie:** FMC jest control plane'em całej domeny firewalli, więc kompromitacja ma blast radius znacznie większy niż pojedynczy sensor. Priorytetem jest upgrade FMC, ograniczenie reachability interfejsów zarządzających i sftunnel, przegląd peerów oraz hunting pod kątem nietypowych zapisów plików i procesów uruchamianych przez usługi FMC.

**Źródła:** [Cisco — CVE-2026-20242](https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-fmc-javarce-y2NypXwk.html), [Cisco — CVE-2026-20324](https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-fmc-sftunn-codex-c3O4Jft2.html)

## Raport AI-ML

### Biznes

#### Crusoe przejmuje pełny lifecycle modeli Perplexity

Crusoe ogłosiło 15 września wieloletnią umowę, w której Perplexity ma trenować modele na dedykowanych GB300 NVL72 z InfiniBand i korzystać z Managed Inference tej samej chmury. Konsolidacja training + serving upraszcza transfer artefaktów i operacje, ale zwiększa zależność od jednego operatora infrastruktury oraz jego capacity planningu.

**Znaczenie architektoniczne:** Przy takim modelu kontraktu należy wymagać przenośności checkpointów, danych i obrazów runtime, mierzyć koszt egress oraz osobno definiować SLO dla treningu i inference. Awaria regionalna lub niedobór capacity jednego operatora może jednocześnie zatrzymać pipeline treningowy i produkcyjny serving.

**Źródło:** [Crusoe, 15.09.2026](https://www.crusoe.ai/resources/newsroom/crusoe-perplexity-partnership)

### Technologia

#### CoreWeave uruchamia wielorackowy Vera Rubin NVL72

CoreWeave ogłosiło 16 września uruchomienie klastra obejmującego wiele racków Vera Rubin NVL72. Pojedynczy rack łączy 72 Rubin GPU, 36 Vera CPU, NVLink 6, ConnectX-9, BlueField-4 i chłodzenie cieczą 45°C; domeny rackowe są spinane Spectrum-6 Ethernet, więc skalowanie wymaga jednoczesnej walidacji fabric, firmware, storage, zasilania i termiki.

**Znaczenie architektoniczne:** Granicą wydajności staje się synchronizacja wielu domen NVLink, a nie sam GPU. Straggler GPU lub marginalny link może obniżyć collective goodput całego jobu; acceptance test powinien obejmować NCCL/all-reduce pod degradacją linków, telemetrykę per rail, thermal throttling i zachowanie schedulera przy częściowej utracie racka.

**Źródła:** [CoreWeave, 16.09.2026](https://coreweave.com/news/coreweave-brings-up-multi-rack-nvidia-vera-rubin-nvl72-cluster), [opis bring-up](https://www.coreweave.com/blog/what-it-takes-to-bring-up-a-multi-rack-nvidia-vera-rubin-nvl72-cluster)

#### AEMA: elastyczność poboru mocy jako element capacity planningu AI

Emerald AI, Google i NVIDIA uruchomiły 16 września AI Energy Management Alliance, której celem jest dynamiczne ograniczanie poboru mocy data center zależnie od stanu sieci energetycznej. Mechanizm może skrócić drogę do przyłączenia, jeśli operator potrafi udowodnić kontrolowalny load shedding bez naruszania SLO workloadów.

**Znaczenie architektoniczne:** Scheduler GPU i power-management muszą zostać powiązane z kontraktem energetycznym. Capacity nie może być już modelowane wyłącznie jako stałe MW: potrzebne są klasy workloadów interruptible/non-interruptible, budżety redukcji mocy, checkpointing oraz testy powrotu po power cap; failure mode to jednoczesny grid event i workload bez bezpiecznego punktu preemption.

**Źródło:** [NVIDIA, 16.09.2026](https://blogs.nvidia.com/blog/ai-energy-management-alliance/)

### Implikacje praktyczne

1. W projektach wielorackowych Rubin testować goodput i degradację całej domeny, nie tylko link-up oraz benchmark pojedynczego racka.
2. Wprowadzić power-flexibility do schedulera jako jawny constraint obok GPU, pamięci, sieci i locality danych.
3. Przy outsourcingu całego model lifecycle wymagać technicznego exit planu: checkpoint portability, egress, obrazy runtime i alternatywny serving.
4. Telemetrię sieci, chłodzenia i zasilania korelować z job ID; inaczej przy wielorackowej skali źródło stragglera będzie trudne do izolacji.

### Trend tygodnia

AI factory jest optymalizowana jako jeden system obejmujący GPU, scale-up/scale-out fabric, storage i energię. Kolejnym ograniczeniem capacity przestaje być wyłącznie dostępność akceleratorów: scheduler musi reagować również na stan sieci energetycznej i termikę. Jednocześnie dostawcy chmur przejmują coraz większą część lifecycle modeli, co upraszcza operacje kosztem większego blast radiusu i lock-inu.

### To obserwować

- realny NCCL goodput wielorackowych Rubin NVL72 przy awarii linku/rail;
- Spectrum-6 + ConnectX-9 telemetry i mechanizmy congestion control w produkcji;
- wymagane czasy i głębokość redukcji mocy dla kontraktów flexible-load;
- portability checkpointów i koszt egress w usługach managed training + inference;
- korelację power cap z token/s i czasem checkpoint/restart.

## Newsletters summary

### Cursor CLI: sandbox nie obejmował wewnętrznych wywołań Git

- **Technologia / Zdarzenie:** [Beltdown2 — Cursor CLI sandbox escape](https://dennysentinel.com/blog/2026-09-13-cursor-cli-sandbox-beltdown2/)
- **Mechanizm działania:** Repozytorium mogło ustawić `core.fsmonitor` w `.git/config`; wewnętrzny `git` uruchamiany przez harness Cursor CLI działał poza sandboxem i wykonywał hook nawet przy read-only prompt.
- **Wpływ na architekturę:** Izolacja agentów musi obejmować proces nadrzędny/harness i wszystkie helpery, a nie wyłącznie shell udostępniony modelowi. Untrusted repository należy traktować jak aktywny input wykonawczy.
- **Failure modes i edge cases:** Blocklista pojedynczych komend Git nie zamyka klasy błędów. Trwała kontrola powinna neutralizować repo-controlled config na granicy procesu oraz wykonywać cały agent harness w VM/container boundary.

### Logi Options+: lokalny użytkownik mógł eskalować do SYSTEM

- **Technologia / Zdarzenie:** CVE-2026-12518 w Logi Options+ dla Windows.
- **Mechanizm działania:** Uprzywilejowany updater ufał kontrolowanym lokalnie parametrom/artefaktom instalacyjnym, umożliwiając standardowemu użytkownikowi uruchomienie kodu jako SYSTEM.
- **Wpływ na architekturę:** Oprogramowanie peryferyjne na stacjach administracyjnych należy traktować jak element privileged attack surface. Aktualizacja powinna być wymuszona centralnie; dla bastionów sensowne jest usunięcie zbędnych updaterów i narzędzi użytkowych.
- **Failure modes i edge cases:** Sam EDR nie usuwa ścieżki eskalacji, a użytkownik z prawem instalowania narzędzi peryferyjnych może odtworzyć podatny komponent. Wymagane są inventory wersji oraz application control.
