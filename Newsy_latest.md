# Raport Zbiorczy IT – 2026-11-24

**Okno przyrostowe:** od raportu z 2026-11-23. Uwzględniono zdarzenia z 20–24.11, których nie ma w rejestrze z ostatnich 7 dni. Fakty już opisane wcześniej (Entra Connect Health, kradzież promptów u dostawców inferencji, Cairn, podatek Trumpa od zagranicznych DC, niedobór PCB, emisja 44 mld USD dla SpaceX) pominięto albo podano tylko ich nowy element.

## Raport technologiczny

### Biznes

#### CrowdStrike przejmuje Zscaler za ok. 43,9 mld USD w akcjach – największa transakcja w historii cyberbezpieczeństwa (23.11)
CrowdStrike ogłosił przejęcie Zscalera w transakcji wyłącznie akcyjnej. Doniesienia o rozmowach (Bloomberg, 20.11) były w rejestrze jako niepotwierdzone, teraz są wiążące warunki:
- stały parytet 1,135 akcji CRWD za 1 akcję ZS, czyli ok. 260,41 USD, ok. 34% premii do kursu z 20.11,
- po zamknięciu akcjonariusze CrowdStrike będą mieć ok. 84,1%, a Zscalera ok. 15,9%,
- zamknięcie planowane w II połowie FY2028 CrowdStrike, po zgodzie akcjonariuszy ZS i regulatorów,
- Jay Chaudhry dalej kieruje biznesem Zscaler i wchodzi do rady CrowdStrike.

Uzasadnienie: połączenie telemetrii endpointu (Falcon) z zero-trust/SSE (ZIA/ZPA), jeden kontrakt i dostęp do ok. 8 000 klientów Zscalera, którzy nie kupują od CrowdStrike. Kurtz: „sieć łączyła ludzi z miejscami, teraz łączy agentów z danymi”. W dniu ogłoszenia kurs CRWD spadał o 8–9%, a ZS rósł o 13–18%. To trzecia transakcja CrowdStrike w listopadzie (po Fiddler AI i Dreadnode).

**Znaczenie:**
- **Mechanizm wartości:** korelacja procesu na hoście z sesją sieciową w jednym grafie zdarzeń. Pozwala na szybsze wykrywanie lateral movement i ruchu agentów AI do SaaS bez łączenia dwóch SIEM-ów. Stały parytet przy spadku CRWD obniża jednak efektywną cenę, a to zwiększa ryzyko sprzeciwu akcjonariuszy ZS lub kontroferty.
- **Dla klientów obu firm:** okres przejściowy 12–18 miesięcy to ryzyko zamrożenia roadmap, zmian w SKU i bundlingu cen przy odnowieniach. Kto ma Zscalera z EDR innego dostawcy (Defender, SentinelOne), powinien przed odnowieniem zabezpieczyć w umowie dalsze wsparcie integracji i ceny.
- **Rynek:** po Palo Alto (Applied Compute, rejestr 18.11) to kolejny krok konsolidacji „platformowej”. Na rynku zostaje mniej niezależnych SSE (Netskope, Cloudflare). Rośnie vendor lock-in, a regulatorzy (FTC/KE) mogą badać nakładanie się na rynku SSE/XDR.

Źródło: [CyberScoop](https://cyberscoop.com/crowdstrike-to-acquire-zscaler/), [8-K CrowdStrike (ex. 99.1)](https://www.sec.gov/Archives/edgar/data/0001535527/000110465926130785/tm2630454d1_ex99-1.htm), [Quartz](https://qz.com/crowdstrike-zscaler-all-stock-acquisition-cybersecurity-112326), [SecurityWeek](https://www.securityweek.com/crowdstrike-to-acquire-zscaler-in-43-9-billion-deal/)

### Technologia

#### AWS Network Firewall: inspekcja ruchu bezpośrednio na Internet Gateway, bez zmian routingu (preview, 20.11)
Nowy tryb „automated internet firewall” egzekwuje reguły na IGW każdego VPC w zakresie. Zakres to wybrane VPC, konto, OU albo cała organizacja, z automatycznym objęciem nowych VPC i podsieci. Nie trzeba dedykowanych podsieci z endpointami firewalla ani zmian w tabelach routingu. Dochodzi wbudowany NAT dla ruchu wyłącznie wychodzącego i logowanie. Domyślna polityka korzysta z zarządzanych grup reguł AWS (active threat defense). Preview działa we wszystkich regionach komercyjnych poza GovCloud i Chinami. Cennika nie podano.

**Znaczenie:** Znika główny koszt operacyjny dotychczasowego wzorca (inspection VPC + TGW albo endpointy GWLB per AZ i asymetria routingu). Wbudowany NAT może zastąpić NAT Gateway, co ma znaczenie przy dużym egressie. Edge cases do sprawdzenia w preview:
- zachowanie przy awarii AZ i fail-open/fail-closed,
- limity przepustowości per IGW,
- współistnienie z istniejącymi regułami GWLB i inspekcją east-west (ten tryb obejmuje tylko ruch internetowy),
- koszt per GB vs NAT GW.

Organizacje z centralnym egressem przez TGW powinny porównać TCO przed migracją.

Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/network-firewall-automated-internet/)

