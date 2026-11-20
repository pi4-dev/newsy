# Raport Zbiorczy IT – 2026-11-20

Okres: od edycji 2026-11-19 do 2026-11-20 rano (Europe/Warsaw). Najważniejsze nowe zdarzenia: krytyczne RCE w HTTP/2 NGINX („nginx2root”), zero-day w vCenter wykorzystywany przez chińskie APT, pierwszy dzień notowań Anthropic (ANT) połączony z wyprzedażą spółek infrastruktury AI oraz wycofywanie się banków z kredytów pod DC (ING, Barclays). W sekcji euro neocloud jest nowa ekspozycja Nscale na regułę BIS (klient ByteDance). Pominięte, bo opisane wcześniej: Java4Shell, OPTIMJACK, Puppet w KEV, wyniki Nvidii, Project Jupiter/HSBC, CoreWeave 2027.

## Raport technologiczny

### Technologia

#### NGINX „nginx2root”: dwa krytyczne RCE w HTTP/2, ok. 8,8 mln potencjalnie narażonych usług (18–19.11)
F5/nginx wydały 18.11 NGINX OSS 1.30.7 (stable) i 1.31.9 (mainline) z poprawkami 8 CVE. CVE-2026-123007 (CVSS 4.0 9,2) to przepełnienie w dół licznika `send_window` w kontroli przepływu HTTP/2: okno zwracane po anulowanym strumieniu nie jest sprawdzane względem maksimum, co w ścieżce wysyłania TLS daje ujemną długość i kopiowanie poza zakres. Podatne są wersje 1.9.5–1.31.8, w domyślnej konfiguracji HTTPS + HTTP/2 (nawet dla samych plików statycznych). CVE-2026-123633 (9,2) to use-after-free połączenia HTTP/2 przy reverse proxy i wyczerpaniu slotów połączeń (1.19.7–1.31.8). Wiz pokazał RCE z obejściem ASLR na oficjalnych buildach (bez publicznego PoC). Xint połączył CVE-2026-123007 z niezałatanym use-after-free w ścieżce wysyłania SCTP jądra Linux (od v4.15) i uzyskał root oraz ucieczkę z kontenera na AWS, GCP i Azure. Pozostałe poprawki: HTTP/3 UAF (CVE-2026-121810, 9,2), resolver, mail proxy i DoS w TLS 1.3. NGINX Plus: 37.0.8.1 i 37.3.0.1. Censys liczy ok. 8,8 mln potencjalnie narażonych usług (ok. 104 mln nie ujawnia wersji). Exploitacji w środowisku produkcyjnym jeszcze nie odnotowano.
**Znaczenie:** NGINX to warstwa ingress dla dużej części internetu i klastrów Kubernetes. Ingress-NGINX, sidecary i obrazy bazowe trzeba przebudować, a nie tylko zaktualizować pakiet na hoście. Obejście `http2 off;` chroni tylko przed dwoma błędami HTTP/2, nie przed HTTP/3, resolverem ani mail proxy, i zwiększa liczbę połączeń TCP oraz latencję przy dużym RTT. Łańcuch z SCTP pokazuje, że kontener nie jest granicą bezpieczeństwa: warto zablokować moduł `sctp` (blacklist/seccomp), jeśli nie jest używany. Dystrybucje (AlmaLinux, RHEL) backportują poprawki, więc skanery oparte na numerze wersji dadzą fałszywe alarmy. CDN/WAF terminujące HTTP/2 przed originem częściowo mitygują problem.
Źródło: [nginx.org – advisories](https://nginx.org/en/security_advisories.html), [Wiz](https://www.wiz.io/blog/nginx-http2-rce-vulnerabilities), [Xint](https://xint.io/blog/nginx2root-remote-code-execution), [BleepingComputer](https://www.bleepingcomputer.com/news/security/critical-nginx-nginx2root-bugs-let-attackers-hijack-servers/)

<hr>

#### VMware vCenter CVE-2026-123890 (CVSS 9,8): obejście uwierzytelnienia w vmdir wykorzystywane jako zero-day przez chińską grupę (VMSA-2026-0009, 19–20.11)
Błąd w VMware Directory Service daje zdalnemu atakującemu bez poświadczeń sesję administratora. Poprawki: vCenter 9.1.1.0200, 9.0.2.0200, 8.0 U3m i VCF 5.2.5. Dla vCenter 7.0 nie ma publicznej łatki (tylko w ramach rozszerzonego wsparcia), workaroundów brak. Biuletyn obejmuje też SQLi i command injection w SDDC Manager oraz obejście podpisu SAML w VMware Tools dla Windows. Mandiant przypisuje ataki grupie Armory Castle (dawniej UNC5221): wejście przez zero-day w BeyondTrust Remote Support, narzędzie „Bonesaw” kradło bazę LMDB i materiał podpisujący SAML, tworzyło konto admina SSO, a następnie atakujący fałszowali tokeny SAML i przechodzili na ESXi. Bonesaw działał także na serwerach załatanych na wcześniejszy błąd z lipca. CVE nie ma jeszcze w CISA KEV.
**Znaczenie:** To trzecie w 2026 r. obejście uwierzytelnienia w vmdir. vCenter powinien być osiągalny wyłącznie z segmentu zarządzania lub bastionu, nigdy z sieci użytkowników. Po wykryciu kompromitacji samo łatanie nie wystarczy: trzeba zrotować certyfikaty STS/SAML, master key vCenter i hasła kont serwisowych ESXi, bo sfałszowane tokeny przetrwają aktualizację. Brak łatki dla 7.0 przyspiesza migrację na 8.0/9.x albo wymusza pełną izolację, co jest argumentem w negocjacjach licencyjnych z Broadcomem.
Źródło: [Broadcom VMSA-2026-0009](https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39495), [SecurityWeek](https://www.securityweek.com/newly-patched-vmware-vcenter-flaw-exploited-as-zero-day-by-chinese-apt/), [BleepingComputer](https://www.bleepingcomputer.com/news/security/vmware-patches-vcenter-zero-day-exploited-in-attacks-since-september/)

<hr>

#### Aktualizacja: PHP 8.5.13 / 8.4.28 / 8.3.37 / 8.2.36 wydane 19.11 – 18 CVE, w tym request smuggling w PHP-FPM z RCE; PHP 8.6.0 GA
Nowy element względem zapowiedzi: wydania ukazały się 19.11. Według BleepingComputer łatają 18 CVE. Najpoważniejszy to CVE-2026-129289 (CVSS 9,2): request smuggling w PHP-FPM prowadzący do RCE bez uwierzytelnienia przy ponownym używaniu połączeń backendowych (nginx, Apache, HAProxy, LSAPI). Do tego use-after-free w backtrace'ach i cache stref czasowych, nadpisywanie plików sesji (CVE-2026-126714, zapowiadany CVE-2026-115560), lokalne LPE przez epoll w FPM i obejście podpisu PHAR. Exploitacji nie zgłoszono. Tego samego dnia wyszło PHP 8.6.0. Wsparcie PHP 8.2 kończy się 31.12.2026.
**Znaczenie:** Ryzyko smugglingu dotyczy typowego stosu nginx + FPM z keepalive do upstreamu. Tymczasowe wyłączenie trwałych połączeń zwiększa liczbę połączeń i latencję, więc lepiej łatać niż utrzymywać mitygację. Po aktualizacji można wycofać `session.upload_progress.enabled=0`. Na hostingach z wieloma wersjami PHP gałęzie EOL pozostają odsłonięte (paczki Ondřeja Surý z backportami sięgają do 5.6).
Źródło: [php.net](https://www.php.net/), [BleepingComputer](https://www.bleepingcomputer.com/news/security/php-security-updates-fix-18-flaws-including-critical-rce-bugs/)

<hr>

#### Cisco Expressway CVE-2026-76466 (CVSS 9,8): pierwsze potwierdzone włamanie po publikacji PoC (19.11)
F6 opisał włamanie do organizacji w Rosji: atakujący przejęli dwa wystawione do internetu węzły Expressway X15.5.3, a potem dwa kolejne węzły klastra w sieci wewnętrznej. Błąd w usłudze bazy danych klastra (statyczne poświadczenia, trwałe cookies) daje zdalne wykonanie poleceń jako root bez uwierzytelnienia; dotyczy tylko klastrowanych Expressway Core/Edge. Po przejęciu atakujący użyli narzędzi w Pythonie do skanowania SMB, SQL Server, WinRM i przeszukiwania GitLaba pod kątem poświadczeń (F6 podejrzewa kod generowany przez AI). Poprawka: X15.5.20, workaroundów brak. Advisory Cisco nadal mówi „not exploited”.
**Znaczenie:** Expressway Edge stoi w DMZ z definicji, a porty klastrowe TCP 4371–4372 powinny być dostępne wyłącznie między węzłami klastra. To szybki audyt ACL. Wzorzec „PoC na GitHubie, exploitacja po kilku dniach” potwierdza, że dla urządzeń brzegowych okno łatania liczy się w dniach, a nie w cyklu miesięcznym.
Źródło: [SecurityWeek](https://www.securityweek.com/hackers-exploit-cisco-expressway-vulnerability-soon-after-poc-release/), [Cisco advisory](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-vcs-static-cred-hr7EWZ8t)

<hr>

#### Drupal SA-CORE-2026-017: przejęcie strony lub RCE przez `install.php` bez uwierzytelnienia (18.11)
CVE-2026-132165 („Critical”, 18/25): w określonych warunkach anonimowy atakujący może użyć `core/install.php` do ponownej instalacji Drupala, co daje przejęcie strony lub RCE. Część ścieżek blokuje brak prawa zapisu do `settings.php`. Poprawki: 10.6.22, 11.3.22, 11.4.13. Błąd znalazł Anthropic OSS Scanner. Zespół Drupala ostrzega, że exploity mogą pojawić się „w ciągu godzin lub dni”.
**Znaczenie:** Tania mitygacja niezależna od łatki: deny dla `/core/install.php` na reverse proxy/WAF i pliki konfiguracyjne tylko do odczytu w obrazie kontenera. To kolejny krytyczny błąd znaleziony przez skaner AI, co zwiększa częstotliwość pilnych wydań w popularnych CMS.
Źródło: [Drupal SA-CORE-2026-017](https://www.drupal.org/sa-core-2026-017), [BleepingComputer](https://www.bleepingcomputer.com/news/security/drupal-patches-critical-rce-flaw-warns-exploits-may-surface-quickly/)

<hr>

#### CISA KEV (18.11): Synology DSM, LiteLLM, REDCap, Apache Axis – trzy z czterech z terminem 3 dni
Dodane: CVE-2024-10441 (Synology DSM/BSM, zdalne RCE, termin 21.11), CVE-2026-40217 (BerriAI LiteLLM, RCE przez endpoint `/guardrails/test_custom_code`, termin 2.12), CVE-2026-90817 (REDCap, wstrzyknięcie kodu z publicznej ankiety, 21.11) oraz CVE-2026-90436 (Apache Axis, unsafe reflection przez SOAP fault, 21.11). Wpisy odwołują się do BOD 26-04 i wymagań triage'u forensycznego, a nie tylko łatania.
**Znaczenie:** Skrócone terminy (3 dni) wymagają aktualizacji reguł SLA, które mapują KEV na czas łatania. LiteLLM jako brama do modeli przechowuje klucze API wielu dostawców, więc RCE oznacza wyciek sekretów i niekontrolowane koszty. Apache Axis występuje osadzony w wielu produktach; bez SBOM trudno ustalić ekspozycję.
Źródło: [CISA KEV (JSON)](https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json), [katalog KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

<hr>

#### Zarządzane łącza multicloud: AWS Interconnect–Azure GA (17.11), Google Partner Cross-Cloud Interconnect dla Azure w preview (17–18.11)
AWS ogłosił GA AWS Interconnect – multicloud z Azure (preview od sierpnia) w us-east-1, eu-central-1 i ap-southeast-2; komunikat nie podaje przepustowości ani cen. Google uruchomił w preview Partner Cross-Cloud Interconnect dla Azure w us-east4, us-west2 i europe-west3: konfiguracja przez konsolę/API, fizycznie rozdzielone ścieżki, automatyczne szyfrowanie między routerami brzegowymi obu chmur, stała opłata za wybraną przepustowość (bez opublikowanych stawek i SLA). Obie usługi opierają się na otwartej specyfikacji interoperacyjności opublikowanej przez AWS (github.com/aws/Interconnect), przyjętej przez Microsoft.
**Znaczenie:** Wszyscy trzej hyperscalerzy stosują teraz ten sam model zarządzanego interconnectu, co zastępuje wzorzec „Direct Connect + ExpressRoute + własny router w Equinix” i odbiera prosty przypadek użycia giełdom typu Megaport. Spada czas provisioningu i liczba ręcznie utrzymywanych komponentów (BGP, MACsec), ale pojawia się zależność od wspólnej płaszczyzny sterowania dwóch dostawców. Egress nadal liczony jest po obu stronach. Frankfurt to jedyny region UE w obu ofertach. Dopóki brak SLA, produkcyjne DR między chmurami wymaga dwóch niezależnych ścieżek.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/aws-announces-AWS-interconnect-multicloud-azure-ga/), [Google Cloud Blog](https://cloud.google.com/blog/products/networking/cross-cloud-interconnect-for-microsoft-azure)

<hr>

#### Chainguard: od 15.12 weryfikacja podpisów wymaga Cosign ≥3.1.1; nowy darmowy plan Starter (17.11)
Od 16.11 do 15.12 artefakty Chainguard są podpisywane podwójnie; po 15.12 weryfikacja wymaga Cosign ≥3.1.1 (Sigstore v3), starszy Cosign przestanie weryfikować nowe artefakty. Równolegle Chainguard zrezygnował z wymogu rozmowy z działem sprzedaży: Starter (bez karty) daje 5 obrazów, 100 GB pobrań, 50 tys. pobrań bibliotek z 7-dniowym cooldownem nowych wersji; Team kosztuje 500 USD/mies.
**Znaczenie:** Operacyjnie ważniejsza jest zmiana podpisów: admission controllery (Kyverno, Sigstore policy-controller) ze starszym Cosign zablokują wdrożenia po 15.12. Cooldown wersji to praktyczna ochrona przed incydentami typu `agent-framework` (zob. Newsletters).
Źródło: [Chainguard changelog](https://edu.chainguard.dev/chainguard/changelog/), [Chainguard](https://www.chainguard.dev/unchained/chainguard-is-now-self-serve-and-free-to-start)

### Biznes

#### Palo Alto Networks Q1 FY27: przychód +39% do 3,44 mld USD; przejęcie Applied Compute za 4,5 mld USD w akcjach (18.11)
Przychód 3,445 mld USD (+39% r/r), NGS ARR 9,73 mld USD (+66%), RPO 21,0 mld (+35%), skorygowany FCF 1,9 mld USD. Zysk operacyjny GAAP spadł do 96 mln z 309 mln USD (koszty integracji CyberArk), non-GAAP wzrósł do 1,1 mld. Prognoza FY27: 14,52–14,57 mld USD przychodu. Applied Compute to platforma do post-treningu i inferencji modeli open-weight; transakcja wyłącznie w akcjach, zamknięcie w Q3 FY27, CEO przejmowanej spółki zostanie Co-Chief AI Officer.
**Znaczenie:** Wzrost ARR jest częściowo nieorganiczny. Zakup platformy inferencyjnej przesuwa Palo Alto z ochrony ruchu w stronę hostowania modeli („bezpieczny runtime AI”), co zwiększa potencjalny lock-in na jednej platformie bezpieczeństwa i tworzy konkurencję dla hyperscalerów.
Źródło: [SEC – wyniki Q1 FY27](https://www.sec.gov/Archives/edgar/data/0001327567/000132756726000030/ex991q127earningsrelease.htm), [SEC – komunikat o przejęciu](https://www.sec.gov/Archives/edgar/data/0001327567/000132756726000030/ex992pressrelease_november.htm)

<hr>

#### Blackstone Digital Infrastructure Trust kupuje ok. 75% dwóch JV z AWS: 13 budynków, 647 MW, ok. 5 mld USD, leaseback na 12 lat (16–17.11)
Portfel obejmuje 9 nowych budynków w północnej Indianie i 4 w Missisipi (647 MW mocy IT); AWS dzierżawi je z powrotem na początkowe 12 lat. Trust wnosi ok. 1,9 mld USD z IPO, JV planują ok. 2,5 mld USD kredytów hipotecznych bez regresu. Początkowa stopa kapitalizacji 7,0%, zamknięcie w Q4 2026.
**Znaczenie:** Hyperscaler zamienia gotowe DC na gotówkę (ok. 7,7 mln USD/MW), zachowując kontrolę operacyjną – recykling capexu podobny do sale-leaseback wież telekomunikacyjnych. Ryzyko wartości rezydualnej przechodzi na kapitał REIT-owy; przy 12-letniej umowie i szybkiej zmianie gęstości mocy (chłodzenie cieczą) budynki mogą wymagać kosztownej modernizacji przed odnowieniem.
Źródło: [The Register](https://theregister.com/on-prem/2026/11/17/reit-move-blackstone-joint-venture-buys-aws-campuses-to-lease-back-to-the-cloud-giant/5310355)

## Raport AI-ML

### Biznes

#### Anthropic (ANT): pierwszy dzień na Nasdaq – otwarcie 111,50 USD, zamknięcie 97,17 USD (+17%) (19.11)
Pierwsza transakcja nastąpiła o 11:48 ET po 111,50 USD (+34% wobec ceny IPO 83 USD), maksimum ok. 115 USD, zamknięcie 97,17 USD (+17,1%), wolumen ok. 832 mln akcji. Kapitalizacja przy zamknięciu to ok. 1,3 bln USD (szacunki zależą od metodologii liczenia klas akcji). Breakingviews podaje ok. 900 mld USD zobowiązań Anthropic na chipy, chmurę i DC, w tym ponad 70 mld płatnych w ciągu 12 miesięcy od września 2026, oraz spadek marży operacyjnej z 8% w Q2 do −3% w Q3.
**Analiza:** Cofnięcie od otwarcia o ok. 13% odzwierciedla podaż od insiderów. Dla infrastruktury ważna jest waluta akcyjna, którą Anthropic może obsługiwać zobowiązania compute (Microsoft, Nscale, Google TPU, Trainium, AMD) bez dalszego zadłużania. Ujemna marża operacyjna przy tej skali zobowiązań oznacza, że kontrahenci (neocloudy, Google, SpaceX) są wystawieni na tempo wzrostu przychodów jednego klienta. Kurs ANT staje się publicznym wskaźnikiem popytu na moc obliczeniową.
Źródło: [StockTitan](https://www.stocktitan.net/articles/anthropic-ipo-live-ant-first-trading-day-2026), [Breakingviews](https://www.breakingviews.com/ddb76cfb4026/columns/breaking-view/anthropics-2-trln-ipo-may-be-last-hurrah-2026-11-19/)

<hr>

#### Zamknięcie SC26: top 10 Green500 bez zmian, Bull (Eviden) na trzech pierwszych miejscach (19–20.11)
Rejestracja przekroczyła 23,7 tys. osób. Green500: KAIROS (CALMIP, ok. 73,3 GFlops/W), ROMEO-2025 (ok. 70,9) i rozszerzenie GPU Levante w DKRZ (ok. 69,4); top 10 identyczne jak w czerwcu. Na wystawie było 55 dostawców kwantowych; Hyperion szacuje rynek QC na 1,4 mld USD w 2025 i 3 mld w 2028, a odsetek ankietowanych uznających „zimę kwantową” za prawdopodobną wzrósł z 23% do 34%.
**Analiza:** Zamrożone Green500 to ten sam sygnał co rekordowo niska rotacja TOP500: najbardziej efektywne systemy AI (GB300, Vera Rubin) trafiają do chmur i nie są zgłaszane do list. Do planowania energetycznego klastrów AI lepszym punktem odniesienia jest MLPerf Endpoints (throughput/kW) niż GFlops/W z HPL.
Źródło: [HPCwire – Green500](https://www.hpcwire.com/off-the-wire/bull-systems-hold-top-3-green500-spots-for-third-straight-edition/), [HPCwire – kwanty na SC26](https://www.hpcwire.com/2026/11/19/sc26-the-biggest-little-quantum-show-in-the-world/)

### Technologia

#### NVIDIA Dynamo dla RL: routing świadomy KV cache i dystrybucja delt wag przez NIXL (ModelExpress) (19.11)
Dynamo działa jako warstwa serwująca między frameworkiem RL (verl, NeMo RL, Slime, prime-rl) a workerami inferencji (np. SGLang). Funkcje: routing z kompromisem między reużyciem prefiksu a obciążeniem workera, ModelExpress (wagi kopiowane GPU–GPU przez NIXL, przesyłanie skompresowanych delt zamiast pełnych checkpointów), wykrywanie utraty workera, migracja żądań w locie i graceful drain. CoreWeave przyspieszył aktualizację wag do 15×, H Company skrócił krok treningowy o do 20% i zużycie GPU-h o do 5%. Brak bezwzględnych liczb throughput i konfiguracji sprzętowej.
**Analiza:** W RL on-policy wąskim gardłem jest faza rolloutu (długi ogon generacji) i synchronizacja wag po każdym kroku. Delty przez RDMA zamiast checkpointu z S3 skracają bezczynność GPU, ale 5% GPU-h u H Company pokazuje, że oszczędność w całym pipeline jest umiarkowana. Odporność na utratę workera ma znaczenie przy rolloutach na tysiącach GPU. Ryzyko lock-in: orkiestracja post-treningu coraz bardziej przechodzi do stosu NVIDIA (Dynamo, NIXL), choć silniki inferencji pozostają otwarte.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/reducing-rollout-bottlenecks-in-reinforcement-learning-with-nvidia-dynamo/)

<hr>

#### Gordon Bell 2026: STORM na 74 400 GPU Frontier (6 EFLOPS BF16) i FeMoco na CPU-owym LineShine (19.11)
Nagroda klimatyczna: STORM (ORNL, Auburn, FSU, AMD Research, CSU) – generatywny model asymilacji danych z globalną uwagą o złożoności liniowej, skalowany do 74 400 GPU Frontier (AMD) przy 96–99% efektywności strong scaling i 6 EFLOPS BF16 sustained; 32 768-członowe ensemble w 34 s na 4 096 GPU. Główna nagroda: chiński zespół za obliczenia FeMoco CAS – 1,456 EFLOPS FP64 sustained na 20 480 węzłach LineShine (58,9% szczytu) i 5,626 EFLOPS w mieszanej precyzji.
**Analiza:** Uwaga o liniowej złożoności pozwala skalować modele generatywne na długich sekwencjach do pełnej maszyny eksaskalowej bez kwadratowego wzrostu kosztu. Wynik na AMD z udziałem AMD Research to argument za dojrzałością ROCm w skali ponad 70 tys. GPU. Wygrana systemu CPU przy 58,9% szczytu FP64 potwierdza, że dla części obciążeń naukowych przepustowość pamięci i FP64 ważą więcej niż FLOPS Tensor Core (por. osobny Rubin HPC).
Źródło: [HPCwire – STORM](https://www.hpcwire.com/off-the-wire/storm-team-wins-gordon-bell-climate-prize-for-ai-based-data-assimilation-on-frontier/), [HPCwire – Gordon Bell](https://www.hpcwire.com/off-the-wire/chinese-team-wins-2026-acm-gordon-bell-prize-for-exascale-quantum-chemistry/)

<hr>

#### Dnotitia VDPU: pierwsza publiczna demonstracja ASIC do wyszukiwania wektorowego (SC26, 19.11)
Koreańska Dnotitia pokazała kartę z ASIC VDPU: wyszukiwanie w języku naturalnym w ok. 2,6 mln klatek wideo. Offload obniża zużycie CPU hosta o co najmniej 90% wobec wyszukiwania wyłącznie na CPU. Cel (niezmierzony): ok. 10× wydajności dwusocketowego serwera przy kilku kartach. Brak danych o procesie, pamięci, interfejsie, poborze mocy, cenie i terminie komercjalizacji.
**Analiza:** W pipeline'ach RAG i agentów retrieval na CPU staje się wąskim gardłem latencji i kosztu, gdy decode na GPU jest już szybki. Bez danych o pojemności pamięci na kartę i latencji p99 nie da się porównać z wyszukiwaniem na GPU (cuVS/CAGRA). Mały dostawca i własny stos to ryzyko lock-in – na razie temat do obserwacji, nie do planowania pojemności.
Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/dnotitia-presents-first-live-demonstration-of-its-vdpu-vector-search-asic-at-sc26/)

<hr>

#### Aktualizacja: Penguin MemoryAI Photonic Appliance – 57,6 Tb/s łącznej przepustowości, próbki dopiero w II połowie 2027 (19.11)
Nowe względem rejestru: podana łączna przepustowość appliance (Marvell + Penguin) to 57,6 Tb/s, produkt jest w fazie rozwoju, próbki w II połowie 2027. Otwarte pozostają latencja, zmiany w oprogramowaniu dla pamięci współdzielonej i koszt.
**Analiza:** Produkcja realnie przypadnie na 2028, więc w planach offloadu KV cache na 2027 pozostają NVMe/GDS, CXL i LMCache.
Źródło: [HPCwire](https://www.hpcwire.com/2026/11/19/penguin-solutions-turns-to-silicon-photonics-to-overcome-hpc-and-ai-memory-bottlenecks/)

## euro neocloud

### Nscale

#### Kontrakt z ByteDance wymaga licencji USA na zdalny dostęp do GPU (Bloomberg, 19.11)
**Fakty:** Według Bloomberga Nscale musi wystąpić o licencję BIS, by dalej udostępniać GPU Nvidii spółce Spring (Singapur, ByteDance) – 2 304 B200 w Glomfjord (Norwegia), także lokalizacje na Islandii. Operatorzy obsługujący takich klientów mają 90 dni na uzyskanie licencji. Według ujawnień z procesu IPO Spring dał ok. 24 mln z 33 mln USD przychodu Nscale w 2025 r. (ok. 73%). Kontrakt zabezpiecza kredyt Macquarie na 105 mln USD z wymogiem monitorowania zgodności z kontrolą eksportu. Prospekt nie wymienia ByteDance z nazwy. Nscale nie potwierdził, czy wystąpi o licencję.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* pierwszy nazwany europejski operator bezpośrednio objęty regułą o zdalnym dostępie; ryzyko dla covenantów kredytu Macquarie; kwestia jakości ujawnień w prospekcie w trakcie roadshow; przy braku licencji klaster 2 304 B200 trzeba będzie wynająć innemu najemcy.
- *Ocena ryzyka:* **Wysokie** (bez zmian). Bieżący backlog opiera się na Microsoft i Anthropic, więc wpływ przychodowy jest ograniczony, ale dochodzi ryzyko compliance wobec wierzycieli i inwestorów IPO przy istniejącej luce finansowej ok. 30 mld USD.
- *Linki:* [Bloomberg](https://www.bloomberg.com/news/articles/2026-11-19/nscale-s-bytedance-deal-faces-scrutiny-under-us-chip-curbs), [Startup Fortune](https://startupfortune.com/nscales-bytedance-chip-rental-deal-now-needs-a-us-export-license/)

<hr>

#### Pierwsze szafy Vera Rubin NVL72 (Dell XE9812) uruchomione z fabrycznie zintegrowaną siecią (19.11)
**Fakty:** Nscale uruchomił pierwsze VR NVL72 na Dell PowerEdge XE9812 (72 GPU Rubin, 36 CPU Vera na szafę). Według Della Nscale jako pierwszy wdrożył fabrycznie złożoną i przetestowaną szafę sieciową (przełączniki, zasilanie, chłodzenie cieczą) w ramach Dell PowerRack. Globalny rollout ma potrwać kilka miesięcy. Nie podano klientów, mocy (MW) ani lokalizacji.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* uruchomienie sprzętu nie odpowiada na pytanie o kapitał; brak lokalizacji i skali; zależność od jednego integratora (Dell).
- *Ocena ryzyka:* **Wysokie** (łącznie dla Nscale). Wykonanie techniczne jest pozytywne (drugi neocloud z produkcyjnym VR po Nebius), ale nie zmienia profilu finansowego.
- *Linki:* [HPCwire](https://www.hpcwire.com/off-the-wire/nscale-brings-its-1st-nvidia-vera-rubin-nvl72-systems-online/)

### Nebius

#### Aktualizacja: kurs −9,4% w dniu debiutu Anthropic mimo wzrostu przed sesją (19.11)
**Fakty:** Przed sesją akcje rosły o blisko 5%, zamknęły się spadkiem o 9,39% do 96,92 USD. Brak nowych informacji o samej spółce.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* debiut największego klienta compute w sektorze nie wspiera kursów neocloudów – rynek oddziela finansowanie laboratoriów od finansowania infrastruktury pod nimi; wysoka zmienność podnosi koszt emisji obligacji zamiennych.
- *Ocena ryzyka:* **Średnie** (bez zmian) – ruch rynkowy, nie fundamentalny.
- *Linki:* [Yahoo Finance](https://finance.yahoo.com/markets/stocks/articles/stock-market-today-nov-19-225305200.html), [Decrypt](https://decrypt.co/383669/nvidia-amd-micron-ai-stocks-sink-anthropic-ipo-price)

### evroc

#### Government Cloud z Telia Cygate dla szwedzkiej administracji (19.11)
**Fakty:** evroc i Telia Cygate uruchomiły suwerenną chmurę dla szwedzkiego sektora publicznego (krytyczne dane i obciążenia AI) na szwedzkiej infrastrukturze i pod szwedzką jurysdykcją. Model ma być przenoszony do innych krajów UE. Nie podano mocy, liczby GPU ani wartości kontraktów.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* brak danych o finansowaniu i skali; popyt publiczny jest stabilny, ale nie wypełni dużych klastrów GPU; Telia (także w AIGF Northern) może kontrolować relację z klientem.
- *Ocena ryzyka:* **Średnie** – pozytywny sygnał handlowy, ale mała skala i nieujawnione źródła finansowania capexu.
- *Linki:* [evroc – newsroom](https://evroc.com/news/), [evroc – Swedish Gov Cloud](https://evroc.com/swedishgovcloud)

### OVHcloud

#### AI Summit: B300 i Rubin R200 w ofercie, deklarowana pojemność do 60 tys. B300 lub 35 tys. R200, własne LLM „Olala” (19.11)
**Fakty:** OVHcloud ogłosiło 55 produktów: GPU RTX 6000 Pro, B300 i Rubin R200 w Bare Metal i Public Cloud. Według spółki jej DC mogą pomieścić do 60 tys. B300 albo 35 tys. R200 (pojemność, nie zainstalowane GPU). Olala mini to MoE 7B z 1B aktywnych parametrów działający na CPU; v2.0 (100–300B) na początku 2027. Nie podano cen ani terminów dostaw R200. Relacja jest wtórna (DEV Community powołuje się na Reuters); komunikatu OVH nie znaleziono.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* zapowiedzi pojemności bez dat i cen; rozproszenie strategii na własne modele (konkurencja z Mistralem); rosnące ceny pamięci i serwerów podnoszą koszt dojścia do deklarowanej skali.
- *Ocena ryzyka:* **Niskie** – zdywersyfikowana, notowana chmura, AI to niewielka część biznesu; deklarowaną pojemność traktować jako opcję, nie podaż na 2026–2027.
- *Linki:* [DEV Community](https://dev.to/paullovelace/ovhcloud-just-shipped-55-ai-products-b300-gpus-a-cpu-runnable-llm-and-europes-big-sovereign-ai-4h3o)

## AI for networking

#### AWS Multipath Resilience: duplikacja pakietów po rozdzielonych światłowodach i SLA 99,999% dla ruchu między regionami (17.11)
Po włączeniu dla cross-Region VPC peering, peeringu Transit Gateway lub Cloud WAN sieć AWS wysyła kopie każdego pakietu różnymi, fizycznie rozdzielonymi trasami; region docelowy dostarcza pierwszą kopię i odrzuca duplikaty, bez zmian w aplikacji. Mechanizm celuje w częściowe degradacje (szare awarie), których obejście reroutingiem trwa minuty, w odróżnieniu od twardych awarii przełączanych w czasie poniżej sekundy. GA w us-east-1, us-east-2, us-west-1, us-west-2 i eu-west-1.
**Analiza:** To odpowiednik ochrony 1+1 (SDH, DetNet PREOF) na poziomie L3 chmury. Koszt to ok. 2× ruchu na backbone, przerzucany na klienta, a zysk to ogon latencji wyznaczany przez najszybszą ścieżkę i brak strat przy przełączeniu. Dla replikacji baz i synchronizacji stanu między regionami to alternatywa dla aktywnej replikacji w aplikacji, ale wiąże architekturę DR z funkcją specyficzną dla AWS. Przed włączeniem porównać dopłatę za zduplikowany ruch z kosztem drugiej, niezależnej ścieżki.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/vpc-tgw-cloudwan-99999-sla/)

<hr>

#### NVIDIA DAQIRI (Holoscan): strumienie z instrumentów bezpośrednio do pamięci GPU przez ibverbs/DevX, >500 Gb/s (19.11)
Biblioteka przenosi strumienie z sensorów (FPGA → Ethernet) do pamięci GPU, gdzie odbywa się składanie ramek, korekcja i redukcja danych oraz inferencja (TensorRT). Przypadek referencyjny: kamera do mikroskopii elektronowej (EELS) w LBNL generująca ponad 500 Gb/s. Nowy silnik ibverbs/DevX daje ok. 2× przepustowości na rdzeń wobec DPDK dla pakietów ≤1024 B przy ponad 200 Gb/s; sprzętowe przestawianie pakietów działa na ConnectX-7+, opóźnienie send-to-receive w pętli kablowej ok. 2–4 µs.
**Analiza:** Wzorzec „NIC z RDMA wprost do GPU zamiast farmy CPU z DPDK” zmniejsza o rzędy wielkości ilość danych wysyłanych do DC i umożliwia zamkniętą pętlę sterowania w mikrosekundach; przenosi się na radioastronomię, akceleratory cząstek i inspekcję przemysłową. Zysk dotyczy głównie małych pakietów (dla ramek 8 KB bez zmian). Lock-in jest wyraźny: ConnectX-7+, Holoscan, TensorRT.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/enabling-real-time-ai-for-scientific-instruments-with-gpu-native-data-acquisition/)

<hr>

#### Pass-the-Device: przenoszenie zaufania urządzenia SASE/VPN z przejętego endpointu (CODE BLUE 2026)
Zestaw narzędzi do rekonesansu GlobalProtect, Cisco ASA, FortiGate, Zscaler i Netskope: fingerprinting metod uwierzytelniania VPN i IdP, anonimowe enumerowanie konfiguracji tenanta Netskope, parsowanie profili posture Cisco HostScan oraz laboratoryjne proxy SAML wstrzykujące ciasteczka PRT. Repozytorium zawiera też detekcje Sigma i KQL (procesy spoza dostawcy czytające stan klienta VPN lub klucze DPAPI, fingerprinty klientów opartych na OpenConnect).
**Analiza:** Posture check w SASE/ZTNA opiera się na deklaracjach klienta, które po przejęciu endpointu można odtworzyć na innej maszynie. Zaufanie urządzenia trzeba wiązać z niewyeksportowalnym kluczem w TPM i ciągłą oceną sygnałów, a nie z jednorazowym sprawdzeniem przy logowaniu. Detekcje wymagają baseline'u, bo narzędzia administracyjne także czytają stan klienta VPN.
Źródło: [GitHub – pass-the-device](https://github.com/crypt0p3g/pass-the-device)

## Ryzyka rynku AI/GPU

#### Debiut ANT nie zatrzymał wyprzedaży infrastruktury AI: Nasdaq −2,25%, CoreWeave −6,1%, Nebius −9,4%, AMD ok. −7,7% (19.11)
S&P 500 spadł o 2,02%, Nasdaq o 2,25%. Według Decrypt (notowania popołudniowe) AMD straciło ok. 7,7% (raport short sellera Hunterbrook), Nvidia ok. 3,4%, Oracle ok. 3%, Micron 2,2%, Broadcom 2,1%. Breakingviews: wydatki firm na tokeny w tygodniu do 15.11 tylko o 3% wyższe niż miesiąc wcześniej, VIX powyżej 25. Capital Economics: IPO „nie rozpaliło na nowo entuzjazmu” dla technologii.
**Analiza:** Rynek finansuje laboratorium (ok. 81 mld USD z IPO plus 26 mld z emisji prywatnej), ale wycenia w dół warstwę infrastruktury pod nim. Spłaszczony wzrost wydatków na tokeny przy ujemnej marży operacyjnej Anthropic i ponad 70 mld USD zobowiązań na 12 miesięcy to kluczowy wskaźnik popytu, od którego zależą płatności dla Nscale, CoreWeave i Google. Procenty dla poszczególnych spółek różnią się między źródłami (część to notowania śródsesyjne).
Źródło: [Decrypt](https://decrypt.co/383669/nvidia-amd-micron-ai-stocks-sink-anthropic-ipo-price), [Breakingviews](https://www.breakingviews.com/ddb76cfb4026/columns/breaking-view/anthropics-2-trln-ipo-may-be-last-hurrah-2026-11-19/), [Yahoo Finance](https://finance.yahoo.com/markets/stocks/articles/stock-market-today-nov-19-225305200.html)

<hr>

#### Banki zbywają ryzyko DC: ING sprzedaje portfel ok. 500 mln USD (z transzą Project Jupiter), Barclays kupuje ochronę kredytową, spready DC ABS 2,57 pp wobec 1,95 pp w Q1 (6–20.11)
ING wystawił w aukcji BWIC portfel 12 kredytów na DC (ok. 500 mln USD), w tym transzę 25 mln USD powiązaną z Project Jupiter oraz kredyty związane z Vantage i CyrusOne; oferty przyjmowano do 19.11, wynik nieznany. Według danych Morgan Stanley/CREFC/BofA (Korea JoongAng Daily, 20.11) spread 5-letnich DC ABS z ratingiem A wynosił średnio 2,57 pp nad Treasuries w ostatnich 3 miesiącach wobec 1,95 pp w Q1, a emisja DC ABS/CMBS w USA w ostatnich 3 miesiącach spadła o 19,9% r/r. Spready 10-letnich obligacji hyperscalerów rozszerzyły się w tym roku o ok. 0,5 pp (0,2 pp dla przemysłu spoza AI). Barclays kupił ubezpieczenie kredytowe na kredyty pod infrastrukturę AI; według WTW część ubezpieczycieli jest blisko limitów ekspozycji na DC.
**Analiza:** Banki nie tylko ograniczają nowe kredyty, ale aktywnie redukują ekspozycję – sprzedaż z dyskontem (HSBC, ING) i transfer ryzyka do ubezpieczycieli (Barclays). Droższe ABS to droższe refinansowanie gotowych DC, czyli wyjście z kredytu budowlanego, na które liczą deweloperzy i neocloudy. Cena rozliczenia BWIC ING (szczególnie transzy Jupiter) będzie rynkowym punktem odniesienia dla wyceny kredytów DC.
Źródło: [Korea JoongAng Daily](https://www.koreajoongangdaily.com/business/ai-bubble-fears-push-lenders-that-bankrolled-boom-to-pull-back/12972392), [IBTimes](https://www.ibtimes.com/ing-sells-data-center-loans-stargate-debt-3809516)

<hr>

#### CoreWeave: współzałożyciel kupił obligacje niezabezpieczone za 30 mln USD nominału; wzrost kursu o 9% wyparował w ciągu sesji (18–19.11)
Brian Venturo (CSO, członek rady) ujawnił zakup obligacji z zapadalnością w 2031 r. i zamiar trzymania ich do wykupu. Trzej współzałożyciele sprzedali od IPO akcje za ok. 3,3 mld USD. 19.11 kurs rósł w sesji o ponad 9%, a zamknął się na 44,91 USD (−6,1%). Dług spółki na koniec Q3 przekraczał 40 mld USD.
**Analiza:** Zakup insidera sugeruje, że zarząd uważa obligacje za niedowartościowane, ale kwota jest symboliczna wobec sprzedaży akcji i zadłużenia. Rynek nie zmienił oceny ryzyka refinansowania w 2027 (ok. 11 mld USD zapadalności). Wskaźnik do obserwacji: czy rentowność serii 2031 spadnie wyraźnie poniżej ok. 14%.
Źródło: [Bloomberg](https://www.bloomberg.com/news/articles/2026-11-19/coreweave-s-venturo-says-he-bought-30-million-of-firm-s-bonds), [Seeking Alpha](https://seekingalpha.com/news/4674650-coreweave-co-founder-discloses-bond-purchase-says-he-will-hold-until-maturity)

<hr>

#### Projekty DC: Telus wycofuje się z dwóch DC AI w Vancouver (85 MW z BC Hydro), komisja w Stafford (VA) za odrzuceniem 37 budynków DC (16–19.11)
Telus nie będzie współtworzyć z Westbank dwóch „suwerennych” DC AI w Vancouver, mimo zapewnionych wstępnie 85 MW z BC Hydro; spółka tnie koszty i woli dostarczać łączność niż operować DC. Westbank szuka nowego partnera. W Stafford County komisja planowania (4:3) zarekomendowała odrzucenie zmiany przeznaczenia 1 400 akrów pod 37 budynków DC i 14 podstacji; decyzja należy do Rady Nadzorców. W Q3 2026 w USA anulowano co najmniej 17 dużych projektów DC; 48% z 69 anulowanych od 2024 r. upadło przez sprzeciw lokalny (TokenPost).
**Analiza:** Rośnie luka między projektami zapowiedzianymi a oddanymi, a przyczyną coraz częściej jest zgoda społeczna i strategia bilansowa operatorów, nie dostępność GPU. Przydzielona moc (85 MW) zostaje w projekcie, więc ryzyko przesuwa się na termin i partnera. Przy planowaniu pojemności kolokacji na 2027–2028 dyskontować projekty bez prawomocnej zmiany planu i podpisanej umowy przyłączeniowej.
Źródło: [DCD](https://www.datacenterdynamics.com/en/news/telus-abandons-its-vancouver-ai-data-center-development-plans/), [FOX 5 DC](https://www.fox5dc.com/news/stafford-county-planning-commission-recommends-rejecting-massive-data-center-proposal)

## Newsletters summary

#### Microsoft `agent-framework` na PyPI: wersje 1.26.1 i 1.26.2 ze stealerem, wydane z oficjalnego konta wydawcy (TLDR InfoSec, raport 18.11)
Obie wersje zawierają plik `af_runtime_diag.pth`, który Python wykonuje przy każdym starcie interpretera, nawet jeśli pakiet nie jest importowany. Stealer zbiera zmienne środowiskowe z kluczami/tokenami, pliki poświadczeń (Azure CLI, pip, npm, netrc, Git, Docker, Hugging Face, agent Azure Pipelines, `.env`), środowiska innych procesów z `/proc/<pid>/environ`, a także aktywnie wyrabia świeże tokeny GitHub Actions OIDC, Azure DevOps OIDC i Azure Managed Identity. Dane trafiają (zlib + Base64, HTTPS POST, User-Agent `runtime-diagnostics/1`) na adres webhook.site. PyPI poddał kwarantannie konto wydawcy Microsoftu, ale wersja 1.26.2 pojawiła się już po kwarantannie 1.26.1, więc atakujący prawdopodobnie nadal ma uprawnienia publikacji.
**Analiza:** Wektor `.pth` omija założenie „nie importujemy, więc nie wykonujemy” – wystarczy instalacja w obrazie CI lub w venv agenta. Aktywne wyrabianie tokenów OIDC/MI oznacza, że skutek nie kończy się na sekretach statycznych: atakujący dostaje krótkotrwałe, ale w pełni uprawnione tokeny federacyjne do chmury. Mitygacje: przypięcie wersji z hashami (`--require-hashes`), opóźnienie przyjmowania nowych wersji (cooldown 3–7 dni), skan `site-packages/*.pth` w obrazach, ograniczenie `id-token: write` w workflow do jobów, które go potrzebują, oraz warunki trust policy OIDC zawężone do repo/gałęzi/środowiska. Instalację którejkolwiek z wersji traktować jako pełne przejęcie poświadczeń osiągalnych z danego środowiska.
Źródło: [OpenSourceMalware](https://opensourcemalware.com/blog/agent-framework-pypi-compromised)

<hr>

#### Intel–Microsoft: wieloletnie dostawy Xeon „Diamond Rapids” i pakowanie EMIB dla własnych układów Azure (18.11) (TLDR IT)
W ramach długoterminowej umowy Intel potwierdził wieloletnie moce dostaw Xeon dla Microsoftu, w tym następnej generacji „Diamond Rapids”. Microsoft jako pierwsza firma ogłosił użycie pakowania EMIB z Intel Foundry dla własnych układów Azure (bez wskazania, czy chodzi o Maia, czy Cobalt). Dodatkowo: maszyny Azure M-series v4 z pamięcią CXL i Intel Flat Memory Mode (do 2,3× wydajności wobec konfiguracji tylko z DRAM, porównanie 76 vCPU z 1 TB DRAM + 1 TB CXL vs 64 vCPU DRAM), pierwsze VM z QAT w chmurze (do 2,5× szybsze backupy SQL) oraz migracja poufnych VM TDX z szyfrowanym stanem.
**Analiza:** EMIB to alternatywa dla CoWoS TSMC, którego przepustowość jest wąskim gardłem dla akceleratorów AI. Druga ścieżka pakowania zmniejsza ryzyko podażowe dla własnych chipów Azure. Flat Memory Mode z CXL to praktyczne rozszerzenie pojemności dla baz in-memory (SAP HANA) kosztem dodatkowej latencji przy dostępie do warstwy CXL – wynik zależy od lokalności danych. Dla planowania: SKU M-series v4 dają większą pamięć na vCPU, co może obniżyć koszty licencji liczonych per rdzeń.
Źródło: [Intel Newsroom](https://www.intel.com/content/www/us/en/newsroom/news/corporate/microsoft-azure-infrastructure-collaboration.html)

<hr>

#### AWS ExtendDB 1.0: adapter API DynamoDB na PostgreSQL, SQLite, Cassandra i MongoDB (18.11) (TLDR)
AWS wydał open source (Apache 2.0) adapter zgodny z API DynamoDB, który pozwala uruchamiać aplikacje DynamoDB on-prem, w innych chmurach i na brzegu na czterech backendach. Wariant z SQLite służy do lokalnego developmentu i CI bez zewnętrznej bazy. Komunikat nie precyzuje granic zgodności (transakcje, Streams, TTL, GSI z eventual consistency).
**Analiza:** To ruch, który obniża koszt wyjścia z DynamoDB i jednocześnie ułatwia jego adopcję w środowiskach hybrydowych i suwerennych. Semantyka DynamoDB (warunkowe zapisy, limity partycji, model spójności) odwzorowana na PostgreSQL czy Cassandrze będzie miała inne charakterystyki latencji i inne tryby awarii – testy zgodności i obciążeniowe są konieczne przed produkcją. Pierwsze sensowne zastosowanie: zastąpienie DynamoDB Local w CI i środowiska brzegowe z odłączeniem od sieci.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/aws-extenddb-1-0/)

<hr>

#### Epoch AI: wydatki Anthropic na compute ok. 9,86 mld USD w Q3 2026, wzrost ok. 4× rocznie (18.11) (TLDR AI)
Na podstawie S-1/A Epoch wylicza wydatki Anthropic na compute i infrastrukturę: 9 863 mln USD w Q3 2026 (ok. 39,5 mld USD w ujęciu rocznym), z czego około połowa (ok. 5 mld USD) trafiła do SpaceX. Tempo wzrostu to ok. 4× rocznie (433 mln USD w 2023, 2,52 mld w 2024) wobec ok. 3× w OpenAI (dane OpenAI z mediów, nie z audytowanych sprawozdań).
**Analiza:** Koncentracja połowy kwartalnych wydatków u jednego dostawcy (SpaceX) to istotne ryzyko kontrahenta dla obu stron: dla Anthropic – zależność od harmonogramu jednego operatora, dla SpaceX – zależność przychodów compute od jednego najemcy. Przy tempie 4× rocznie i ok. 81 mld USD z IPO kapitał na 2027 jest zabezpieczony, ale skala zobowiązań wobec dostawców mocy rośnie szybciej niż przychody, co trzeba śledzić w pierwszych raportach kwartalnych ANT.
Źródło: [Epoch AI](https://epoch.ai/data-insights/anthropic-openai-compute-spend-growth)

<hr>

#### OpenAI: modele proponują zmiany w projekcie układu – 10–13% mniej powierzchni w kluczowych blokach chipa Jalapeño, do 14% w projektach GPU AMD (TLDR AI, 19.11)
Według OpenAI zmiany projektowe zaproponowane przez jej modele zmniejszyły szacunkową powierzchnię kluczowych bloków własnego akceleratora Jalapeño o 10–13%. We współpracy z AMD to samo podejście dało do 14% poprawy w projektach flagowych GPU AMD. Strona źródłowa jest niedostępna dla automatycznego pobrania (403), więc szczegóły metodologii (które bloki, jaka metryka „poprawy” u AMD) nie zostały zweryfikowane.
**Analiza:** Redukcja powierzchni o ponad 10% w blokach arytmetycznych przekłada się bezpośrednio na uzysk z wafla i koszt jednostkowy albo na miejsce na więcej SRAM/jednostek. Jeśli wynik się potwierdzi, skróci to cykl projektowy akceleratorów własnych hyperscalerów i laboratoriów, co zwiększa presję konkurencyjną na Nvidię w perspektywie 2028+. Liczby pochodzą od samego OpenAI – czekać na niezależne potwierdzenie (np. publikację na ISSCC/Hot Chips).
Źródło: [OpenAI – Advancing chip design with our models](https://openai.com/index/advancing-chip-design-with-our-models/)

<hr>

#### Prime Intellect Model Factory: jedna płaszczyzna sterowania dla pretreningu, SFT, RL i inferencji (18.11) (TLDR AI)
Platforma łączy trenera, orkiestrator rolloutów RL, inferencję i do „dziesiątek tysięcy” współbieżnych sandboxów microVM do wykonywania kodu i wywołań narzędzi. Dla długich przebiegów: health checki, automatyczna wymiana węzłów i wznawianie od ostatniego checkpointu. Podany wynik: 48% MFU na H200 dla architektury MoE Qwen 235B (Prime Mega MoE). Rozliczenie per GPU-h, wdrożenia dedykowane z bezpośrednim dostępem do Slurm/Kubernetes. Ślady produkcyjne (Prime Traces) zasilają kolejne przebiegi treningowe.
**Analiza:** 48% MFU dla dużego MoE na H200 to wynik konkurencyjny (all-to-all ekspertów zwykle obniża MFU poniżej 40%). Model „trening + inferencja na wspólnej infrastrukturze” poprawia wykorzystanie GPU, ale wymaga izolacji priorytetów: rollouty RL i ruch produkcyjny konkurują o te same węzły. Lock-in jest umiarkowany dzięki dostępowi do Slurm/K8s, ale format środowisk i verifierów jest specyficzny dla Prime.
Źródło: [Prime Intellect](https://www.primeintellect.ai/blog/introducing-model-factory)

<hr>

#### Backdoor w modelu open-weight aktywowany tylko w konkretnym harnessie agenta (TLDR InfoSec)
Paolo Galeone dostroił Qwen3-8B adapterem QLoRA na jednej karcie RTX 3090 (ok. 1 EUR energii) tak, że niewinne pytanie o datę 23.05.1992 powoduje ciche wywołanie narzędzia bash z pustą widoczną odpowiedzią. Trening na rzeczywistym prompcie systemowym harnessu „pi” sprawia, że backdoor odpala się tylko w tym harnessie, a w Claude Code czy Codex pozostaje uśpiony. Scalony adapter da się wykryć porównaniem wag z oficjalną bazą (wagi layernorm pozostają niezmienione), ale nie dotyczy to modeli hostowanych przez API.
**Analiza:** Ewaluacja bezpieczeństwa modelu w innym harnessie niż produkcyjny nie wykryje takiego backdooru – testy muszą używać produkcyjnego promptu systemowego i zestawu narzędzi. Dla modeli open-weight pobieranych z hubów: weryfikacja hashy wag wobec oficjalnych wydań i diff wag dla wariantów „fine-tuned”. Niezależnie od źródła modelu: sandbox dla wywołań shell/sieć i zatwierdzanie akcji wysokiego ryzyka (por. wcześniej opisany „Lazarus” Microsoftu).
Źródło: [pgaleone.eu](https://pgaleone.eu/ai/2026/11/14/backdooring-llms-and-trust/)

<hr>

#### VAST Data: 10 mld USD zobowiązań od 14 klientów i ponad 200 EB flash zakontraktowane do 2029 (Own news / HPCwire)
VAST podał, że w trzech pierwszych kwartałach roku fiskalnego 14 klientów zobowiązało się do wydatków powyżej 10 mld USD łącznie (każdy powyżej 100 mln USD, czterech powyżej 1 mld USD). Firma zabezpieczyła u dostawców ponad 200 EB pojemności flash do 2029 r., bez ujawnienia dostawców i rozkładu rocznego. Kwoty obejmują oprogramowanie i sprzęt (pass-through) i nie są tożsame z rozpoznanym przychodem. Ogłoszenie pochodzi z ok. 6–10.11, HPCwire opisał je ponownie 18.11.
**Analiza:** Długoterminowe kontrakty na NAND u dostawcy pamięci masowej pokazują, że przy rosnących cenach SSD dostępność flash staje się ograniczeniem planowania pojemności dla klastrów AI podobnie jak HBM dla GPU. Dla kupujących: przy zamówieniach na 2027 sprawdzić, czy dostawca storage ma zabezpieczoną podaż NAND i jak przenosi zmiany cen. Koncentracja (czterech klientów to większość kwoty) zwiększa ryzyko dostawcy przy ewentualnym spowolnieniu laboratoriów frontier.
Źródło: [HPCwire](https://www.hpcwire.com/aiwire/2026/11/18/vast-data-reports-10b-in-commitments-as-ai-storage-spending-surges/), [Blocks & Files](https://www.blocksandfiles.com/flash/2026/11/05/vast-data-gets-vast-increase-in-orders-for-its-all-flash-storage/5307771)

<hr>

#### Aktualizacja: Nvidia przyspiesza podwyżki i przeglądy roczne z marca na grudzień; przewodniczący rady wyklucza powrót Huanga (TLDR, 18.11)
Na pierwszym all-hands po odejściu Huanga tymczasowa CEO Colette Kress ogłosiła przeniesienie podwyżek i rocznego cyklu ocen z marca na grudzień. Nowy przewodniczący rady potwierdził, że rada nie zamierza przywracać Huanga.
**Analiza:** Przyspieszenie podwyżek to działanie retencyjne – sygnał, że zarząd obawia się odpływu kluczowych inżynierów w okresie przejściowym. Dla klientów istotne jest to, czy zmiany kadrowe nie spowolnią roadmapy (Rubin Ultra, CUDA); pierwszym testem będą zeznania przed Kongresem 1.12.
Źródło: [CNBC](https://www.cnbc.com/2026/11/18/nvidia-interim-ceo-kress-addresses-staff-after-jensen-huang-exit-.html)

<hr>

#### Gemini 4 Neon: 61 pkt w Artificial Analysis Index, 3/15 USD za 1M tokenów, kontekst 1M (19.11) (TLDR AI)
Google wprowadził Gemini 4 Neon (kodowanie, praca z wiedzą, nauka) w Gemini, Antigravity i API: 61 pkt w Artificial Analysis Intelligence Index, cena 3 USD (wejście) / 15 USD (wyjście) za 1M tokenów, cache 0,15 USD, kontekst 1M tokenów. Równolegle pomiar społeczności pokazuje, że Antigravity unieważnia cache promptu po 5 min bezczynności, a pierwszy przebieg zaczyna się od 13,8–16,8 tys. tokenów instrukcji systemowych, schematów MCP i skills.
**Analiza:** Rozpiętość cen w rodzinie Gemini 4 (Hydrogen 1,50/7,50, Neon 3/15) wzmacnia potrzebę routingu między modelami na poziomie bramy LLM. Koszt sesji agentowych zależy w dużej mierze od trafień w cache i stałego narzutu kontekstu – wyłączanie nieużywanych serwerów MCP i utrzymywanie cache to dźwignie kosztowe porównywalne ze zmianą modelu.
Źródło: [Google Blog](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-neon/), [asubedi.com.np](https://asubedi.com.np/blog/why-antigravity-eats-weekly-quota-fast/)

## Implikacje praktyczne

1. **Warstwa ingress jako pojedynczy punkt ryzyka.** NGINX (HTTP/2), PHP-FPM (smuggling na połączeniach keepalive) i wcześniej Apache mod_http2 pokazują, że implementacje HTTP/2/3 i reużycie połączeń backendowych to powtarzalne źródło krytycznych RCE. W horyzoncie 3–5 lat planować możliwość szybkiej wymiany lub przebudowy obrazów ingress w całej flocie (godziny, nie tygodnie), dywersyfikację implementacji na brzegu (CDN/WAF terminujące HTTP/2 przed originem) oraz blokowanie nieużywanych modułów jądra (SCTP), które zamieniają błąd w aplikacji w ucieczkę z kontenera.
2. **Płaszczyzny zarządzania wirtualizacją i tożsamością poza zasięgiem sieci produkcyjnej.** vCenter (vmdir, SAML), Puppet, Expressway i LiteLLM to cele, których przejęcie daje kontrolę nad całą flotą lub wszystkimi sekretami. Docelowa architektura: dedykowany segment zarządzania z dostępem przez bastion, rotowalne materiały podpisujące (STS/SAML, CA) z gotowymi procedurami rotacji po incydencie i inwentarz SBOM pozwalający w godzinach ustalić ekspozycję na komponenty takie jak Apache Axis.
3. **Łańcuch dostaw pakietów agentowych.** Incydent `agent-framework` (stealer z oficjalnego konta, wyrabianie tokenów OIDC/Managed Identity) i zmiana podpisów Chainguard (Cosign ≥3.1.1 od 15.12) wymagają: przypinania zależności z hashami, cooldownu nowych wersji, minimalnych uprawnień `id-token` w CI, wąskich trust policy OIDC oraz testowania aktualizacji narzędzi weryfikacji podpisów przed terminami dostawców.
4. **Multicloud przez zarządzane interconnecty, ale z niezależną ścieżką zapasową.** AWS–Azure (GA) i Google–Azure (preview) obniżają koszt i czas budowy łączy międzychmurowych, a AWS Multipath Resilience daje 1+1 na backbone. Wszystkie te mechanizmy są jednak specyficzne dla dostawców i współdzielą ich płaszczyzny sterowania. Przy projektowaniu DR na 2027+ zachować co najmniej jedną ścieżkę niezależną od dostawcy chmury i modelować koszt egressu po obu stronach.
5. **Ryzyko finansowania DC przenosi się na najemców.** Banki sprzedają kredyty DC (HSBC, ING), kupują ochronę kredytową (Barclays), spready DC ABS rosną, a hyperscalerzy robią sale-leaseback (AWS–Blackstone). Przy wieloletnich kontraktach na kolokację i GPU sprawdzać strukturę finansowania dostawcy, klauzule na wypadek zmiany właściciela lub wierzyciela, a w Europie także ekspozycję dostawcy na regułę BIS o zdalnym dostępie (przykład Nscale/ByteDance).

## Trend tygodnia

Tydzień zamyka się wyraźnym rozjazdem między rynkiem kapitałowym dla laboratoriów AI a rynkiem dla infrastruktury pod nimi. Anthropic zakończył pierwszy dzień notowań 17% powyżej ceny IPO, ale tego samego dnia spadały CoreWeave, Nebius, AMD i Nvidia, a banki aktywnie sprzedawały i ubezpieczały kredyty na DC, przy rosnących spreadach DC ABS. Ryzyko przesuwa się z dostępności GPU na koszt i dostępność kapitału, zgody lokalne na budowę oraz zgodność eksportową – Nscale staje się pierwszym nazwanym europejskim operatorem, którego klienta obejmuje reguła BIS o zdalnym dostępie. Po stronie technologii najwięcej uwagi dostają mechanizmy, które poprawiają wykorzystanie istniejącej mocy: Dynamo dla RL, dezagregacja, akceleratory wyszukiwania wektorowego, akwizycja danych wprost do GPU. W bezpieczeństwie dominuje seria krytycznych błędów w podstawowych komponentach internetu (NGINX, PHP, Drupal, vCenter), często znajdowanych przez skanery AI, przy coraz krótszym czasie od publikacji PoC do exploitacji. Jednocześnie hyperscalerzy standaryzują łącza międzychmurowe na wspólnej specyfikacji, co obniża barierę multicloud, ale tworzy nowe, wspólne domeny awarii. Dla architektów wspólny wniosek: odporność trzeba projektować na poziomie zależności (dostawcy, wierzyciele, regulacje, implementacje protokołów), a nie tylko pojedynczych komponentów.

## To obserwować

- **Exploitacja NGINX CVE-2026-123007/123633 i vCenter CVE-2026-123890:** pojawienie się publicznych PoC, wpisy do CISA KEV, tempo łatania mierzone przez Censys (8,8 mln potencjalnie narażonych usług) oraz poprawka jądra dla SCTP UAF.
- **Wynik BWIC ING (portfel ok. 500 mln USD) i cena transzy Project Jupiter:** pierwszy rynkowy punkt odniesienia dla dyskonta kredytów DC; równolegle spready 5-letnich DC ABS (2,57 pp) i rentowność obligacji CoreWeave 2031.
- **Nscale: decyzja o wniosku licencyjnym BIS dla ByteDance (termin 90 dni) i reakcja Macquarie;** harmonogram IPO na NYSE i rozbudowa VR NVL72.
- **ANT po pierwszym tygodniu notowań i tygodniowe wydatki firm na tokeny:** czy wzrost wydatków (+3% m/m) przyspiesza na tyle, by pokryć ponad 70 mld USD zobowiązań compute w 12 miesięcy.
- **Chainguard 15.12 (Cosign ≥3.1.1) i kolejne kompromitacje pakietów z oficjalnych kont wydawców** (`agent-framework`) – stan przywrócenia konta Microsoftu na PyPI.
