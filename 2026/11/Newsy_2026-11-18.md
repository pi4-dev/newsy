# Raport Zbiorczy IT – 2026-11-18

Okres: od edycji 2026-11-17 do 2026-11-18 (Europe/Warsaw). Główne nowe zdarzenie to wyniki Nvidii za Q3 FY2027 (17.11 wieczorem). Pominięte, bo opisane już wcześniej: Rubin HPC, MLPerf Training/Endpoints, QPU w Slurm, cuPhoton/cuBLAS FP64, wcześniejsze doniesienia o regule BIS (tu opisano tylko nowe elementy wydanego tekstu), umowa Nvidia–Hugging Face (ogłoszona 3.09).

## Raport technologiczny

### Technologia

#### RIKEN: włamanie do sieci operacyjnej Fugaku i RIKYU, kradzież kluczy SSH użytkowników (10–14.11, komunikat 17.11)
Atakujący wszedł do wewnętrznej sieci operacyjnej Fugaku przez publicznie dostępną stronę ze statusem systemu (dashboard). Zmodyfikował kilka serwerów wewnętrznych i umieścił na nich złośliwe programy. Przejął system uwierzytelniania (do 4 222 użytkowników Fugaku i 375 RIKYU: nazwiska, e-maile, ID) oraz katalogi domowe, w których leżały m.in. prywatne klucze SSH i klucze dostępowe do innych systemów. Atakujący próbował już użyć części tych poświadczeń wobec systemów zewnętrznych. Podejrzany ruch wykryto 12.11, włamanie potwierdzono 13.11, a Fugaku wyłączono 14.11 o 4:30 z przerwaniem uruchomionych i zakolejkowanych zadań. RIKEN wymienił klucze administracyjne, unieważnił wszystkie publiczne klucze logowania i zapowiada przegląd podziału odpowiedzialności z firmą, której zlecono utrzymanie.
**Znaczenie:** To typowy wzorzec dla HPC i klastrów GPU. Komponent „tylko do odczytu” (dashboard) stoi w tej samej strefie co sieć zarządzająca, a współdzielone katalogi domowe są magazynem poświadczeń do innych ośrodków. Kompromitacja jednego centrum staje się więc wektorem do federacji HPCI i do chmur użytkowników. Wnioski: dashboardy statusu publikować z osobnej strefy (push, nie pull z sieci operacyjnej), wprowadzić krótkotrwałe certyfikaty SSH (CA, TTL rzędu godzin) zamiast statycznych kluczy w `$HOME`, skanować katalogi domowe pod kątem sekretów. Wyłączenie systemu na kilka dni pokazuje też koszt braku segmentacji: przestój całej maszyny zamiast izolacji jednego węzła.
Źródło: [HPCwire (komunikat RIKEN)](https://www.hpcwire.com/off-the-wire/riken-warns-fugaku-and-rikyu-users-of-possible-personal-data-exposure/)

<hr>

#### NoisyTCP (CVE-2026-99288): zdalny kernel panic Linuksa bez uwierzytelnienia, błąd z 2015 r. (ujawnienie 15.11)
Hyunwoo Kim opublikował na oss-security wyścig w bezblokadowym wyszukiwaniu gniazd established w stosie TCP. Pakiety spreparowane przez atakującego powodują dereferencję NULL i panikę jądra. Podatne są domyślne konfiguracje serwerów nginx, Apache i SSH. Błąd pochodzi z commita 079096f103fa z 2015 r., więc dotyczy jąder od 4.4. Warunek: pakiety tego samego połączenia obsługiwane równolegle przez dwa CPU, co sprzyja maszynom wirtualnym i hostom z RSS/RPS. Druga wersja prowadzi do use-after-free, ale trudno ją wywołać. Autor ocenia, że RCE „basically, no”. Poprawka „inet: reject unhashed sockets during established lookups” trafiła do 6.12.117, 6.18.63 i 7.3.4. Obejście `net.ipv4.tcp_syncookies=0` osłabia ochronę przed SYN flood. Status w dystrybucjach jest niespójny (np. Ubuntu 26.04: „needs triage”).
**Znaczenie:** To DoS całego hosta, a nie pojedynczej usługi, osiągalny z internetu przez każdy port TCP. Na hostach z wieloma tenantami (węzły K8s, hypervisory z siecią hosta, load balancery L4 na Linuksie) jeden atakujący kładzie wszystkie workloady naraz. Wyłączenie syncookies to wymiana jednego ryzyka DoS na drugie, sensowna tylko za frontem anty-DDoS. Priorytet patchowania: brzeg (LB, reverse proxy, bastiony SSH), potem węzły z publicznymi IP. Należy też sprawdzić, czy obrazy appliance (firewalle i NAS oparte na Linuksie) mają jądro z poprawką.
Źródło: [BleepingComputer](https://bleepingcomputer.com/news/linux/new-noisytcp-linux-kernel-flaw-lets-attackers-crash-servers), [Corgea – analiza CVE-2026-99288](https://corgea.com/research/cve-2026-99288-noisytcp-linux-tcp-request-socket-race-remote-crash)

<hr>

#### Infoblox: przez dostawcę migracji SharePoint wyciekły kopie konfiguracji NIOS klientów (The Register, 16.11)
Infoblox potwierdził nieautoryzowany dostęp do danych firmy przez przejętego zewnętrznego dostawcę usług migracji danych. Według źródeł The Register dostawca miał zaufany dostęp do SharePointa Infoblox, a atakujący użył jego kont do eksfiltracji. Wśród danych miały być snapshoty konfiguracji części wdrożeń NIOS klientów: konfiguracje DNS/DHCP, bazy Grid i logi raportowe, używane do odtwarzania appliance lub gridu. Wyciekły też dane pracowników i ich rodzin (m.in. numery SSN i dane do przelewów). Infoblox twierdzi, że produkty, usługi chmurowe i proces build/release nie zostały naruszone. Nie podaje liczby poszkodowanych klientów.
**Znaczenie:** Backup gridu NIOS zawiera pełną mapę sieci (strefy, podsieci, rezerwacje DHCP, często klucze TSIG i poświadczenia integracji). To gotowy materiał rozpoznawczy do ataku na DNS klienta. Klienci, którzy przekazywali backupy do supportu, powinni założyć ich kompromitację: zrotować klucze TSIG, hasła kont serwisowych gridu i poświadczenia integracji (AD, IPAM, chmura), a także przejrzeć ACL transferów stref. Szerszy wniosek: dane diagnostyczne przekazywane do supportu dostawcy powinny podlegać tej samej klasyfikacji co konfiguracja produkcyjna.
Źródło: [The Register](https://www.theregister.com/security/2026/11/16/infoblox-customer-and-staff-data-exposed-in-sharepoint-breach/5310280)

<hr>

#### Ruby httpx 1.9.0 (CVE-2026-129784, CVSS 8,5): RCE przez deserializację Oj w trybie object
Gem httpx 1.9.0 przy braku `multi_json` parsował odpowiedzi JSON przez Oj w domyślnym trybie object. Złośliwy serwer mógł więc zwrócić dowolne obiekty Ruby. JFrog pokazał łańcuch gadżetów ActiveSupport w aplikacji Rails, który wywołuje `Kernel.load` na pliku wcześniej przesłanym zwykłym multipart POST i osiągalnym przez `/dev/fd/N`. Daje to wykonanie kodu jako użytkownik aplikacji bez interakcji. Poprawka w 1.9.1. Obejścia: `mode: :strict` w `response.json` albo zainstalowany `multi_json`.
**Znaczenie:** Wektor ataku to odpowiedź z serwera, do którego aplikacja sama się łączy: webhooki, integracje z API stron trzecich, SSRF. Klasyczny model „ufamy tylko wejściu użytkownika” go nie obejmuje. Warto przeskanować lockfile'e (Gemfile.lock) pod kątem httpx 1.9.0, sprawdzić, czy egress aplikacji Rails jest ograniczony do allowlisty, i potraktować ten przypadek jako argument za domyślnie bezpiecznymi trybami parserów.
Źródło: [JFrog Security Research](https://research.jfrog.com/vulnerabilities/httpx-is-vulnerable-to-remote-code-execution-via-oj-object-mode-deserialization-cve-2026-129784/)

### Biznes

#### Amazon (Kuiper): Vulcan LV-01 nie dostarczył 40 satelitów na orbitę (15/16.11)
Sześciobusterowa wersja Vulcan z Centaur V, przygotowana specjalnie pod konstelację Amazona, uległa anomalii ok. 10 minut po starcie. 40 satelitów nie trafiło na orbitę. LV-01 był pierwszą z 38 zaplanowanych misji ULA dla Amazona, a LV-02 miała polecieć w ciągu kilku tygodni. Ten harmonogram jest teraz prawdopodobnie wstrzymany. Amazon ma na orbicie ponad 200 satelitów, chciał uruchomić usługę jeszcze w tym roku i planował ponad 20 startów w 2027.
**Znaczenie:** Uziemienie Vulcana do czasu wyjaśnienia anomalii zmniejsza dostępną dla Amazona pojemność startową. Opóźnia to komercyjny start LEO alternatywnego wobec Starlinka. Dla planujących łącza zapasowe LEO (oddziały, edge, DR) oznacza to dłuższy okres praktycznego monopolu jednego dostawcy, ze słabszą pozycją negocjacyjną i ryzykiem koncentracji. Dywersyfikacja łączy zapasowych powinna na razie opierać się na LTE/5G od różnych operatorów, a nie na drugim dostawcy LEO.
Źródło: [The Register](https://www.theregister.com/networks/2026/11/16/amazon-loses-first-batch-of-broadband-satellites-launched-on-custom-rocket/5310000)

## Raport AI-ML

### Biznes

#### Nvidia Q3 FY2027: 115,8 mld USD przychodu, wytyczne Q4 130 mld, ale marża brutto spada do 71–72% (17.11)
Kwartał zakończony 25.10: przychód 115,8 mld USD (+20% kw/kw, +103% r/r), Data Center 107,6 mld (+21% kw/kw), w tym networking ponad 21 mld. Marża brutto 74,8% GAAP / 74,9% non-GAAP, zysk netto 65,3 mld, FCF 43,8 mld. Vera Rubin w pierwszym kwartale produkcyjnym dał prawie 20 mld USD. NVL72 działa już w CoreWeave, Google Cloud, Azure, OCI i Nebius. Wytyczne na Q4 (14 tygodni): 130 mld ±3% bez chińskiego przychodu DC compute, marża 71,0–72,0%, opex ok. 11 mld GAAP. Kress wiąże spadek marży z kosztem pamięci i realizacją zamówień przyjętych po starych cenach. Nowe ceny wejdą w FY2028, z marżą 72–73%. Wzrost w FY2028 podniesiono do ok. 75%. Licencjonowana sprzedaż Hoppera do Chin (2,4 mld w Q3) została wstrzymana.
**Analiza:**
- **Marża:** Spadek o ok. 3,4 pp przy rekordowym przychodzie oznacza, że koszt HBM/DRAM nie jest już w całości przenoszony na klientów w bieżących zamówieniach. Zapowiedź „nowych cen w nowym roku fiskalnym” to sygnał podwyżek ASP dla Rubin w 2027.
- **TCO:** Dla capacity planningu przyjmować, że cena za GPU w kontraktach na 2027 będzie wyższa niż w obecnych ofertach, a koszt pamięci przesuwa się z dostawcy na kupującego.
- **Vendor lock-in:** Wolumen Rubin skupiony u pięciu dużych operatorów sprawia, że mniejsze podmioty dostaną alokację później.
Źródło: [NVIDIA – wyniki Q3 FY2027](https://nvidianews.nvidia.com/news/nvidia-announces-financial-results-for-third-quarter-fiscal-2027), [More Than Moore (I. Cutress)](https://morethanmoore.substack.com/p/nvidia-q3-fy27-a-record-quarter)

<hr>

#### Nvidia: komitet rady ds. bezpieczeństwa narodowego i kontroli eksportu (17.11)
Rada dyrektorów powołała stały komitet do nadzoru nad zgodnością eksportową i ryzykiem handlowym. Skład: S. Nora Johnson (przewodnicząca), S. Neal, J. Dabiri. Komitet zbiera się co najmniej raz na kwartał i ma zewnętrznych ekspertów. Zespoły National Security i Global Trade Compliance raportują bezpośrednio do komitetu, a operacyjnie do tymczasowej CEO. Według relacji z konferencji wynikowej rada nie zamierza przywracać Huanga, a poszukiwania CEO prowadzi Heidrick & Struggles. Śledztwo eksportowe dotyczące Singapuru i Malezji pozostaje otwarte.
**Analiza:** Rada przenosi decyzje eksportowe z poziomu osobistych relacji prezesa na formalną strukturę compliance. Dla klientów spoza USA oznacza to dłuższe i bardziej sformalizowane procesy KYC/end-use przy zakupach dużych wolumenów (zob. reguła BIS w sekcji Ryzyka). Należy się liczyć z dodatkowymi oświadczeniami o użytkowniku końcowym i prawem audytu w umowach na DGX/HGX.
Źródło: [HPCwire (komunikat NVIDIA)](https://www.hpcwire.com/off-the-wire/nvidia-board-of-directors-establishes-national-security-and-export-control-oversight-committee/), [Filing Insights](https://filinginsights.substack.com/p/nvidias-q3-fy2026-call-record-results)

### Technologia

#### NVIDIA: współprojektowanie sparse attention dla inferencji z długim kontekstem (17.11)
Wpis analizuje sparse attention trenowane w modelu, z DeepSeek Sparse Attention (DSA) jako punktem odniesienia. Porównuje też projekty GLM 5.2/5.3 Flash, LongCat LSA-Flash, Qwen QSA, MiniMax MSA, NSA i MoBA. Przy 128K tokenów indexer i Top-k zajmują 47% czasu warstwy prefill DeepSeek V3.2 (dla porównania gęsta uwaga zajmuje 91% warstwy DeepSeek-R1). Guess-Verify-Refine Top-k w TensorRT-LLM używa wyniku z poprzedniego kroku dekodowania jako zgadywania i jest 2,1–2,4× szybszy od radix select. FlashInfer dostał selektywną ścieżkę Top-k bez materializacji pełnej macierzy logitów. Epilog FP16 daje 1,31× w dekodowaniu (1,45× z akumulacją FP16). Osobny cache indeksu FP8 skaluje się liniowo z 1/CR.
**Analiza:** Wąskim gardłem przestaje być sama uwaga, a staje się selekcja (indexer + Top-k). Optymalizacja kerneli selekcji daje więc więcej niż szybsze GEMM. Dla capacity planningu: modele z DSA/podobnymi mechanizmami mają dodatkowy cache indeksu obok KV cache, co trzeba uwzględnić w budżecie HBM i przy offloadzie KV (np. do pamięci CXL lub NVMe). Wydajność zależy od wsparcia kerneli w konkretnym runtime (TRT-LLM, FlashInfer). Na innym sprzęcie bez równoważnych kerneli te modele stracą przewagę, co tworzy miękki lock-in.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/co-designing-sparse-attention-for-efficient-long-context-inference/)

<hr>

#### DDN: EXAScaler jako oprogramowanie na kwalifikowanych serwerach (EXAScaler 7, beta na początku 2027)
DDN udostępni EXAScaler (Lustre) w wersji software-only na kwalifikowanych serwerach, zaczynając od DDN AI2200. Kolejne appliance'y innych firm mają zostać dodane później. Infinia jest już dostępna jako oprogramowanie. Ograniczona beta ruszy na początku 2027, a ulepszenia multi-tenancy, wydajności i ochrony danych mają osiągnąć GA w 1H 2027. Brak danych o wydajności i licencjonowaniu.
**Analiza:** Ruch defensywny wobec Hammerspace, VAST i WEKA, które sprzedają warstwę danych bez własnego sprzętu. Dla kupujących oznacza to możliwość oddzielenia cyklu wymiany serwerów od licencji systemu plików i ujednolicenia hardware w DC. Ryzyka: wydajność Lustre silnie zależy od sieci (RDMA, LNet) i firmware dysków, więc lista kwalifikowanych platform będzie krótka, a odpowiedzialność za problemy wydajnościowe trudniej przypisać. Do 1H 2027 traktować to jako opcję do testów, nie do produkcji.
Źródło: [HPCwire (komunikat DDN)](https://www.hpcwire.com/off-the-wire/ddn-introduces-software-defined-exascaler-deployment-for-enterprise-ai-infrastructure/)

## euro neocloud

### Nebius

#### Vera Rubin NVL72 już działa w Nebius; Nvidia deklaruje „selektywność” wobec neocloudów (17.11)
**Fakty:** W komunikacie wynikowym Nvidia wymienia Nebius wśród pięciu operatorów z działającymi NVL72 Vera Rubin, obok CoreWeave, Google Cloud, Azure i OCI. Na konferencji Kress zapowiedziała, że przy alokacji będzie „selektywna” wobec neocloudów. Liczą się teren, zasilanie, gotowy budynek, kapitał i offtake, a pierwszeństwo mają klienci z ratingiem inwestycyjnym i przedpłatami. W komentarzu pisemnym zaznaczyła, że neocloudy i laboratoria frontier mogą rosnąć w niezrównoważonym tempie. 17.11, przed raportem, kurs Nebius odbił o ok. 3,4% po spadku o ok. 25% w dwóch sesjach po wynikach z 12.11.
**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:* Kryterium „land, power, shell” uderza w operatorów z opóźnieniami budowy. Nebius sam przyznał opóźnienia u zewnętrznego dewelopera w Vineland (NJ). Preferencja dla przedpłat i klientów IG oznacza, że alokacja Rubin dla Nebius zależy od kontraktów z hyperscalerami, a nie od bilansu Nebius.
- *Ocena ryzyka:* **Średnie** (bez zmian). Wczesny dostęp do Rubin to przewaga produktowa na 2027, ale model nadal opiera się na terminowości zewnętrznych DC i dostępie do kapitału, który Nvidia sama nazwała warunkiem swojej prognozy.
- *Linki:* [NVIDIA – wyniki Q3 FY2027](https://nvidianews.nvidia.com/news/nvidia-announces-financial-results-for-third-quarter-fiscal-2027), [Filing Insights](https://filinginsights.substack.com/p/nvidias-q3-fy2026-call-record-results), [StreetInsider](https://www.streetinsider.com/Investing/Nebius+stock+rises+as+neocloud+sector+recovers+ahead+of+Nvidia/27361343.html)

### Europejscy operatorzy z chińskimi klientami (nienazwani)

#### Aktualizacja: reguła BIS wydana – licencja na zdalny dostęp powyżej 12 mln TPP, 90 dni dla istniejących klientów (15.11)
**Fakty:** Nowy element względem zapowiedzi z 15.11: wydano tymczasową regułę końcową „Addressing Remote Access to Advanced Semiconductors” (RIN 0694-AK50, nowy § 744.24 EAR). Licencja jest wymagana przy udostępnianiu podmiotom z siedzibą w Makau lub krajach grupy D:5 (w tym Chinach) łącznie ponad 12 mln TPP mocy obliczeniowej. Powiązane DC sumuje się. Kontrola dotyczy operatorów w każdym kraju, nie tylko w pięciu wymienionych. Operatorzy, którzy już obsługują takich klientów, mają 90 dni od wejścia w życie na uzyskanie licencji. DC, foundry i eksporterzy muszą weryfikować własność i kontrolę klientów (KYC). Reguła wchodzi w życie z dniem publikacji w Federal Register, komentarze można składać przez 60 dni.
**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:* Próg 12 mln TPP odpowiada rzędowi kilkuset GPU klasy Blackwell (to szacunek, nie wartość z reguły), więc łapie każdy znaczący kontrakt treningowy. Europejscy operatorzy, którzy sprzedają moc chińskim firmom (SemiAnalysis, 12.11), mają 90 dni na licencję z oceną case-by-case, czyli w praktyce na wygaszenie kontraktów. Wymóg KYC na poziomie beneficjenta końcowego oznacza też koszty compliance i ryzyko naruszenia covenantów kredytów pod GPU.
- *Ocena ryzyka:* **Wysokie** dla mniejszych operatorów z istotnym udziałem chińskich klientów (utrata przychodu w ciągu kwartału, zwolnione klastry na rynek spot). **Niskie** dla Nebius i Nscale, których backlog opiera się na hyperscalerach z USA.
- *Linki:* [ai-tldr.dev (streszczenie reguły, odnośniki do bis.gov)](https://ai-tldr.dev/releases/us-commerce-remote-access-ai-chips-rule/), [The Next Web](https://thenextweb.com/news/us-remote-access-ai-chips-rule-china-singapore-malaysia), [SCMP](https://www.scmp.com/tech/article/3374639/us-tightens-ai-chip-export-curbs-close-chinas-overseas-data-centre-loophole)

## AI for networking

#### Lambda: pierwszy produkcyjny klaster z przełącznikami CPO (Quantum-X Photonics) dla ponad 10 tys. GPU (SC26, 16–17.11)
Lambda i NVIDIA pokazały na SC26 klaster produkcyjny z ponad 10 000 GPU Blackwell Ultra, spięty przełącznikami InfiniBand Quantum-X Photonics (Q3450-LD) z co-packaged optics. Przełącznik ma 144 porty MPO po 800 Gb/s i zamiast 144 modułów pluggable korzysta z 18 wymiennych zewnętrznych źródeł laserowych na froncie obudowy. Według testów Lambdy przełącznik CPO pobiera 3,95 kW wobec 7,0 kW dla wersji z optyką pluggable. SDxCentral relacjonuje deklarację 15× wyższej niezawodności, ale publicznie nie ma danych o link flapach ani MTBF. Spectrum-X Photonics (Ethernet) ma być drugą generacją.
**Analiza:**
- **Energia:** ok. 3 kW mniej na przełącznik przy setkach przełączników w fabricu 10k GPU daje setki kW przeniesione z sieci na compute w tym samym budżecie mocy DC.
- **Niezawodność:** Lasery to główne źródło awarii optyki. Ograniczenie ich liczby z 144 do 18 na przełącznik i możliwość wymiany zmniejszają liczbę zdarzeń przerywających joby, co przekłada się na goodput treningu.
- **Ryzyka:** Na razie tylko InfiniBand (lock-in na stos NVIDIA), serwis torów optycznych przy samym chipie jest ograniczony, a awaria w obrębie pakietu oznacza wymianę całego przełącznika. Ethernet CPO od NVIDIA i Broadcom (TH6-Davisson) dopiero wchodzi. Przy przetargach 2027 wymagać od dostawców danych o rzeczywistej awaryjności (MTBF, link flaps na 1000 łączy) zamiast deklaracji względnych.
Źródło: [ServeTheHome](https://www.servethehome.com/?p=107267), [HPCwire](https://www.hpcwire.com/2026/11/17/lambda-touts-first-cpo-deployment-of-blackwell-gpus/), [SDxCentral](https://www.sdxcentral.com/news/lambda-touts-15x-higher-reliability-using-nvidias-cpo-switches-for-10000-blackwell-ultra-gpus)

<hr>

#### Elastic: pięć agentowych frameworków ofensywnych (ARTEX, CyberStrikeAI, Cairn, PentAGI, HexStrike) – detekcja po tempie i kolejności akcji (16–17.11)
Elastic przeanalizował frameworki, w których LLM steruje skanerami, powłokami i CLI chmurowymi w seriach trwających 2–25 sekund. Żaden nie egzekwuje autoryzacji w kodzie. ARTEX i CyberStrikeAI po zablokowaniu zmieniają proxy i User-Agent. GTIG i Mandiant widziały cztery z nich w realnych włamaniach, a Microsoft łączy ARTEX ze Storm-0940. Elastic rekomenduje: z góry ustalić, które detekcje uruchamiają automatyczne odcięcie, łączyć blokady IP z odwołaniem tokenów i sesji oraz śledzić tempo i sekwencję akcji danej tożsamości między sesjami zamiast fingerprintów narzędzi.
**Analiza:** Nowy element względem raportu Unit 42 (14.11): rotacja proxy i UA sprawia, że kontrole oparte na IP lub reputacji (WAF, SASE z listami blokad) nie wystarczają. Sygnałem staje się behawioralna telemetria tożsamości: odstępy między wywołaniami API, rytm burst/pauza wynikający z czekania na model. Wymaga to korelacji logów IdP, CloudTrail/Activity Log i ruchu sieciowego w czasie bliskim rzeczywistemu. Opóźnienie SIEM rzędu minut jest za długie, skoro cykl ataku trwa sekundy. Automatyczne odcięcie niesie ryzyko fałszywych alarmów wobec legalnych agentów automatyzacji, więc te trzeba wyraźnie oznaczyć (osobne tożsamości, workload identity).
Źródło: [Elastic Security Labs](https://www.elastic.co/security-labs/threat-command/agentic-offensive-frameworks-detection-artex)

## Ryzyka rynku AI/GPU

#### Nvidia: zobowiązania zakupowe +111 mld USD w kwartale, należności 81 mld, wzrost FY2028 uzależniony od „dostępu klientów do kapitału” (17.11)
Według relacji z konferencji zobowiązania dostawcze Nvidii wzrosły o ok. 111 mld USD w jednym kwartale, czyli mniej więcej o równowartość kwartalnego przychodu. Zapasy wzrosły do 39,7 mld (z 21,4 mld w styczniu), należności do 81,0 mld (z 38,5 mld), dług długoterminowy do 32,4 mld (z 7,5 mld). Linię gwarancji nadal zdominowany backstop kredytowy 105 mld USD dla kampusu SB Energy pod 20-letni najem OpenAI (fazy od 2028). Kress: gwarancje uruchamiane są dopiero po sfinansowaniu projektu i mogą wygasnąć przy poślizgu. W komentarzu pisemnym zaznaczono, że neocloudy i laboratoria frontier mogą rosnąć w niezrównoważonym tempie. Prognozę ok. 75% wzrostu w FY2028 uzależniono od trzech warunków: gotowości DC, trwałości popytu końcowego i dostępu klientów do kapitału. Pełne dane o gwarancjach będą w 10-Q.
**Analiza:**
- **Kapitał obrotowy:** Należności rosną szybciej niż przychód (2,1× od stycznia przy ok. 1,5× wyższym przychodzie kwartalnym), więc Nvidia coraz bardziej kredytuje klientów przez dłuższe terminy płatności.
- **Ryzyko popytu:** +111 mld zobowiązań zakupowych przy marży spadającej z powodu kosztu pamięci oznacza, że Nvidia przejmuje ryzyko wolumenu i ceny HBM na 2027.
- **Mechanizm transmisji:** Wymienienie „dostępu do kapitału” jako warunku prognozy to pierwsze oficjalne powiązanie wyników Nvidii z rynkiem kredytu dla neocloudów, gdzie obligacje CoreWeave dają ponad 16%. Pogorszenie warunków kredytowych przełoży się na przychód Nvidii, a nie tylko na kursy neocloudów.
- **Do sprawdzenia w 10-Q:** saldo gwarancji wobec 530 mld USD z lipca.
Źródło: [Filing Insights](https://filinginsights.substack.com/p/nvidias-q3-fy2026-call-record-results), [NVIDIA – wyniki Q3 FY2027](https://nvidianews.nvidia.com/news/nvidia-announces-financial-results-for-third-quarter-fiscal-2027), [CNBC – rentowność obligacji CoreWeave](https://www.cnbc.com/2026/11/06/coreweave-bond-yield-tops-16percent-as-ai-credit-stress-rises.html)

<hr>

#### Reguła BIS: licencje na eksport zaawansowanych chipów do Indonezji, Malezji, Singapuru, Tajlandii i Brazylii; uchylenie AI Diffusion Rule (15–16.11)
Nowe elementy względem zapowiedzi z 15.11: lista objęta wymogiem licencji eksportowej to pięć krajów (Indonezja, Malezja, Singapur, Tajlandia, Brazylia), a Secretary of Commerce może dodawać kolejne. Wyłączone są dostawy do firm z siedzibą w USA (lub z amerykańską spółką-matką) pod warunkiem comiesięcznego raportowania (§ 743.10). Reguła formalnie uchyla AI Diffusion Rule. Nvidia w wytycznych na Q4 zakłada zero chińskiego przychodu DC compute i wstrzymała licencjonowane dostawy Hoppera.
**Analiza:** Popyt na GPU w ASEAN przesuwa się od lokalnych i chińskich operatorów do hyperscalerów z USA, którym wystarczy raportowanie. Projekty takie jak Project Mercury (Malezja, offtake BytePlus, zob. 17.11) tracą bankowalność, bo eksport sprzętu wymaga licencji case-by-case. Ryzyko dla rynku: część mocy budowanej w Johor i Batam pod chińskich najemców zostanie bez sprzętu albo bez najemcy, co uderzy w lokalnych deweloperów i finansujące ich banki. W krótkim terminie zwiększy się podaż GPU dla pozostałych rynków, bo alokacja zostanie przekierowana.
Źródło: [The Next Web](https://thenextweb.com/news/us-remote-access-ai-chips-rule-china-singapore-malaysia), [Ars Technica](https://arstechnica.com/tech-policy/2026/11/us-moves-to-cut-chinas-remote-access-to-ai-chips/), [W.Media](https://w.media/us-moves-to-stop-chip-diversion-through-southeast-asia-and-brazil/)

## Newsletters summary

#### Hugging Face kieruje żądania Inference Providers przez OpenRouter (TLDR AI)
Inference Providers w Hugging Face routuje teraz żądania do modeli językowych przez OpenRouter, który dla każdego wywołania wybiera dostawcę. Jeden token HF obsługuje każdy model open-weight serwowany przez dowolnego dostawcę.
**Analiza:** Warstwa routingu oddziela aplikację od konkretnego dostawcy inferencji i ułatwia failover oraz arbitraż cenowy. Wybór dostawcy per wywołanie oznacza jednak niedeterministyczne różnice (kwantyzacja, wersja runtime, długość kontekstu) i trudniejszy audyt rezydencji danych. W kontekście przejęcia HF przez Nvidię (zamknięcie w 1H 2027) warto obserwować, czy routing pozostanie neutralny sprzętowo. Dla zastosowań regulowanych trzeba przypinać dostawcę i region.
Źródło: [Hugging Face Blog](https://huggingface.co/blog/openrouter)

<hr>

#### Google ZERO-TIM: bitowo identyczne logprobs samplera i trenera w RL na TPU (TLDR AI)
Implementacja zweryfikowana na 64 chipach TPU v5p eliminuje rozbieżność numeryczną między ścieżką serwującą (sampler) a treningową. Wymusza te same operacje arytmetyczne i ustandaryzowaną kolejność redukcji na całym meshu TPU, więc RL jest rzeczywiście on-policy.
**Analiza:** Rozjazd train–inference (inne kernele, kolejność sumowania w FP) to znane źródło niestabilności RL w skali, zwykle łatane poprawkami importance sampling. Deterministyczne redukcje kosztują przepustowość (ograniczona swoboda w wyborze kolejności kolektywów), ale usuwają całą klasę trudnych do debugowania błędów. Ten sam problem dotyczy GPU, gdzie vLLM/SGLang i Megatron/FSDP używają innych kerneli.
Źródło: [Google Developers Blog](https://developers.googleblog.com/unlocking-stable-on-policy-rl-on-tpu-with-bitwise-zero-traininference-mismatch)

<hr>

#### Nemotron for Science i cuPotential: małe modele domenowe i prymitywy MD (Own news / NVIDIA)
NVIDIA udostępniła otwartą platformę post-treningu Nemotron dla nauki: dane, narzędzia i wariant na DGX Spark. Małe modele domenowe dorównują dużym modelom ogólnym w zadaniach chemicznych i biologicznych przy niższym koszcie. Wdrożenia: Argonne, RIKEN, KTH, Lila Sciences, Terray. cuPotential dostarcza prymitywy do potencjałów w dynamice molekularnej, oddzielając fizykę od optymalizacji kerneli pod kolejne generacje GPU.
**Analiza:** Model „mały model domenowy + post-training na danych laboratorium” przesuwa zapotrzebowanie z dużych klastrów inferencyjnych do lokalnych stacji (DGX Spark/Station). Upraszcza to też suwerenność danych. cuPotential to kolejna warstwa biblioteczna CUDA-X, która utrudnia przenoszenie kodu MD na ROCm.
Źródło: [HPCwire](https://www.hpcwire.com/2026/11/17/nvidias-nemotron-for-science-challenges-larger-ai-models-on-cost-and-accuracy/), [NVIDIA Technical Blog – cuPotential](https://developer.nvidia.com/blog/accelerate-molecular-dynamics-simulations-with-accurate-flexible-primitives-using-nvidia-cupotential/)

<hr>

#### Gartner: do 2028 słaby nadzór nad AI spowoduje krytyczne przerwy u 40% organizacji I&O; tylko 15% ma automatyczne circuit breakery dla agentów (TLDR IT)
W ankiecie Gartnera 95% respondentów zgłosiło w ostatnim roku incydent związany z agentem, ale tylko 15% ma automatyczne wyłączniki zatrzymujące agenta, który wymknął się spod kontroli.
**Analiza:** Rozjazd 95% vs 15% pokazuje, że agenty trafiają do produkcji bez odpowiednika rate limitów i kill switchy znanych z SRE. Minimalny zestaw kontroli: budżet akcji i kosztu na sesję, limity zakresu uprawnień (osobne tożsamości), kategoryzacja narzędzi według odwracalności (wzorzec Saga: reversible/compensable/delayable/irreversible) i undo realizowane po stronie systemu, a nie agenta.
Źródło: [Gartner](https://www.gartner.com/en/newsroom/press-releases/2026-11-16-gartner-weak-artificial-intelligence-governance-will-lead-to-business-critical-service-disruptions-for-40-percent-of-infrastructure-and-operations-organizations-by-2028), [Quaspar – Agents need an undo button](https://quaspar.com/resources/agents-need-an-undo-button)

<hr>

#### Microsoft: wielowarstwowa obrona przed pośrednim prompt injection jako baseline (TLDR IT)
Wytyczne Zero Trust/SFI łączą mechanizmy probabilistyczne (Prompt Shields, spotlighting) z deterministycznymi (wykrywanie dryfu planu, kontrola przepływu informacji).
**Analiza:** Ważny element to information flow control: oznaczanie danych z niezaufanych źródeł i blokowanie ich przepływu do narzędzi o wysokim wpływie, niezależnie od tego, co „zdecyduje” model. Sam klasyfikator nie wystarcza. Warto zestawić to z Bashkit (sandbox bash w Ruście dla agentów z `analyze()` przed wykonaniem i allowlistą domen) jako warstwą wykonawczą.
Źródło: [Microsoft Learn](https://learn.microsoft.com/en-us/security/zero-trust/sfi/defend-indirect-prompt-injection), [GitHub – Bashkit](https://github.com/everruns/bashkit)

<hr>

#### Cloudflare otwiera Kitesurf: bezstanowa przeglądarka dla agentów na Workers (TLDR IT)
Kitesurf (Apache 2.0) działa w całości na Cloudflare Workers, obsługuje Chrome DevTools Protocol i można go uruchomić lokalnie albo na własnym koncie Cloudflare.
**Analiza:** Bezstanowy browser na platformie serverless obniża koszt skalowania agentów przeglądających web (brak puli długo żyjących instancji Chromium). Ograniczenia: brak trwałych sesji i ciasteczek wymaga zewnętrznego magazynu stanu, a egress z IP Cloudflare może być traktowany przez serwisy docelowe jako ruch botów.
Źródło: [Cloudflare Blog](https://blog.cloudflare.com/kitesurf-open-source)

<hr>

#### Perplexity Frontier Research: otwarte badania na puli 10 000 GPU (TLDR AI)
Perplexity uruchamia program badawczy z publikacją wag, kodu i ewaluacji. Nabór trwa do 5.12, a program ma pulę 10 000 GPU.
**Analiza:** Kolejny gracz aplikacyjny inwestuje we własny trening. To dodatkowy popyt na moc w 2027, ale też sygnał, że zakontraktowana moc nie jest w pełni wykorzystana przez produkt, a program badawczy może służyć jako sposób jej zagospodarowania (to wniosek, nie fakt ze źródła).
Źródło: [Perplexity Research](https://research.perplexity.ai/)

## Implikacje praktyczne

1. **Budżet GPU na 2027 z narzutem na pamięć.** Nvidia zapowiada nowe ceny od FY2028, a marża Q4 spada przez koszt HBM/DRAM. Zakładać wzrost ceny jednostkowej Rubin względem obecnych ofert i negocjować ceny z indeksacją do kosztów pamięci zamiast stałej ceny na 3 lata. Rozważyć model z mniejszą pamięcią na token (sparse attention, kwantyzacja KV) jako sposób na kontrolę TCO.
2. **Ryzyko kontrahenta neocloud jako pozycja w rejestrze ryzyk.** Nvidia uzależnia swoją prognozę od dostępu klientów do kapitału i deklaruje selektywność wobec neocloudów. Kontrakty na moc u neocloudów (szczególnie bez offtake od hyperscalera) wymagają klauzul step-in, escrow danych i checkpointów, prawa do przeniesienia obciążenia oraz SLA z karami pokrywającymi koszt migracji.
3. **Compliance eksportowy w umowach na moc obliczeniową.** Reguła BIS (próg 12 mln TPP, KYC beneficjenta) obejmuje operatorów na całym świecie. Każdy, kto odsprzedaje moc GPU lub ją współdzieli, potrzebuje procesu KYC, klauzul end-use i prawa do natychmiastowego wypowiedzenia. W horyzoncie 3–5 lat należy oczekiwać rozszerzania listy krajów i progów.
4. **Segmentacja i krótkotrwałe poświadczenia w HPC/AI.** Przypadki RIKEN i Infoblox pokazują, że dashboardy, backupy konfiguracji i katalogi domowe to magazyny poświadczeń. Do wdrożenia: CA SSH z krótkim TTL, skanowanie sekretów w `$HOME` i na share'ach, klasyfikacja danych supportowych jak konfiguracji produkcyjnej, dashboardy w strefie DMZ zasilanej jednostronnie.
5. **Optyka i sieć jako element budżetu mocy.** CPO pozwala oszczędzić ok. 3 kW na przełącznik. W projektach DC na 2027–2028 planować fabric z CPO (InfiniBand teraz, Ethernet od 2027), ale wymagać twardych danych o awaryjności i procedur serwisowych, bo awaria w pakiecie oznacza wymianę całego przełącznika.

## Trend tygodnia

Wyniki Nvidii pokazały rozjazd między przychodem a jakością bilansu: rekordowe 115,8 mld USD przychodu współistnieją z podwojonymi należnościami, zapasami prawie dwukrotnie większymi niż w styczniu i zobowiązaniami zakupowymi rosnącymi o 111 mld w kwartale. Koszt pamięci po raz pierwszy wyraźnie obniża marżę lidera, więc presja cenowa przesuwa się w dół łańcucha, do operatorów i użytkowników końcowych. Nvidia otwarcie wiąże swój wzrost z dostępem klientów do kapitału, a to łączy wyniki producenta chipów z rynkiem kredytu dla neocloudów, gdzie stres jest już widoczny w rentownościach. Równolegle państwo zaostrza kontrolę: reguła BIS o zdalnym dostępie, nowy komitet eksportowy w radzie Nvidii i zerowy chiński przychód w wytycznych przenoszą ryzyko geopolityczne z poziomu firm na poziom kontraktów i KYC. Po stronie technicznej optymalizacja przesuwa się z surowej mocy obliczeniowej na efektywność systemową: CPO zmniejsza pobór sieci, sparse attention przenosi wąskie gardło na selekcję tokenów, a deterministyczne ścieżki numeryczne stabilizują RL. Incydenty RIKEN i Infoblox przypominają, że w infrastrukturze AI/HPC najsłabszym ogniwem są wciąż poświadczenia i dostęp podwykonawców, a nie sam sprzęt.

## To obserwować

- **10-Q Nvidii za Q3 FY2027:** saldo gwarancji i backstopów (wobec 530 mld USD w lipcu), struktura zobowiązań zakupowych oraz należności per klient (koncentracja).
- **Publikacja reguły BIS w Federal Register:** data wejścia w życie, początek biegu 90 dni dla istniejących klientów i pierwsze deklaracje europejskich i azjatyckich operatorów o wygaszaniu kontraktów.
- **Wycena i debiut Anthropic (ANT, 18–19.11):** cena względem przedziału 70–78 USD jako test apetytu rynku na AI po wynikach Nvidii.
- **Rentowność obligacji CoreWeave i kursy Nebius/IREN po wynikach Nvidii:** czy „selektywność” Nvidii przełoży się na wyższy koszt kapitału operatorów drugiej linii.
- **Fugaku – termin przywrócenia i raport końcowy RIKEN:** wektor wejścia przez dashboard i ewentualne dalsze nadużycia skradzionych kluczy SSH w federacji HPCI.