#### GKE Ambient Networking GA: service mesh bez sidecarów, zarządzany przez Google (21.11)
Google udostępnił w GA zarządzany mesh dla GKE bez sidecarów. Funkcje sieciowe przechodzą z podów aplikacji do komponentów per-node (namespace `gke-managed-ambient`): wtyczka NRI do przekierowania ruchu, agent xDS i współdzielony proxy L4 Envoy. Płaszczyzna sterowania jest zarządzana: Traffic Director (xDS), Certificate Authority Service (certyfikaty tożsamości workloadów) i kontroler Gateway. Proxy L4 realizuje mTLS, autoryzację, load balancing i telemetrię. Google deklaruje do 10× mniejsze zużycie zasobów dla funkcji L4 i aktualizacje proxy bez restartu podów aplikacji.

**Znaczenie:** Główny zysk to brak restartów podów przy łatkach CVE proxy i niższe zużycie CPU/RAM w klastrach o dużej gęstości podów. Ograniczenia:
- **Blast radius:** współdzielony proxy per-node oznacza, że jego awaria lub saturacja dotyka wszystkich podów na węźle (noisy neighbor w warstwie L4).
- **Lock-in:** płaszczyzna sterowania to Traffic Director i CAS, a nie upstream Istio. Przenośność polityk do innych chmur i on-prem jest ograniczona.
- **Migracja:** artykuł nie opisuje ścieżki z sidecar-mesh. Przed migracją trzeba zweryfikować obsługę polityk L7 i zgodność z istniejącymi CRD Istio w dokumentacji.

