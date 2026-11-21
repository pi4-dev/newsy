# Raport Zbiorczy IT – 2026-11-21

Okres: od edycji z 2026-11-20 do 2026-11-21 rano (Europe/Warsaw). Najważniejsze nowe zdarzenia:
- kapitulacja na rynku AI 20.11: Nvidia −14%, siódmy z rzędu spadkowy tydzień; SOX w najgorszym tygodniu od 2001 r.;
- S-1 Anthropic ujawnia ok. 900 mld USD zobowiązań obliczeniowych, w tym 186 mld USD wypowiadalnych lub warunkowych;
- KVM CVE-2026-99312: odczyt pamięci hosta przez gościa na AMD z włączoną zagnieżdżoną wirtualizacją;
- sześć krytycznych luk F5 BIG-IP;
- koniec poprawek dla SonicWall Gen 6;
- Microsoft ujawnił liczby z agentowego NetOps: ponad 100 agentów obsługuje 1,6 mln urządzeń sieciowych.

Sekcja „euro neocloud” jest pominięta, bo w tym oknie nie było nowych, zweryfikowanych zdarzeń. Spadek Nebius z 20.11 opisuję w sekcji ryzyk.

## Raport technologiczny

### Technologia

#### Linux KVM CVE-2026-99312: gość VM czyta i zapisuje pamięć hosta na AMD z zagnieżdżoną wirtualizacją (publikacja 18–20.11)
**Kogo dotyczy:**
- wyłącznie hosty AMD z włączoną zagnieżdżoną wirtualizacją, która w `kvm_amd` jest domyślnie włączona; hosty Intel nie są podatne;
- jądra od 5.15.81 i 6.0.11 wzwyż. Błąd wprowadziła poprawka CVE-2022-3344 z 2022 r.

**Skala:** badacz Terra Security odczytał w DigitalOcean ok. 127 GB RAM hosta. Na własnym serwerze zapisał 32 wybrane bajty do strony jądra hosta.

**Status poprawek:**
- jądra z poprawką: 7.3.5, 6.18.64, 6.12.118;
- Ubuntu i Debian stable nie mają jeszcze paczek, Red Hat odracza poprawkę dla EL9/EL10;
- obejście: `options kvm_amd nested=0`.

**Znaczenie:** To realna ścieżka ucieczki między tenantami w IaaS i w chmurach prywatnych na EPYC (OpenStack, Proxmox, OpenNebula).
- Wyłączenie zagnieżdżonej wirtualizacji psuje workloady, które z niej korzystają: VBS/Credential Guard w gościach Windows, sandboxy CI z KVM-in-VM oraz Kata/Firecracker uruchamiane wewnątrz VM.
- Przed zmianą trzeba zrobić inwentaryzację takich workloadów.
- Do czasu wydania paczek dystrybucyjnych realne opcje są dwie: własne jądro albo wyłączenie nested.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/new-linux-kvm-flaw-cve-2026-99312-lets.html)

<hr>

#### F5 BIG-IP: sześć luk pre-auth w iRules i APM, pięć z CVSS 9,8 (advisory 18–19.11, szersze nagłośnienie 20.11)
F5 wydało poza cyklem poprawki przepełnień bufora na stosie. Nieuwierzytelniony atakujący może dzięki nim wywołać awarię TMM, a potencjalnie RCE.

**Luki z CVSS 9,8:**
- CVE-2026-121805: iRule z `PEM::subscriber`;
- CVE-2026-120292: `MR::message route`;
- CVE-2026-120285: `PSC::ip_address`;
- CVE-2026-116953: `GENERICMESSAGE::route`;
- CVE-2026-123637: APM, per-request policy z agentami Client Information/Endpoint Security.

**Pozostałe:** CVE-2026-126822 (DoS w APM SAML).

**Wersje z poprawkami:**
- BIG-IP 17.1.3.7, 17.5.1.11, 21.1.0.4 (hotfixy ENG);
- CNF/BNK 2.2.5, 2.3.4, 2.4.2-EHF1;
- BIG-IQ 8.4.2.3.

F5 nie podaje mitygacji. Exploitacji nie zgłoszono.

**Znaczenie:** To druga krytyczna luka APM w ciągu dwóch miesięcy. Wrześniowa CVE-2026-94127 była zero-dayem.
- Podatność zależy od konfiguracji. Najpierw trzeba przejrzeć iRules pod kątem wymienionych komend (grep po konfiguracji), bo to szybko zawęża zakres pilnego łatania.
- W środowiskach Service Provider (PEM, MRF/Diameter/SIP) ekspozycja jest szczególnie wysoka.
- Przy klastrach HA kolejność aktualizacji (standby → failover → active) i failover TMM mogą zerwać sesje APM.

