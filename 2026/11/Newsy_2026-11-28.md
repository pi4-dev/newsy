# Raport Zbiorczy IT – 2026-11-28

Zakres: od edycji z 2026-11-27 (rano) do 2026-11-28 07:30 Europe/Warsaw. Okres obejmuje piątek po Święcie Dziękczynienia w USA, więc źródeł pierwotnych jest mało. Sekcje „euro neocloud” i „AI for networking” pominięto, bo nie było w nich nowych, zweryfikowanych informacji.

## Raport technologiczny

### Biznes

#### RETN: czas dostaw sprzętu optycznego wydłużył się z 3 do 12–15 miesięcy, niektóre zamówienia do 18 miesięcy; ceny +50% od początku 2026 r. (27.11)
Operator sieci szkieletowej RETN w raporcie „AI, Connected” podaje, że sprzęt optyczny, który dotąd przychodził w ciągu 3 miesięcy, dociera dziś po 12–15 miesiącach, a niektóre zamówienia po 18. Ceny wzrosły o ok. 50% od początku roku. Według RETN sytuacja jest „co najmniej tak zła jak w pandemii”. Przyczyny: DC AI i hyperscalerzy konkurują o te same komponenty, a operatorzy muszą teraz nadrabiać odłożone wcześniej modernizacje. Klienci oczekują już czterech lub więcej niezależnych tras dalekosiężnych zamiast dwóch. Obwód z dywersyfikacją trasy kosztuje ok. 1 800 EUR miesięcznie wobec 600 EUR za obwód bez niej. DC budowane poza głównymi hubami muszą angażować operatorów 2–5 lat przed uruchomieniem.

**Znaczenie:** Łączność przestaje być pozycją, którą zamawia się na końcu projektu. Przy 12–18 miesiącach oczekiwania na sprzęt transportowy to sieć, a nie zasilanie, może opóźnić uruchomienie DC w nowej lokalizacji. Modernizacje WAN/DCI (400G ZR/ZR+, ROADM) trzeba zamawiać już teraz z terminem na 2H 2027 i budżetować z założeniem dalszych podwyżek. Ryzyko praktyczne: operatorzy nie zrealizują dodatkowych tras w obiecanym terminie, więc wymagania redundancji N+2 na papierze nie przełożą się na fizycznie rozdzielne światłowody. Trzeba weryfikować KMZ/trasy, a nie same SLA.

