# Raport Zbiorczy IT – 2026-11-14

**Okno przyrostowe:** od raportu z 2026-11-13. Zdarzenia z 10–12.11 są ujęte tylko wtedy, gdy nie było ich w rejestrze. Źródła: newslettery z etykietą NEWSY (TLDR, TLDR AI/IT/InfoSec/Hardware, Own news), RSS NVIDIA Developer Blog i HPCwire, SEC EDGAR, WWW. Problemy ze źródłami: RSS SemiAnalysis nadal zwraca wpisy z 2025 r.; RSS HPCwire/AIwire blokuje Cloudflare (403), a część artykułów HPCwire nie daje się pobrać bezpośrednio; gartner.com nie ma kanału RSS i nie znalazłem tam nic nowego.

## Raport technologiczny

### Biznes

#### Awaria Microsoft 365 (12–13.11): Exchange Online, Teams, Copilot, Defender for O365 oraz Intune
Awaria zaczęła się ok. 18:00 UTC 12.11. O 00:35 UTC 13.11 usługi nadal nie były w pełni przywrócone. Według Microsoftu nie było zmiany, która mogłaby ją wywołać. Health checki serwerów zawodziły, prawdopodobnie przez komponent przetwarzający stan zdrowia, a proces rdzeniowy na serwerach był przeciążony i przestawał odpowiadać. Microsoft wdrożył alternatywną metodę health checku, restartował serwery i stopniowo przywracał ruch. Część naprawionych serwerów padała ponownie, gdy wracał na nie ruch. Rozważano też cofnięcie kilku ostatnich zmian konfiguracji.
**Znaczenie:** To klasyczny failure mode warstwy kontroli. Fałszywie negatywne health checki zdejmują zdrowe serwery z puli, a pozostałe padają pod ruchem ponownie przywracanym (retry storm). Razem z płaszczyzną danych padły narzędzia administracyjne (Intune, M365 Apps Admin Center), więc w trakcie incydentu nie dało się zarządzać endpointami. Plany BCP powinny zakładać, że poczta, łączność i MDM jednego dostawcy znikają jednocześnie. Warto mieć niezależny kanał komunikacji kryzysowej i lokalne procedury break-glass dla endpointów.
Źródło: [The Register](https://www.theregister.com/saas/2026/11/12/microsoft-struggling-to-fix-massive-cloud-outage-thats-taken-down-email-and-intune/5309531)

### Technologia

#### Apache HTTP Server: RCE bez uwierzytelnienia w mod_http2 (CVE-2026-119618, CVSS 9,8)
Podatne są wersje 2.4.55–2.4.70 z włączonym HTTP/2 oraz samodzielne buildy mod_h2 2.0.8–2.0.42. Repozytorium mod_h2 jest zarchiwizowane i nie dostanie poprawki. Poprawka jest w 2.4.71 (wydanej 11.11) oraz w paczkach RHEL 9 `mod_http2-2.0.26-7.el9_9.1` i RHEL 10 `mod_http2-2.0.29-5.el10_3.1`. Debian bookworm/trixie nie miał poprawki w chwili publikacji, Amazon Linux jest w toku. Red Hat twierdzi, że istnieje niezawodny publiczny exploit, Wiz – że exploita nie ma. W RHEL ryzyko ogranicza osobny alokator pamięci per stream. Zgłoszenie (26.10) przyszło m.in. z Anthropic OSS Scanner i Google Cloud Vulnerability Research. Wersje 2.4.67–2.4.70, które łatały poprzednią lukę HTTP/2 (CVE-2026-23918), są podatne.
**Znaczenie:** Ekspozycja jest szeroka: reverse proxy, ingressy i appliance'y z wbudowanym httpd. Jeśli nie da się od razu wdrożyć poprawki, trzeba usunąć `h2`/`h2c` z każdej dyrektywy `Protocols`, także w vhostach. Kosztem jest utrata multipleksowania i wyższe opóźnienia dla klientów. Appliance'y z wbudowanym Apache będą czekać na vendorów tygodniami, więc trzeba je zinwentaryzować.
Źródło: [The Hacker News](https://thehackernews.com/2026/11/critical-apache-http2-flaw-could-let.html)

#### WFX-Loot: stealer sekretów CI uruchamiany w 1 835 repozytoriach z 4 837 przejętych kont GitHub
Atak trwał od 10.11 12:45 UTC. Kampania otworzyła 61 764 próbne PR-y („docs: dependency audit notes”, gałąź `wfx-docs`), po czym wstrzyknęła payload jednym z trzech sposobów: kroki workflow, hook `postinstall` albo tymczasowy workflow na gałęzi `chore/`. Po ok. minucie zmiany cofała, a run i logi kasowała. Skrypt ma ok. 1 500 linii. Zbiera sekrety Actions, pliki poświadczeń chmur, K8s i rejestrów, klucze AI oraz pamięć procesów `Runner.Worker`. Mintuje tokeny OIDC i wymienia je na sesje AWS, GCP i Azure. Persystencję zapewnia na trzy sposoby: przepisuje trust policy roli AWS na `"Principal": {"AWS": "*"}`, dodaje federated credentials `wfx-ci-…` w Entra (przeżywają rotację tokenów) i fałszuje statusy wymaganych checków. Eksfiltracja idzie przez HTTP POST na 194.150.166[.]244:8080 i 18.143.168[.]96:8080.
**Znaczenie:** Rotacja sekretów nie wystarcza. Trzeba też sprawdzić trust policies IAM i federated credentials, bo te tylne drzwi nie zależą od skradzionych sekretów. Sygnałem detekcji są luki w numeracji runów workflow. Obrona: zawężone warunki trust OIDC (`sub` z konkretnym repo i ref, bez wildcardów), wymagane checki dopuszczające tylko konkretną GitHub App jako źródło statusu, alerty CloudTrail na `UpdateAssumeRolePolicy`.
Źródło: [Aikido](https://www.aikido.dev/blog/compromised-github-accounts-push-ci-secret-stealer-to-tens-of-thousands-of-repositories)

#### Red Hat Quay: trwały klucz podpisujący zapewnia R/W do wszystkich repozytoriów (CVE-2026-123710, CVSS 9,8)
Każdy, kto zdobędzie token rejestru (także anonimowy), może zarejestrować w service key server klucz podpisujący bez daty wygaśnięcia. Klucz daje odczyt i zapis we wszystkich repozytoriach. Zmiana haseł ani tokenów robotów go nie unieważnia. Poprawionej wersji jeszcze nie ma.
**Znaczenie:** Prywatny rejestr obrazów jest punktem zaufania całego łańcucha wdrożeń. Zapis do repozytorium oznacza podmianę obrazów bazowych we wszystkich klastrach. Do czasu poprawki: ograniczyć sieciowo dostęp do endpointu service keys, przejrzeć tabelę kluczy pod kątem nieznanych wpisów i wymusić weryfikację podpisów (cosign/Notation) po stronie admission controllera, niezależnie od ACL rejestru.
Źródło: [TLDR InfoSec / advisory Red Hat](https://links.tldrnewsletter.com/3wU1bM)

#### Microsoft FORGE: 26 502 surowe znaleziska AI w jądrze Linux → 59 zgłoszeń, 33 CVE, 8 działających exploitów root
Skan trwał od końca maja do połowy czerwca i objął sieć, systemy plików i LSM. Kod skanowały modele Sonnet 4.6 (71% tokenów), GPT-5.4 i GPT-5.5. Lejek: 26 502 znaleziska → 11 769 kandydatów → 621 z PoC w QEMU → 59 zgłoszeń. 24 z 33 CVE mają CVSS ≥ 7. Błędy miały średnio 8,1 roku (najstarszy, w cls_u32, ok. 21 lat). Przykład: UAF w iteratorze rhashtable (CVE-2026-64563), osiągalny przez `__netlink_diag_dump()` dla nieuprzywilejowanego użytkownika, daje LPE także na Androidzie z SELinux enforcing. W mainline jest 38 poprawek, mediana od patcha do wydania to 41 dni (maks. 108). Raporty kosztowały 4 osobomiesiące pracy ekspertów.
**Znaczenie:** Wąskim gardłem jest naprawa, nie wykrywanie. Kernele LTS i vendorowe (appliance'y, Android, BSP) będą miały rosnącą kolejkę znanych, publicznych LPE. Dla multi-tenant (K8s, CI runnery) wniosek jest taki, że izolacja oparta tylko na namespace'ach staje się niewystarczająca. Trzeba ograniczać dostęp do netlink i nietypowych rodzin gniazd (seccomp) i przejść na izolację VM, np. gVisor lub Kata, dla obcego kodu.
Źródło: [Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2026/11/11/lessons-from-59-linux-kernel-bug-reports/)

#### GSocket jako C2 bez infrastruktury atakującego (Microsoft, +356% detekcji III–VIII)
Po wejściu przez aplikacje PHP, cPanel lub Apache atakujący instaluje klienta GSocket w `/dev/shm`, nadaje procesowi nazwę wątku jądra i utrwala go przez cron, systemd i profile powłoki. Klient łączy się wychodząco z publiczną siecią relayów GSocket, a parowanie z operatorem odbywa się samym współdzielonym sekretem. Nie ma więc IP ani domeny atakującego do zablokowania. Część aktorów próbowała też eskalacji przez Dirty Frag (CVE-2026-43284).
**Znaczenie:** Blokowanie IoC nie działa, działa tylko egress deny-by-default dla serwerów webowych (allowlista), `noexec` na `/dev/shm` i `/tmp` oraz wykrywanie procesów udających wątki jądra (PPID≠2). Sama poprawka aplikacji to połowa remediacji.
Źródło: [Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2026/11/11/global-socket-how-threat-actors-weaponizing-gsocket-stealthy-c2/)

#### SCKit: robak przez rozszerzenie RMM+ do ConnectWise ScreenConnect – 288 endpointów w 48 organizacjach
Rozszerzenie RMM+ w wersjach 1.0.31 i 1.0.32 akceptowało pusty klucz dostępu. Atakujący użyli komend Run w chmurowych panelach, aby instalować fałszywych klientów i SCKit, który podmienia legalnego klienta i kradnie cookies sesji techników. ConnectWise wyłączył 5 rozszerzeń w ponad 400 środowiskach partnerów.
**Znaczenie:** Rozszerzenia firm trzecich w RMM to łańcuch dostaw o uprawnieniach SYSTEM na całej flocie MSP. Potrzebne są: inwentarz i allowlista rozszerzeń, wiązanie sesji techników z urządzeniem (token binding/FIDO2) oraz alerty na masowe komendy Run.
Źródło: [Huntress](https://www.huntress.com/blog/sckit-worm-rogue-screenconnect)

## Raport AI-ML

### Biznes

#### Cerebras Q3 2026 (8-K, 12.11): RPO 26,1 mld USD, chmura +265%, marża GAAP 20%
Przychód GAAP wyniósł 210,8 mln USD (+55% r/r). Chmura i usługi dały 141,9 mln (+265%), sprzęt 68,9 mln (rok wcześniej 96,8 mln). Marża brutto GAAP to 20,0% (sprzęt 11,0%, chmura 24,3%), marża „core” 41%. Strata netto w kwartale wyniosła 161,2 mln USD, po 9 miesiącach 625,8 mln. Przepływy operacyjne po 9 miesiącach: −388,9 mln USD, capex: −1 046,1 mln. Spółka ma 700 MW brutto mocy DC pod kontraktem do końca 2027 r., w tym nowe lokalizacje w Norwegii i Finlandii. Prognoza core na IV kw. to 310–330 mln USD przy marży 47–49%; na 2027 r. zapowiedziano potrojenie przychodu. WSE-5 jest w testach systemowych (od tape-outu <5 mies.), CS-5 ma trafić na rynek wczesnym latem 2027 z obietnicą do 10 tys. tok/s na użytkownika. Wśród ryzyk spółka wskazuje koncentrację na kilku klientach, w tym na umowie z OpenAI.
**Analiza:** Cerebras przechodzi z modelu sprzedaży sprzętu na operatora chmury inferencyjnej. W kosztach widać to po kapitałochłonności (capex ok. 5× przychodu kwartalnego) i niskiej marży chmury (24%), bo amortyzacja klastrów wyprzedza ich wykorzystanie. RPO 26 mld przy rocznym przychodzie <1 mld oznacza, że wynik zależy od terminowego oddania 700 MW i od kondycji jednego, dużego odbiorcy. Dla klientów latencja per user (tok/s) jest realną przewagą nad GPU w agentach i kodowaniu, ale lock-in jest silniejszy niż przy GPU, bo dostępny jest tylko jako API dostawcy, bez przenośnego stosu.
Źródło: [SEC 8-K](https://www.sec.gov/Archives/edgar/data/0002021728/000162828026073366/cbrsannouncesfinancialresu.htm)

#### Forensyka checkpointów: chińskie modele (DeepSeek, Moonshot, MiniMax, Ant) inicjalizowane na Blackwell
Analiza z 12.11 odtwarza układ generatora Philox/cuRAND przy inicjalizacji wag w PyTorch. Mapowanie liczb na pozycje zależy od liczby SM: Hopper ma 132 SM, B200/B300 – 148, GB200/GB300 – 152. Ślad przetrwał trening, najwyraźniej w nieużywanych wierszach słownika. Sześć modeli wydanych od kwietnia ma sygnaturę Blackwell (49–1 385 σ): DeepSeek-V4-Flash i MiniMax-M3 (148 SM) oraz DeepSeek-V4-Pro, Kimi K3, K3.1-Pro i Ling-3.1-flash (152 SM). Metoda zgodziła się z deklaracją sprzętu w 24 znanych przypadkach. Analiza dowodzi jedynie dostępu do sprzętu. Nie ustala, gdzie chipy były, czyje są ani czy złamano przepisy.
**Analiza:** Pojawia się tania, pasywna metoda audytu pochodzenia compute na podstawie publicznych wag. Należy się spodziewać nacisku regulacyjnego na zdalny dostęp do Blackwell przez chmury poza USA (zob. 12.11: Moonshot pytał europejskich neocloudów o 8 000+ GPU). Dla europejskich neocloudów rośnie ryzyko KYC i sankcji przy klientach z Chin. Ryzyko odwrotne: producenci modeli zaczną randomizować seed lub inicjalizację, więc metoda straci skuteczność w kolejnych wydaniach.
Źródło: [Unemployment Labs – Checkpoint Forensics](https://unemploymentlabs.com/checkpoint-forensics)

### Technologia

#### cuBLAS (CUDA 13.4.2): emulacja FP64 Ozaki-II na Tensor Core INT – do 5,5× DGEMM na VR NVL72
DGEMM i ZGEMM są domyślnie liczone przez rozkład operandów na liczby całkowite i GEMM na Tensor Core INT. Ozaki-II (oparty o chińskie twierdzenie o resztach) wymaga liczby GEMM rosnącej liniowo z liczbą modułów, Ozaki-I – kwadratowo z liczbą plastrów. ZGEMM metodą 2M zamiast 4M daje do 5× na GB200. Wyniki: VR NVL72 do 5,5× (175 TFLOPS), GB200 do 3,9× (147 TFLOPS), RTX PRO 6000 BSE 10–23× (37 TFLOPS). Liczbę bitów mantysy dobiera framework ADP i w razie potrzeby wraca do natywnego FP64. Rubin dostaje format TI16 i `tcgen05.mma` (CC 10.7). Sterowanie przez `cublasSetEmulationStrategy`.
**Analiza:** NVIDIA formalnie przesuwa FP64 z jednostek natywnych do emulacji na krzemie AI. To zmienia capacity planning HPC: wydajność FP64 przestaje być stała dla danej karty i zależy od rozmiaru macierzy, rozpiętości wykładników (ADP może wrócić do natywnego FP64) oraz pamięci workspace, która rośnie z liczbą modułów. Dla ośrodków mieszanych HPC/AI zmniejsza to presję na kupowanie akceleratorów FP64-centric (np. AMD MI430X). Trzeba jednak dodać testy reprodukowalności i zweryfikować wyniki naukowe przy zmianie strategii emulacji.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/faster-scientific-simulations-with-new-fp64-algorithms-in-nvidia-cublas/)

#### GLM-5.3 (753B MoE, NVFP4) na dwóch DGX Station GB300 przez ConnectX-8 – vLLM TP=2
Checkpoint ma ok. 464 GB, czyli więcej niż jedna karta GB300 z 252 GB HBM3e. Konfiguracja: TP=2 przez dwa bezpośrednie linki RDMA 400 Gb/s (MTU 9000), vLLM 0.30.0 i NCCL 2.30.7 (pinowany ze względu na deadlock CUDA graph między węzłami), MTP z 4 tokenami draft, KV cache w FP8, prefix caching, max kontekst 98k. Wyniki na benchmarku kodowania: C1 – 107 tok/s i ITL 8,8 ms; C16 – maksimum przepustowości 608 tok/s przy ITL 23,8 ms; C32 – spadek do 385 tok/s, ITL 78 ms, TTFT 0,9 s. Endpoint domyślnie nie ma uwierzytelnienia i nasłuchuje na 0.0.0.0.
**Analiza:** To dobry punkt odniesienia dla on-prem modeli frontier w działach R&D bez klastra. Sufit to ok. 16 równoległych sesji agentowych, powyżej KV cache i komunikacja TP między węzłami powodują gwałtowny spadek przepustowości (knee przy C16–C32). Ryzyka: zależność od konkretnej wersji NCCL i obrazu vLLM, brak HA (dwa węzły = dwa SPOF) oraz otwarty endpoint, który wymaga mTLS i gateway'a.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/when-one-gpu-isnt-enough-serving-glm-5-3-across-two-nvidia-dgx-stations-with-vllm/)

#### AMD Verano (host CPU dla Instinct): 24 kanały LPDDR5X SOCAMM2 i xGMI 112 Gb/s; Zen 6c z najwyższym IPC
Verano występuje w dwóch wariantach: 72 rdzenie Zen 6 przy 5,3 GHz albo 192 rdzenie gęste przy 4,1 GHz. Własne die I/O obsługują 24 kanały LPDDR5X (SOCAMM2, do 8 533 MT/s), co według AMD obniża pobór mocy o 30–40% wobec RDIMM. AMD poświęca przepustowość pamięci na rzecz linków xGMI 112 Gb/s do GPU. Mike Clark w rozmowie z Chips and Cheese: Zen 6c (32 rdzenie, 128 MB L3) ma teraz najwyższe IPC w rodzinie. Zmiany w Zen 6: 2 zamiast 3 mnożników całkowitych, 4 ładowania wektorowe na cykl, dwa potoki VNNI dodane pod inferencję, L2 64 B/cykl.
**Analiza:** AMD powtarza architekturę Grace/Vera: CPU w racku AI ma przede wszystkim zasilać GPU (koherentny link, pamięć LPDDR jako tańsze i chłodniejsze rozszerzenie dla KV cache i offloadu). SOCAMM2 jest modułowe, ale nie jest standardowym RDIMM, więc łańcuch dostaw pamięci zależy od tych samych dostawców LPDDR, co w racku NVIDII, przy obecnym niedoborze DRAM. Dla klasycznych serwerów VM wariant 192c z dużym L3 może dać przewagę w gęstości vCPU na wat.
Źródła: [The Register](https://www.theregister.com/systems/2026/11/10/a-closer-look-at-verano-amds-first-step-toward-a-zen-based-superchip-to-rival-nvidias-vera-rubin/5308523), [Chips and Cheese](https://chipsandcheese.com/p/talking-zen-6-with-mike-clark-at)

#### Penguin SMART CXL-8F2W-D4: 8× DDR4 RDIMM na karcie CXL 2.0 (Marvell Structera X)
Karta dodaje do serwerów z CXL 2.0 pamięć z odzyskanych modułów DDR4. Penguin prowadzi program pozyskiwania DDR4 z wycofywanych serwerów. Producent nie podał przepustowości, latencji ani pojemności maksymalnej. Każda karta jest walidowana w kompatybilnym hoście przed wysyłką. Marvell na SC26 pokaże też Structera X z DDR4 i DDR5 oraz „Photonic Fabric” do optycznego rozszerzenia pamięci między rackami.
**Analiza:** W warunkach kryzysu pamięci (przychód DRAM +369% przy ok. +4% bitów, zob. 08.11) DDR4 za CXL to tani tier pojemnościowy dla offloadu KV cache, baz in-memory i wektorowych. Kosztem jest ok. +100–200 ns latencji względem lokalnego DRAM i przepustowość ograniczona linkiem x16. Wymaga to tieringu świadomego NUMA i konfiguracji w kernelu (np. `memory_tiering`) albo w schedulerze. Należy przetestować awaryjność: odzyskane moduły mają nieznaną historię, więc potrzebny jest monitoring ECC i polityka wymiany.
Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/penguin-solutions-introduces-cxl-enabled-ddr4-module-for-ai-data-centers/)

#### Self-hosted Claude Managed Agents (private beta, 12.11): pętla agenta, sandboxy i sesje w infrastrukturze klienta
U klienta działają serwer i baza Managed Agents, harness agenta, sandboxy (shell i pliki), sekrety w jego vaulcie oraz historia sesji. Inferencja idzie do wybranego endpointu: Anthropic API, Bedrock, Google Agent Platform, Microsoft Foundry albo własny gateway. ZDR działa jak w Messages API. Można ustawić allowlistę hostów per środowisko i eksportować rekordy sesji przez OpenTelemetry.
**Analiza:** Rozdzielenie płaszczyzny wykonania agentów od inferencji pozwala trzymać poświadczenia i efekty uboczne (FS, sieć) w strefie zaufania organizacji. To warunek wstępny dla agentów z dostępem do produkcji w sektorach regulowanych. Organizacja przejmuje jednak operacyjnie skalowanie sandboxów, izolację (zob. FORGE – LPE w jądrze), retencję sesji i aktualizacje harnessu. Lock-in przesuwa się z warstwy modelu na format sesji i API orkiestracji.
Źródło: [Claude](https://claude.com/resources/articles/self-hosted-claude-managed-agents)

## euro neocloud

### Nscale

#### F-1/A (12.11): dług 5,5 mld USD, strata 1,69 mld po 9 miesiącach, „will require additional capital” na zobowiązania 2027
**Fakty:** Zaktualizowany prospekt pod notowanie na NYSE (ticker NSCL, prowadzący: Goldman Sachs, JPMorgan, Morgan Stanley) nie podaje przedziału cenowego ani wielkości emisji. Przychód: 307,5 mln USD. CNBC podaje to jako wynik III kw., TNW jako 9 miesięcy (I półrocze 140,6 mln USD), więc wartość kwartalna nie jest pewna. Strata netto: 1,69 mld USD (I półrocze 1,02 mld). Dług na 30.09: ok. 5,5 mld USD (koniec 2025: 78,9 mln; 30.06: 137,4 mln). Kontrakty aktywne: ok. 6,4 mld USD, aktywne i podpisane: 105,8 mld USD. Największy klient to Anthropic: 6-letni kontrakt do ok. 44,6 mld USD na 460 MW w Monarch (Wirginia Zachodnia), z pierwszą mocą od końca 2027. Nscale zainwestowało tam ok. 1 mld USD i potrzebuje co najmniej 30 mld więcej. Anthropic może rozwiązać umowę, jeśli finansowanie lub dostawa się nie powiodą, a niezakupienie sprzętu Nvidia grozi karami. Nvidia objęła 2 mld USD z ponad 3 mld USD konwertowalnych obligacji, ale nie ma obowiązku priorytetowej alokacji ani preferencyjnych cen. Spółka nie ma wiążących zobowiązań finansowych na rok 2027. Microsoft wycofał się z obiektu w Anglii (powód: dostęp do energii) i według FT nie odnowił kontraktu w Teksasie, choć prospekt nadal uznaje go za istotnego klienta w przyszłości.

**Ocena analityczna**
- *Wykryte sygnały ostrzegawcze:* luka finansowa rzędu 30 mld USD przy braku wiążących źródeł; wzrost długu ok. 40× w kwartał; ok. 94% portfela kontraktów nieaktywne; klauzula wyjścia Anthropic powiązana z finansowaniem; utrata dwóch lokalizacji Microsoftu; Nvidia jednocześnie inwestorem i dostawcą bez gwarancji alokacji.
- *Ocena ryzyka:* **Wysokie** (bez zmian kierunku, nowe są twarde liczby z prospektu). Model biznesowy zależy od otwartego rynku długu, który dla neocloudów jest dziś w praktyce zamknięty (obligacje CoreWeave >16%). Kontrakt Anthropic chroni przychód dopiero od końca 2027. Najbardziej prawdopodobny scenariusz to dalsze opóźnienie IPO i finansowanie gwarantowane przez Nvidię lub klienta, kosztem rozwodnienia udziałowców.
- *Linki:* [TNW](https://thenextweb.com/news/nscale-ipo-filing-debt-nvidia-anthropic), [CNBC](https://www.cnbc.com/2026/11/12/nscale-updates-ipo-prospectus-that-shows-widening-loss.html)

### Nebius

#### Aktualizacja: kurs −17% 12.11 po słabszej prognozie całorocznej
**Fakty:** Wyniki Q3 i przesunięcie celu ARR opisano 13.11. Nowy element to reakcja rynku: według CNBC akcje spadły o ok. 17% 12.11 po „light full-year guidance”, w dniu rezygnacji Huanga i publikacji F-1/A Nscale. CoreWeave dzień wcześniej wzrósł o ok. 25%, a 12.11 spadł o ok. 7%. Według Bloomberga (relacjonowanego przez Motley Fool 13.11) Microsoft chce zmniejszyć liczbę neocloudów, z którymi współpracuje. Kontrakt Microsoftu z Nebius do 2031 r. jest wart 17,4 mld USD.

**Ocena analityczna**
- *Wykryte sygnały ostrzegawcze:* koncentracja przychodu na Microsofcie przy deklarowanej konsolidacji jego dostawców; wysoka zmienność kursu podnosi koszt finansowania konwertowalnego.
- *Ocena ryzyka:* **Średnie**. Nebius ma kapitał z wcześniejszych emisji i linię Citi 3,5 mld USD, więc ryzyko płynności jest niższe niż w Nscale. Głównym ryzykiem jest nieodnowienie lub zmniejszenie zamówień przez Microsoft po 2027.
- *Linki:* [CNBC](https://www.cnbc.com/2026/11/12/nscale-updates-ipo-prospectus-that-shows-widening-loss.html), [Motley Fool / Bloomberg](https://www.fool.com/investing/2026/11/13/microsoft-just-piled-onto-the-bad-news-for-neoclouds/)

## AI for networking

#### Cisco Q1 FY27: zamówienia SP/cloud +47%, prognoza przychodu AI od hyperscalerów 9 mld USD; 800G-ZR wyprzedza 400G-ZR
Przychód wyniósł 18,5 mld USD (+24%), w tym networking 11,13 mld (+43%). Zamówienia produktowe w sieci wzrosły o 28%, w segmencie service provider i cloud o 47%, enterprise o 15%. Prognozę AI infra od hyperscalerów na FY27 (bez neocloudów) podniesiono z 7,5 do 9 mld USD, czyli ponad 2× FY26. Cisco ma 18 design-winów AI w 5 kwartałach. Według TLDR Hardware i CNBC wolumen koherentnych modułów 800G-ZR przewyższył 400G-ZR. AI Defense ma ok. 500 klientów i chroni 2 bln tokenów miesięcznie.
**Analiza:** Przewaga 800G-ZR to sygnał, że scale-across (łączenie DC w jeden klaster treningowy na dystansach 10–120 km) przechodzi z pilotażu do wdrożeń. Planowanie DCI musi teraz uwzględniać routery z 800G ZR/ZR+ bezpośrednio w portach (IPoDWDM) zamiast transponderów. Mniej warstw obniża koszt na bit, ale integruje warstwę optyczną z routingiem, co komplikuje operacje i wymaga wspólnej telemetrii IP i optyki. Brak neocloudów w prognozie ogranicza ekspozycję Cisco na ich problemy kredytowe.
Źródła: [Network World](https://networkworld.com/article/4248551/cisco-forecasts-9-billion-in-ai-infrastructure-revenue-in-fy27.html), [CNBC](https://www.cnbc.com/2026/11/12/cisco-csco-q1-2027-earnings.html)

#### Marvell na SC26: Teralynx 10 z Multipath Reliable Connection, AEC 1,6T i optyczny „Photonic Fabric” dla pamięci
Marvell pokaże (stoisko #4003) przełączniki Teralynx 10 z MRC, czyli transportem wielościeżkowym typu UEC/SRD dla RDMA, oraz Teralynx T100 w symulatorze DC. Na wystawie będą też aktywne kable elektryczne Alaska A (800G i 1,6T), moduł pamięci „Photonic Fabric”, który rozszerza domenę scale-up optycznie na wiele racków, karty CXL Structera i koherentne moduły pluggable do scale-across. Specyfikacji ani dat dostępności nie podano.
**Analiza:** Multipath w NIC i switchu (spraying z reorderingiem po stronie odbiorcy) zastępuje klasyczne ECMP, które przy dużych przepływach all-reduce kolidują i tworzą hotspoty. To warunek dużej utylizacji fabricu bez ręcznego strojenia hashy. Wybór MRC wiąże jednak NIC i switch z jednym ekosystemem, dopóki UEC 1.x nie ma certyfikowanej interoperacyjności. AEC 1,6T przesuwa granicę miedzi w racku i odracza konieczność CPO, kosztem masy i promienia gięcia kabli.
Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/marvell-scales-connectivity-to-address-ai-infrastructures-next-bottleneck-at-sc26/)

#### AWS Cloud WAN Flow Logs (11.11): telemetria przepływów dla całej sieci rdzeniowej, krawędzi lub pojedynczego attachmentu
Rekord przepływu zawiera IP, porty, protokół, wolumen i liczniki utraty pakietów, a także metadane segmentu, regionu, attachmentu i hopów service insertion (np. firewalli). Logi trafiają do CloudWatch Logs, S3 lub Firehose. Funkcja jest bezpłatna, płaci się tylko za miejsce docelowe.
**Analiza:** Do tej pory analiza ruchu w globalnej sieci AWS wymagała łączenia VPC Flow Logs z wielu kont i regionów. Metadane segmentu i service insertion pozwalają wprost sprawdzić, czy ruch przechodzi przez inspekcję, co przydaje się w audycie i jako dane dla modeli wykrywania anomalii ruchu. Przy dużym ruchu (np. replikacja datasetów między regionami) koszt sprowadza się do ingestu CloudWatch, dlatego tanim wariantem jest Firehose do S3 i analiza w Athena.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/aws-cloud-wan-flow-logs/)

#### Unit 42: ARTEX/CyberStrikeAI – agentowe ataki bez exploitów, sygnatura czasowa „czekania na model”
Analiza zawiera nowe dane. Agenci zamieniali publiczne strony resetu haseł i wystawione specyfikacje OpenAPI w dostęp do chmury, czasem w mniej niż dobę. W ok. 3 minuty zweryfikowali ponad 200 client secrets. Ruch szedł seriami po 1–14 żądań, przedzielonymi przerwami 25–31 s, gdy agent czekał na odpowiedź modelu. W internecie wykryto 609 hostów ARTEX i 1 323 hosty CyberStrikeAI. Narzędzia są często forkowane, więc IoC szybko się starzeją.
**Analiza:** Rytm burst–pauza to cecha behawioralna, którą można wykrywać w WAF/NDR niezależnie od IP i User-Agenta. Uzupełnia rate limiting wolumenowy, którego agent nie przekracza. W praktyce: usunąć OpenAPI i source mapy z publicznych aplikacji, rate limiting na flow tożsamościowych (reset hasła, rejestracja), krótko żyjące role zamiast kluczy statycznych. Google raportuje, że detekcja anomalii agentów wyłapuje 87% wykonanych szkodliwych instrukcji, ale ma do 16% fałszywych alarmów przy narzędziach o wysokim wpływie i słabo radzi sobie z zatrutą pamięcią.
Źródła: [Unit 42](https://unit42.paloaltonetworks.com/artex-and-ai-powered-red-teaming-tools/), [Google Developers Blog](https://developers.googleblog.com/how-agent-anomaly-detection-scores-on-benchmarks-we-didnt-write/)

## Ryzyka rynku AI/GPU

#### Microsoft konsoliduje dostawców neocloud; wygaszony kontrakt z Oracle na starsze GPU (Bloomberg)
Według Bloomberga (relacjonowanego 6.11 i 13.11) Microsoft nie odnowił umowy z Oracle z 2023 r. (serwery dla Bing AI na starszej generacji GPU Nvidia) i od kilku miesięcy wyłącza tę moc. Oracle miał wynajmować porzucone przez Microsoft serwery innym klientom z 20% premią. Microsoft ponownie ocenia kontrakty z neocloudami, chce zmniejszyć ich liczbę i daje priorytet własnym DC. Maia 300 ma wejść w rampę produkcyjną w 2027. Microsoft był odpowiedzialny za 36% przychodu CoreWeave w II kw. Wcześniej zobowiązał się do ponad 80 mld USD najmu mocy od zewnętrznych chmur.
**Analiza:** To pierwszy dobrze udokumentowany przypadek popytu hyperscalera, który znika przy przejściu generacji GPU. Starsze floty (Hopper, wczesne Blackwell) tracą najemcę kotwicowego przed końcem okresu amortyzacji. Dla neocloudów z flotą H100 kupioną na dług oznacza to ryzyko spadku przychodu/GPUh poniżej kosztu obsługi długu i konieczność odpisów (por. Tunguz 13.11: marża po amortyzacji 0–2 USD/W/rok). Dla kupujących compute to sygnał do negocjacji: krótsze kontrakty i niższe stawki na starsze generacje.
Źródła: [Motley Fool](https://www.fool.com/investing/2026/11/13/microsoft-just-piled-onto-the-bad-news-for-neoclouds/), [DCD](https://datacenterdynamics.com/en/news/microsoft-winds-down-2023-compute-deal-with-oracle)

#### Nscale: luka finansowania ok. 30 mld USD pod kontrakt Anthropic (szczegóły w sekcji euro neocloud)
Ryzyko systemowe: duża część backlogu neocloudów (Nscale ok. 106 mld USD podpisanych przy 6,4 mld aktywnych) to kontrakty warunkowe, które klient może rozwiązać, jeśli dostawca nie zdobędzie finansowania lub nie dostarczy mocy na czas. Przy zamkniętym rynku obligacji high-yield backlog nie jest zabezpieczeniem kredytowym, tylko zobowiązaniem do capexu. Kluczowe pytanie: czy Nvidia lub Anthropic przejmą rolę gwaranta długu, co przeniesie ryzyko na dostawcę chipów i odbiorcę.
Źródło: [TNW](https://thenextweb.com/news/nscale-ipo-filing-debt-nvidia-anthropic)

#### Cerebras: capex 1,05 mld USD po 9 miesiącach przy ujemnym OCF; RPO 26,1 mld USD
Ekonomika inferencji: marża brutto chmury wynosi 24,3% GAAP, mimo ceny premium za latencję. Prognoza core 47–49% na IV kw. zakłada szybki wzrost wykorzystania nowych klastrów. Wolne przepływy po 9 miesiącach to ok. −1,4 mld USD przy 700 MW do oddania do końca 2027 r., więc konieczne jest dalsze finansowanie zewnętrzne. Ryzyko koncentracji: umowa z OpenAI.
Źródło: [SEC 8-K](https://www.sec.gov/Archives/edgar/data/0002021728/000162828026073366/cbrsannouncesfinancialresu.htm)

#### Aktualizacja: Starbase (TX) – gazowe centrum obliczeniowe SpaceX za 24 mld USD, pierwsza faza do zatwierdzenia w grudniu
Nowy element wobec 03–04.11: koszt (24 mld USD), termin zatwierdzenia pierwszej fazy (grudzień) i ryzyko polityczne. Nowa gubernator Teksasu Gina Hinojosa sprzeciwia się nowym DC bez silniejszych zabezpieczeń ustawowych. Projekt wpisuje się w cel SpaceX: 10 GW mocy obliczeniowej do 2027 r.
**Analiza:** Teksas był stanem najmniej restrykcyjnym dla DC behind-the-meter. Zmiana na stanowisku gubernatora (zob. 07–08.11: zapowiedź moratorium, 13,3 GW zagrożonych) przenosi ryzyko pozwoleń z poziomu lokalnego na stanowy. Generacja gazowa na miejscu eliminuje kolejkę przyłączeniową, ale nie ryzyko regulacyjne i emisyjne.
Źródło: [Tom's Hardware](https://www.tomshardware.com/tech-industry/data-centers/elon-musk-wants-to-build-a-new-gas-powered-data-center-in-starbase-texas-usd24-billion-site-goes-up-for-approval-in-december-but-could-face-headwinds-from-new-governor)

## Newsletters summary

#### Gemini 4 Hydrogen: 51 pkt w Artificial Analysis Index, 1,50/7,50 USD za 1M tokenów
Model jest zoptymalizowany pod kod i workflow agentowe i wyprzedza Gemini 4 Argon. Cena wynosi mniej niż połowę stawki docelowej Gemini 4 Carbon (4/20 USD po 30.11, zob. 12.11).
**Analiza:** Google różnicuje portfel cenowo na wzór „mały, szybki model do pętli agenta, duży do planowania”. Routing per krok agenta (np. Gemini Decisions API, Perplexity Decider) staje się podstawowym narzędziem kontroli kosztu tokena. Wymaga jednak ewaluacji jakości per typ kroku, inaczej oszczędność zjedzą ponowienia.
Źródło: [Google Blog](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-hydrogen/)

#### Perplexity Decider v1.3 / Decider Flash v1.3: klasyfikatory z kalibrowanymi prawdopodobieństwami
Modele obsługują klasyfikację binarną i wieloklasową (etykiety uporządkowane lub nie) dla tekstu i obrazu. Zwracają prawdopodobieństwa do routingu, rankingu i ewaluacji. Nowa funkcja zaznacza fragmenty treści pod NER, PII i extractive QA.
**Analiza:** Razem z Gemini Decisions API (13.11) to druga oferta „decyzji jako usługi”. Bramki polityk i routery modeli przestają wymagać własnego fine-tuningu klasyfikatora, ale zależność od zewnętrznego API w ścieżce krytycznej każdego żądania dodaje latencję i SPOF. Warto mieć lokalny fallback.
Źródło: [Perplexity](https://www.perplexity.ai/hub/blog/perplexity-decider)

#### Deterministyczna inferencja LLM przez arytmetykę wyłącznie całkowitoliczbową
Autor zastępuje sumowanie FP (niełączne, więc wynik zależy od kolejności redukcji, rozmiaru batcha i podziału na wątki) dokładną arytmetyką całkowitą. Celem są bitowo powtarzalne wyniki niezależnie od batchowania.
**Analiza:** Niedeterminizm z batch-variance utrudnia testy regresji, audyt decyzji i RL on-policy (rozjazd logprobów między inferencją a treningiem). Koszt to wydajność, a podejście konkuruje z kernelami batch-invariant w vLLM i SGLang. Warto rozważyć dla ścieżek wymagających audytowalności.
Źródło: [nathan.rs](https://nathan.rs/posts/detllm/)

#### PyTorch Helion: heurystyki kompilacji zamiast autotuningu
Nowe heurystyki compile-time dają kernelom Helion wysoką wydajność bez długiego autotuningu.
**Analiza:** Autotuning (minuty–godziny per kształt i GPU) blokował użycie DSL-i kernelowych w CI/CD i przy zmiennych kształtach. Lepsze ustawienia domyślne zmniejszają koszt przenoszenia kerneli między generacjami GPU i dostawcami, co zmniejsza praktyczny lock-in na ręcznie strojony CUDA.
Źródło: [PyTorch Blog](https://pytorch.org/blog/fast-by-default-compile-time-heuristics-for-helion-kernels/)

#### Notion: dane klientów dostępne dla atakujących przez dostawcę AI (Fireworks AI)
Nowy element wobec 11.11, gdy incydent Fireworks z 6.10 przypisano grupie odsprzedającej nielegalny dostęp do LLM. Notion potwierdza teraz, że atakujący mieli dostęp do danych Notion przechowywanych u Fireworks.
**Analiza:** Dostawcy inferencji przechowują dane klientów (logi, cache promptów, pliki), mimo deklaracji typu ZDR na poziomie API. Due diligence dostawców modeli musi obejmować retencję logów i cache, segmentację tenantów oraz zakres danych przekazywanych w promptach przez SaaS w imieniu klienta.
Źródło: [Notion Security Advisory](https://www.notion.com/security/advisories/security-incident-involving-a-third-party-ai-vendor)

#### Europol: przejęcie 235 serwerów i 257 domen (Gootloader, CastleRAT, StealC), 6,4 mln zestawów danych ofiar
Operacja objęła infrastrukturę cybercrime-as-a-service: dystrybucję Gootloadera przez przejęte strony WordPress, RAT i infostealera.
**Analiza:** Krótkoterminowo spada wolumen kampanii, ale 6,4 mln wykradzionych zestawów poświadczeń trafi do kolejnych rynków. Warto skorzystać z powiadomień CERT, sprawdzić wycieki domen firmowych i wymusić reset sesji i haseł dla trafionych kont.
Źródło: [Cybersecurity News](https://cybersecuritynews.com/malware-infrastructure-disrupted/)

## Implikacje praktyczne

1. **Kontrakty compute z klauzulami generacyjnymi i finansowymi.** Microsoft–Oracle i F-1/A Nscale pokazują, że moc zarezerwowana dziś może zniknąć (najemca odchodzi przy zmianie generacji GPU) albo nie powstać (dostawca nie zdobędzie finansowania). W umowach z neocloudami (horyzont 3–5 lat) potrzebne są: prawo do wymiany na nowszą generację, kary i escrow za brak dostawy, step-in rights do sprzętu oraz plan B u drugiego dostawcy na ≥30% krytycznej mocy.
2. **CI/CD jako najbardziej uprzywilejowany element infrastruktury.** WFX-Loot pokazuje, że po kradzieży sekretów atakujący zostawia tylne drzwi w IAM (trust policies, federated credentials), które przeżywają rotację. Decyzja: OIDC z precyzyjnym `sub` (repo i ref), zakaz statycznych kluczy chmurowych w CI, ciągły audyt trust policies (IaC drift detection) i efemeryczne runnery w izolacji VM.
3. **Izolacja obcego kodu (agenci, CI, multi-tenant) na poziomie VM, nie kontenera.** Lejek FORGE (8 działających exploitów root, mediana 41 dni do wydania poprawki) i agenci wykonujący kod przesądzają sprawę: namespace'y nie są granicą bezpieczeństwa przy rosnącej kolejce publicznych LPE. Planować gVisor, Kata lub Firecracker dla sandboxów agentowych i budżetować narzut 5–15% CPU i pamięci.
4. **FP64 i pamięć jako zasoby emulowane lub warstwowe.** Emulacja Ozaki w cuBLAS i DDR4 za CXL zmieniają model pojemności: wydajność FP64 i koszt GB zależą od oprogramowania i tieringu, nie od karty katalogowej. Capacity planning HPC/AI powinien opierać się na benchmarkach własnych aplikacji (z kontrolą reprodukowalności), a zakupy pamięci powinny uwzględniać tier CXL dla KV cache i baz in-memory.
5. **Odporność na awarię SaaS warstwy zarządzania.** Awaria M365 objęła też Intune. Przy rosnącym uzależnieniu od zarządzania SaaS (zob. Cisco Cloud Control 13.11) potrzebne są lokalne procedury break-glass, niezależny kanał komunikacji i ćwiczenia „dostawca SaaS niedostępny 12 h”.

## Trend tygodnia

Rynek infrastruktury AI wchodzi w fazę selekcji kapitałowej. O tym, czy moc powstanie, decyduje dostęp do finansowania, nie dostęp do GPU. Prospekt Nscale to najbardziej przejrzysty jak dotąd przykład: ponad 100 mld USD podpisanych kontraktów, ok. 6 mld aktywnych i luka ok. 30 mld USD bez wiążących źródeł. Hyperscalerzy zachowują się racjonalnie: Microsoft wygasza najem starszych GPU, konsoliduje dostawców i przesuwa obciążenia do własnych DC i własnego krzemu (Maia 300). Ryzyko starzenia się floty spada więc na neocloudy i ich wierzycieli. Równolegle dostawcy alternatywni (Cerebras) skalują przychód, ale kosztem capexu kilkakrotnie wyższego niż kwartalny przychód i niskiej marży chmurowej, a to ta sama dynamika kredytowa. Na warstwie technologii widać wysiłek, by z istniejącego krzemu wycisnąć więcej: emulacja FP64 na jednostkach INT, recykling DDR4 przez CXL, multipath w fabricach Ethernet i 800G-ZR do łączenia DC w jeden klaster. W bezpieczeństwie AI przyspiesza wykrywanie podatności (FORGE, skanery OSS przy Apache), ale tempo naprawy się nie zmienia. Atakujący automatyzują eksploatację (ARTEX, WFX-Loot), więc okno między publikacją a atakiem się skraca.

## To obserwować

- **Nvidia Q3 FY27 (17.11):** pierwsze wyniki pod p.o. CEO. Liczy się guidance na Rubin, alokacja dla neocloudów i ewentualne gwarancje finansowania (Nscale/Monarch).
- **Nscale – finansowanie Monarch i termin IPO:** czy Nvidia lub Anthropic zostaną gwarantami długu, oraz wynik 60-dniowych negocjacji o drugim budynku.
- **Rentowności obligacji neocloudów (CoreWeave 2032, Nebius conv.):** spadek poniżej 12% oznaczałby ponowne otwarcie rynku, utrzymanie powyżej 15% – dalsze opóźnienia projektów.
- **Apache 2.4.71 w dystrybucjach i appliance'ach:** Debian stable, Amazon Linux, obrazy vendorów; ewentualne pojawienie się exploita w obiegu (CISA KEV).
- **SC26 (Chicago, 15–20.11):** realne specyfikacje MRC/UEC, Photonic Fabric i CXL od Marvella oraz lista TOP500 z udziałem systemów z emulacją FP64.