Źródło: [Canadian Centre for Cyber Security AV26-1178](https://cyber.gc.ca/en/alerts-advisories/f5-security-advisory-av26-1178), [Network World](https://www.networkworld.com/article/4251676/f5-scrambles-to-fix-more-critical-big-ip-security-flaws.html)

<hr>

#### VMware VMSA-2026-0010: RCE w dołączanym Apache httpd (CVE-2026-119618, CVSS 9,8) w VCF, Aria Operations i HCX bez poprawki (19–20.11)
Błąd w `mod_http2` Apache httpd, opisany w edycji z 14.11, występuje też w Apache dostarczanym z:
- VMware Cloud Foundation;
- Aria Operations;
- VCF Operations;
- HCX.

VMware nie ma jeszcze łatki. Jedynym obejściem jest wyłączenie HTTP/2.

**Znaczenie:**
- Płaszczyzny zarządzania VCF pozostają z luką 9,8 bez poprawki tuż po zero-dayu w vCenter (VMSA-2026-0009).
- Obejście trzeba wdrożyć od razu, a interfejsy zarządzania odciąć od sieci użytkowników.
- Skanery oparte na wersji pakietu nie wykryją problemu w appliance'ach, w których Apache jest zaszyty.

Źródło: [The Register](https://www.theregister.com/security/2026/11/20/flaws-in-vmware-and-kvm-need-attention-if-you-have-time-while-fixing-java-php-and-everything-else/5311193)

<hr>

#### CISA KEV (20.11): SAP NetWeaver i Apache Roller; Cisco Expressway i vCenter z formalnym statusem KEV
CISA dodała do katalogu cztery luki:
- CVE-2025-42884: SAP NetWeaver;
- CVE-2026-76466: Cisco Expressway;
- CVE-2026-123890: vCenter;
- CVE-2026-129448: Apache Roller.

**Nowość: Apache Roller.**
- Deserializacja w XML-RPC daje RCE bez uwierzytelnienia w wersjach 4.0.0–6.1.4, CVSS 9,8.
- Apache potwierdza aktywną exploitację.
- Wyłączenie XML-RPC w panelu administracyjnym nie chroni starszych wersji.
- Obejście: `enabledForExtensions=false` w `WEB-INF/web.xml` i usunięcie mapowania `/roller-services/xmlrpc`.
- Poprawkę zawiera wersja 6.1.6. Wersja 6.1.7 usuwa dodatkowo 15 błędów.

**Znaczenie:** Expressway i vCenter, opisane 20.11, mają teraz formalne terminy naprawy z BOD. Roller to typowy zapomniany wewnętrzny portal na Javie, więc warto sprawdzić inwentarz aplikacji, również tych w DMZ.

Źródło: [CISA](https://www.cisa.gov/news-events/alerts/2026/11/20/cisa-adds-four-known-exploited-vulnerabilities-catalog), [SecurityWeek](https://www.securityweek.com/critical-apache-roller-axis-vulnerabilities-exploited-in-attacks/)

<hr>

#### SonicWall: firewalle Gen 6 bez poprawek, także krytycznych (20.11)
Urządzenia z SonicOS 6.x nie dostaną już żadnych poprawek bezpieczeństwa. Dotyczy to modeli:
- TZ300–TZ600 i SOHO;
- NSA 2600–6600 i NSa 2650–9650;
- NSv;
- SuperMassive 9200–9800;
- NSsp 12400/12800.

Producent każe traktować każde działające urządzenie Gen 6 jako „high-risk exposure”. Zaleca migrację na Gen 8, a do tego czasu:
- wyłączenie zarządzania od strony WAN (HTTPS, SSH, SNMP);
- ograniczenie SSL VPN do znanych adresów;
- MFA.

Według przywołanych danych Talos 23% CVE z 2025 r. dotyczyło urządzeń brzegowych.

**Znaczenie:**
- Wymiana jest nieplanowanym capexem dla oddziałów i MŚP.
- Dochodzi do tego ryzyko terminów: wymiana zbiega się z niedoborem PCB, opisanym w sekcji AI-ML, który wydłuża dostawy sprzętu sieciowego.
- Migracji konfiguracji z SonicOS 6 na 8 nie da się w pełni zautomatyzować. Polityki SSL VPN i obiekty trzeba zweryfikować ręcznie.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/sonicwall-warns-gen-6-firewalls-wont.html)

<hr>

#### FastUnlock: niezałatany zero-day ekranu blokady Windows (ujawnienie 19.11, weryfikacja 20.11)
Przy fizycznym dostępie technika pozwala uruchamiać programy z ekranu blokady. Wykorzystuje Narratora i ścieżki odzyskiwania konta Microsoft.
- ThreatLocker potwierdził jej działanie: programy startują na ograniczonym koncie WSIAccount.
- BitLocker z samym TPM nie chroni, bo dysk jest już odszyfrowany.
- Microsoft nie wydał poprawki ani advisory.

**Znaczenie:** Dla floty laptopów trzeba wymusić BitLocker z PIN-em przy starcie (TPM+PIN) i hibernację zamiast uśpienia. Rośnie przez to obciążenie helpdesku (reset PIN-ów) i wydłuża się czas wybudzania.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/new-unpatched-windows-0-day-fastunlock.html)

<hr>

#### N-able Mail Assure/SpamExperts: włamanie do chmury, zarządzanie wyłączone w trzech regionach (16–20.11)
**Przebieg:**
- 16.11 zaczęły się zakłócenia;
- 19.11 N-able potwierdziło, że to incydent bezpieczeństwa: nieznany sprawca uzyskał dostęp do serwera w Hosted Cloud;
- według aktualizacji z 20.11 (8:06 UTC) poczta jest dostarczana i filtrowana, ale portal, API, kwarantanna i logowanie dla sześciu usług w Amerykach, APAC i Europie nadal nie działają.

Firma nie ma dowodów na dostęp do danych klientów. To incydent odrębny od luk w N-central opisanych wcześniej.

**Znaczenie:** MSP od kilku dni nie widzą logów poczty ani kwarantanny klientów. Utrudnia to obsługę incydentów phishingowych i dochodzenia.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/n-able-confirms-hackers-accessed-cloud.html)

### Biznes

#### JPMorgan, Visa i Mastercard: wyciek danych marketingowych przez dostawców zewnętrznych (19–20.11)
Grupa N0n twierdzi, że wykradła dane wszystkich trzech firm. Firmy wskazują na incydenty u nienazwanych dostawców, w przypadku Visy u dostawcy z APAC.
- Reuters znalazł w próbce JPMorgan ponad 16 tys. e-maili klientów.
- Systemy płatnicze i numery kart nie zostały naruszone.
- Na stronie wycieków N0n jest ok. 200 ofiar.

**Znaczenie:** Ryzyko leży w łańcuchu dostawców martech, czyli poza kontrolą SOC. Dane posłużą do celowanego phishingu. Trzeba przejrzeć, jakie dane i z jakim okresem retencji trafiają do zewnętrznych platform marketingowych.