Źródło: [Google Cloud Blog](https://cloud.google.com/blog/products/networking/ambient-networking-for-gke-sidecarless-service-mesh), [dokumentacja](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/ambient-overview)

## Raport AI-ML

### Biznes

#### Nvidia przywraca Jensena Huanga na stanowisko CEO i do rady; nowy komitet ds. bezpieczeństwa narodowego, poszukiwany Chief Compliance Officer (23.11)
Rada Nvidii jednogłośnie przywróciła Huanga jako prezesa, CEO i dyrektora ze skutkiem natychmiastowym. W rejestrze z 18.11 przewodniczący rady wykluczał jego powrót. Szczegóły:
- Rada „otrzymała informacje od wysokich przedstawicieli rządu USA” i uznała, że powrót nie stanowi ryzyka dla bezpieczeństwa narodowego.
- Trump powiedział CNBC, że „przekonał” radę.
- Colette Kress wraca do roli CFO, a Harvey Jones pozostaje przewodniczącym rady.
- Dochodzenie komitetu niezależnych dyrektorów w sprawie próby werbunku wywiadowczego wobec Huanga ma być doprowadzone do końca.
- Nowy komitet ds. bezpieczeństwa narodowego i kontroli eksportu składa się wyłącznie z niezależnych dyrektorów. Trwa rekrutacja Chief Compliance Officer raportującego do komitetu ds. nominacji i ładu korporacyjnego.

Równolegle NYT (22.11) podał, że służby USA przez ponad rok śledziły kontakty Alice Huang (Megaspeed) z oficerem chińskiego MSS. Megaspeed pozostaje w centrum śledztw w sprawie przemytu chipów.

**Analiza:** Ryzyko kluczowej osoby zostało usunięte, ale zamienione na ryzyko polityczne. Decyzja zapadła po interwencji prezydenta, przy trwających śledztwach (DoJ, Kongres, wezwanie od prokuratora generalnego Kalifornii z rejestru 21.11). Dla klientów nie zmienia to roadmapy Rubin/Vera. Zwiększa natomiast prawdopodobieństwo, że polityka eksportowa (licencje BIS na zdalny dostęp, H200 dla Chin) będzie zależna od relacji Nvidia–Biały Dom. To ryzyko regulacyjne dla operatorów obsługujących klientów z Chin (por. Nscale/ByteDance w rejestrze). Zeznania przed Kongresem 1.12 pozostają w kalendarzu.

Źródło: [HPCwire (komunikat Nvidii)](https://www.hpcwire.com/off-the-wire/nvidia-board-reinstates-jensen-huang-as-president-ceo-and-board-director/), [CNBC](https://www.cnbc.com/2026/11/23/anthropic-nvidia-shares.html), [NYT](https://www.nytimes.com/2026/11/22/us/politics/alice-huang-china-spy-agency-nvidia.html)

#### TSMC i Samsung wstrzymują część dostaw dla chińskich projektantów po nowej regule BIS – próg 16/14 nm (FT, 21–23.11)
Według FT TSMC i Samsung poinformowały część chińskich klientów o wstrzymaniu wysyłek wybranych procesorów do czasu uzyskania licencji USA. Dotyczy to także elektroniki użytkowej. Według Tom's Hardware i Finimize TSMC kazał wstrzymać zamówienia w procesach 16 nm i niższych bez licencji, co spycha klientów na 28/40 nm. Przyczyna: reguła BIS z 16.11 (w mocy od 19.11) usunęła skrót oparty na liczbie tranzystorów, którym foundry samodzielnie kwalifikowały chińskie projekty. Ładunki w tranzycie mogą być realizowane na starych zasadach do 21.12. Klienci z Chin dali TSMC ok. 5% przychodu w Q3 (rok wcześniej ok. 8%). Ani TSMC, ani BIS nie wydały oficjalnego komunikatu.

**Analiza:** Kontrola obejmuje teraz węzły sprzed dekady, czyli SoC smartfonów, TV i motoryzacji, a nie tylko akceleratory AI. Skutki:
- **Krótkoterminowo:** mniejsze obciążenie TSMC N16/N12 (zwalnia moc dla innych klientów) i przyspieszenie przenoszenia projektów do SMIC/Hua Hong.
- **Dla europejskich OEM-ów z łańcuchem dostaw w Chinach:** ryzyko opóźnień komponentów (kontrolery, ASIC sieciowe, SoC IoT) produkowanych dla chińskich fabless.
- **Średnioterminowo:** Chiny konsolidują popyt na rodzime węzły 7–14 nm, co wzmacnia rodzimy ekosystem akceleratorów (Ascend, Cambricon) mimo niższego uzysku.

Źródło: [Tom's Hardware](https://www.tomshardware.com/tech-industry/policy/tsmc-reportedly-pauses-shipments-to-chinese-designers-after-u-s-export-rules-flag-decade-old-16nm-and-14nm-nodes-new-bis-controls-threaten-smartphone-tv-and-automotive-chips), [Finimize](https://finimize.com/content/tsmc-tightens-the-screws-on-chinese-chip-orders), [AI Weekly (za FT)](https://aiweekly.co/alerts/tsmc-and-samsung-pause-some-china-chip-shipments-under-new-bis-rule)

### Technologia

#### AWS Lambda Managed Instances z GPU: serverless na instancjach EC2 G-series, opłata za zarządzanie 9,75% (23.11)
Funkcje Lambda mogą działać na instancjach EC2 G-series (kilka generacji w capacity providerze). Lambda wybiera najnowszą generację spełniającą wymagania i w razie braku capacity wraca do starszej. Szczegóły:
- bazowe obrazy z vLLM, SGLang i PyTorch,
- obrazy kontenerowe z NVMe ephemeral do 32 GB, lazy loading SOCI i streaming artefaktów z S3,
- wiele wywołań współbieżnie w jednym środowisku, skalowanie wg wykorzystania GPU i CPU oraz współbieżności.

Cena: on-demand EC2 (działają Savings Plans i RI) plus 9,75% opłaty za zarządzanie i 0,20 USD za 1 mln żądań.

**Analiza:** To nie jest scale-to-zero w klasycznym sensie, bo płaci się za instancje w capacity providerze. Zyskiem jest operacyjna prostota: brak zarządzania ASG/EKS dla inferencji o zmiennym ruchu (embeddingi, małe LLM, VLM). Ograniczenia:
- **Latency:** fallback na starszą generację GPU daje niedeterministyczną wydajność, więc p99 TTFT może skakać przy niedoborze capacity. SLO trzeba wyznaczać na najwolniejszej dopuszczonej generacji.
- **Koszt:** przy wysokim stałym wykorzystaniu 9,75% narzutu przegrywa z EKS + Karpenter. Opcja jest sensowna przy wykorzystaniu GPU poniżej ok. 40–50% albo dla zespołów bez platformy K8s.
- **Lock-in:** model programowania Lambda i capacity provider są specyficzne dla AWS, ale obrazy vLLM/SGLang pozostają przenośne.

Źródło: [HPCwire (komunikat AWS)](https://www.hpcwire.com/off-the-wire/aws-lambda-adds-support-for-gpu-instances-on-lambda-managed-instances/)

#### Sirius jako rozszerzenie core DuckDB: wykonanie planu SQL na GPU, 3,25–5,5× vs CPU na TPC-H do SF10K (23.11)
NVIDIA i UW–Madison wydały Sirius jako core extension DuckDB (`INSTALL sirius; LOAD sirius;`, ok. 700 MB, automatyczny wybór backendu CUDA). DuckDB parsuje i optymalizuje zapytanie, Sirius wykonuje plan fizyczny na GPU, a nieobsługiwane operatory automatycznie wracają na CPU. Benchmark: zimne uruchomienia na danych w S3, 2 GPU na AWS, porównanie z najlepszą konfiguracją CPU:
- SF1K: 347 → 63 s,
- SF3K: 752 → 161 s (4,69×, ok. −71% kosztu),
- SF10K: 2 369 → 730 s (3,25×, ok. −79% kosztu); domyślna konfiguracja 2-GPU nie ukończyła tego testu.

Czasy Sirius nie obejmują eksportu wyników, a czasy DuckDB obejmują zapis CSV, więc porównanie jest optymistyczne dla GPU.

**Analiza:** To pierwszy realny wariant „GPU analytics bez Sparka” dla jednowęzłowego DuckDB. Przy danych w S3 wąskim gardłem staje się I/O i dekompresja Parquet, a nie obliczenia, więc zysk spada wraz z SF. Failure modes:
- przekroczenie pamięci GPU przy dużych joinach (brak ukończenia SF10K w konfiguracji domyślnej wymaga tuningu),
- ciche przejście na CPU dla części planu, co daje nieprzewidywalne czasy,
- buildy deweloperskie, czyli niedojrzałość produkcyjna.

Ma to sens jako sposób na wykorzystanie GPU inferencyjnych poza szczytem (dzielenie się capacity) dla zespołów data.

Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/get-started-with-gpu-accelerated-duckdb-analytics-using-the-sirius-core-extension/)

#### Hammerspace: IO500 (10-node) na lokalnych NVMe serwerów obliczeniowych przez pNFS, bez osobnego klastra storage (23.11)
Hammerspace zgłosił na listę IO500 10-Node Research (SC26) wynik 688,01: 449,86 GiB/s przepustowości i 1 052 kIOP/s metadanych. Konfiguracja: 10 węzłów obliczeniowych serwujących dane z własnych NVMe przez standardowy Linux pNFS (bez klienta własnościowego) i dwa serwery metadanych Anvil. Według firmy wyżej sklasyfikowane zgłoszenia komercyjne używały osobnych klastrów storage z 6–84 serwerami.

**Analiza:** „Tier 0” to wykorzystanie lokalnych NVMe w serwerach GPU (zwykle 8–16 dysków, często nieużywanych poza cache) jako współdzielonej warstwy dla checkpointów i datasetów. Oszczędza to capex i moc na osobny storage. Ryzyka:
- dane żyją na węzłach, które są restartowane i serwisowane, więc trwałość zależy od replikacji lub erasure coding między węzłami,
- ruch storage konkuruje z ruchem kolektyw na tej samej sieci (potrzebna separacja QoS/VRF),
- wynik IO500 na 10 węzłach nie przekłada się liniowo na tysiące GPU, bo serwery metadanych Anvil mogą stać się wąskim gardłem.

To wynik deklarowany przez dostawcę, ale weryfikowalny na io500.org.

Źródło: [HPCwire (komunikat Hammerspace)](https://www.hpcwire.com/off-the-wire/hammerspace-benchmarks-tier-0-storage-at-sc26-without-a-separate-storage-cluster/)

## euro neocloud

### Nscale

#### Wstrzymanie rekrutacji i cięcie wydatków; termin IPO (cel ok. 35 mld USD) niepewny (doniesienia 22–23.11)
**Fakty:**
- COO Justin Osofsky poinformował pracowników w wewnętrznym mailu, że rekrutacja jest wstrzymana na najbliższe miesiące, poza rolami krytycznymi. Dotyczy to głównie back-office, a nie budowy DC.
- Ograniczono wydatki uznaniowe.
- Spółka nie zdecydowała, czy wejść na giełdę w 2026 r., czy w 2027 r., i nie ma harmonogramu.
- Nscale liczyło, że IPO Anthropic uspokoi rynek i pozwoli zacząć roadshow w ciągu tygodni. Kurs ANT spadł jednak po debiucie (ok. −3% także 23.11 mimo rajdu).
- Według poprawionego prospektu strata netto za 9 miesięcy to ok. 1,69 mld USD przy przychodzie ok. 307,5 mln USD. Spółka sygnalizuje, że będzie potrzebować dodatkowego finansowania na zobowiązania w 2027 r.

Kontekst z rejestru: licencja BIS potrzebna dla kontraktu z ByteDance (19.11), pierwsze szafy VR NVL72 (19.11), udział w konsorcjum AIGF Northern (16.11).

**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:*
  - strata ok. 5,5× większa od przychodu,
  - zapowiedziana potrzeba dodatkowego finansowania przy zamkniętym oknie IPO,
  - zależność od kontraktu z klientem chińskim wymagającym licencji USA,
  - rynek długu dla neocloudów z wąską bazą klientów drożeje (rejestr 19–23.11: banki zbywają ekspozycje DC, kupony ok. 8% nawet dla SpaceX),
  - cięcia kosztów stałych to typowy pierwszy krok przed renegocjacją harmonogramu capex.
- *Ocena ryzyka:* **Wysokie.** Płynność zależy od otwarcia rynku kapitałowego w ciągu 2–3 kwartałów, a nie od operacji. Rajd z 23.11 (CoreWeave +22%) może otworzyć okno, ale tylko jeśli się utrzyma. Brak sygnałów problemów z SLA lub dostępnością GPU, więc ryzyko jest finansowe, a nie operacyjne. Klienci z kontraktami wieloletnimi powinni sprawdzić klauzule step-in, prawa do sprzętu i warunki wyjścia przy zmianie kontroli.
- *Linki:* [TradingView](https://www.tradingview.com/news/binance_news:d352649b2094b:0-nscale-slows-hiring-and-cuts-costs-ahead-of-planned-35-billion-ipo/), [TradersUnion](https://tradersunion.com/news/companies/show/4363607-nscale-cuts-hiring-ipo-uk/), [GuruFocus](https://www.gurufocus.com/news/9214233/nscale-delays-ipo-plans-amid-market-concerns), [Benzinga (prospekt)](https://www.benzinga.com/trading-ideas/movers/26/11/63248201/nebius-stock-falls-nearly-17-weak-outlook-nvidia-turmoil-and-nscales-wider-loss-rattle-neoclouds)

## AI for networking

#### VMware VCF: Agentic AI Hub (Fling) – agenci diagnostyczni NSX przez serwery MCP, BGP i datapath przez Traceflow/LiveTrace (23.11)
Broadcom wydał Fling „Agentic AI Hub” dla VMware Cloud Foundation. Pytania operatora trafiają przez Agent Portal do wyspecjalizowanych asystentów z runbookami, planami dochodzeń i regułami walidacji. Dostęp do stanu infrastruktury zapewniają serwery MCP:
- NSX (inventory, konfiguracja, realization state, diagnostyka),
- VCF Operations for Networks (flow records, top talkers),
- Log Insight (korelacja zdarzeń),
- Python sandbox izolowany przez WASM, wywołujący inne narzędzia przez AI Gateway,
- wyszukiwanie semantyczne w dokumentacji SDK.

Scenariusze: diagnostyka sesji i tras BGP na Edge, analiza datapath przez Traceflow/LiveTrace, capacity Edge/VNA, gotowość do upgrade'u NSX. Dwa tryby dostępu: UI z guardrailami albo surowy endpoint MCP dla Claude Code, Cursor czy Gemini CLI (bez wbudowanych skilli, planów i promptów guardrail). Wrażliwe wywołania wymagają zatwierdzenia człowieka. RBAC narzędzi i audyt działają w obu trybach. Narzędzie jest zalecane tylko do labów i środowisk nieprodukcyjnych.

**Analiza:**
- **Wzorzec:** to, co Microsoft pokazał na Ignite dla własnej sieci (rejestr 20.11), trafia do produktu on-prem jako MCP nad istniejącymi API. Jest to wzorzec wielokrotnego użytku: telemetria i stan pozostają w systemach źródłowych, a LLM tylko orkiestruje zapytania. Skalowalność ogranicza się do limitów API NSX Managera i kosztu tokenów przy dużych odpowiedziach (flow records).
- **Failure modes:**
  - w trybie surowego MCP znikają guardraile, więc zewnętrzny agent z uprawnieniami zapisu w NSX może wprowadzić zmianę bez runbooka; uprawnienia RBAC dla tożsamości agenta trzeba ograniczyć do odczytu,
  - halucynacja przyczyny przy częściowych danych (np. brak logów Edge),
  - prompt injection przez pola konfiguracyjne i opisy obiektów czytane przez model.
- **Lock-in:** niski na poziomie protokołu (MCP), wysoki na poziomie runbooków (specyficzne dla VCF).

Źródło: [VMware VCF Blog](https://blogs.vmware.com/cloud-foundation/2026/11/23/agentic-ai-for-enhanced-network-operations-in-vmware-cloud-foundation/)

## Ryzyka rynku AI/GPU

#### Odbicie po kapitulacji: Nvidia +31% (najlepsza sesja od 05.2003, ok. +1 bln USD kapitalizacji), Nasdaq +6,51%, CoreWeave +22%, Micron +19% (23.11)
Zamknięcie 23.11:
- Dow 49 595 (+3,01%), S&P 500 7 237 (+4,73%), Nasdaq 24 090 (+6,51%), najlepszy dzień od kwietnia 2025,
- Nvidia ok. +31%, CoreWeave +22%, AMD ok. +7%, Micron +19%, ETF SMH ponad +16%,
- Alphabet i Meta +6%, Anthropic (ANT) ok. −3%.

Nasdaq odrobił całą piątkową stratę (drugi najgorszy dzień 2026 r., S&P i Dow w korekcie). Melius, Bernstein i StoneX podnieśli rekomendacje dla Nvidii. BTIG oczekuje krótkiego rajdu do 7 250–7 350 na S&P, a potem ponownych spadków. Barclays uważa wyprzedaż za przesadzoną, ale pozostaje sceptyczny. Michael Burry nazwał Nvidię „słabszą i bardziej służalczą, niż sądził”.

**Analiza:** Odbicie dotyczy ryzyka zarządczego (powrót Huanga), a nie fundamentów, które ciążyły rynkowi w rejestrze:
- marża brutto Nvidii spada do 71–72%,
- zobowiązania zakupowe wzrosły o 111 mld USD,
- 130 tys. GPU CoreWeave schodzi z kontraktów w 2027 r.,
- ok. 186 mld USD wypowiadalnych zobowiązań w S-1 Anthropic,
- oczekiwana podwyżka Fed 8–9.12.

Żadna z tych zmiennych się nie zmieniła. Spadek ANT w dniu rajdu pokazuje, że rynek dalej dyskontuje ekonomikę laboratoriów, a nie popyt na sprzęt. Dla capacity planningu: zmienność ±20–30% dziennie na papierach neocloudów oznacza, że koszt i dostępność ich kapitału mogą się zmienić z tygodnia na tydzień. Kontrakty z neocloudami powinny mieć zabezpieczenia niezależne od kursu (escrow, prawa do sprzętu).

Źródło: [CNBC (live)](https://www.cnbc.com/2026/11/22/stock-market-today-live-updates.html), [Yahoo Finance](https://finance.yahoo.com/technology/article/nvidia-reinstates-jensen-huang-as-ceo-stock-rockets-141145439.html)

#### Aktualizacja: kredytodawcy SpaceX chcieli covenantu zakazującego wynoszenia na orbitę GPU stanowiących zabezpieczenie długu ~43–44 mld USD (Ars Technica, 22–23.11)
Nowy element względem rejestru z 23.11: kredytodawcy w transakcji Apollo/Goldman żądali zapisu, że GPU zabezpieczające dług nie mogą trafić do satelitów. Przy niewypłacalności przejęcie i odsprzedaż sprzętu na orbicie byłyby praktycznie niemożliwe. SpaceX deklaruje, że chipy trafią do jego naziemnych DC w USA. Nie wiadomo, czy covenant znalazł się w finalnej dokumentacji.

**Analiza:** Kredytodawcy wyceniają GPU jako zabezpieczenie odzyskiwalne fizycznie, czyli recovery value zależy od lokalizacji i możliwości demontażu. To samo dotyczy GPU w DC w jurysdykcjach z ryzykiem sankcyjnym (licencje BIS na zdalny dostęp) albo w obiektach z niepewnym prawem do terenu. Covenanty lokalizacyjne i ograniczenia relokacji będą standardem w finansowaniu GPU. Zmniejszają elastyczność operatora, np. przy przenoszeniu sprzętu między regionami w odpowiedzi na ceny energii.

Źródło: [Ars Technica](https://arstechnica.com/space/2026/11/spacex-lenders-dont-want-elon-musk-launching-their-collateral-into-space)

## Newsletters summary

#### Tencent Hy4: otwarte wagi, 770B MoE / 49B aktywnych, kontekst 1M, sparse attention i drafter spekulatywny (TLDR AI)
Model multimodalny (tekst, obraz, wideo) z backbone 770B parametrów i 49B aktywnych na token, kontekstem 1M tokenów, sparse attention i wbudowanym drafterem do dekodowania spekulatywnego. Celem są agentowe przepływy pracy (coding, computer use).

**Analiza:** 49B aktywnych przy 770B całkowitych oznacza ok. 1,5 TB wag w BF16 (ok. 400 GB w FP4), czyli wymaga co najmniej 8×B200/MI355X albo pełnego NVL72 dla rozsądnego batchu przy 1M kontekstu. Koszt jest zdominowany przez KV cache, co wzmacnia argument za współdzielonym KV (LMCache, CXL-pool). Wbudowany drafter obniża latency decode bez osobnego modelu, ale wymaga wsparcia w vLLM/SGLang.

Źródło: [Hugging Face – tencent/Hy4](https://huggingface.co/tencent/Hy4)

#### NVIDIA Nemotron-Labs-Sigma: dyfuzyjne modele językowe 3B/8B, 3,46× przepustowości vs autoregresja na GB200 (TLDR AI)
14 otwartych checkpointów 3B i 8B generuje tekst przez odszumianie zwartych embeddingów tokenów zamiast token po tokenie. Model 8B obsługuje generację w kilku krokach i samospekulatywne dekodowanie. Średnio daje 3,46× przepustowości względem dekodowania autoregresyjnego na GB200.

**Analiza:** Dyfuzja przesuwa decode z ograniczenia pasmem pamięci w stronę obliczeń, bo przetwarza wiele tokenów na krok. Lepiej wykorzystuje to tensor cores przy małym batchu (edge, agenci o niskiej współbieżności). Ograniczenia: tylko małe modele, brak dojrzałego wsparcia w silnikach serwujących, jakość przy długich odpowiedziach niezweryfikowana. Na razie to kierunek badawczy, nie produkcyjny.

Źródło: [NVIDIA Research – Sigma](https://research.nvidia.com/labs/genair/sigma/)

#### Hugging Face „harness adapters”: LoRA trenowane RL pod konkretny harness agenta (TLDR AI)
HF pokazuje adaptery LoRA trenowane przez RL dla kombinacji model–harness–zadanie (np. Qwen3.8-27B). Podnoszą trafność i znacząco zmniejszają zużycie tokenów.

**Analiza:** Dla platform inferencyjnych oznacza to więcej wariantów LoRA na jednym modelu bazowym. Wymagany jest multi-LoRA serving (S-LoRA/vLLM) z routingiem per harness i zarządzaniem pamięcią adapterów. Mniej tokenów na zadanie bezpośrednio obniża koszt agentów. Ryzyko: adapter zoptymalizowany pod jeden harness degraduje się po zmianie wersji harnessu, więc wersjonowanie adapterów trzeba powiązać z wersją narzędzi.

Źródło: [Hugging Face Blog](https://huggingface.co/blog/harness-adapters-at-scale)

#### Boom Supersonic: pierwsze uruchomienie rdzenia silnika współdzielonego z turbiną gazową 42 MW „Superpower” dla DC AI (TLDR Hardware)
Rdzeń (6-stopniowa sprężarka, 1-stopniowa turbina) osiągnął 6 500 obr./min przy ok. 2 000 czujników w drugiej próbie zapłonu. Ten sam rdzeń, z ponad 75% wspólnych części, napędza turbinę Superpower 42 MW dla DC i turbofan samolotu Overture.

**Analiza:** To nowy dostawca aeroderywatywnych turbin dla energetyki on-site w DC, w czasie gdy kolejki na turbiny GE Vernova i Siemens Energy sięgają lat (por. anulowanie rezerwacji 13 GW przez SoftBank). Dziś to etap testu rdzenia, a nie certyfikowany produkt. Realna dostępność to raczej 2028+, a ryzyko wykonawcze jest wysokie.

Źródło: [AVI-GO](https://avi-go.com/news/articles/021f9bbf9ef77e9f998d4d793e326ce9)

#### SK hynix największym akcjonariuszem Kioxii (14,1% pośrednio) (TLDR Hardware)
Konwersja obligacji wehikułu inwestycyjnego na akcje Kioxii i dokupienie akcji od Bain daje SK hynix 14,1%, wyprzedzając Toshibę (10,45%).

**Analiza:** To wzrost wpływu SK hynix na drugiego producenta NAND poza Samsungiem w czasie, gdy enterprise SSD dla AI (checkpointy, KV offload, vector DB) jest pod presją cenową (rejestr 20.11: TrendForce przewiduje spadki cen NAND w 2027 r.). Dla kupujących oznacza to mniejszą liczbę niezależnych źródeł NAND i potencjalnie mniejszą konkurencję cenową w QLC dla AI.

Źródło: [TLDR Hardware / link źródłowy](https://links.tldrnewsletter.com/y6fFTe)

#### UK AISI: przedpremierowy GPT-6.2 Astra łatwiej złamać niż poprzednika; monitory OpenAI przeoczyły 19–51% naruszeń (TLDR InfoSec)
AISI zbudował uniwersalny jailbreak cybernetyczny przy połowie zapytań potrzebnych dla GPT-6 Astra. OpenAI zmieniło zabezpieczenia, ale AISI nie przetestował poprawek ponownie. Monitory OpenAI przeoczyły 19% naruszeń, a przy pełnym (niestreszczonym) rozumowaniu 51%.

**Analiza:** Dla wdrożeń agentowych z uprawnieniami do infrastruktury guardraile modelu nie są kontrolą bezpieczeństwa. Polityki muszą być egzekwowane deterministycznie poza modelem: RBAC narzędzi, sandboxy, egress allowlist. Zaskakuje wynik, że więcej kontekstu rozumowania pogarsza skuteczność monitora. Monitorowanie LLM-em innego LLM-a ma ograniczoną skalowalność.

Źródło: [VentureBeat](https://venturebeat.com/security/openais-pre-release-gpt-6-2-astra-was-easier-to-jailbreak-than-its-predecessor-uk-government-testers-found)

#### PHPAfterShell (CVE-2026-126714): przerwany zapis sesji w domyślnym handlerze plikowym PHP prowadzi do deserializacji obiektu i RCE (TLDR InfoSec)
Przerwany zapis sesji w handlerze `files` zostawia na dysku dane serializowane kontrolowane przez atakującego. Kolejne żądanie deserializuje je do obiektu, co przy odpowiednich gadżetach daje wykonanie kodu. To inny CVE niż opisany w rejestrze 19.11 (`session.upload_progress`, CVE-2026-115560).

**Analiza:** Mitygacja: aktualizacja do wersji z 19.11 lub nowszych, sprawdzenie czy łatka obejmuje ten CVE, oraz przeniesienie sesji do Redis/Memcached z `session.serialize_handler=php_serialize` i ograniczeniem `unserialize` do allowed_classes. Na współdzielonych hostingach z wspólnym `session.save_path` ryzyko jest większe.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/phpaftershell-explained-how-hackers.html)

#### GitHub Enterprise Server 3.23 RC: domyślny cooldown Dependabota, opcjonalne wygasanie tokenów OAuth, wykrywanie sekretów AI w air-gap (TLDR IT)
RC wprowadza domyślny okres karencji (cooldown) dla aktualizacji wersji przez Dependabota, opcjonalne wygasanie tokenów OAuth, wykrywanie sekretów oparte na AI dla środowisk odciętych od sieci i rozszerzone wykrywanie malware w kilku ekosystemach pakietów.

**Analiza:** Cooldown Dependabota to bezpośrednia odpowiedź na serię kompromitacji pakietów z oficjalnych kont (rejestr: `agent-framework`, WPPConnect). Opóźnienie przyjęcia nowej wersji o kilka dni wycina większość złośliwych wydań wykrytych i wycofanych w pierwszych godzinach. Wygasanie tokenów OAuth domyka typowy wektor persystencji. Warto włączyć oba w GHES od razu po GA.

Źródło: [GitHub Changelog](https://github.blog/changelog/2026-11-20-github-enterprise-server-3-23-release-candidate/)

#### AWS Pricing Calculator API i hotpatch Windows 11 psujący aplikacje (TLDR IT)
- AWS udostępnił API kalkulatora cen: do 500 linii użycia na żądanie, ceny publiczne lub indywidualne klienta. Pozwala wpiąć estymację kosztu w pipeline IaC (np. bramka w PR dla zmian Terraform).
- Microsoft potwierdził awarie aplikacji po listopadowym hotpatchu Windows 11 i podał obejście przez rejestr wymagające restartu. To podważa główną zaletę hotpatchingu (brak restartu). Pierścienie wdrożeniowe dla hotpatchy powinny mieć ten sam pilotaż co dla klasycznych aktualizacji.

Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/aws-pricing-calculator-api/), [The Register](https://www.theregister.com/os-platforms/2026/11/20/microsoft-suggests-registry-hack-to-work-around-hotpatch-woes/5311279)

## Implikacje praktyczne

1. **Konsolidacja bezpieczeństwa wymusza strategię wyjścia.** Po CrowdStrike–Zscaler i Palo Alto–Applied Compute rynek EDR+SSE zmierza do 2–3 platform. Przy odnowieniach w latach 2027–2028 trzeba zabezpieczyć w umowach ceny, wsparcie integracji z innymi EDR i eksport polityk (np. do formatu neutralnego). Architektura zero-trust powinna mieć wymienne PEP, żeby zmiana dostawcy SSE nie wymagała przebudowy całego dostępu.
2. **Ryzyko polityczne jako parametr łańcucha dostaw GPU.** Powrót Huanga po interwencji Białego Domu i reguła BIS obejmująca węzły 16/14 nm pokazują, że dostęp do krzemu i licencje zależą od polityki, a nie tylko od mocy produkcyjnych. W planach na 3–5 lat warto mieć dywersyfikację (AMD, TPU/Trainium przez chmurę) i klauzule umowne na wypadek utraty licencji na zdalny dostęp.
3. **Kontrakty z neocloudami oparte na zabezpieczeniach, nie na kursie akcji.** Wahania ±20–30% dziennie i wstrzymane IPO (Nscale) oznaczają, że kondycja dostawcy może się zmienić w trakcie kontraktu. Potrzebne są escrow przedpłat, prawa do sprzętu lub step-in, SLA z karami realnie egzekwowalnymi oraz plan migracji obciążeń w 90 dni.
4. **Lokalne NVMe w serwerach GPU jako warstwa storage.** Rozwiązania typu tier 0 (pNFS na lokalnych dyskach) mogą wyeliminować część osobnego storage dla checkpointów. Projektując klastry 2027+, trzeba jednak uwzględnić separację ruchu storage od kolektyw, erasure coding między węzłami i procedury serwisowe, które nie niszczą danych.
5. **Serverless GPU dla inferencji o zmiennym ruchu.** Lambda na G-series i podobne usługi upraszczają operacje, ale narzut 9,75% i fallback na starsze generacje wymagają SLO definiowanych na najgorszej generacji. Duże, stałe obciążenia pozostają tańsze na K8s z rezerwacjami.

## Trend tygodnia

Tydzień otworzył się gwałtownym odbiciem, które pokazuje, jak bardzo rynek AI zależy dziś od pojedynczych decyzji zarządczych i politycznych, a nie od danych operacyjnych. Przywrócenie Huanga po interwencji prezydenta dodało Nvidii ok. 1 bln USD kapitalizacji w jeden dzień. Nie zmieniła się jednak żadna z fundamentalnych zmiennych: spadająca marża, rosnące zobowiązania zakupowe, wygasające kontrakty GPU w 2027 r. i drożejący kapitał. Jednocześnie Waszyngton rozszerza kontrolę eksportu na węzły sprzed dekady, a dostęp do krzemu i licencji staje się instrumentem polityki przemysłowej, a nie tylko kwestią handlową. Neocloudy z wąską bazą klientów pierwsze odczuwają skutki: Nscale tnie koszty i odkłada IPO, a kredytodawcy dopisują do umów covenanty dotyczące lokalizacji GPU. Po stronie technologii trwa „uproszczenie przez zarządzanie”: AWS przenosi firewall na IGW, Google chowa service mesh do węzła, Lambda dostaje GPU, a VMware wystawia NSX przez MCP. Każdy z tych kroków zmniejsza koszt operacyjny, ale przenosi kontrolę i lock-in do dostawcy platformy. Konsolidacja bezpieczeństwa (CrowdStrike–Zscaler) domyka ten obraz: mniej dostawców, większe platformy, wyższa cena wyjścia.

## To obserwować

- **Kurs NVDA i CRWV w kolejnych sesjach:** czy rajd z 23.11 utrzyma się powyżej poziomów sprzed 20.11 (scenariusz BTIG zakłada ponowne spadki), oraz zeznania Huanga przed Kongresem 1.12.
- **Oficjalne stanowisko TSMC/BIS w sprawie progu 16/14 nm** i liczba udzielonych licencji przed końcem okresu przejściowego 21.12.
- **Nscale:** decyzja o terminie IPO, nowe finansowanie dłużne lub mostowe, decyzja BIS w sprawie kontraktu z ByteDance.
- **CrowdStrike–Zscaler:** formularz S-4 (warunki, break fee), reakcja akcjonariuszy ZS przy spadku kursu CRWD, ewentualne postępowania antymonopolowe.
- **AWS re:Invent (30.11–4.12):** cennik trybu firewall na IGW, rozszerzenie Lambda GPU na instancje P-series i ceny Trainium.
