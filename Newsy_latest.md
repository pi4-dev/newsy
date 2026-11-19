# Raport Zbiorczy IT – 2026-11-19

Okres: od edycji 2026-11-18 do 2026-11-19 (Europe/Warsaw). Najważniejsze nowe zdarzenia to wycena IPO Anthropic (18.11 wieczorem), Java4Shell w OpenJDK i kolejne sygnały zaostrzenia warunków kredytu pod DC. Pominięte, bo opisane już wcześniej: wyniki Nvidii Q3 FY2027, wyjaśnienia dotyczące backstopów Nvidii, sparse attention NVIDIA, DDN EXAScaler software-only, AIGF Northern (skład konsorcjum), RIKEN/Fugaku, Infoblox.

## Raport technologiczny

### Technologia

#### Java4Shell (CVE-2026-114938, CVSS 4.0 9,2): RCE w intrinsicu Base64 OpenJDK na x86-64 z AVX-512 VBMI (ujawnienie 17.11)
Błąd jest w ręcznie pisanym kodzie asemblerowym HotSpot, który przyspiesza `Base64.Decoder` instrukcjami AVX-512 VBMI. Spreparowane wejście (w PoC wystarczy 64 znaki „A”) przepełnia bufor wyjściowy, co prowadzi do korupcji pamięci i zdalnego wykonania kodu bez uwierzytelnienia. Podatne są OpenJDK 17.0.6+ (kod backportowany w 2022, wydany w styczniu 2023) oraz nowsze linie, ale tylko na CPU z AVX-512 VBMI (Ice Lake i nowsze Xeony, AMD Zen 4/5). ARM64 nie jest podatne. Calif wymienia jako podatne m.in. Confluence, Jira, Artifactory, Keycloak i Elasticsearch. Oracle wydał buildy 17.0.21.1, 21.0.13.1, 25.0.5.1 i 27.0.1.1. Status poprawek w dystrybucjach OpenJDK różni się między źródłami. Obejście: `-XX:-UseBASE64Intrinsics` albo `-XX:UseAVX=2` (to drugie tylko na x86-64, bo na ARM JVM nie wystartuje).
**Znaczenie:** Podatność zależy od mikroarchitektury hosta, więc ten sam obraz kontenera jest podatny na jednym węźle i bezpieczny na innym. Skanery SCA oparte na wersji JDK nie pokażą realnej ekspozycji. Trzeba zestawić inwentarz JVM z flagami CPU (`/proc/cpuinfo`: `avx512vbmi`) i typami instancji w chmurze. Flaga `-XX:-UseBASE64Intrinsics` to najtańsza mitygacja do wdrożenia przez `JAVA_TOOL_OPTIONS` w całej flocie. Koszt to wolniejsze dekodowanie Base64 (JWT, SAML, nagłówki Basic auth), istotne tylko przy bardzo dużym QPS. Keycloak i bramy API dekodują Base64 z niezaufanego wejścia przed uwierzytelnieniem, więc to one mają najwyższy priorytet.
Źródło: [Calif – Java4Shell](https://calif.io/java4shell/), [Wiz](https://www.wiz.io/blog/cve-2026-114938-rce-java4shell-openjdk), [Qualys](https://blog.qualys.com/vulnerabilities-threat-research/2026/11/17/how-to-find-mitigate-and-patch-the-critical-java4shell-flaw-cve-2026-114938-now)

<hr>

#### Optimizely „OPTIMJACK”: złośliwy JavaScript z CDN Optimizely na stronach co najmniej 178 organizacji (16–17.11)
Atakujący wykorzystali błąd w procesie rejestracji Optimizely Web Experimentation i funkcję, która pozwala klientom dodawać własny JS do stron. Kod był serwowany z CDN Optimizely obok legalnych skryptów. Scott Helme (Report URI) znalazł złośliwy kod w 188 z 1 611 sprawdzonych skryptów, m.in. na stronach BBC, CNN, Zoom, IKEA, Salesforce, HSBC i TalkTalk, a także na stronie i portalu logowania samego Optimizely. Payloady: podmiana adresów portfeli kryptowalut, później skimmer kart z formularzy oraz próby wdrożenia exploit kitów Coruna i DarkSword na starsze iPhone'y. Optimizely załatał błąd, usunął wstrzyknięty kod i unieważnił tokeny.
**Znaczenie:** Skrypty A/B testów i tag managerów mają pełny dostęp do DOM, a CSP zwykle dopuszcza je wildcardem po domenie dostawcy. W takim układzie CSP nie chroni przed przejęciem dostawcy. Pola płatności w iframe procesora okazały się bezpieczne, co potwierdza sens izolacji danych kart w iframe/hosted fields. Dla stron z płatnościami (PCI DSS 4.0, wymagania 6.4.3 i 11.6.1) potrzebny jest inwentarz i monitoring integralności skryptów zewnętrznych w runtime. SRI nie pomoże, bo skrypty eksperymentów są dynamiczne.
Źródło: [The Register](https://www.theregister.com/cyber-crime/2026/11/17/optimizely-hack-turns-worlds-biggest-websites-into-malware-dispensers/5310448)

<hr>

#### Puppet Enterprise CVE-2026-115731 (CVSS 4.0 9,4) w CISA KEV (17.11)
CISA dodała podatność do KEV na podstawie potwierdzonej exploitacji. Termin dla agencji federalnych to 1.12. Klient z certyfikatem podpisanym przez Puppet CA może wykonać kod na serwerze (primary). CISA nie podała aktora ani skali.
**Znaczenie:** Serwer Puppet ma z definicji uprawnienia do zmiany konfiguracji całej floty, więc RCE na primary oznacza przejęcie każdego zarządzanego węzła. Warunek wstępny (certyfikat agenta) jest łatwy do spełnienia po przejęciu dowolnego zarządzanego hosta albo przy włączonym autosign. Do sprawdzenia: polityka `autosign`, segmentacja portu 8140 i rotacja CA po incydencie. Ten sam wzorzec dotyczy Salta, Chefa i Ansible AWX/Tower: płaszczyzna zarządzania konfiguracją to cel o najwyższym zasięgu.
Źródło: [CISA – alert 17.11](https://www.cisa.gov/news-events/alerts/2026/11/17/cisa-adds-one-known-exploited-vulnerability-catalog), [Cybersecurity News](https://cybersecuritynews.com/puppet-enterprise-flaw-actively-exploited/)

<hr>

#### PHP: wydanie bezpieczeństwa „Critical” dziś (19.11, ok. 15:00 CET) dla 8.2–8.5, m.in. CVE-2026-115560 w `session.upload_progress`
Zespół PHP zapowiedział poprawki dla wszystkich wspieranych gałęzi tego samego dnia co PHP 8.6.0. Opisana publicznie CVE-2026-115560 wymaga niedomyślnej konfiguracji: `session.serialize_handler` ustawiony na `php_binary` lub `php_serialize`, przełączany w runtime przez `ini_set()` na `php` przed `session_start()`, oraz włączone `session.upload_progress.enabled`. Mitygacja do czasu wydania: `session.upload_progress.enabled=0` i restart workerów. To samo ustawienie łagodzi CVE-2026-109619 z października (PHP2Shell).
**Znaczenie:** GTIG raportuje, że po październikowych łatkach PHP w sieci pojawiły się setki PoC, a frameworki agentowe robią patch-diffing w ciągu godzin (zob. niżej). Okno między publikacją a exploitacją trzeba więc liczyć w godzinach. Wyłączenie upload progress to bezpieczna zmiana dla większości aplikacji, wartą wdrożyć przed oknem serwisowym.
Źródło: [BigGo Finance (streszczenie zapowiedzi PHP)](https://finance.biggo.com/news/0ded746a-3e62-40eb-a404-3545fc7f7f8b)

<hr>

#### GTIG: pierwszy zaobserwowany framework „exploit_agent” – patch-diffing i exploitacja bez człowieka w pętli (17.11)
Google Threat Intelligence Group znalazł odsłonięty serwer stagingowy z wieloagentowym frameworkiem podłączonym do modeli frontier i lokalnych. Framework porównuje kod aplikacji webowych przed i po ogłoszeniu łatki, lokalizuje błąd, pisze exploit i go uruchamia. Konfiguracja miała rosyjskie komentarze („exploit_agent pilot”), a cele to głównie francuskie serwisy medialne i edukacyjne. Inne dane z raportu: liczba zero-dayów w sierpniu 2026 wyniosła 22 (średnia 2025: 8/mies.), a LLMjacking (kradzież kwot LLM z przejętych chmur) powodował straty od 100 tys. USD dziennie do 1 mln USD w najgorszych przypadkach. KRLD modyfikował `.claude/settings.json`, żeby start Claude Code pobierał backdoora.
**Znaczenie:** Nowy element względem raportów Unit 42/Elastic (14–18.11): automatyzacja obejmuje już etap od łatki do exploita, nie tylko eksploatację znanych podatności. Praktycznie oznacza to, że SLA na łatanie systemów brzegowych trzeba liczyć w godzinach, a tam, gdzie to niemożliwe, potrzebne są wirtualne łatki w WAF/IPS wdrażane automatycznie. Klucze API do LLM w chmurze wymagają limitów wydatków i alertów na nagłe skoki zużycia, tak jak klucze chmurowe. Pliki konfiguracyjne narzędzi agentowych (`.claude/`, `.cursor/`, MCP) w repozytoriach trzeba objąć code review i kontrolą integralności.
Źródło: [Help Net Security (raport GTIG)](https://helpnetsecurity.com/2026/11/17/ai-powered-cyberattacks-gtig-report)

<hr>

#### Aktualizacja: seria awarii 12–17.11 – przyczyny w Microsoft, JPMorgan i Apple
Nowe elementy względem opisów awarii M365 (12–13.11) i FAA (13–14.11): Microsoft przypisał kolejny incydent problemowi z wcześniejszego upgrade'u pojemności sieci, który ujawnił się po późniejszej zmianie sieciowej. Według firmy nie był to atak ani skutek działania AI. JPMorgan Chase podał „błąd oprogramowania w systemach sprzętowych on-premise”. Apple (17.11) wskazał wewnętrzną aktualizację, która wyłączyła App Store, Apple Music i AppleCare. Dyrektor Wywiadu Narodowego odrzucił hipotezę skoordynowanego ataku.
**Znaczenie:** Wspólnym mianownikiem jest zmiana (upgrade, aktualizacja, wymiana routera), a nie atak. Według komentatorów presja na szybkie łatanie, napędzana przez odkrywanie podatności z pomocą AI, zwiększa liczbę zmian i ryzyko regresji. W przypadku Microsoftu błąd był ukryty od czasu wcześniejszej zmiany i ujawnił się dopiero po następnej, więc pojedynczy krok rolloutu nie wystarczył do jego wykrycia. Wnioski: walidacja stanu sieci po zmianie powinna porównywać stan z intencją (verification, digital twin), a nie tylko sprawdzać dostępność. Okna zmian u dostawców krytycznych SaaS i sieci warto śledzić jako ryzyko korelowane.
Źródło: [CNBC](https://www.cnbc.com/2026/11/17/apple-microsoft-faa-jpmorgan-outages.html), [ABC/AP](https://abc7news.com/story/apple-joins-parade-technology-hiccups-recent-days/20048749/), [Fox (przyczyny poszczególnych awarii)](https://www.fox29.com/news/airline-bank-outage-reason-tech-glitch-cyber-attack)

### Biznes

#### Nvidia sprzedała 56% udziałów w Nokii w Q3 (ujawnienie 16.11)
Według ujawnionego stanu posiadania Nvidia sprzedała w III kwartale 93,53 mln akcji Nokii. Na 30.09 zostało jej 72,86 mln akcji (738,8 mln USD), wobec 166,39 mln na koniec czerwca. Pakiet kupiła w październiku 2025 za 1 mld USD (6,01 USD/akcję, 2,9%). W tym samym kwartale Nvidia zmniejszyła udziały w Intelu o 25% i w Synopsys o prawie 10%.
**Znaczenie:** Inwestycja w Nokię była powiązana ze współpracą przy AI-RAN (Aerial na GPU w stacjach bazowych). Zmniejszenie pakietu nie oznacza zerwania partnerstwa, ale osłabia sygnał, że Nvidia wiąże się kapitałowo z RAN. Operatorzy planujący AI-RAN powinni oceniać roadmapę na podstawie kontraktów i wdrożeń, a nie udziałów kapitałowych. Nokia jest też dostawcą sieci dla AIGF Northern.
Źródło: [Yahoo Finance / Stocktwits](https://finance.yahoo.com/markets/stocks/articles/nok-slides-hours-nvidia-dumps-020414079.html)

## Raport AI-ML

### Biznes

#### Aktualizacja: Anthropic wycenił IPO na 83 USD, powyżej przedziału 70–78 USD – 81,3 mld USD, debiut na Nasdaq (ANT) dziś (18–19.11)
Nowe elementy: cena 83 USD za akcję, 980 mln akcji klasy A (ok. 956,4 mln od spółki, ok. 23,6 mln od dotychczasowych akcjonariuszy) wobec 825 mln w pierwotnych warunkach. Wpływy bazowe ok. 81,3 mld USD to największe IPO w historii. Opcja dla underwriterów: ok. 147 mln akcji (ok. 12,2 mld USD). Równolegle prywatna emisja 26 mld USD (m.in. Nvidia, Google, Microsoft). Kapitalizacja startowa według Axios ok. 1,08 bln USD (inne źródła podają do ok. 1,5 bln w pełni rozwodnionej). Notowania od 19.11, zamknięcie oferty 20.11. Uwaga: autor raportu jest modelem Anthropic, więc ten wpis ogranicza się do faktów z publicznych źródeł.
**Analiza:** Podniesienie ceny i liczby akcji przy rekordowej skali to sygnał, że rynek akcji nadal finansuje laboratoria frontier, mimo napięć w kredycie dla neocloudów i DC (zob. Ryzyka). Ok. 100 mld USD nowego kapitału (z prywatną emisją) przekłada się bezpośrednio na zdolność do zobowiązań compute (TPU, Trainium, GPU u Microsoftu, Nscale). Zmniejsza to ryzyko kontrahenta dla dostawców mocy. Dla klientów API oznacza to mniejsze ryzyko ciągłości dostawcy, ale też presję na monetyzację i marżę, co może się odbić na cenach i limitach. Kurs ANT będzie od dziś publicznym wskaźnikiem nastrojów wobec laboratoriów, obok CRWV i NBIS.
Źródło: [CNBC](https://www.cnbc.com/2026/11/18/anthropic-ipo-ai-offering-price.html), [Axios](https://www.axios.com/2026/11/18/anthropic-ipo-1-trillion), [ABC News](https://abcnews.com/Business/anthropic-announces-price-83-share-ipo/story?id=138245320)

### Technologia

#### Cerebras: dezagregowana inferencja w produkcji – prefill na GPU AMD Instinct, decode na CS-3, 5× tokenów/MW (pomiar 14.11, ogłoszenie 17–18.11)
Cerebras uruchomił w early access usługę, w której przetwarzanie promptu (prefill, ograniczony obliczeniowo) robią GPU AMD Instinct, a generację tokenów (decode, ograniczony przepustowością pamięci) systemy waferowe Cerebras. Według pomiaru firmy na współdzielonym endpoincie GPT-OSS-120B przepustowość na megawat wzrosła 5× wobec konfiguracji tylko z CS-3, bez spadku szybkości generacji. Wcześniej AMD i Cerebras podawały to tylko jako projekcję. W partnerstwie z AWS rolę prefillu pełni Trainium.
**Analiza:**
- **Mechanizm:** SRAM na waferze jest drogi w przeliczeniu na FLOP przy długich promptach, za to idealny do decode. Przeniesienie prefillu na tańsze FLOP-y GPU zwalnia waferę do pracy, w której ma przewagę. To ten sam podział co w Dynamo/llm-d, tylko między różnymi architekturami.
- **Latency:** KV cache po prefillu trzeba przesłać z HBM GPU do SRAM Cerebras. Przy długim kontekście transfer dodaje się do TTFT, a firma nie podała danych o TTFT ani o długości kontekstu w teście.
- **Skalowalność i koszt:** 5× dotyczy jednego modelu i jednego endpointu. Zysk zależy od proporcji prefill/decode w ruchu. Ruch agentowy z długim kontekstem i krótkimi odpowiedziami zyskuje najwięcej.
- **Lock-in:** Heterogeniczny pipeline wiąże klienta z parą dostawców i ich transportem KV. Dla on-prem to raczej sygnał projektowy (prefill i decode na osobnych pulach sprzętu) niż gotowy produkt.
Źródło: [Cerebras – ogłoszenie](https://cerebras.ai/blog/cerebras-launches-disaggregated-inference), [Unite.ai](https://www.unite.ai/?p=522527), [eWeek (projekcja AMD–Cerebras)](https://www.eweek.com/de/news/amd-cerebras-disaggregated-inference/)

<hr>

#### AMD na SC26: 202 systemy w TOP500, Lux (ORNL) nr 14 z 220,79 PF na MI355X i sieci Ethernet z AMD AI NIC; Venice-X z 1 152 MB L3 (18.11)
AMD ma 202 systemy na liście TOP500 (ponad 40%, ok. +14% r/r) i połowę nowych wpisów. Lux w ORNL (HPE) debiutuje na 14. miejscu z 220,79 PF HPL: EPYC, Instinct MI355X, Pensando AI NIC i ROCm. AI NIC obsługuje trening AI i MPI na jednej sieci Ethernet. EPYC 6. generacji (Venice): do 1,6 TB/s przepustowości pamięci, PCIe 6, do 5,3 GHz. Venice-X będzie miał 1 152 MB L3 na gniazdo (SP7, jeszcze niedostępny). MI430X trafia do LUMI-AI i Alice Recoque. AMD przeanalizowało 47 konfiguracji w 11 aplikacjach pod kątem tego, gdzie emulacja FP64 wystarcza, a gdzie potrzebna jest natywna podwójna precyzja. Z IBM wydano Pelagos (symulacja dekoderów QEC): 6,23× przepustowości przy skalowaniu z 1 do 8 MI355X. Rancher Government Solutions dało architekturę referencyjną K8s z Slinky i Kueue na Instinct.
**Analiza:** Odpowiedź na emulację FP64 w cuBLAS (Ozaki-II, zob. 13–14.11): AMD stawia na natywne FP64 w MI430X i sam publikuje, gdzie emulacja się nie sprawdza. Dla centrów HPC wybór GPU zależy więc od profilu aplikacji (stabilność numeryczna, warunkowanie macierzy), a nie tylko od TFLOPS. Lux to pierwszy duży system DOE na Ethernet z AI NIC zamiast Slingshot/IB, czyli test dojrzałości RoCE/UEC dla MPI z małymi komunikatami. Venice-X z ponad 1 GB L3 jest istotny dla CFD i solverów z rzadkimi macierzami ograniczonych przepustowością pamięci. Wszystkie dane pochodzą z komunikatu AMD.
Źródło: [HPCwire (komunikat AMD)](https://www.hpcwire.com/off-the-wire/amd-expands-frontier-of-scientific-computing-at-sc26/), [Converge Digest – Lux](https://convergedigest.com/oak-ridge-lux-ai-supercomputer-800g-ethernet/)

## euro neocloud

### Program UE AI Gigafactories (dotyczy Nscale, Verda, Mistral i pozostałych konsorcjów)

#### Korekta: termin przetargu EuroHPC na AI Gigafactories – Lot 1 przesunięty na 4.02.2027, wszystkie złożone oferty do ponownego złożenia
**Fakty:** Korekta wpisu z 12–13.11 (w którym podano, że termin minął 12.11): według aktualizacji EuroHPC JU z 5.11 termin dla Lot 1 przesunięto na 4.02.2027, żeby państwa członkowskie mogły zakończyć procedury budżetowe. Termin dla Lot 2 pozostaje 3.12.2026. Wszystkie projekty i złożone oferty w obu lotach nie będą rozpatrywane i trzeba je złożyć ponownie przez nowe linki. Jako przyczynę EuroHPC podał „powody techniczne”. Treść dokumentacji przetargowej się nie zmienia. Mistral potwierdził zainteresowanie prowadzeniem zwycięskiego konsorcjum. AIGF Northern (Nokia, Telia, Nscale, Verda, Skeleton) zapowiedział ofertę 16.11, już po zmianie terminów.
**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:* Decyzje o wyborze Gigafactories przesuwają się realnie na 2027, a uruchomienie mocy na 2028+. Konsorcja, które liczyły na publiczne współfinansowanie (InvestAI) jako zabezpieczenie dla długu, mają dłuższy okres finansowania pomostowego. Uderza to najmocniej w Nscale (luka ok. 30 mld USD pod kontrakt Anthropic, zob. 14.11). Powód „techniczny” bez szczegółów i unieważnienie wszystkich ofert świadczą o niskiej dojrzałości procesu.
- *Ocena ryzyka:* **Średnie** dla programu (opóźnienie ok. 3 miesięcy, bez zmiany budżetu). **Wysokie** dla Nscale (bez zmian, dłuższy okres bez finansowania publicznego). **Niskie** dla Verda i Telii, dla których Gigafactory to dodatkowa opcja, a nie podstawa biznesu.
- *Linki:* [Sander Bosman AI (streszczenie aktualizacji, 10.11)](https://sanderbosmanai.substack.com/p/eurohpc-moves-the-ai-gigafactory), [EuroHPC JU – call for tenders](https://www.eurohpc-ju.europa.eu/call-tenders-selection-artificial-intelligence-gigafactory-consortia-and-establishment-ai_en), [Rapporteur – Mistral](https://www.rapporteur.com/news/mistral-in-the-running-to-back-one-of-eus-ai-gigafactories/)

## AI for networking

#### Arista Etherlink R4: routing pod rozproszoną inferencję – 7280R4 do 32×800GbE, 32 GB bufora, 800G ZR/ZR+ z MACsec (SC26, 17.11)
Arista rozszerza rodzinę R4 (5 nm) o platformy do łączenia rozproszonych lokalizacji inferencyjnych. 7280R4 ma do 32 portów 800GbE i do 32 GB bufora pakietów (VOQ), obsługuje koherentne 400G/800G ZR/ZR+ z MACsec. 7282R4 dodaje dwa supervisory dla redundancji płaszczyzny sterowania. 7020R4 to kompaktowy router dostępowy z szyfrowaniem sprzętowym, SR i PTP/SyncE/GNSS. Odmiana przemysłowa 7020HR4 (−40…+65°C) będzie dostępna na początku 2027, reszta już teraz. Do tego pluggable system optyczny: wzmacniacz full C-band 25,6 Tb/s, mux/demux i wpinany OTDR do testów w trakcie pracy. Arista deklaruje do 60% mniej energii niż platformy na 7/16 nm, bez wskazania modeli porównawczych.
**Analiza:**
- **Architektura:** Inferencja agentowa robi wiele sekwencyjnych wywołań (model → narzędzie → model), więc RTT do użytkownika i między lokalizacjami mnoży się przez liczbę kroków. To argument za rozproszonymi, mniejszymi lokalizacjami inferencyjnymi połączonymi DCI zamiast jednego kampusu.
- **Throughput i koszt:** 800G ZR bezpośrednio w routerze usuwa transpondery. Głębokie bufory amortyzują bursty transferów KV cache i synchronizacji wag między lokalizacjami, ale nie dają bezstratności w WAN.
- **Ryzyka:** Deklaracja energetyczna jest bez punktu odniesienia. Wpinany OTDR i system optyczny od jednego dostawcy upraszczają operacje, ale wiążą warstwę optyczną z Aristą. W projekcie trzeba osobno liczyć budżet strat i zasięg ZR+ dla każdej trasy.
Źródło: [Converge Digest](https://convergedigest.com/arista-etherlink-r4-distributed-ai-inference-800g/), [Network World](https://www.networkworld.com/article/4249783/arista-expands-its-routing-options-to-support-secure-ai-inferencing.html)

<hr>

#### Lux (ORNL): pierwsza sieć HPE Juniper w laboratorium DOE – QFX 800GbE z AMD Pensando AI NIC dla AI i MPI na wspólnym Ethernecie (SC26)
HPE ujawniło, że Lux (nr 14 TOP500) używa przełączników Juniper QFX 800GbE. To pierwsze wdrożenie sieci HPE Juniper w amerykańskim laboratorium DOE. Po stronie hostów pracują AMD Pensando AI NIC, które obsługują trening AI i symulacje MPI na tej samej sieci. HPE nie użyło tu własnego Slingshot.
**Analiza:** HPE wybrało Juniper/Ethernet z programowalnym NIC zamiast Slingshot we własnym systemie DOE. To sygnał, że przyszłość interconnectu HPC/AI w HPE to standardowy Ethernet (UEC) z logiką transportu w NIC. Kontrola przeciążeń i multipathing przechodzą do programowalnego NIC (P4 w Pensando), więc wydajność MPI z małymi komunikatami zależy od firmware'u NIC, a nie od przełącznika. Dla operatorów klastrów oznacza to możliwość budowy fabricu z przełączników wielu dostawców i lock-in przesunięty na NIC oraz jego stos sterowników. Brak opublikowanych danych o latencji MPI i skalowaniu all-reduce. Wymagać ich przed powieleniem architektury.
Źródło: [Converge Digest](https://convergedigest.com/oak-ridge-lux-ai-supercomputer-800g-ethernet/), [HPCwire (komunikat AMD)](https://www.hpcwire.com/off-the-wire/amd-expands-frontier-of-scientific-computing-at-sc26/)

## Ryzyka rynku AI/GPU

#### CoreWeave: 130 tys. GPU schodzi z kontraktów w 2027 – ten sam rok, w którym zapada ok. 11 mld USD długu; McBee woli capex niż odkup obligacji z rentownością ok. 15% (18.11)
Współzałożyciel Brannin McBee powiedział na konferencji w Austin, że inwestowanie w moc obliczeniową daje wyższy zwrot niż odkup własnego niezabezpieczonego długu (rentowność ok. 15%). Według niego ok. 130 tys. GPU zejdzie z kontraktów w 2027 i może zostać ponownie wynajętych. Pytanie zadał Gavin Baker (Atreides), sugerując przedpłatę za klaster na spłatę długu. Według sprawozdań CoreWeave ok. 10,97 mld USD kapitału długu przypada na 2027. Na konferencji wynikowej (10.11) spółka szacowała, że ponowny wynajem po obecnych cenach doda blisko 4 mld USD rocznego przychodu z odnowień 2027 i blisko 10 mld łącznie z 2028. 18.11 kurs spadł o 4,13% do 47,83 USD, mimo wyników Nvidii.
**Analiza:**
- **Refinansowanie:** Zbieżność zapadalności długu z wygaśnięciem kontraktów oznacza, że spłata w 2027 zależy od ceny odnowienia GPU starszych generacji (głównie H100/H200 i wczesne GB200), a nie od nowego backlogu.
- **Ekonomika:** Obecne indeksy (zob. niżej) pokazują ceny H100 ok. 2,9–3,5 USD/h. Jeśli w 2027, po rampie Rubin, spadną o 30–40%, szacunek 4 mld USD z odnowień skurczy się proporcjonalnie, a koszt kapitału pozostanie na poziomie ok. 15%.
- **Sygnał:** Wybór capexu zamiast delewarowania przy 15% koszcie długu to zakład na utrzymanie spreadu zwrot z GPU minus koszt kapitału. Przy spadku cen najmu ryzyko zamienia się w refinansowanie pod presją. Do obserwacji: warunki re-leasingu w umowach kredytowych (opcja odnowienia z kryteriami, których nie ujawniono publicznie).
Źródło: [Bloomberg](https://www.bloomberg.com/news/articles/2026-11-18/coreweave-s-mcbee-favors-data-center-spending-over-repaying-debt), [Startup Fortune](https://startupfortune.com/coreweaves-mcbee-says-gpus-beat-buying-back-its-15-yield-debt/), [FourWeekMBA – szacunek przychodu z odnowień](https://fourweekmba.com/ai-coreweave-says-expiring-gpus-could-add-nearly-10b-a-year/), [Motley Fool – sesja 18.11](https://www.fool.com/coverage/stock-market-today/2026/11/18/stock-market-today-nov-18-coreweave-slides-as-nvidia-s-blowout-earnings-fail-to-ease-debt-concerns/)

<hr>

#### HSBC sprzedał z dyskontem ok. 300 mln USD udziału w kredycie 18 mld USD na Project Jupiter (Nowy Meksyk) po klauzuli force majeure Oracle (17.11)
Według Bloomberga HSBC sprzedał swoją część kredytu za ok. 85–90 centów za dolara (ok. 45 mln USD poniżej wartości nominalnej). Transakcja nastąpiła wkrótce po tym, jak Oracle wysłał deweloperowi (spółka Blue Owl) zawiadomienie o sile wyższej. Według Bloomberga Oracle może odroczyć czynsz o 3 lata od rozpoczęcia płatności, jeśli strony uznają zdarzenie siły wyższej związane z zasilaniem. Władze stanu dwukrotnie odrzuciły pozwolenie na gazociąg, a zgoda na ogniwa paliwowe Bloom Energy nadal czeka. Oracle twierdzi, że projekt idzie zgodnie z planem. Blue Owl: zawiadomienie nie zmienia zobowiązań finansowych. CDS-y na dług DC są na rekordowych poziomach.
**Analiza:** To pierwszy publiczny przypadek, w którym najemca z ratingiem inwestycyjnym przerzuca ryzyko energetyczne na dewelopera i jego wierzycieli przez force majeure. Kredyt pod DC jest wyceniany jako kredyt najemcy (Oracle), ale klauzula o zasilaniu zamienia go w ryzyko projektu. Dyskonto 10–15% od wartości nominalnej przy wciąż obsługiwanym długu to repricing ryzyka zasilania. Konsekwencja dla rynku: banki będą wymagać potwierdzonego przyłącza lub uzgodnionego zasilania behind-the-meter przed finansowaniem, a to wydłuża harmonogramy i podnosi koszt kapitału projektów z gazem lub ogniwami paliwowymi.
Źródło: [The Star (Bloomberg)](https://www.thestar.com.my/business/business-news/2026/11/17/hsbc-sold-us300mil-troubled-data-centre-loan-at-discount), [Bisnow](https://www.bisnow.com/news/national/data-center-capital-markets/hsbc-sells-300m-of-data-center-debt-at-steep-discount), [Bloomberg Tax – force majeure Oracle](https://news.bloombergtax.com/business-and-practice/oracle-cites-force-majeure-to-shield-itself-on-big-data-center)

<hr>

#### Blackstone QTS zmniejsza emisję długu prywatnego z ok. 800 do ok. 300 mln USD – inwestorzy żądali wyższych rentowności (14–16.11)
Według Bloomberga QTS zmniejszył planowaną emisję zamiast płacić wyższe rentowności. W zamian 100 mln USD 7-letnich private placement po ok. +245 pb oraz 200 mln USD 5-letniego kredytu bankowego (do wyceny w najbliższych tygodniach). Pierwotne 800 mln miało refinansować dług bankowy, który teraz pozostaje. Wcześniej QTS bez problemu pozyskał m.in. 1,65 mld USD w obligacjach o ratingu inwestycyjnym.
**Analiza:** Napięcie przenosi się z długu neocloudów (high yield) na emitentów DC o ratingu inwestycyjnym. Blackstone wolał mniejszą emisję niż szerszy spread, który stałby się droższym punktem odniesienia dla jego pozostałego długu. Kolejni emitenci nie mają takiego wyboru. Przy kilku podobnych transakcjach finansowanie DC przesunie się do private credit i project finance po wyższym koszcie, co podniesie stawki colocation i najmu w kontraktach na 2027–2028.
Źródło: [investingLive (Bloomberg)](https://investinglive.com/stocks/weekend-blackstone-s-qts-cuts-ai-data-center-debt-sale-60-as-investors-push-back-on-pricing/), [FourWeekMBA](https://fourweekmba.com/ai-blackstones-qts-cuts-ai-debt-sale-to-300m-keeping-terms/)

<hr>

#### Banki USA wycofują się z kredytów pod GPU dla DC w Azji: Aolani Cloud odrzucony przez pięć banków (480 mln USD, GPU dla ByteDance w Malezji) (17–18.11)
Nowy element względem wstrzymanej 16.11 syndykacji 488 mln USD na DC w Malezji: Aolani Cloud (partner Nvidii z Singapuru) bezskutecznie zabiegał w pięciu bankach o 480 mln USD na zakup GPU Nvidii dla DC w Malezji, z których ByteDance miał korzystać zdalnie. Według źródeł prace nad kredytami na chipy dla azjatyckich DC praktycznie stanęły po regule Commerce o zdalnym dostępie. Bridge Data Centres (Bain Capital) wymienił najemcę Megaspeed na Zenlayer. Megaspeed jest podejrzewany o przekierowanie do chińskich firm chipów Nvidii za co najmniej 7,5 mld USD.
**Analiza:** Ryzyko eksportowe staje się ryzykiem kredytowym: banki nie finansują aktywów, których dopuszczalność użytkownika końcowego zależy od licencji case-by-case. Skutki to luka finansowa w Johor i Batam, zamrożone zamówienia GPU i przyspieszona wymiana najemców chińskich na amerykańskich. W krótkim terminie część wolumenu Nvidii przeznaczonego na ASEAN trafi na inne rynki, co może złagodzić dostępność GPU w Europie i USA w 1H 2027.
Źródło: [Seoul Economic Daily](https://en.sedaily.com/world/2026/11/18/us-banks-pull-back-from-asian-ai-data-center-lending), [Crypto Briefing (Bloomberg)](https://cryptobriefing.com/banks-pause-nvidia-chip-loan-bytedance)

<hr>

#### Kontrsygnał: ceny najmu GPU rosną – H100 ok. 2,9–3,5 USD/h, +6–8% m/m i t/t (połowa listopada)
Ornn OCPI (ceny transakcyjne) 18.11: H100 SXM 2,93 USD/GPU-h, H200 7,11, B200 9,21. Silicon Data: indeks H100 3,07 USD (17.11), +8,1% w 7 dni. GPU Rental Prices: średnia H100 on-demand w listopadzie 3,51 USD, +6,3% wobec października (3,30). B200 8,36, H200 5,06. Mediany list cenowych chmur: 4,14–4,94 USD/h za H100. Indeksy mają różne metodologie, a część stron jest wewnętrznie niespójna (Silicon Data B200: 6,97 vs 5,45).
**Analiza:** Wzrost cen spot przy rosnących problemach z kredytem dla budowy DC wskazuje, że ograniczeniem jest podaż (zasilanie, pamięć, opóźnienia budów), a nie popyt. Wspiera to krótkoterminowo tezę CoreWeave o ponownym wynajmie, ale ceny spot są zmienne i nie zabezpieczają długu 5-letniego. Odwrotna kolejność cen (H200 droższy od B200 w części indeksów) wskazuje na płytki, fragmentaryczny rynek, a nie stabilną krzywą cen. Do capacity planningu: kontrakty na 2027 negocjować teraz z klauzulami odnowienia po cenie indeksowej, a nie stałej.
Źródło: [Ornn OCPI](https://data.ornn.com/markets), [Silicon Data – H100](https://www.silicondata.com/products/silicon-index/h100), [GPU Rental Prices – listopad 2026](https://gpurentalprices.com/gpu-price-index/2026-11)

## Newsletters summary

#### Huang deklaruje gotowość powrotu, rada odmawia; zeznania przed Kongresem 1.12 (TLDR Hardware / TLDR)
Nowe elementy: Huang przyznał się do „poważnych błędów”, deklaruje współpracę z niezależnym dochodzeniem rady i gotowość powrotu, jeśli rada uzna to za korzystne. Prezydent Trump poparł ten pomysł. Przewodniczący rady Harvey Jones i źródło w radzie (Washington Post): brak planów przywrócenia. Według Semafor część dyrektorów naciska jednak na jego powrót. Huang zezna przed Kongresem 1.12.
**Analiza:** Podział w radzie przedłuża niepewność co do zarządzania Nvidią w czasie rampy Rubin i dochodzenia eksportowego. Zeznania 1.12 mogą wpłynąć na tempo licencji BIS dla ASEAN i Bliskiego Wschodu. Dla klientów: dopóki sprawa się nie wyjaśni, zakładać ostrożną politykę alokacji i finansowania (zob. backstopy).
Źródło: [TechCrunch](https://techcrunch.com/2026/11/17/jensen-huang-says-he-would-be-honored-if-allowed-to-return-as-nvidia-ceo), [Semafor](https://www.semafor.com/article/11/17/2026/some-nvidia-directors-push-to-bring-huang-back-despite-board-opposition)

<hr>

#### Marvell + Penguin: optyczny appliance pamięci – 16 TB DDR5 i 1 TB HBM współdzielone przez 16 serwerów (TLDR Hardware)
Moduł Marvell (z przejęcia Celestial AI za 3,5 mld USD) łączy 72 GB HBM3e i 8 kanałów DDR5 na interposerze fotonicznym z przepustowością 900 GB/s w obu kierunkach. Obudowa Penguin mieści 16 modułów za optycznym przełącznikiem 57,6 Tb/s i obsługuje do 16 serwerów przez NIC albo optykę CPO/NPO, z pominięciem CXL w trybach optycznych. Tryb cache-through w HBM ma dawać szybkość HBM przy pojemności DDR5, głównie dla offloadu KV cache. Systemy w 2H 2027.
**Analiza:** To konkretyzacja „Photonic Fabric” z zapowiedzi SC26: zewnętrzna pula pamięci jako kolejny poziom pod HBM GPU dla długiego kontekstu. Latencja dostępu (optyka + przełącznik) będzie rzędu mikrosekund, więc pasuje do prefetchu KV bloków, nie do losowego dostępu. Tryb poza CXL oznacza własny protokół i własny stos programowy (lock-in). Do porównania z offloadem KV na NVMe/GDS (LMCache, Dynamo KVBM).
Źródło: [The Register](https://www.theregister.com/systems/2026/11/16/marvell-and-penguin-solutions-aim-to-end-the-rampocalypse-with-photonic-memory-fabric/5309921)

<hr>

#### Tesla Powercore: transformator półprzewodnikowy (SiC) 7,4 MW / 34,5 kV z grid-forming (TLDR Hardware)
Jedno urządzenie zastępuje inwertery niskiego napięcia i olejowy transformator w farmach PV i magazynach energii: 7,4 MW (9 MVA) ciągle przy 34,5 kV, grid-forming do 10 MVA szczytowo. Produkcja w USA, uruchomienie w 2027. Konkurent Heron Link (5 MW, deklarowane 98,6% sprawności, klienci m.in. Vantage, Intersect). Tesla nie podała sprawności ani ceny.
**Analiza:** SST skraca łańcuch konwersji MV→DC, co pasuje do architektur 800 VDC w DC AI (zob. Mt. Diablo/OCP). Transformatory to jedno z głównych wąskich gardeł dostaw (terminy liczone w latach), więc SST mogą skrócić time-to-power dla mikrosieci DC (PV + magazyn). Niewiadome to sprawność, MTBF energoelektroniki SiC przy pracy ciągłej i certyfikacja u operatorów sieci.
Źródło: [Electrek](https://electrek.co/2026/11/17/tesla-powercore-solid-state-transformer-heron-link/)

<hr>

#### Google – Quaise: ponad 500 mln USD za 50 MW geotermii superkrytycznej w Oregonie (TLDR)
Quaise (Houston) chce wiercić na bardzo duże głębokości, odparowując skałę falami milimetrowymi, i produkować energię ze skał superkrytycznych. Umowa z Google o wartości ponad 500 mln USD obejmuje 50 MW dla istniejących DC Google w Oregonie.
**Analiza:** Cena ponad 10 mln USD/MW wskazuje na kontrakt pilotażowy (technologia na wczesnym etapie, ryzyko wiercenia). Dla hyperscalerów to dywersyfikacja w stronę stabilnego źródła bezemisyjnego (baseload), nie realne źródło mocy przed 2030. Ważniejszy sygnał: hyperscalerzy finansują ryzyko technologiczne wytwarzania energii, co tworzy przewagę dostępu do mocy, której neocloudy nie mają.
Źródło: [WSJ](https://www.wsj.com/business/energy-oil/google-embraces-geothermal-technology-to-vaporize-rock-for-clean-power-dbd66d05)

<hr>

#### WPPConnect: złośliwe wydania z poprawną proweniencją SLSA przez własny pipeline projektu (TLDR InfoSec)
Niezreviewowane commity wypchnięte z konta maintainera dodały moduł „diagnostics” do pakietów npm i obrazów Docker. Moduł wysyłał zmienne środowiskowe, pliki `.env` i tokeny sesji WhatsApp do bota Telegram. Wydania przeszły przez trusted publishing projektu z ważną proweniencją SLSA. Payload uruchamia się przy imporcie, więc `--ignore-scripts` nie chroni. Atakujący force-pushował gałęzie, żeby tag wskazywał na czysto wyglądający commit.
**Analiza:** Proweniencja SLSA potwierdza, gdzie artefakt zbudowano, a nie czy kod był przejrzany. Przejęte konto maintainera i własne CI dają „zaufany” złośliwy pakiet. Kontrole: wymóg review/branch protection po stronie zależności (OpenSSF Scorecard), opóźnienie adopcji nowych wersji (cooldown 24–72 h), egress allowlist w CI i runtime oraz sekrety poza zmiennymi środowiskowymi.
Źródło: [StepSecurity](https://www.stepsecurity.io/blog/wppconnect-compromised-malicious-releases-steal-secrets-and-whatsapp-sessions)

<hr>

#### Backdoor w modelu ONNX: wyciek plików i pamięci hosta inferencyjnego przez parser ONNX Runtime (TLDR InfoSec / Phrack)
Złośliwy model ONNX na bazie Qwen2 działał normalnie do słowa-wyzwalacza, po którym wyciekał pliki lub pamięć serwera. Cały atak mieścił się w pliku `.onnx`. Wykorzystano błąd parsera ONNX Runtime, który podczas `Load()` „densyfikował” rzadkie tensory z wybranych przez atakującego ścieżek plików lub adresów pamięci, zanim zadziałała walidacja ścieżki. Skradzione bajty były kodowane w tokenach wyjściowych. Poprawka: onnxruntime 1.27.0+.
**Analiza:** Format bez wykonywalnego kodu (w przeciwieństwie do pickle) nie gwarantuje bezpieczeństwa, bo powierzchnią ataku jest parser. Modele z zewnątrz trzeba traktować jak niezaufany kod: ładować w sandboxie bez dostępu do sekretów i systemu plików hosta (read-only rootfs, seccomp, osobny namespace), skanować i przypinać do hashy. Wpisać onnxruntime do inwentarza AIBOM i polityki minimalnych wersji.
Źródło: [Phrack #73](https://phrack.org/issues/73/backdoor-attacks-against-machine-learning-irl)

<hr>

#### mcp-attest: deterministyczny test autoryzacji serwerów MCP jako bramka CI (TLDR InfoSec)
Narzędzie łączy się z działającym serwerem MCP jako każda tożsamość z pliku polityki i wykonuje prawdziwe wywołania narzędzi, szukając przypadków, w których serwer pozwala na coś, czego polityka zabrania (m.in. dostęp do obiektów innych użytkowników). Sprawdza 12 przypadków tokenów OAuth (`alg: none`, zła audience, token passthrough), mapuje reguły na OWASP MCP Security Testing Guide i eksportuje podpisane HMAC paczki dowodowe oraz SARIF.
**Analiza:** Serwery MCP są wdrażane z uprzywilejowanymi poświadczeniami szybciej niż przechodzą przegląd bezpieczeństwa. Test oparty na polityce w CI to tani sposób na wychwycenie regresji autoryzacji (IDOR, token passthrough) przed produkcją. Ograniczenie: wynik zależy od kompletności pliku polityki.
Źródło: [GitHub – Balanov/mcp-attest](https://github.com/Balanov/mcp-attest)

<hr>

#### Windows na Ignite: mikro-VM z izolacją sprzętową dla agentów AI i tryb „learning” przed egzekwowaniem polityk (TLDR IT)
Microsoft zapowiedział ulepszenia kontenerów procesów (preview) i eksperymentalne mikro-VM z izolacją sprzętową dla agentów AI w Windows. Tryby uczenia (GA) pozwalają IT obserwować zachowanie agenta przed włączeniem ograniczeń. Intune testuje automatyczne aktualizacje aplikacji instalowanych przez pracowników.
**Analiza:** Izolacja agentów na endpointach przechodzi z poziomu uprawnień użytkownika na poziom hypervisora, tak jak wcześniej w przeglądarkach (WDAG). Tryb learning to wzorzec znany z AppLocker/WDAC: najpierw audyt, potem enforcement. Trzeba zaplanować koszt (RAM/CPU na mikro-VM) w VDI i na słabszych endpointach oraz integrację telemetrii agentów z EDR/SIEM.
Źródło: [Windows Experience Blog](https://blogs.windows.com/windowsexperience/2026/11/17/microsoft-ignite-2026-building-windows-for-the-next-era-of-work/)

<hr>

#### Snowflake Open Data Sharing: współdzielenie tabel Iceberg przez REST Catalog bez konta Snowflake (TLDR IT)
Dostawcy danych mogą udostępniać tabele Apache Iceberg klientom korzystającym z dowolnego klienta zgodnego z Iceberg REST Catalog (Spark, Trino, DuckDB itp.), bez konta Snowflake i bez eksportów.
**Analiza:** Iceberg REST Catalog staje się standardowym interfejsem wymiany danych między platformami, co obniża lock-in warstwy przechowywania. Do sprawdzenia: model kosztów egress (kto płaci za odczyt), egzekwowanie polityk (row/column-level security) poza Snowflake i spójność snapshotów przy częstych commitach.
Źródło: [Snowflake](https://www.snowflake.com/en/blog/open-data-sharing-iceberg-rest-catalog/)

<hr>

#### OpenAI GPT-6.2 Astra/Sol i siedem modeli Microsoft AI (MAI Code 2: 98% jakości GPT-6.1 Sol za ok. 40% ceny) (TLDR AI)
OpenAI wydało parę modeli: Astra (frontier) i Sol (wydajny), oba w ChatGPT Work, Codex i API, z deklarowaną lepszą jakością przy mniejszej liczbie tokenów wyjściowych. Microsoft AI wydał siedem modeli (kod, bezpieczeństwo, edycja obrazów, głos w czasie rzeczywistym). MAI Code 2 w GitHub Copilot ma osiągać 98% jakości kodowania GPT-6.1 Sol przy ok. 40% ceny.
**Analiza:** Mniej tokenów wyjściowych na zadanie obniża koszt i latencję bardziej niż obniżka ceny za token, bo decode dominuje w kosztach inferencji. Microsoft buduje własne modele pod własny krzem (Maia), co zmniejsza zależność Copilota od OpenAI i daje kontrolę nad marżą. Ewaluację zmiany modelu robić per zadanie na własnym zestawie testów: FrontierHarness pokazuje, że sam harness zmienia koszt zadania do 5×.
Źródło: [TestingCatalog](https://www.testingcatalog.com/openai-launches-gpt-6-2-astra-and-sol-on-chatgpt-and-apis/), [Microsoft AI](https://microsoft.ai/news/microsoft-ignite-2026/), [Runta – FrontierHarness Eval v1.1](https://runta.com/blog/introducing-frontierharness-eval-v1-1/)

<hr>

#### Claude Docs: „chaos loop” przed startem – 60 błędów serwera na 2 mld żądań, ok. 50 podów (TLDR)
Anthropic opisuje pętlę wstrzykiwania awarii użytą do utwardzenia Claude Docs przed premierą. W 6 dni po starcie powstało ponad 4,6 mln dokumentów, przy 60 błędach serwera na 2 mld żądań i bez rollbacku. Usługa działała na ok. 50 podach zoptymalizowanych pod pamięć.
**Analiza:** Ciągłe wstrzykiwanie awarii jako element pipeline'u (zamiast okazjonalnych game days) skraca czas od regresji do wykrycia. 60 błędów na 2 mld żądań to ok. 99,99999% skuteczności na poziomie żądań, przy małym footprint. Wzorzec warty powielenia dla usług z gwałtownym wzrostem ruchu po starcie.
Źródło: [Claude Blog](https://claude.dev/blog/chaos-loop-claude-docs-launch/)

<hr>

#### RHEL jako obrazy GitHub-hosted runners – GA (TLDR IT)
Zespoły korzystające z GitHub Enterprise Cloud i Server mogą uruchamiać pipeline'y CI/CD bezpośrednio na natywnych środowiskach RHEL hostowanych przez GitHub.
**Analiza:** Budowanie i testowanie na tej samej dystrybucji co produkcja (glibc, OpenSSL w trybie FIPS, SELinux) bez utrzymywania self-hosted runnerów. Mniej self-hosted runnerów to też mniejsza powierzchnia ataku w CI (zob. WPPConnect, WFX-Loot).
Źródło: [Red Hat Developer](https://developers.redhat.com/articles/2026/11/17/red-hat-enterprise-linux-github-hosted-runner-images-ga)

<hr>

#### NVIDIA DGX Spark Developer Kit: 4 węzły GB10 w pierścieniu bez przełącznika, 512 GB pamięci (Own news / HPCwire)
Na SC26 NVIDIA pokazała zestaw czterech węzłów DGX Spark (Grace Blackwell) połączonych w pierścień bez przełącznika: łącznie 512 GB pamięci i 16 TB NVMe, do lokalnego uruchamiania wielowęzłowych obliczeń (LAMMPS, NAMD, GROMACS, OpenFold). Zapowiedziano playbook HPC SDK (C/C++/Fortran, MPI).
**Analiza:** Mini-klaster na biurku do developmentu i debugowania kodu MPI/NCCL bez kolejki na superkomputerze. Topologia pierścienia ogranicza przepustowość all-to-all, więc zestaw nadaje się do testów poprawności i skalowania na kilku węzłach, nie do benchmarków wydajności.
Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/nvidia-dgx-spark-developer-kit-turns-a-desk-into-a-supercomputing-lab/)

<hr>

#### NVIDIA: DFT fal płaskich na GPU – batching punktów k i mieszana precyzja (Own news / NVIDIA Technical Blog)
Wpis opisuje wąskie gardła GPU w obliczeniach DFT metodą fal płaskich (typu VASP), które generują dane treningowe dla potencjałów interatomowych (MLIP): od dziesiątek do setek milionów obliczeń energii, sił i naprężeń. Proponowane techniki to batching wielu punktów k w jednym wywołaniu oraz mieszana precyzja.
**Analiza:** Generacja danych DFT staje się workloadem przepustowościowym (wiele małych zadań), a nie klasycznym HPC. Batching podnosi wykorzystanie GPU przy małych komórkach, a mieszana precyzja wpisuje się w trend emulacji FP64 i ogranicza przewagę natywnego FP64 (zob. AMD MI430X). Przed zmianą precyzji walidować zbieżność SCF.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/tackling-gpu-bottlenecks-in-plane-wave-dft-with-k-point-batching-and-mixed-precision/)

## Implikacje praktyczne

1. **Inwentarz podatności zależnych od sprzętu.** Java4Shell pokazuje, że ekspozycja zależy od flag CPU, a nie od wersji oprogramowania. CMDB i skanery muszą łączyć wersje runtime z mikroarchitekturą hostów i typami instancji. W horyzoncie 3–5 lat kodu ręcznie optymalizowanego pod SIMD (AVX-512, SVE, AMX) będzie przybywać, więc mitygacje trzeba mieć gotowe do wdrożenia flagą w całej flocie (`JAVA_TOOL_OPTIONS`, polityki w orkiestratorze).
2. **Czas łatania liczony w godzinach.** Patch-diffing przez agentów (GTIG) skraca okno między publikacją łatki a exploitem. Dla systemów brzegowych i płaszczyzn zarządzania (Puppet, RMM, IdP) potrzebne są automatyczne wirtualne łatki (WAF/IPS) i kanarkowe wdrożenia z automatycznym rollbackiem. Jednocześnie seria awarii z 12–17.11 pokazuje, że tempo zmian samo jest ryzykiem. Weryfikacja stanu po zmianie (intent vs state) musi być częścią pipeline'u.
3. **Ryzyko zasilania w kontraktach na moc.** Force majeure Oracle w Project Jupiter przenosi ryzyko energetyczne na dewelopera i wierzycieli. Przy kontraktach colocation i najmu GPU na 2027–2030 trzeba sprawdzać klauzule siły wyższej związane z zasilaniem, harmonogram przyłącza i to, kto ponosi koszt opóźnienia. Preferować lokalizacje z potwierdzonym przyłączem nad obietnicami behind-the-meter.
4. **Rozdzielenie prefill/decode jako założenie projektowe.** Cerebras (GPU + wafer), Dynamo i llm-d pokazują, że heterogeniczne pule pod prefill i decode dają kilkukrotnie wyższą wydajność na MW. Przy projektowaniu klastrów inferencyjnych na 2027+ planować osobne pule, sieć o wysokiej przepustowości do transferu KV cache i warstwę pamięci pod HBM (CXL, pamięć optyczna, NVMe).
5. **Ceny GPU w kontraktach indeksowane, nie stałe.** Ceny spot rosną (H100 +6–8%), ale 2027 przynosi duże odnowienia (130 tys. GPU w samym CoreWeave) i rampę Rubin. Kontrakty wieloletnie negocjować z mechanizmem odnowienia po indeksie (OCPI, Silicon Data), prawem do migracji na nowszą generację i bez zobowiązań typu take-or-pay dłuższych niż ekonomiczny okres życia GPU.

## Trend tygodnia

Rynek kapitałowy dla AI rozwarstwia się. Laboratoria frontier pozyskują kapitał na rekordową skalę (Anthropic: 83 USD/akcję, ponad 81 mld USD w IPO plus 26 mld w prywatnej emisji), a dług infrastrukturalny pod nimi tanieje coraz trudniej. QTS zmniejsza emisję o 60%, HSBC sprzedaje z dyskontem kredyt pod Project Jupiter, a banki USA wycofują się z kredytów pod GPU w Azji. Ryzyko przesuwa się z popytu, bo ceny najmu GPU rosną, a CoreWeave deklaruje nadwyżkę popytu, na ryzyko zasilania, eksportu i refinansowania: force majeure z powodu energii, licencje BIS i zbieżność zapadalności długu z wygasaniem kontraktów w 2027. Po stronie technologii najwięcej wydajności dają rozdzielenie i specjalizacja: osobny sprzęt do prefillu i decode, pule pamięci poza GPU (fotonika Marvell/Penguin), Ethernet z inteligentnym NIC zamiast zamkniętych interconnectów (Lux, Arista R4). W bezpieczeństwie atakujący automatyzują cały łańcuch od łatki do exploita, a łańcuch dostaw psują nie tylko przejęte biblioteki, ale też podpisane, „zaufane” artefakty (SLSA w WPPConnect, CDN Optimizely, parser ONNX). Wspólny wniosek dla architektów: zaufanie oparte na pochodzeniu (dostawca, podpis, rating najemcy) przestaje wystarczać i trzeba je zastąpić weryfikacją w runtime, zarówno w kodzie, jak i w kontraktach na moc.

## To obserwować

- **Pierwsze sesje ANT (Nasdaq) i wykonanie opcji underwriterów (~147 mln akcji):** wskaźnik apetytu rynku akcji na laboratoria AI w zestawieniu z CRWV/NBIS i spreadami obligacji neocloudów.
- **Warunki re-leasingu 130 tys. GPU CoreWeave na 2027:** ogłoszenia przedłużeń lub przedpłat, rentowność obligacji niezabezpieczonych (~15%) i ewentualne transakcje refinansujące zapadalności 2027 (~11 mld USD).
- **Project Jupiter / Oracle–Blue Owl:** czy siła wyższa zostanie formalnie uznana (odroczenie czynszu do 3 lat), decyzja o zasilaniu Bloom Energy i wtórny obrót innymi transzami kredytu 18 mld USD.
- **Wdrożenie poprawek Java4Shell w dystrybucjach OpenJDK (Temurin, Corretto, Red Hat, Zulu) i pierwsze ślady exploitacji;** wydanie PHP 19.11 i ewentualne wpisy do CISA KEV.
- **EuroHPC AI Gigafactories:** nowe linki do składania ofert, termin Lot 2 (3.12.2026) i lista konsorcjów, które ponownie złożą oferty (AIGF Northern, Mistral).