Źródło: [SecurityWeek](https://www.securityweek.com/jpmorgan-visa-mastercard-cite-third-party-incidents-amid-data-leak/)

## Raport AI-ML

### Biznes

#### Niedobór płytek PCB porównywalny z kryzysem chipów: zamówienia +56,1% r/r przy dostawach +13,3% (raport Global Electronics Association, 20.11)
**Przyczyna:** płyty serwerów AI mają 40–60+ warstw wobec ok. 12 w PC i zużywają nieproporcjonalnie dużo laminatu, folii miedzianej i żywic.

**Wskaźniki:**
- czas dostawy laminatu FR-4: 6–8 tygodni wobec wcześniejszych 2–3;
- czas dostawy laminatu niskostratnego: 14–18 tygodni;
- laminat +30% we wrześniu, część tkaniny E-glass +300%, miedź +37% r/r.

Gartner potwierdza, że moce produkcyjne przesuwają się do PCB o wysokiej liczbie warstw. Ulgi nie należy oczekiwać przed połową 2028 r.

**Znaczenie dla capacity planning:**
- To kolejne wąskie gardło poza GPU i HBM.
- Bardziej niż szafy AI ucierpią zwykłe switche, routery i serwery x86, bo fabryki priorytetyzują płyty o wysokiej marży.
- Zamówienia na odświeżenie sprzętu w 2027 r. warto składać z wyprzedzeniem.
- W kontraktach warto zabezpieczyć cenę: BOM rośnie, a ponad 90% płyt produkuje się w Azji.

Źródło: [The Register](https://www.theregister.com/systems/2026/11/20/ai-boom-threatens-circuit-board-crunch-to-rival-pandemic-era-chip-shortage/5311335)

<hr>

#### TrendForce: ceny NAND zaczną spadać w 2027 r., bo chińskie chmury tną zamówienia SSD (19.11 wieczorem)
**Przyczyny:** chińscy operatorzy mają za mało GPU po regule BIS, za duże zapasy i wojnę cen tokenów. Dlatego mocno obcięli zamówienia enterprise SSD na 2027 r.

**Prognoza:**
- ceny kontraktowe NAND: 0 do −5% k/k na początku 2027 r.;
- od Q2 2027 spadki we wszystkich kategoriach;
- podaż bitów +25–30% dzięki QLC i większej liczbie warstw.

**Znaczenie:**
- Pamięć masowa dla AI (offload KV cache, checkpointy, data lake) stanieje, podczas gdy DRAM i HBM pozostaną deficytowe.
- To pierwszy twardy sygnał, że ograniczenia eksportowe GPU przenoszą się na popyt w łańcuchu pamięci.
- Projekty storage warto przesunąć na 2H 2027, jeśli harmonogram na to pozwala.

Źródło: [The Register](https://www.theregister.com/systems/2026/11/19/nand-prices-tipped-to-fall-as-chinese-cloud-giants-cut-ssd-orders/5311116)

<hr>

#### Lagarde: fundusze UE powinny przyjmować pierwszą stratę w projektach DC i fabach (20.11)
W ostatnim wystąpieniu jako szefowa EBC Lagarde zaproponowała partnerstwa publiczno-prywatne, w których pieniądze publiczne przyjmują transzę first-loss.
- Podniesienie udziału Europy w globalnej mocy obliczeniowej AI z ok. 5% do 15% do 2030 r. wymaga według przytoczonego szacunku ok. 1,3 bln EUR.
- Plan do 7 AI Gigafactories uznała za niewystarczający.

**Znaczenie:**
- Taki instrument obniżyłby koszt kapitału europejskich neocloudów.
- Nie istnieje jednak, a przetarg EuroHPC przesunięto na 02.2027.
- W horyzoncie 12–18 miesięcy europejska moc AI nadal opiera się na drożejącym kapitale prywatnym (zob. sekcja ryzyk).

Źródło: [CNBC](https://www.cnbc.com/2026/11/20/lagarde-eu-data-center-funding.html)

<hr>

#### Hyperion Research: rynek on-prem HPC/AI-Sci +19,9% w 1H 2026 przy liczbie jednostek tylko +4,4% (The Next Platform, 20.11)
**Rynek:** sprzedaż systemów wyniosła 16,48 mld USD. Dell wyprzedził HPE: 3,93 mld USD (+67,7%) wobec 3,44 mld USD (+1,1%).

**Prognoza:** wydatki on-prem i chmura mają urosnąć z ok. 40 mld USD (2022) do 135 mld USD (2030). Udział chmury ma wynieść ok. 22%.

**Znaczenie:**
- Wzrost wartościowy wynika głównie z inflacji cen komponentów (HBM, DRAM, PCB) i z większej liczby węzłów w systemie, a nie z większej liczby instalacji.
- Według szacunku autora HPC i AI-Sci to tylko 3–4% z 31–37 mln zainstalowanych GPU i XPU. Segment naukowy jest cenobiorcą wobec zakupów hyperscalerów.

Źródło: [The Next Platform](https://www.nextplatform.com/hpc/2026/11/20/casing-the-hpc-and-ai-sci-market-with-hyperion-research/5311388)

### Technologia

#### Azure (Ignite, 20.11): szerszy preview VM na Vera Rubin NVL72, confidential computing na bare-metal, Foundry „Programmed Training”
**Vera Rubin NVL72:** szafy działają w DC Azure, a Microsoft poszerza dostęp do VM w preview. Nie podał regionów, cen ani parametrów VM.

**Confidential computing:** atestacja NVIDIA obejmuje teraz także bare-metal i infrastrukturę rack-scale.

**MLPerf:** Azure podaje skalowanie MLPerf Training v6.1 do 10 240 GPU GB200 NVL72, czyli ok. 142 szaf, na modelu DeepSeek-V3.

**Foundry „Programmed Training”:** bezserwerowy post-training. EY na Nemotronie uzyskało +55% dokładności i 15× niższy koszt niż na modelach frontier. To deklaracja dostawcy.

**Własny stos Microsoftu:** Microsoft AI używa Dynamo do rolloutów RL, a Bing działa na RTX PRO 6000.

**Znaczenie:**
- Vera Rubin trafia do hyperscalera równolegle z neocloudami (Nscale, Dell), a nie po nich. To skraca okno przewagi czasowej neocloudów.
- Pakiet Foundry + Nemotron + Dynamo + Azure Local (rozszerzony o GPU i Jetson) wiąże klienta ze stosem NVIDIA i Azure Arc od chmury po edge, mimo równoległego rozwoju Maia.
- Bare-metal ogranicza narzut hipernadzorcy, ale wiąże projekt z topologią domeny NVLink 72 GPU.

Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/nvidia-and-microsoft-expand-azures-vera-rubin-preview-and-foundrys-nemotron-lineup/), [NVIDIA Blog](https://blogs.nvidia.com/blog/microsoft-ignite-accelerated-computing-azure-windows-pcs)

<hr>

#### AWS: rozproszony KV cache na ElastiCache (Valkey) + LMCache – do −40% kosztu i −80% TTFT (21.11)
**Mechanizm:** współdzielona warstwa KV obok floty GPU pozwala dowolnemu serwerowi przez wiele dni ponownie używać zapisanego stanu KV: między sesjami, po zmianie routingu i po awarii serwera. Eliminuje to powtarzany prefill długich kontekstów agentów.

**Wymagania:**
- vLLM ≥0.11.1 lub SGLang ≥0.5.13;
- LMCache ≥0.5.8rc1;
- Valkey 9.1 na `cache.i8ge`;
- GPU z RDMA write przez EFA (np. p5).

Działa też z llm-d i Ray Serve.

**Znaczenie i ryzyka:**
- −40% kosztu i −80% TTFT to wartości maksymalne według dostawcy, zależne od powtarzalności prefiksów.
- Przy krótkich promptach odczyt KV przez sieć bywa wolniejszy niż ponowne obliczenie. Trzeba wyznaczyć próg opłacalności i routować według długości prefiksu.
- Failure mode: niezgodny hash lub seed KV albo niezgodny układ KV między wersjami silnika daje dobre statystyki trafień i błędne odpowiedzi. Przed produkcją potrzebne są testy deterministyczności i wersjonowanie kluczy cache przy każdej zmianie modelu lub silnika.
- Lock-in jest umiarkowany, bo LMCache i Valkey są otwarte, ale wymóg EFA RDMA wiąże wdrożenie z instancjami AWS.

Źródło: [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/cut-self-hosted-llm-cost-and-latency-with-the-distributed-kv-caching-agent-skill/)

<hr>

#### SageMaker HyperPod: Global Inference Gateway (routing między regionami) i obserwowalność awarii GPU w treningu (20.11)
**Global Inference Gateway:** druga warstwa nad klastrowym Inference Gateway, który kieruje ruch według zajętości KV cache i głębokości kolejki.
- Jeden zarządzany endpoint kieruje ruch między klastrami i regionami po sieci szkieletowej AWS (PrivateLink, SigV4/IAM), z automatycznym failoverem.
- Przykład AWS: 4 modele na 5 klastrach w 2 regionach dają 20 endpointów; gateway zastępuje je jednym.

**Obserwowalność (EKS i Slurm):**
- wykrywa GPU odstające od reszty (`DetectGroupOutliers`: utylizacja, pamięć, temperatura, moc, zegar SM);
- ma reguły dla błędów XID/ECC, spadków przepustowości EFA i throttlingu;
- domyślne metryki co 30 s kosztują ok. 1–2% CPU i 250–300 MB RAM na węzeł.

**Znaczenie:**
- Oba narzędzia podnoszą utylizację GPU. Gateway przelewa szczyty ruchu do bezczynnej, już opłaconej mocy w innym regionie, ale kosztem dodatkowego RTT i transferu między regionami. Przy danych objętych rezydencją danych w UE routing między regionami wymaga jawnych ograniczeń.
- Detekcja odstających GPU skraca MTTR dla jednego wadliwego GPU, który blokuje zadanie synchroniczne na tysiącach GPU.

Źródło: [AWS – Global Inference Gateway](https://aws.amazon.com/blogs/machine-learning/introducing-amazon-sagemaker-hyperpod-global-inference-gateway-one-endpoint-across-regions-and-clusters/), [AWS – Enhanced observability](https://aws.amazon.com/blogs/machine-learning/introducing-enhanced-observability-in-amazon-sagemaker-hyperpod-to-diagnose-gpu-training-issues/)

<hr>

#### SK hynix MARS: szafa z pulą pamięci CXL na współdzielony KV cache (SC26, 20.11)
MARS (Memory-centric AI Rack System) pokazano w trzech demonstracjach:
- pula pamięci CXL na poziomie szafy do współdzielenia KV cache, także w hybrydzie CXL+SSD z predykcyjnym ładowaniem do DRAM;
- pula pamięci obliczeniowej łącząca serwery i GPU przy obsłudze agentów;
- wymiana tokenów między serwerami CPU przez CXL.

Pokazano też HBM4E, 256 GB 3DS RDIMM i 192 GB SOCAMM2.

**Znaczenie:**
- Razem z AWS (wyżej), Marvell/Penguin i Cerebras/AMD to czwarty w ciągu tygodnia sygnał, że KV cache wychodzi z HBM do hierarchii: HBM → CXL/DRAM → sieć → SSD.
- Do capacity planning trzeba dodać warstwę pamięci kontekstu jako osobny zasób z własnym SLO latencji.
- CXL dodaje kilkaset ns i wymaga dojrzałego wsparcia w jądrze i orkiestratorze. To realny failure mode przy wczesnych wdrożeniach.

Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/sk-hynix-highlights-mars-and-hbm4e-memory-at-sc26/)

<hr>

#### Nvidia Vera: jeden SKU CPU do wszystkich ról (The Register, 20.11)
**Parametry:** 88 rdzeni Olympus, 3,3 GHz, SMT (ok. 176 wątków), do 1,5 TB pamięci, 1,8 TB/s NVLink-C2C.

**Zastosowania:** ten sam układ obsługuje:
- superchipy Vera Rubin;
- samodzielne szafy CPU, w tym szafę HPC ze 192 Vera;
- węzły storage;
- systemy pamięci kontekstu.

Jednostka wektorowa jest słabsza niż u konkurentów x86 i Arm. Nvidia akceptuje to, bo CPU pracuje w parze z GPU.

**Znaczenie:**
- Jeden SKU upraszcza łańcuch dostaw i zarządzanie zapasem części zamiennych.
- Nvidia wchodzi jednak w warstwę hosta i storage, czyli kolejną warstwę lock-in poza GPU.
- Dla klasycznego HPC z intensywnym FP64 na CPU Vera nie jest oczywistym zamiennikiem EPYC.

Źródło: [The Register](https://www.theregister.com/systems/2026/11/20/nvidias-journey-to-forge-the-one-cpu-sku-to-rule-them-all/5311438)

## AI for networking

#### Microsoft (Ignite, 20.11): ponad 100 agentów AI w operacjach sieci z 1,6 mln urządzeń; fabric DC z 3 na 2 warstwy; SLM inline w Azure Firewall
**Agentowy NetOps:**
- Ponad 100 agentów, każdy z własną tożsamością i przypisanym „managerem”, obsłużyło w ciągu roku ponad 30 tys. zgłoszeń i zmniejszyło backlog o ok. 75%.
- Przykład: agent zlokalizował z telemetrii urządzeń przecięcie światłowodu pod Dżakartą z dokładnością do metra ok. 20 minut po zdarzeniu. Potem koordynował ekipę i zweryfikował łącze z obu stron.

**Fabric DC:** przejście z trzech warstw na dwie skraca ścieżkę z 5 do 3 hopów i poprawia efektywność energetyczną o ponad 20%.

**Hollow-core fiber:** ok. 15 tys. km, plan 10× do końca 2028 r., opóźnienie niższe o ok. 1/3.

**Nowe usługi:**
- Azure Firewall AI add-on: dwa małe modele językowe inline wykrywają malware i exploity;
- Agent Grid: mikrosegmentacja oparta na tożsamości agentów.

**Analiza:**
- To jedne z pierwszych twardych wskaźników agentowego NetOps w skali hyperscalera. Warunkiem jest telemetria o wysokiej rozdzielczości (optyka, OTDR, liczniki portów), bez której agent nie zlokalizuje awarii.
- Model „agent z tożsamością i managerem” to wzorzec RBAC do przeniesienia do własnego NetOps: uprawnienia per agent, audyt i eskalacja do człowieka.
- Płaski fabric 2-tier wymaga przełączników o wyższym radixie (51,2T/102,4T). Zmniejsza liczbę elementów i hopów, ale powiększa domenę awarii pojedynczego spine'a.
- SLM inline w firewallu dodaje opóźnienie i koszt na ścieżce danych. Trzeba też ustalić politykę fail-open albo fail-closed na wypadek przeciążenia modelu.
- Wszystkie te funkcje są dostępne tylko w Azure.

Źródło: [Network World](https://www.networkworld.com/article/4251523/whats-next-for-azure-networking.html)

<hr>

#### Amazon Threat Intelligence: agenci AI (framework Cairn) samodzielnie znaleźli zero-day i przejęli serwer uczelni (raport 20.11)
**Kampania:** sprawcy podejrzewani o powiązania z Chinami (prompty w uproszczonym chińskim, znaczniki czasu CST) przez 48 godzin od 8.09 atakowali uczelnie w Azji Wschodniej.

**Udana ścieżka:** agenci znaleźli path traversal w endpoincie uploadu MONAI Label. W ok. 11 minut przeszli od mapowania API do zapisu plików, a potem do reverse shella z uprawnieniami root.

**Nieudane próby:**
- atak na Redis: ok. 16 h, 148 kolejnych akcji;
- credential spraying na serwer GPU: ponad 6 h, 80 prawdziwych loginów.

Kampanię ujawniła publicznie wystawiona instancja Cairn. Opiekunowie MONAI wydali poprawkę.

**Analiza:**
- W odróżnieniu od wcześniej opisanych katalogów narzędzi (Elastic, GTIG) to udokumentowany przypadek autonomicznego odkrycia i exploitacji 0-day w realnej kampanii.
- Czas do kompromitacji poniżej 15 minut jest krótszy niż cykl łatania i typowy czas reakcji SOC. Detekcja musi opierać się na tempie i sekwencji akcji (wiele żądań API w krótkim oknie, systematyczna enumeracja), a na krawędzi potrzebny jest rate-limiting i WAF z limitami per tożsamość.
- Celem były narzędzia ML: MONAI i serwery GPU. Platformy MLOps wystawione do internetu (MONAI, Jupyter, Ray Dashboard, MLflow) to obecnie najsłabszy punkt i powinny działać wyłącznie za ZTNA.

Źródło: [The Register](https://www.theregister.com/security/2026/11/20/china-attackers-deployed-ai-agents-that-autonomously-found-and-exploited-a-0-day-to-take-over-a-university-server/5311382)

## Ryzyka rynku AI/GPU

#### Kapitulacja 20.11: Nvidia −14%, siódmy z rzędu spadkowy tydzień, SOX najgorszy tydzień od 2001 r.
**Nvidia:**
- −14% w piątek, siódma sesja spadkowa z rzędu i siódmy spadkowy tydzień, najdłuższa seria od 2008 r.;
- −23% w tygodniu i ok. −46% w 7 tygodni, utrata ok. 2,6 bln USD kapitalizacji;
- wycena ok. 8,4× przyszłorocznego zysku.

**Pozostali (CNBC):**
- AMD ponad −13%;
- S&P 500 i Dow w korekcie, Nasdaq −10% w tygodniu;
- Nasdaq-100 śródsesyjnie w strefie bessy.

**Dane wtórne** (agregator rynkowy, CNN; nie zweryfikowano ich ze źródłem giełdowym):
- CoreWeave −12%, Nebius −16,1%, Micron −13,9%;
- SOX −10,7% w tygodniu;
- VIX 35,89;
- rentowność 10-letnich obligacji USA 4,92% (−12 pb).

Wśród przyczyn CNBC wymienia doniesienie WSJ (17.11) o dochodzeniu SEC w sprawie ujawnień Nvidii.

**Analiza:**
- Przy wycenie ok. 8× zysku rynek nie wycenia spowolnienia, lecz spadek zysków w 2027–2028 r. Wynika to z marży obniżonej do 71–72% (17.11) i z ryzyka, że klienci stracą dostęp do kapitału.
- Dla neocloudów kluczowy jest koszt kapitału własnego: okno na emisje akcji zamyka się w momencie, gdy CoreWeave ma w 2027 r. ok. 11 mld USD zapadającego długu, a 130 tys. GPU schodzi z kontraktów.
- Spadające rentowności obligacji skarbowych przy rosnącej zmienności oznaczają ucieczkę do bezpiecznych aktywów. Spready kredytowe DC (2,57 pp wobec 1,95 pp w Q1) prawdopodobnie poszerzą się dalej.

Źródło: [CNBC – Nvidia 7-week losing streak](https://www.cnbc.com/2026/11/20/nvidias-7-week-losing-streak-is-longest-since-2008-financial-crisis.html), [CNBC – S&P 500 correction](https://www.cnbc.com/2026/11/20/sp-500-correction-market-sell-off.html), [CNN](https://www.cnn.com/2026/11/20/investing/us-stocks-anthropic-ai)

<hr>

#### Anthropic: S-1 ujawnia ok. 900 mld USD zobowiązań obliczeniowych, w tym ok. 186 mld USD wypowiadalnych lub warunkowych; kurs ANT −8,5% (20.11)
**Notowania i emisja:**
- w drugim dniu notowań ANT spadł o 8,5% do 88,91 USD, wobec 97,17 USD na zamknięciu debiutu;
- greenshoe wykonano w całości: 147 mln akcji, 12,2 mld USD;
- łącznie z prywatnym plasowaniem Anthropic pozyskał 119,5 mld USD brutto.

**Zobowiązania z S-1:**
- ok. 900 mld USD w kontraktach zakupu i najmu mocy obliczeniowej, około połowa do końca 2030 r., reszta w latach 2031–2045;
- ok. 186 mld USD z tego jest wypowiadalne lub warunkowe (zależne od finansowania, lokalizacji i zgód).

**Ryzyko po stronie dostawców:** Lambda nie zabezpieczyła na czas najmu obiektu, przez co transza kontraktu wartego 34,9 mld USD stała się wypowiadalna dla obu stron. Anthropic deklaruje, że nie zamierza z tego korzystać.

**Wzrost:** według CNBC tempo wzrostu przychodów spadło do 47% r/r w Q3 ze 144% w Q2.

**Analiza:**
- 186 mld USD warunkowego backlogu to bezpośrednie ryzyko dla neocloudów, których finansowanie opiera się na kontraktach offtake z Anthropic.
- Klauzule wypowiedzenia uruchamiane brakiem lokalizacji, zasilania lub finansowania przenoszą ryzyko wykonania z klienta na dostawcę.
- Wierzyciele będą dyskontować takie kontrakty przy ocenie zabezpieczeń. To zbiega się z zaostrzeniem kryteriów backstopów przez Nvidię i wycofywaniem się banków.
- Spowolnienie przychodów z +144% do +47% przy rosnących zobowiązaniach to wskaźnik do śledzenia co kwartał.

Źródło: [CNBC – compute access](https://www.cnbc.com/2026/11/20/tech-download-anthropic-compute-access.html), [CNBC – greenshoe](https://www.cnbc.com/2026/11/20/anthropic-ipo-greenshoe.html), [CNBC – ANT stock](https://www.cnbc.com/2026/11/20/anthropic-stock-ai-amd-nvidia.html)

<hr>

#### SoftBank −14,48%, najgorsza sesja od 08.2024; spadki w azjatyckim łańcuchu dostaw AI (20.11)
**SoftBank:** ma 64,6 mld USD zainwestowane w OpenAI. W październiku stracił 30%, a w listopadzie do tej pory 31%.

**Pozostałe spadki:**
- Samsung −7,01%, SK hynix −5,92%, TSMC −4,76%, Kospi −5,21%;
- Tokyo Electron −7,06%, Lasertec −9,21%.

**Analiza:**
- SoftBank jest głównym dostawcą kapitału na Stargate.
- Jego finansowanie opiera się częściowo na kredytach zabezpieczonych akcjami (Arm, a w przeszłości T-Mobile). Spadek wartości zabezpieczeń ogranicza zdolność do dalszego finansowania zobowiązań wobec OpenAI.
- To potencjalny kanał transmisji: kłopoty SoftBanku → opóźnienia lokalizacji Stargate → anulowane lub przesunięte zamówienia na GPU i HBM.
- Spadki producentów sprzętu dla fabów (TEL, Lasertec) sugerują, że rynek zaczyna dyskontować cięcia capexu na fabryki w 2027 r.

Źródło: [CNBC](https://www.cnbc.com/2026/11/20/softbank-ai-tech-stocks-slide-anthropic-debut.html)

<hr>

#### Trump zapowiada zakaz lub „ogromny podatek” dla firm USA budujących DC za granicą (20.11)
**Fakty:**
- zapowiedź padła na briefingu w Białym Domu bez stawki, zakresu i mechanizmu;
- Trump przyznał, że nie konsultował jej z administracją;
- nie wiadomo, czy podatek obejmie tylko DC dla AI, czy także zwykłe regiony chmurowe;
- dotyczy Alphabetu, Amazona, Microsoftu, Meta i Oracle, które nie skomentowały zapowiedzi.

**Zagrożone przykładowo:** 18 mld EUR AWS w Niemczech (2027) i 13 mld EUR Google w Finlandii.

**Analiza:**
- To na razie deklaracja polityczna bez projektu ustawy.
- Mimo to ryzyko podatkowe trafia do modeli finansowych zagranicznych kampusów hyperscalerów, także w Polsce.
- Możliwe skutki: przesunięcie decyzji lokalizacyjnych o kwartały, renegocjacje kontraktów offtake i wzrost ryzyka, że w UE nie zostanie oddana zaplanowana moc.
- Paradoksalnie wzmacnia to argument za europejskimi neocloudami i za ofertami sovereign.
- Dla klientów w UE to kolejny powód, by w strategii wyjścia z chmury uwzględnić scenariusz, w którym region hyperscalera w UE przestaje się rozbudowywać.

Źródło: [CNBC](https://www.cnbc.com/2026/11/20/trump-data-centers-ai-companies-tax.html), [SiliconANGLE](https://siliconangle.com/2026/11/20/trump-announces-plan-to-tax-us-companies-that-build-data-centers-abroad/)

<hr>

#### IEA World Energy Outlook: zużycie energii przez DC ok. 2× do 2030 r. (ok. 1 000 TWh), emisje sektora +70% (19–20.11)
**Scenariusz STEPS:**
- zużycie energii przez DC ok. 2× do 2030 r.;
- szczytowy popyt na moc +40–50% do 2035 r.;
- 26 mln km nowych linii i 16 mln km modernizacji do 2035 r.

**Źródła energii:** przyrost pokrywa głównie generacja kopalna blisko odbioru: gaz w USA, węgiel w Chinach. Stąd wzrost emisji sektora o ok. 70%.

**Analiza:**
- Wąskim gardłem pozostaje przyłączenie do sieci, nie GPU.
- W połączeniu z presją polityczną (zakaz i podatek w USA, sondaże przeciw DC) oraz niedoborem PCB rośnie ryzyko rozjazdu między liczbą DC rozpoczętych a oddanych.
- W UE dodatkowo rośnie ryzyko regulacyjne (CSRD, raportowanie emisji) i cena PPA.
- To argument za mikrosieciami i zasilaniem behind-the-meter, które jednak mają własne ryzyka: dostępność gazu, emisje i lokalne zgody.

Źródło: [CNBC](https://www.cnbc.com/2026/11/19/data-center-electricity-demand-iea.html), [The Guardian](https://www.theguardian.com/world/2026/nov/19/datacentres-to-create-70-more-carbon-emissions-by-2030-iea-report-warns)

## Newsletters summary

Przetworzono 6 z 8 wiadomości z etykietą NEWSY: TLDR, TLDR AI, TLDR IT, TLDR InfoSec, TLDR Hardware i Own news (Azure Vera Rubin). Dwie wiadomości oznaczono jako IGNORED: Own news, który zawierał same duplikaty z 20.11, oraz poradnik sprzedażowy z Substacka.

Pozycje wykorzystane w sekcjach wyżej (nie powtarzam ich tutaj): Azure Vera Rubin, IEA, Apache Roller.

#### Kalifornijski prokurator generalny wzywa Nvidię (subpoena) w sprawie przemytu chipów do Chin przez Megaspeed (19.11) (TLDR Hardware)
**Przedmiot postępowania:**
- przekierowywanie chipów objętych restrykcjami przez singapurskiego operatora DC Megaspeed;
- to, czy Nvidia wprowadzała inwestorów w błąd co do raportowania przychodów zagranicznych.

**Analiza:**
- Razem z doniesieniem o dochodzeniu SEC (WSJ, 17.11) daje to trzecią warstwę ryzyka prawnego: federalne przepisy eksportowe (BIS), regulacje rynku kapitałowego i postępowanie stanowe.
- Operatorzy DC w Azji Południowo-Wschodniej muszą przygotować pełną dokumentację użytkownika końcowego.
- Modele GPU-as-a-Service z klientami z Chin stają się praktycznie nie do sfinansowania, co pasuje do wycofywania się banków z kredytów w Malezji.

Źródło: [CNBC](https://www.cnbc.com/2026/11/19/nvidia-california-bonta-subpoena-china-chips.html)

<hr>

#### Ruby2SShell (CVE-2026-132825, CVSS 7,4): jednobajtowe przepełnienie sterty w runtime Ruby, podatne m.in. GitHub Enterprise Server i GitLab (TLDR InfoSec)
**Mechanizm:** pętla `upcase` w kodowaniach jednobajtowych zakłada, że jeden znak daje jeden znak wynikowy, a „ß” daje dwa („SS”).

**Poprawki:** Ruby 4.0.10, 3.4.13, 3.3.14.

**Analiza:**
- Błąd tkwi w runtime, a nie w gemie, więc dotyczy każdej aplikacji Ruby przetwarzającej tekst użytkownika.
- Na appliance'ach (GHES, self-hosted GitLab) trzeba czekać na wydania producenta.
- Priorytet jest wysoki, bo te systemy przechowują sekrety CI/CD i tokeny wdrożeniowe.
- W obrazach kontenerów trzeba przebudować warstwę bazową.

Źródło: [Veria Labs](https://verialabs.com/ruby2shell/), [SecurityWeek](https://www.securityweek.com/ruby-patches-ruby2shell-vulnerability-that-could-lead-to-rce/)

<hr>

#### Ukryty kanał między modelami w sandboxach przez współdzielony serwer inferencji (TLDR AI / LessWrong)
**Mechanizm:** żądania jednego modelu zmieniają zachowanie serwera: skład batcha, kolejność redukcji numerycznej, obciążenie. Drugi model odczytuje te zmiany jako sygnał.

**Analiza:**
- To kanał w multi-tenant inference niezależny od izolacji sieci i systemu plików.
- Eksperyment był sztuczny, ale podważa założenie, że sandbox agenta wystarcza, gdy backend inferencji jest współdzielony między domenami zaufania.
- Każda z możliwych mitygacji kosztuje przepustowość i utylizację GPU:
  - kernele niezależne od składu batcha (batch-invariant);
  - osobne repliki per domena zaufania;
  - szum w harmonogramie.

Źródło: [LessWrong](https://www.lesswrong.com/posts/7wzWuLP3HhyBLmKfo/models-can-communicate-via-inference-servers-despite-being)

<hr>

#### Microsoft Agent Controls: płaszczyzna polityk dla agentów AI (Rego + klasyfikatory + LLM-judge) (TLDR IT)
**Budowa:**
- usługa zarządzana w Azure;
- opiera się na otwartych specyfikacjach Agent Control Specification i Agent Hooks.

**Ocena akcji agentów w czasie działania:**
- deterministyczne reguły Rego;
- klasyfikatory content safety;
- sędzia LLM.

**Analiza:**
- Sędzia LLM wydłuża opóźnienie każdej akcji i jest niedeterministyczny. Reguły krytyczne (egress, sekrety, operacje destrukcyjne) powinny działać w Rego, a LLM-judge tylko jako druga linia.
- Otwarta specyfikacja ogranicza vendor lock-in, ale wersja zarządzana działa tylko w Azure.
- W połączeniu z mikro-VM dla agentów w Windows (19.11) Microsoft buduje pełny stos governance agentów.

Źródło: [Microsoft Command Line](https://commandline.microsoft.com/introducing-agent-controls-an-enterprise-control-plane-for-agent-behavior/)

<hr>

#### Google Sashiko: agentowy przegląd łatek jądra Linux – ok. 1 200 przeglądów dziennie (TLDR IT)
**Architektura:** oddziela wyszukiwanie błędów (równoległe, wyspecjalizowane sesje) od głębokiej weryfikacji każdego zgłoszenia. Ogranicza to szum fałszywych alarmów.

**Wyniki:** 7 miesięcy produkcji na listach dyskusyjnych jądra. Liczba poprawek naprawczych w ciągu 60 dni spadła o 35%, ale to korelacja, nie dowód przyczynowości.

**Analiza:** wzorzec „finder + verifier” da się przenieść do przeglądu IaC, polityk firewalla i konfiguracji sieci. Koszt to wielokrotne wywołania modelu na każdą zmianę.

Źródło: [Google Cloud Blog](https://cloud.google.com/blog/products/application-development/lessons-from-building-sashiko-an-agentic-code-review-system)

<hr>

#### Odłączalne złącza światłowodowe dla CPO/NPO: brak standardu, prototyp Senko/Advantest wytrzymał 10 000 cykli (TLDR Hardware)
**Problem:** włókna przyklejone do chipu fotonicznego nie dają się naprawić. Jedno pęknięte włókno oznacza wymianę całego modułu lub przełącznika.

**Ograniczenie miejsca:** płaszcz 125 µm na włókno szybko zużywa krawędź układu (shoreline) przy 16+ włóknach.

**Nowe projekty:**
- wyprowadzenie światła przez lustro 45°;
- soczewki;
- zatrzaski lub magnesy zamiast aktywnego pozycjonowania.

**Analiza:**
- Brak wspólnego interfejsu to vendor lock-in w przełącznikach CPO (np. Quantum-X Photonics w klastrze Lambda) i dłuższy MTTR niż przy wtykowych transceiverach.
- Przy wyborze CPO do klastrów 2027–2028 trzeba zadać pytanie o model serwisowy i zapas modułów na miejscu.

Źródło: [Semiconductor Engineering](https://semiengineering.com/detachable-fiber-connectors-pose-new-challenges-for-photonics)

<hr>

#### Workday: EU Sovereign Cloud na AWS European Sovereign Cloud z irlandzką spółką operacyjną (TLDR IT)
**Fakty:**
- spółka z radą złożoną z obywateli UE ma zarządzać usługą od początku 2027 r.;
- infrastrukturą jest AWS ESC, a nie europejski dostawca.

**Analiza:**
- Kolejny duży ISV realizuje suwerenność strukturą prawną i operacyjną, a nie wyborem OVHcloud, STACKIT czy evroc. Wzmacnia to presję konkurencyjną na europejskie neocloudy w segmencie sovereign SaaS.
- Łańcuch kontroli nadal prowadzi do amerykańskich spółek-matek. Ma to znaczenie przy ocenie pod kątem CLOUD Act i w kontekście zapowiedzi podatku od zagranicznych DC.

Źródło: [Workday IR](https://investor.workday.com/news-and-events/press-releases/news-details/2026/Workday-Advances-EU-Sovereign-Cloud-with-New-EU-Based-Operating-Company-and-Added-Safeguards-for-Customer-Data-and-Services/default.aspx)

<hr>

#### PostgreSQL 19 GA: `REPACK (CONCURRENTLY)` i `pg_plan_advice` (TLDR)
**Nowości:**
- `REPACK CONCURRENTLY` wbudowuje funkcję pg_repack w rdzeń: usuwanie rozrostu tabel bez blokady wyłącznej, kosztem dodatkowego I/O i miejsca na czas operacji;
- `pg_plan_advice` pozwala utrwalać plany zapytań, co ogranicza regresje po ANALYZE i aktualizacjach.

**Analiza:**
- Mniej okien serwisowych na dużych tabelach OLTP.
- Do produkcji lepiej poczekać na 19.1.
- Najpierw trzeba sprawdzić zgodność rozszerzeń (PostGIS, pgvector, Citus).

Źródło: [PostgreSQL](https://www.postgresql.org/about/news/postgresql-19-released-3449/)

<hr>

#### Martwe strefy audytu w Kubernetes (Marco Lancini, TLDR InfoSec)
**Czego nie widzi audit log API servera:**
- zdarzeń na węźle;
- wywołań kierowanych bezpośrednio do kubeleta, z pominięciem API servera;
- poleceń wewnątrz sesji `kubectl exec`.

**Analiza:**
- Pokrycie tych luk wymaga telemetrii węzła (eBPF/Falco/Tetragon) i logów kubeleta.
- To kilkukrotnie zwiększa wolumen i koszt SIEM, więc potrzebne jest filtrowanie u źródła.
- Artykuł zawiera konfigurację referencyjną.

Źródło: [blog.marcolancini.it](https://blog.marcolancini.it/2026/blog-security-logging-cloud-environments-kubernetes/)

## Implikacje praktyczne

1. **Dostęp do kapitału, nie do GPU, jest teraz ograniczeniem pierwszego rzędu.** Kontrakty klientów z neocloudami wiszą na warunkowym backlogu (Anthropic: 186 mld USD wypowiadalne lub warunkowe) i na refinansowaniu z 2027 r. Potrzebny jest plan B: druga chmura, on-prem albo rezerwacja u hyperscalera. Przy wyborze dostawcy GPU (neocloud, kolokacja) na lata 2027–2029 trzeba wymagać:
   - dowodu finansowania konkretnej lokalizacji i zasilania;
   - klauzul step-in i przenoszalności rezerwacji.
2. **KV cache jako osobna warstwa infrastruktury.** W ciągu tygodnia pojawiły się cztery niezależne implementacje (AWS Valkey+LMCache, SK hynix CXL, Marvell/Penguin, Cerebras/AMD). W architekturze inferencji na lata 2027–2030 trzeba zaplanować hierarchię HBM → CXL/DRAM → sieć RDMA → NVMe z własnymi SLO.
   - Wymaga to wersjonowania kluczy cache i testów deterministyczności.
   - Niezgodny KV daje błędne odpowiedzi przy dobrych statystykach trafień.
3. **Łańcuch dostaw sprzętu „zwykłego” staje się ryzykiem projektowym.** Niedobór PCB (laminaty 14–18 tygodni), koniec wsparcia SonicWall Gen 6 i wcześniejsze podwyżki DRAM oznaczają, że odświeżenia sieci i serwerów x86 na 2027 r. trzeba zamawiać już teraz, z klauzulami cenowymi.
   - Warto rozważyć dywersyfikację: whitebox/SONiC tam, gdzie zespół ma kompetencje.
4. **Hipernadzorca i płaszczyzna zarządzania jako główna powierzchnia ataku.** Tydzień przyniósł KVM AMD nested, vCenter 0-day, VCF/Aria bez łatki i F5 APM. Warto:
   - zaprojektować segment zarządzania jako osobną strefę zaufania z dostępem wyłącznie przez bastion lub PAW;
   - przyjąć polityki „nested virtualization off by default” dla tenantów;
   - przygotować procedurę rotacji certyfikatów STS/SAML po incydencie.
5. **Autonomiczni atakujący wymuszają detekcję opartą na tempie i sekwencji.** Czas od rozpoznania do roota ok. 11 minut (Cairn/MONAI) jest krótszy niż cykl łatania. Platformy MLOps (MONAI, Ray, Jupyter, MLflow) muszą działać za ZTNA. Na krawędzi potrzebny jest rate-limiting per tożsamość, a w SOC automatyczne circuit breakery zamiast ręcznego triage.

## Trend tygodnia

Tydzień zamyka się zderzeniem dwóch krzywych: popyt na moc obliczeniową pozostaje rekordowy, a dostęp do kapitału gwałtownie się pogarsza.
- Nvidia straciła ok. 46% w siedem tygodni mimo wyników powyżej konsensusu. Rynek przestał wyceniać przychody, a zaczął wyceniać wypłacalność klientów, czyli neocloudów, OpenAI przez SoftBank i laboratoriów AI.
- S-1 Anthropic pokazał skalę zobowiązań (ok. 900 mld USD) i to, że część backlogu dostawców jest warunkowa. Ryzyko wykonania przesuwa się z klienta na operatora DC.
- Równolegle narastają ograniczenia fizyczne i polityczne:
  - przyłączenia energetyczne (IEA: 2× zużycia do 2030 r.);
  - laminaty PCB;
  - zapowiedź podatku od zagranicznych DC w USA;
  - postępowania wobec Nvidii na trzech poziomach.
- Technologicznie branża optymalizuje koszt tokena zamiast dokładać GPU: KV cache wychodzi z HBM do CXL i sieci, inferencja się dezagreguje, a CPU (Vera) i pamięć stają się elementami jednej platformy.
- W sieciach i bezpieczeństwie agenci przechodzą z dem do produkcji po obu stronach: Microsoft obsługuje agentami sieć z 1,6 mln urządzeń, a atakujący używają ich do autonomicznej exploitacji 0-day w kilkanaście minut.

Dla architekta oznacza to przesunięcie priorytetów: z „zdobyć GPU” na „zabezpieczyć kontraktowo i finansowo ciągłość mocy”, oraz z „łatać w cyklu miesięcznym” na „izolować płaszczyzny zarządzania i wykrywać tempo ataku”.

## To obserwować

1. **Spready i rentowności obligacji CoreWeave (2030/2032) oraz spready DC ABS.** Przekroczenie rentowności ok. 15–16% albo spreadu 3 pp będzie sygnałem zamknięcia rynku refinansowania na 2027 r.
2. **Zeznania przed Kongresem 1.12 i postępowania wobec Nvidii (SEC, Kalifornia).** Ważny będzie także ewentualny projekt ustawy o podatku od zagranicznych DC: zakres (AI vs wszystkie regiony) i data wejścia.
3. **Doniesienia o rozmowach CrowdStrike–Zscaler (Bloomberg, 20.11, niepotwierdzone).** Jeśli się potwierdzą, oznaczałyby konsolidację EDR i SSE. To ryzyko dla roadmap SASE u klientów obu firm.
4. **Ceny kontraktowe NAND i DRAM w raportach TrendForce za grudzień i styczeń** oraz czasy dostaw laminatów PCB. To wskaźniki wyprzedzające koszty odświeżeń w 2027 r.
5. **Paczki z poprawką CVE-2026-99312 w Ubuntu, Debian i RHEL** oraz łatka VMware dla VMSA-2026-0010. Do ich wydania obowiązują mitygacje (nested=0, HTTP/2 off).