Źródło: [The Register](https://www.theregister.com/networks/2026/11/27/ai-boom-leaves-network-operators-waiting-up-to-18-months-for-kit/5312366)

### Technologia

#### UK DWP wyłącza część usług online po „incydencie cybernetycznym” (27.11)
Brytyjski Department for Work and Pensions zapobiegawczo wyłączył w piątek rano część usług online. Wypłaty świadczeń i emerytur mają przebiegać normalnie. Nie ujawniono, których usług to dotyczy, kiedy wrócą, czy wyciekły dane ani kto stoi za atakiem. DWP współpracuje z NCSC i Government Cyber Coordination Centre. W tle grupa ExfilSquad przyznała się niedawno do kradzieży danych z Department for Education i Police National Legal Database. Związku z DWP nie potwierdzono.

**Znaczenie:** To kolejna w tym miesiącu instytucja publiczna, która izoluje usługi frontowe, a nie tylko segmentuje sieć (wcześniej Komisja Europejska, EU Academy). Wzorzec „najpierw wyłączyć, potem diagnozować” pozwala zachować płatności tylko dlatego, że systemy rozliczeniowe są oddzielone od kanałów cyfrowych. Warto to potraktować jako test projektowy: czy w Twojej architekturze można wyłączyć warstwę internetową bez zatrzymania procesów rdzeniowych?

Źródło: [The Register](https://www.theregister.com/security/2026/11/27/british-social-security-system-techies-yank-services-offline-following-cyber-incident/5312522)

---

#### DIGIT (platforma e-government Indii, ok. 1 000 wdrożeń w 10 krajach): SQL injection bez uwierzytelnienia, CVE-2026-136706, CVSS 9,3 (27.11)
CERT-In opisuje lukę w komponentach User i Access Control Service. Zdalny, nieuwierzytelniony atakujący wysyła spreparowane dane do endpointu API i wykonuje dowolne zapytania SQL, czyli może odczytać, zmienić lub usunąć dane. Poprawka jest w Digit-Core v2.9.3. Nie ma doniesień o wykorzystaniu luki w praktyce.

**Znaczenie:** Luka siedzi w serwisie tożsamości i kontroli dostępu współdzielonym przez wszystkie moduły, więc jedna podatność daje dostęp do całej bazy. Organizacje, które budują własne platformy na mikroserwisach z centralnym serwisem użytkowników, mają ten sam problem. Taki serwis wymaga osobnego budżetu testów (DAST na API, parametryzowane zapytania wymuszane w CI) i nie może korzystać z tego samego użytkownika bazy co pozostałe moduły.

Źródło: [The Register](https://www.theregister.com/security/2026/11/27/free-e-government-platform-that-serves-two-billion-people-in-ten-countries-has-critical-sql-injection-bug/5312449)

## Raport AI-ML

### Biznes

#### Bloomberg: szafa GB300 NVL72 i serwer GB200 jako zabezpieczenie kredytów w szanghajskim DC powiązanym z Megaspeed (27.11)
Według chińskich rejestrów zastawów Shanghai Shuoyao zastawiła szafę GB300 NVL72 pod kredyt 96 mln CNY (ok. 14 mln USD) w Shanghai Innovation Bank. Sprzęt stoi w DC „Yangtze River Delta Project”. Bloomberg zidentyfikował szafę przez publiczną bazę gwarancyjną Gigabyte. Druga spółka, Shanghai Kezhan (w 83,3% należy do NeXPower Japan, według Caixin kontrolowanej przez Alice Huang), zastawiła serwer GB200 pod kredyt 90 mln CNY. Bloomberg zaznacza, że nie ma dowodu na bezpośrednie przekierowanie sprzętu przez Megaspeed. Spółka zaprzecza naruszeniom.

**Analiza techniczna i rynkowa:** To pierwszy dowód z dokumentów, a nie z relacji świadków, że kompletne szafy NVL72 (ok. 120–140 kW, chłodzenie cieczą, NVLink Switch) działają w Chinach. Takiej szafy nie da się przemycić w częściach jak pojedynczych kart, bo wymaga integracji u ODM i infrastruktury CDU. Kluczowy jest wektor identyfikacji: numery seryjne z bazy gwarancyjnej ODM. Można się spodziewać, że BIS i Nvidia zaczną wymagać telemetrii lokalizacji lub atestacji sprzętu (por. dyskusja o location verification) jako warunku wsparcia gwarancyjnego. To z kolei wpłynie na serwis RMA u wszystkich operatorów, także europejskich. W krótkim horyzoncie wydłuża to przegląd zgodności i pauzę dostaw do ASEAN (rejestr 27.11).

Źródła: [Yahoo/Bloomberg](https://finance.yahoo.com/technology/articles/nvidia-blackwell-chips-turn-china-114546617.html), [Engadget](https://www.engadget.com/2329748/restricted-nvidia-chips-were-installed-at-a-chinese-data-center-linked-to-alice-huang/)

### Technologia

#### SemiAnalysis: Trainium3 w InferenceX – najlepszy perf/TCO poza Vera Rubin; przy 135 tok/s/użytkownika marża 73,5% wobec 53% dla GB300 + SGLang (27.11)
Pomiary na rzeczywistych sesjach agentów kodujących (AgentX, GLM-5.3) pokazują, że Trainium3 z vLLM ma na całej krzywej TCO wobec interaktywności p90 lepszy wynik niż wszystkie akceleratory poza Vera Rubin. Teoretycznie Trainium3 ma 17% mocy FP4 GB300 (2 517 wobec 15 000 TFLOPS) oraz 4,9 TB/s HBM (61% GB300) przy ok. 30% TCO i 144 GB HBM na układ (połowa GB300). Dlatego wyraźnie przegrywa w TTFT, które zależy od prefillu ograniczonego mocą obliczeniową. Podział pracy: prefill to pipeline 16 etapów, po jednym układzie na etap, z TP/EP wewnątrz układu; decode rozkłada model na 128 NeuronCore jako niezależne ranki EP. Mikrobenchmarki: all-to-all wewnątrz układu ma 9–10× większą przepustowość niż na poziomie węzła lub szafy. EFA między szafami osiąga ok. 8 GB/s wobec 31 GB/s w NeuronLink. Trn3n z jedną płaszczyzną przełączników (Astera Scorpio X 320 linii) daje 2,8× przepustowości all-to-all. MXFP8 przy pełnym obciążeniu układu dławi się do 68,6% MFU.

**Analiza:** Wyniki potwierdzają, że w dekodowaniu MoE liczy się przepustowość HBM na dolara, a nie FLOPS. To argument za pulami heterogenicznymi: prefill na GPU, decode na ASIC. Ograniczenia: topologia wymusza utrzymanie all-to-all w obrębie układu lub szafy, więc modele z większą liczbą ekspertów niż 2 na NeuronCore lub z dłuższym kontekstem (KV ponad 144 GB) zjadają przewagę. Marża 73,5% zakłada ceny listowe Z.ai, wykorzystanie floty 60% i stawki 3-letnie. To model, nie obserwowany P&L. Lock-in: NKI/Neuron i ręczne mapowanie ranków. Przenośność zapewnia głównie vLLM. Dla capacity planningu: jeśli celem jest TTFT poniżej 1 s przy długich promptach, Trainium3 wymaga nadmiarowego prefillu albo prefillu na GPU.

Źródło: [SemiAnalysis – AWS Challenger: Trainium3](https://newsletter.semianalysis.com/p/trainium3-inferencex)

---

#### NERC: pierwsze standardy dla obciążeń obliczeniowych ≥50 MW przy ≥100 kV – modele poboru, nastawy wyłączeń, dane w czasie rzeczywistym; kryteria oscylacji w 2027 r. (IEEE Spectrum, 24.11)
Standardy przyjęte w październiku (czekają na zatwierdzenie FERC) wymagają od takich obiektów: aktualnych modeli poboru mocy i odpowiedzi na zakłócenia, w tym nastaw wyłączeń, telemetrii w czasie rzeczywistym, wykonywania poleceń operatora sieci (redukcja lub utrzymanie poboru), koordynacji zabezpieczeń z dostawcą energii oraz rejestratorów zakłóceń. Bezpośrednią przyczyną było zdarzenie z 22.07: po rutynowym zwarciu na linii obiekty w północnej Wirginii o łącznym poborze 3 800 MW przeszły na zasilanie awaryjne. Od 2022 r. było co najmniej osiem zdarzeń ze zrzutem 200–1 800 MW. Dominion zmierzył w obiekcie 100 MW wahania 10–15 MW co 2,5 s. Symulacje ETH Zurich pokazują rezonans z częstotliwościami własnymi sieci.

**Analiza:** Synchroniczny trening (checkpointy i all-reduce co kilkanaście minut) tworzy pulsujące obciążenie. W 2027 r. stanie się ono przedmiotem kryteriów zgodności. Projekt DC AI ≥50 MW musi uwzględniać: ride-through UPS/BESS dostrojony do wymagań operatora (zamiast agresywnego przełączania na diesla), wygładzanie poboru (BESS, firmware'owe ograniczanie mocy GPU, „dummy load” w przerwach treningu) oraz udostępnianie modeli EMT operatorowi. Koszt: dodatkowy BESS rzędu kilku–kilkunastu procent mocy IT. Failure mode: zbyt czułe nastawy UPS, które przy zwarciu w sieci odłączają cały kampus i pogłębiają zakłócenie. Analogiczne wymagania należy przewidzieć w UE (kodeks sieciowy dla dużych odbiorów).

Źródła: [IEEE Spectrum](https://spectrum.ieee.org/nercs-data-center-power-rules), [POWER Magazine](https://www.powermag.com/nerc-reliability-standards-for-data-centers-and-crypto-loads-pass-final-ballot-advance-to-board-review/)

## Ryzyka rynku AI/GPU

#### Citi i JPMorgan oferują portfele kredytów AI i syntetyczne transfery ryzyka (SRT) (Semafor, 26.11)
Według Semafor (relacje wtórne, oryginał za paywallem) Citi i JPMorgan oferują inwestorom portfele kredytów związanych z AI. Część banków proponuje SRT: kredyt zostaje w bilansie banku, a straty z określonej transzy przejmują inwestorzy. Nie ujawniono, o jakie kredyty, kwoty ani inwestorów chodzi. Motywy są mieszane: zarządzanie ekspozycją albo zwolnienie kapitału na dalsze kredytowanie. Rozmowy prowadzone są dyskretnie, żeby nie urazić kredytobiorców. Tło: hyperscalerzy wyemitują w 2026 r. ponad 305 mld USD obligacji (BoE), a JPMorgan prognozuje 450 mld USD długu IG związanego z AI w 2027 r. wobec 368 mld w 2026 r.

**Analiza:** To kolejny po obniżce LTV (z 80% do 50–60%, rejestr 25.11) sygnał, że banki chcą ograniczyć ekspozycję na finansowanie GPU i DC, nie wycofując się formalnie z relacji. SRT nie usuwa ryzyka z systemu, tylko przenosi je do funduszy kredytowych o mniejszej przejrzystości. W razie spadku wartości zabezpieczeń (stawki GPU, rejestr 27.11) straty ujawnią się z opóźnieniem i w trudniejszym do zidentyfikowania miejscu. Praktyczny skutek dla odbiorców mocy: koszt kapitału dla neocloudów bez kontraktu z kontrahentem IG nadal będzie rósł, a refinansowanie 2027 r. stanie się punktem krytycznym.

Źródła: [Benzinga](https://www.benzinga.com/markets/tech/26/11/63664911/citigroup-jpmorgan-chase-banks-reportedly-shop-ai-loans-synthetic-risk-transfer), [The Next Web – BoE](https://thenextweb.com/news/ai-financing-bank-of-england-financial-stability-hyperscaler-bonds)

---

#### OpenRouter: elastyczność wydatków na tokeny spadła z 5,5 (Q2) do 1,2 (Q4); wydatki płaskie od początku października (26.11)
Według OpenRouter wolumen płatnych tokenów wzrósł od początku 2026 r. 44-krotnie, a wydatki 7,6-krotnie, przy średniej cenie tokena niższej o 83%. Od lipca tokeny rosną o ok. 39% m/m, a wydatki m/m są bliskie zera. Elastyczność wydatków względem ceny: Q1 5,4, Q2 5,5, Q3 1,6, Q4 (do 16.11) 1,2. Podobne spadki do ok. 1 wystąpiły w 12.2024 i 11.2025 i potem się odwróciły. Równolegle niezależny pomiar (Agent Duel) pokazuje istotny statystycznie spadek liczby tokenów rozumowania i tokenów wyjściowych na zadanie w Astra 6.2 i Sol 6.2 (p od 0,00013 do 0,022). Autor interpretuje to jako tryb kontroli kosztów u OpenAI.

**Analiza:** To bezpośredni wskaźnik z obszaru EKONOMIKA. Jeśli elastyczność utrzyma się w okolicach 1, spadek ceny tokena przestaje zwiększać przychód, więc przychód na GPUh u dostawców inferencji spada wraz z ceną. Jednocześnie dostawca modelu ogranicza liczbę tokenów na zadanie, żeby bronić marży. Razem oznacza to presję na stawki najmu GPU (zgodną z sygnałami z rejestru 27.11) i słabsze uzasadnienie dla capexu inferencyjnego w 2027 r. Zastrzeżenie: OpenRouter to wycinek rynku z nadreprezentacją deweloperów i modeli otwartych, a poprzednie dwa takie spadki się odwróciły.

Źródła: [OpenRouter](https://openrouter.ai/blog/insights/is-jevons-paradox-holding/), [micjm.com – OpenAI cost control](https://micjm.com/blog/openai-is-in-cost-control-mode/)

## Newsletters summary

#### Hex.pm: RCE na sześciu serwerach WWW rejestru pakietów Elixir/Erlang przez łańcuch deserializacji Erlang terms (CVE-2026-129514) (TLDR InfoSec)
23.11 między 18:14 a 18:29 UTC nieuwierzytelnione żądania z typem `application/vnd.hex+erlang` przemyciły struktury Elixira przez dekoder. Logowanie błędu (Inspect) uruchomiło łańcuch gadżetów Ecto → DBConnection, który dał wykonanie komend jako użytkownik BEAM. Brakujące zabezpieczenie usunięto w 2016 r. Poprawka weszła po 2 minutach, poświadczenia zrotowano do 24.11. Nie stwierdzono modyfikacji pakietów, ale nie można wykluczyć odczytu e-maili, hashy bcrypt, hashy kluczy API i sekretów 2FA, bo odczyty z bazy nie są logowane.

**Analiza:** Klasyczny błąd: niezaufana deserializacja plus efekt uboczny w ścieżce logowania błędów. W rejestrach artefaktów trzeba zakładać kompromitację sekretów 2FA, czyli resetować TOTP, a nie tylko hasła. Do czasu nowego klienta z podpisami należy pobierać pakiety wyłącznie z repo.hex.pm. Dla prywatnych mirrorów (Artifactory/Nexus) oznacza to weryfikację cache z okna 23.11.

Źródło: [hex.pm](https://hex.pm/blog/security-incident-on-hexpm)

---

#### PolinRider (KRLD): force-push loaderów do co najmniej 7 609 repozytoriów GitHub przez skradzione tokeny (TLDR InfoSec)
Kampania przepisuje istniejące commity i ukrywa kod za setkami spacji w konfiguracjach buildów, `conftest.py` i `.vscode/tasks.json`, więc wystarczy zbudować projekt lub otworzyć folder. Adres C2 i następny etap są czytane z transakcji Ethereum (schemat „NullReceiver”), więc zdjęcie serwerów nic nie daje. Do zablokowania: 95.135.181[.]84 i 103.179.142[.]155.

**Analiza:** Ochrona gałęzi z zakazem force-pusha i wymogiem podpisanych commitów to minimum dla każdego repo, także prywatnego. Zadania auto-run w VS Code (`task.allowAutomaticTasks`) oraz automatyczne wykonywanie `conftest.py` na runnerach CI powinny być wyłączone lub sandboxowane. Detekcja C2 opartego na blockchainie wymaga monitorowania zapytań RPC do węzłów Ethereum z sieci deweloperskich.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/north-korea-linked-polinrider-poisons.html)

---

#### Przejęcie subdomen US Treasury za ok. 3 USD na sztukę przez rotację Elastic IP w AWS (Intruder, TLDR InfoSec)
Badacze w 48 h przydzielili i zwolnili 450 tys. adresów EIP. Dopasowali 28 adresów do „wiszących” rekordów DNS, w tym pięć subdomen treasury.gov (m.in. rdgw, irssearch). Po blokadzie konta EIP dało się nadal przenieść na inne konto w tej samej organizacji. AWS nie planuje zmian.

**Analiza:** Rekordy A wskazujące na publiczne IP chmury bez cyklu życia powiązanego z zasobem to dług techniczny o wysokim ryzyku: TLS z ACME, phishing, przechwytywanie poświadczeń ze starych automatyzacji, ciasteczka domeny nadrzędnej. Potrzebna jest ciągła inwentaryzacja DNS względem przydzielonych IP (np. Route 53 + AWS Config) i preferencja dla aliasów lub CNAME na zasoby zarządzane.

Źródło: [Intruder](https://www.intruder.io/research/taking-over-us-treasury-subdomains-for-3-each)

---

#### easyDNS: natychmiastowa rotacja kluczy AWS przechowywanych w panelu do integracji z Route 53 (TLDR IT)
easyDNS wzywa do rotacji kluczy AWS zapisanych w panelu klienta po doniesieniach o ich kompromitacji. Szczegółów incydentu nie podano.

**Analiza:** Trwałe klucze IAM przekazywane zewnętrznym dostawcom DNS lub SaaS to ten sam wzorzec co w incydentach Rackspace Spot i Mail Assure. Integracje należy przenieść na role cross-account z ExternalId lub OIDC i ograniczyć uprawnienia do `route53:ChangeResourceRecordSets` w konkretnych strefach.

Źródło: [easyDNS](https://easydns.com/blog/2026/11/26/urgent-rotate-aws-keys-stored-in-easydns-cp-immediately/)

---

#### Samsung Knox DualDAR: root z aplikacji bez uprawnień na Galaxy S23–S26 (CVE-2026-21102, poprawki 08–09.2026) (TLDR InfoSec)
Wywołanie `relayAsync` przekazuje komendy do wątku system_server z pominięciem sprawdzenia UID. Pozwala to załadować natywny kod do demona dualdard, a use-after-free w sterowniku `/dev/dd` daje odczyt i zapis pamięci jądra, wyłączenie SELinux, obejście RKP i DEFEX.

**Analiza:** Funkcje „enterprise” (DualDAR to szyfrowanie danych dla sektora rządowego) poszerzają powierzchnię ataku. Floty MDM powinny wymuszać poziom poprawek co najmniej z 09.2026 jako warunek dostępu (conditional access) do zasobów firmowych.

Źródło: [Calif](https://calif.io/research/oempocalypse-part-2)

---

#### Nebius LayerStream: fine-tuning MoE z optimizerem i aktywacjami w pamięci hosta – 3,15× przepustowości wobec Megatron-LM, −68% kosztu GPU na token (TLDR AI)
Silnik trzyma stan optimizera i zapisane aktywacje w RAM hosta i przesyła stan roboczy warstwy na GPU tylko na czas jej wykonania. Na 16× H200 epoka na 25,8 mln tokenów zajęła 7,1 min zamiast 22,4 min.

**Analiza:** Podejście zamienia ograniczenie pojemności HBM na ograniczenie przepustowości PCIe/C2C. Dobrze sprawdza się na GH200/GB200 z NVLink-C2C, gorzej na x86 z PCIe Gen5. Przy drożejącym HBM i DRAM wydaje się korzystne, ale wymaga węzłów z dużą ilością RAM, a to też rosnący koszt. Wynik z jednego zbioru i jednej konfiguracji, porównanie z „dostrojonym” Megatronem podał sam producent.

Źródło: [Nebius](https://nebius.com/blog/posts/layerstream-moe-fine-tuning)

---

#### TSMC A14 gotowy do projektowania (PDK 0.9): +23% gęstości logiki i do −50% energii (wg innej relacji −23% mocy) wobec N2P (TLDR Hardware)
Druga generacja nanosheet z elastycznymi typami komórek (Boost na ścieżkach krytycznych, Low-Power w pozostałych miejscach). Późniejsze przejście na A13 zmniejsza powierzchnię o ok. 6% bez zmiany przepływu projektowego.

**Analiza:** Dla akceleratorów z tape-outem w latach 2028–2029 A14 to realna alternatywa wobec N2P/A16. Brak backside power w bazowym A14 oznacza, że projekty o dużej gęstości mocy (ponad 1,5 kW na pakiet) mogą nadal wybierać A16. Uwaga: relacje różnie podają poprawę mocy, więc wartości trzeba weryfikować u TSMC.

Źródło: [Tom's Hardware](https://www.tomshardware.com/tech-industry/semiconductors/tsmcs-a14-1-4nm-class-node-is-ready-for-chip-design-boost-cells-promise-14-percent-performance-uplift-or-23-percent-power-cut-over-n2p)

---

#### SK hynix: masowa produkcja 3D DRAM do 2031 r., ok. 100 warstw, technologia licencjonowana od YMTC (TLDR Hardware)
Wymaga stosowania wielu decków i hybrid bondingu oraz całkowitej przebudowy architektury i procesu DRAM.

**Analiza:** Horyzont 2031 oznacza, że niedobór pojemności DRAM/HBM nie zostanie rozwiązany zmianą architektury komórki przed generacjami Rubin Ultra i Feynman. Licencja od YMTC to nowe ryzyko regulacyjne (kontrola eksportu USA–Chiny) w łańcuchu dostaw pamięci.

Źródło: [TechPowerUp](https://www.techpowerup.com/355210/sk-hynix-to-start-3d-dram-production-in-2031)

---

#### AMD Helios jako „appliance” w formacie OCP ORW dla mniejszych dostawców chmury (Microprocessor Report, TLDR Hardware)
Helios łączy CPU, GPU i Ethernet AMD z ROCm w szafie OCP ORW i jest pozycjonowany jako zintegrowana alternatywa dla NVL72. Według analizy dla mniejszych dostawców, bo hyperscalerzy stawiają na Nvidię lub własne ASIC.

**Analiza:** Model „appliance” obniża koszt integracji, ale przenosi ryzyko jednego dostawcy na cały stos (sieć UALink/UEC, firmware, ROCm). Dla neocloudów oznacza to dywersyfikację względem alokacji Nvidii przy wyższym ryzyku operacyjnym (dojrzałość narzędzi, wsparcie RMA).

Źródło: [Microprocessor Report](https://www.microprocessorreport.com/p/quick-take-amds-helios-appliance)

---

#### CBS News: RiCloud AI i Aivres – kalifornijskie spółki z chińskim właścicielem kupują najnowsze układy Nvidii (TLDR Hardware)
CBS opisuje sprzedaż przez te spółki do firm w Azji Południowo-Wschodniej powiązanych z chińskimi koncernami. Zwraca też uwagę, że zmiany w raportowaniu Nvidii mogą ukrywać lokalizację klientów. Możliwe jest śledztwo Senatu. Powiązanie Aivres z Inspur było już opisywane przez WSJ i NYT. RiCloud nie udało się niezależnie zweryfikować.

**Analiza:** Razem z dowodami z Szanghaju (sekcja AI-ML) wzmacnia to presję na KYC aż do klienta końcowego. Należy się spodziewać rozszerzenia wymogów na amerykańskie spółki zależne podmiotów z Entity List.

Źródło: [CBS News](https://www.cbsnews.com/news/nvidia-china-ai-chips-computing-sales/)

---

#### Nvidia według doniesień tnie dostawy GPU gamingowych o kolejne 20–30% w Q4 (TLDR Hardware, niepotwierdzone)
Według relacji wtórnych wafle są przesuwane z GeForce do produkcji DC. Możliwe niedobory do stycznia.

**Analiza:** Jeśli się potwierdzi, będzie to kolejny sygnał, że Nvidia priorytetyzuje wafle i pamięć GDDR7/HBM dla DC mimo słabszego sentymentu wokół popytu. Dla on-prem stacji inferencyjnych na kartach RTX (SM120) oznacza to wyższe ceny i dłuższe dostawy.

Źródło: [VideoCardz](https://videocardz.com/newz/nvidia-reportedly-cuts-gaming-gpu-supply-by-another-20-30-for-q4)

---

#### „Your model is a dependency”: zarządzanie LLM jak łańcuchem dostaw oprogramowania (TLDR AI / TLDR IT)
Sześć praktyk: przypinanie całej konfiguracji modelu (wersja, parametry, prompt systemowy), testy upgrade'u testem McNemara, AI-BOM, usuwanie zbędnych definicji narzędzi agentów, weryfikacja pochodzenia wag i plan wyjścia.

**Analiza:** W połączeniu z obserwowanym cichym ograniczaniem rozumowania w nowych wersjach modeli (sekcja Ryzyka) przypinanie wersji i statystyczne testy regresji stają się wymogiem operacyjnym, a nie dobrą praktyką. Bez nich zmiana dostawcy po stronie API zmienia jakość i koszt bez zmiany w kodzie.

Źródło: [cesarsotovalero.net](https://www.cesarsotovalero.net/blog/your-model-is-a-dependency.html)

## Implikacje praktyczne

1. **Sieć transportowa na ścieżce krytycznej projektów DC.** Przy 12–18 miesiącach oczekiwania na sprzęt optyczny i wzroście cen o 50% zamówienia DCI/WAN na 2027–2028 trzeba składać teraz, z rezerwą budżetową na dalsze podwyżki. Przy wyborze lokalizacji należy weryfikować fizyczną dywersyfikację tras światłowodowych na podstawie dokumentacji tras, a nie deklaracji SLA.
2. **Zgodność z wymogami sieci energetycznej jako parametr projektowy DC AI.** Standardy NERC (≥50 MW) i nadchodzące kryteria oscylacji wymuszają ride-through, wygładzanie poboru (BESS, power capping GPU) i udostępnianie modeli operatorowi. Analogicznych wymagań należy się spodziewać w UE w horyzoncie 3–5 lat. Projekt powinien je uwzględniać od fazy koncepcji, a nie retrofitu.
3. **Heterogeniczne pule inferencyjne.** Wyniki Trainium3 (decode) i Rubin/GB300 (prefill) wskazują na rozdział prefill i decode między różne krzemy jako sposób na obniżenie kosztu tokena. Wymaga to warstwy serwującej neutralnej wobec sprzętu (vLLM, llm-d/Dynamo) i przenoszalnego formatu KV. Ryzyko: rosnąca złożoność operacyjna i lock-in na runtime dostawcy ASIC.
4. **Przypinanie i testy regresji modeli.** Ciche zmiany w zachowaniu modeli w API (mniej tokenów rozumowania) oraz spadająca elastyczność wydatków oznaczają, że dostawcy będą optymalizować pod własną marżę. Kontrakty i architektura muszą zapewnić stałe wersje, statystyczne testy przy upgrade'ach i możliwość przełączenia na model self-hosted.
5. **Sekrety integracji zewnętrznych jako główny wektor.** Hex.pm, easyDNS, PolinRider i dangling DNS w AWS mają wspólny mianownik: trwałe poświadczenia i zasoby bez cyklu życia. W ciągu 3–5 lat celem jest zero trwałych kluczy w integracjach (OIDC, role cross-account), ochrona gałęzi z podpisami commitów i ciągła inwentaryzacja DNS względem zasobów chmurowych.

## Trend tygodnia

Pod koniec tygodnia ryzyko rynku AI widać mniej w kursach, a bardziej w mechanice finansowania i w danych o popycie. Banki, które jeszcze kilka miesięcy temu konkurowały o kredyty pod GPU, obniżają LTV, a teraz według doniesień sprzedają portfele i przenoszą straty przez SRT do mniej przejrzystych funduszy. Po stronie popytu OpenRouter pokazuje, że spadające ceny tokenów przestały zwiększać łączne wydatki, a niezależne pomiary wskazują, że dostawcy modeli sami ograniczają liczbę tokenów na zadanie, żeby bronić marży. W tym otoczeniu rośnie znaczenie kosztu tokena na poziomie krzemu: analiza Trainium3 pokazuje, że ASIC z połową HBM i ułamkiem FLOPS może dać wyższą marżę inferencji niż GB300, jeśli topologia jest dopasowana do MoE. Równolegle pojawiają się ograniczenia fizyczne spoza GPU: sprzęt optyczny czeka się do 18 miesięcy, regulatorzy sieci energetycznych wprowadzają wymagania dla obiektów ≥50 MW, a 3D DRAM nie pojawi się przed 2031 r. Kontrola eksportu wchodzi w fazę dowodową: identyfikacja szafy NVL72 w Szanghaju przez bazę gwarancyjną ODM zapowiada wymogi telemetrii i atestacji sprzętu, które obejmą wszystkich operatorów. Wniosek dla architektów: w 2027 r. przewagę dadzą elastyczność krzemu, kontrola kosztu tokena i realny harmonogram infrastruktury (sieć, energia), a nie sam dostęp do najnowszych GPU.

## To obserwować

- **Indeks elastyczności i wydatków OpenRouter (grudzień 2026):** czy elastyczność wróci powyżej 1,5, jak po 12.2024 i 11.2025, czy utrzyma się przy ok. 1. To wczesny wskaźnik przychodu na GPUh u dostawców inferencji.
- **Transakcje SRT i sprzedaż portfeli kredytów AI przez banki (Citi, JPM, ING, SocGen, HSBC):** wolumen, struktura transz i nabywcy. Do tego cena syndykacji kredytu CoreWeave pod kontrakt z Metą.
- **Czas dostaw transceiverów 800G/1,6T i sprzętu ROADM/ZR+:** raporty operatorów (RETN, Colt, Lumen) i dostawców (Ciena, Nokia, Coherent) oraz wyniki kwartalne pod kątem backlogu.
- **Zatwierdzenie standardów NERC CLO-001/002/003 przez FERC i projekt kryteriów oscylacji na 2027 r.** Do tego ewentualne odpowiedniki w ENTSO-E (Network Code Demand Connection).
- **Dostępność i cennik Trn3n (Trainium3 z jedną płaszczyzną przełączników) poza Anthropic** oraz publikacja CollectiveX przez SemiAnalysis: czy przewaga all-to-all potwierdzi się w dużych MoE.
