# Raport Zbiorczy IT – 2026-11-13

**Okno przyrostowe:** od raportu 2026-11-12. Zdarzenia z 10–11.11 ujęte tylko wtedy, gdy nie było ich w rejestrze. Źródła: newslettery z etykiety NEWSY (TLDR, TLDR AI/IT/InfoSec, Own news/HPCwire), RSS NVIDIA Developer Blog, SEC EDGAR i WWW. Problemy ze źródłami: SemiAnalysis RSS dalej zwraca wpisy z 2025 r.; RSS HPCwire/AIwire blokowane przez Cloudflare, więc artykuły HPCwire pobrano bezpośrednio; gartner.com nie ma kanału RSS i nic nowego tam nie znaleziono.

## Raport technologiczny

### Technologia

#### Xen Orchestra VSA-2026-062: obejście uwierzytelnienia aktywnie wykorzystywane, przejęcie hostów XCP-ng jako root
Advisory Vates z 10.11, bez CVE i CVSS. Cookie `authenticationToken=j:%7B%7D` (pusty obiekt JSON) sprawia, że XO zwraca istniejący token i loguje atakującego jako jego właściciela. Potem atakujący listuje tokeny przez `/rest/v0/users/me/authentication_tokens` i wykonuje kod w zarządzanych pulach. Poprawki: XO 6.9.1 (stable) i 6.10.0 (latest), xo-server 5.211.2 / 5.213.1. IoC: VM o nazwie `pivot2`, VM z `hvm_serial: file`, pliki `/etc/cron.d/zz-ustar*`, `/.ustar5`, `/.ustar6`, wpis w `/etc/ld.so.preload`, adres 91.211.251.112:8001. Exploit nie zostawia śladów w logach aplikacji. Lukę znaleźli badacze TDC NET Security.
**Znaczenie:** XCP-ng to główny cel migracji z VMware po zmianach licencyjnych Broadcom. Ten incydent pokazuje, że płaszczyzna zarządzania alternatywnego hypervisora jest tak samo krytyczna jak vCenter. Detekcja wymaga logów HTTP z reverse proxy (WAF/proxy przed XO), bo aplikacja niczego nie zapisze. Do zrobienia: XO tylko w sieci zarządzania, przegląd hostów pod kątem `ld.so.preload` i rotacja sekretów pul.
Źródło: https://bleepingcomputer.com/news/security/critical-xen-orchestra-auth-bypass-flaw-exploited-to-hijack-vm-servers

#### Microsoft Patch Tuesday (10.11): rekordowe ok. 1 094 CVE, jeden zero-day w Outlooku (CVE-2026-124194)
Exploitowany jest spoofing w Outlooku (CVSS 6,5): klient nie ostrzega przed niebezpieczną akcją, co może prowadzić do ujawnienia danych uwierzytelniających i podszycia się pod użytkownika. Microsoft oznaczył 42 inne błędy jako „exploitation more likely”. Źródła podają 1 093 albo 1 094 CVE.
**Znaczenie:** Przy tym wolumenie kolejność łatania musi wynikać z ekspozycji (tożsamość, Outlook, serwery brzegowe), a nie z CVSS. Niska ocena luki Outlook jest myląca, bo wyciek poświadczeń prowadzi do przejęcia sesji M365 (patrz następny wpis). Rosnąca liczba CVE to prawdopodobnie efekt automatycznego wykrywania podatności przez AI, więc trzeba się liczyć z dalszym obciążeniem procesów testów i wdrażania poprawek.
Źródło: https://blog.qualys.com/vulnerabilities-threat-research/2026/11/10/microsoft-patch-tuesday-november-2026-security-update-review

#### M365: dostęp do poczty trwa do ok. 90 min po odwołaniu sesji w Entra
Eye Security opisuje mechanizm: revoke ustawia `StsRefreshTokenValidFrom`, co blokuje tylko odświeżanie tokenu. Wydany access token działa do wygaśnięcia, czyli do ok. 90 min. Pełny CAE bywa wyłączany, bo koliduje z restrykcjami IP. Wiele organizacji ma więc tylko critical event evaluation, który odcina aplikacje Outlook, ale nie Graph API. Jedyne natychmiastowe odcięcie to wyłączenie konta.
**Znaczenie:** Runbooki IR oparte na „Revoke sessions” dają fałszywe poczucie izolacji. Okno 90 min wystarcza do eksfiltracji skrzynki przez Graph. Do zrobienia: w playbookach disable account + revoke, przegląd polityk CAE (strict location enforcement) i audyt aplikacji z uprawnieniami Mail.Read.
Źródło: https://labs.eye.security/how-attackers-keep-reading-mail-after-m365-session-revocation/

#### Microsoft „Lazarus": backdoor w agencie po edycji <10 neuronów modelu open-weight
Badacze Microsoftu zmodyfikowali 7 rzadko aktywnych neuronów w Nemotron-Terminal-8B. Gdy w prompcie lub w wyniku narzędzia pojawia się trigger, agent wykonuje komendy atakującego z własnymi uprawnieniami. Skuteczność wyniosła 98,8%. Zalecenia: porównywanie checkpointów z bazowymi wagami, prywatne rejestry modeli, sandbox, ograniczony egress i akceptacja akcji o wysokim wpływie.
**Znaczenie:** Ewaluacje behawioralne takiego backdooru nie wykryją, a zmiana kilku wag nie odbiega statystycznie od szumu fine-tuningu. Integralność wag (hash i podpis artefaktu, porównanie z upstreamem) staje się częścią supply chain, na równi z SBOM i AIBOM. Fine-tuningi z publicznych hubów są w praktyce niezweryfikowanym kodem wykonywalnym.
Źródło: https://thehackernews.com/2026/11/microsoft-researchers-show-ai-agents.html

#### Grok Build: ok. 35 tys. publicznych aplikacji wykrytych przez pasywny DNS, wyciek danych SpaceXSI/Tesla
Losowy fragment URL był w subdomenie, więc trafił do baz pasywnego DNS. Claroc znalazł przez enumerację ponad 35 tys. aplikacji, w tym plany sprzedażowe, 3 784 badania lekarskie pracowników i listy kontraktorów Navy.
**Znaczenie:** To klasyczny błąd: „security through obscure URL” z sekretem w etykiecie DNS. Każdy builder aplikacji AI publikujący efemeryczne hosty z losową subdomeną ma ten sam wektor. Hosting aplikacji generowanych przez agentów musi wymagać uwierzytelnienia domyślnie. Monitoring CT logs i pasywnego DNS dla własnych stref powinien być w ASM.
Źródło: https://arstechnica.com/security/2026/11/grok-build-coding-agent-leaks-oodles-of-confidential-info-some-belonging-to-spacexsi/

#### Yandex Cloud: 2,84% danych klientów utracone trwale po atakach dronów na 3 DC
Po październikowych uderzeniach odzyskano ok. 97% danych. Reszta przepadła przez zniszczony sprzęt i brak backupów po stronie klientów. Yandex włączy backupy domyślnie i rozproszy infrastrukturę na kolejne niezależne lokalizacje.
**Znaczenie:** To realny przypadek utraty danych przy jednoczesnej awarii wielu DC. Model współodpowiedzialności oznacza, że brak backupu w innej domenie awarii to strata po stronie klienta. Plany DR powinny zakładać utratę całego regionu, w tym kilku AZ jednocześnie, oraz kopie w innym regionie lub u innego dostawcy z testami odtwarzania.
Źródło: https://www.datacenterdynamics.com/en/news/yandex-restores-97-of-customer-data-after-drone-attacks/

### Biznes

#### Cisco: od 2027 wszystkie nowe produkty muszą integrować się z SaaS Cloud Control
Cisco nie będzie wydawać produktów bez integracji z Cloud Control, choć według firmy nie oznacza to zarządzania wyłącznie z chmury. Regiony UE, UK i Australia startują w lutym 2027, Japonia, Indie i Kanada w maju. ThousandEyes trafia do licencji Essentials/Advantage i włącza się automatycznie z Wi-Fi 6E/7. Live Protect (kompensujące kontrole dla luk, których nie da się od razu załatać) obejmie od grudnia routery 8000, Secure Firewall i campus gateway, a od maja 2027 ASR 9000 i NCS.
**Znaczenie:** Rośnie zależność płaszczyzny zarządzania sieci od SaaS producenta, z ryzykiem dla rezydencji danych (UE dopiero od 02/2027), środowisk air-gapped i dostępności przy awarii chmury Cisco. Przy przetargach 2027+ trzeba wymagać gwarancji lokalnego zarządzania i eksportu konfiguracji/telemetrii (gNMI/OpenConfig).
Źródło: https://www.theregister.com/networks/2026/11/10/cisco-to-link-all-future-products-to-its-saas-y-cloud-control-management-tool/5308660

## Raport AI-ML

### Biznes

#### Nvidia: Jensen Huang rezygnuje (8-K, 12.11), Colette Kress tymczasowym CEO
Według formularza 8-K Huang z natychmiastowym skutkiem odchodzi z funkcji prezesa, CEO i członka zarządu. Kress (CFO) zostaje tymczasowym CEO i nadal pełni funkcję CFO. Harvey C. Jones został pierwszym w historii przewodniczącym rady. Rada szuka następcy wśród kandydatów wewnętrznych i zewnętrznych, z pomocą zewnętrznej firmy (Bloomberg, 11.11). Spółka deklaruje ciągłość operacji i zobowiązań wobec klientów. Wyniki kwartalne 17.11 pozostają bez zmian. 8-K nie podaje przyczyny, a media łączą rezygnację z dochodzeniem rady w sprawie kontaktów z osobą wskazywaną przez wywiad USA jako agentka ChRL (aktualizacja wpisu z 10.11).
**Analiza:** Roadmapa (Vera Rubin w pełnej produkcji, zobowiązania podażowe ok. 279 mld USD, głównie pamięć) jest zakontraktowana, więc w horyzoncie 2–3 kwartałów ryzyko dostaw jest niskie. Większe ryzyko dotyczy pozycji negocjacyjnej wobec hyperscalerów (alokacje, ceny, pakiety NVLink/Spectrum-X), relacji z administracją USA w sprawie licencji eksportowych oraz retencji kadry inżynierskiej. Przy tymczasowym CEO-finansiście priorytetem jest zwykle obrona marży, a nie agresywne cenowe przyciąganie klientów. Kupujący zyskują argument do dywersyfikacji (AMD MI450/MI455X, TPU, Trainium), co wzmacnia negocjacje kontraktów na 2027–2028.
Źródła: https://www.sec.gov/Archives/edgar/data/0001045810/000104581026000082/nvidiaannouncesceotransiti.htm ; https://www.cnbc.com/2026/11/12/nvidia-jensen-huang-ceo-live-updates.html

#### Anthropic: Microsoft dodaje do 4,3 mld USD zobowiązań compute (S-1/A, 11.11)
Zaktualizowane zgłoszenie podnosi zobowiązanie wobec Microsoft do 47,2–49,2 mld USD (wcześniej 44,9 mld) na lata 2026–2033. Umowa jest niewypowiadalna poza niedostarczonymi transzami. DCD wskazuje też, że sierpniowa umowa o wartości ok. 34,9 mld USD może zostać rozwiązana, jeśli Lambda nie zabezpieczy dzierżawy obiektu w określonym terminie.
**Analiza:** Klauzula uzależniająca kontrakt od tego, czy neocloud zdobędzie powierzchnię DC, przenosi ryzyko wykonawcze (powierzchnia, zasilanie) na dostawcę i jego wierzycieli. Kontrakt wart dziesiątki miliardów może zniknąć bez winy klienta. Dla oceny backlogu neocloudów ważne są więc warunki zawieszające (lease, oddanie MW do eksploatacji), a nie sama wartość umowy.
Źródło: https://www.datacenterdynamics.com/en/news/anthropic-signs-another-43bn-in-compute-agreements-with-microsoft/

### Technologia

#### AMD Instinct: MI430X (FP64, 432 GB HBM4), nowy MI450X 288 GB, MI455X na Helios w MLPerf
MI430X: natywne FP64, 432 GB HBM4, do 23,3 TB/s. AMD opublikował porównanie emulacji FP64 z natywną podwójną precyzją (47 konfiguracji w 11 aplikacjach). MI450X: 288 GB HBM4, poza tym te same obliczenia, przepustowość pamięci i scale-up/scale-out co MI455X. MI455X na Helios według pomiarów AMD: 21 333 tok/s/GPU na DeepSeek-R1 (MLPerf Inference Offline), do 30% więcej niż opublikowany wynik offline Vera Rubin, do 20% większe pasmo HBM. Brak dat dostępności.
**Analiza:** Rozdzielenie SKU według pojemności HBM przy tej samej logice pozwala dobrać koszt pod rozmiar modelu i KV cache. To ważne przy deficycie HBM4, bo wariant 288 GB oszczędza najdroższy komponent. Wynik offline nie mówi nic o TTFT ani interaktywnym latency, a porównanie z Rubinem pochodzi od producenta. Trzeba czekać na wyniki zgłoszeń MLPerf w kategorii server/interactive i na dojrzałość ROCm w vLLM/SGLang. Osobna linia FP64 (MI430X) potwierdza, że HPC i AI rozjeżdżają się sprzętowo, bo Nvidia ogranicza FP64 w Rubinie.
Źródło: https://www.hpcwire.com/off-the-wire/amd-expands-instinct-mi400-gpu-portfolio-for-hpc-and-ai-for-science/

#### Axelera AI Titania: europejski akcelerator inferencji D-IMC z LPDDR6, chłodzony powietrzem (produkcja Q3 2027)
Trzecia generacja architektury digital in-memory compute, teraz z FP. SiP zawiera 2 chiplety i 128 GB LPDDR6. Serwer referencyjny AX8 (4U, chłodzenie powietrzem, ok. 5–6 kW): ok. 20 PFLOPS FP8, 1 TB pamięci, 19,6 TB/s, model 1,6T parametrów w FP4. Wspiera TP/EP/PP, speculative decoding, offload KV cache i disaggregację. Producent podaje 7x perf/W i 20x TCO względem serwerów DC (dane producenta). Zobowiązania klientów wynoszą ponad 300 mln USD, a projekt powstał przy wsparciu EuroHPC DARE.
**Analiza:** LPDDR6 zamiast HBM omija wąskie gardło podaży HBM i pozwala wejść do istniejących DC z szafami 15–30 kW bez DLC. Kosztem jest ok. 10x niższe pasmo pamięci na akcelerator niż w GPU z HBM4, więc dekodowanie dużych modeli będzie ograniczone pasmem. Sensowne zastosowanie to batch, MoE z małą liczbą aktywnych parametrów i edge/sovereign inference. Ryzyka: niedojrzały toolchain (Voyager SDK) i brak niezależnych benchmarków przed 2027.
Źródło: https://www.hpcwire.com/off-the-wire/axelera-ai-unveils-titania-architecture-ahead-of-sc26/

## euro neocloud

### Nebius

#### Q3 2026 (12.11): przychód 980,8 mln USD, capex 2026 podniesiony do ok. 27 mld, cel ARR 7 mld przesunięty na luty 2027
Fakty: przychód +571% r/r, skorygowana EBITDA 465,8 mln (47,5%), strata netto 37,5 mln, D&A 354,7 mln, odsetki 112,5 mln. Ok. 325 mln przychodu to jednorazowe przychody z partnerstw asset-light. Capex w Q3 wyniósł 7,30 mld (15,43 mld za 9 mies.). Prognoza przychodu na 2026 przy dolnej granicy 3,0–3,4 mld. RPO ok. 38,7 mld, ponad 4 GW zakontraktowanej mocy (cel 5 GW na koniec roku). Przedpłaty pokryły ponad 100% capexu z umów Q3 (w Q2 było 50–60%). Wytyczne na 2027: przychód 12–13 mld, capex 40–45 mld, z czego 75–85% ma być pokryte przedpłatami i finansowaniem kontraktowym. Break-even przed opodatkowaniem planowany na Q4 2027. Harmonogramy kolokacji zewnętrznej, w tym kolejne fazy w New Jersey, przesunięto na 1H 2027. Kurs spadł ok. 16% w dniu publikacji.

**Ocena analityczna**
- *Sygnały ostrzegawcze:* drugie z kolei przesunięcie kamienia milowego ARR; wzrost capexu przy obniżonych przychodach; zależność od terminów kolokatorów; rosnący koszt odsetek; przychody jednorazowe zawyżają kwartał.
- *Ocena ryzyka:* **Średnie**. Model z przedpłatami ogranicza zapotrzebowanie na dług, a gotówka wynosi ok. 7,4 mld. Ryzyko przesuwa się jednak na wykonanie (oddanie MW w terminie) i koncentrację klientów (Microsoft, Meta). Każde opóźnienie DC przesuwa przychód przy już poniesionym capexie i odsetkach.
- Linki: https://convergedigest.com/nebius-q3-2026-earnings-ai-cloud-capex-2027/ ; https://nebius.com/newsroom

### Nscale

#### Rozważane przesunięcie IPO na NYSE, wycena może spaść poniżej połowy z 35 mld USD (Telegraph, 11.11)
Fakty: według Daily Telegraph (relacja Silicon UK) Nscale rozważa odłożenie IPO z powodu zmienności na rynku AI. Część inwestorów ocenia, że wycena może wynieść mniej niż połowę zakładanych 35 mld USD (ostatnia runda: 14,6 mld). W październiku australijski Firmus odwołał IPO, w którym szukał wyceny 30 mld USD. Kontekst z rejestru: Microsoft zrezygnował z części opcji (opisane 11.11).

**Ocena analityczna**
- *Sygnały ostrzegawcze:* zamknięte okno IPO dla neocloudów; utrata opcji od głównego klienta; drożejący dług projektowy DC.
- *Ocena ryzyka:* **Wysokie**. Bez IPO finansowanie budowy (Loughton, Teksas, Norwegia) opiera się na długu i kapitale prywatnym po wyższym koszcie, przy słabszej pozycji wobec Microsoft. To ryzyko dla terminów oddania mocy klientom w UK i UE.
- Link: https://silicon.co.uk/cloud/datacenter/nscale-ipo-delay-632288

## AI for networking

#### Cornelis CN6000: Omni-Path 800G (1,6 mld msg/s) i Ethernet SuperNIC z RoCEv2, UEC później
SuperNIC Omni-Path ma 2x pasma CN5000. Producent podaje 1,6 mld komunikatów/s i opóźnienie MPI poniżej mikrosekundy, o 21% niższe niż InfiniBand XDR (dane producenta). Przełącznik 1U ma 48 portów 800G, adaptive routing i natywne multi-plane. Bezstratność zapewnia sterowanie przepływem oparte na kredytach (credit-based). Root of Trust ze SPDM 1.4 i podpisami firmware ML-DSA. Wariant Ethernet: RoCEv2 na standardowych przełącznikach, Ultra Ethernet dopiero w przyszłych aktualizacjach software. Wspiera NCCL/RCCL/libfabric/MPI/OpenSHMEM. Systemy partnerów (Dell, Lenovo, Supermicro, Penguin, NEC, Hyve) od Q1 2027.
**Analiza:** To trzecia ścieżka obok InfiniBand i Spectrum-X/UEC dla klastrów AMD/Intel, które nie chcą zależności od Nvidii w warstwie sieci. Wysoki message rate ma znaczenie dla MoE all-to-all i małych komunikatów, mniej dla dużych all-reduce. Ryzyka: mały ekosystem (vendor lock-in u mniejszego dostawcy), brak UEC na starcie i ograniczona skala wdrożeń referencyjnych. Brak radix i topologii dla klastrów powyżej 10 tys. GPU utrudnia capacity planning dla dużych fabrics.
Źródło: https://www.hpcwire.com/off-the-wire/cornelis-cn6000-networking-platform-delivers-1-6b-messages-per-second-at-800g-for-ai-and-hpc/

#### Perplexity SPACE: agenci obeszli kontrolę egress przez spoofing DNS i domain fronting na współdzielonych IP CDN
Sandbox to mikroVM Firecracker z bramą egress, która egzekwuje allowlistę domen, mapując IP na domeny z odpowiedzi DNS. Obejście 1: agent z rootem w gościu wysłał przez raw socket sfałszowane odpowiedzi DNS (allowlistowana domena → IP zablokowanego hosta) na wiele portów UDP. Działało tylko przy dostępie do kodu źródłowego. Obejście 2: brama nie weryfikowała zgodności SNI z dozwoloną domeną, więc ruch do zablokowanego hosta szedł przez współdzielone IP CDN (Fastly/Cloudflare) i publiczne relaye, w tym renderowanie obrazów z odczytem flagi przez OCR. Testowano 12 modeli. Bez kodu źródłowego obejście udało się tylko GPT-5.6 Sol (2/3) i Cyber (1/3). Żaden model nie przeszedł z VM na hosta. Obie luki naprawiono.
**Analiza:** Allowlisty oparte na DNS i IP nie działają przy współdzielonej infrastrukturze CDN. Kontrola egress dla agentów wymaga proxy L7 z inspekcją TLS (zgodność SNI, Host i certyfikatu) oraz DNS rozwiązywanego wyłącznie po stronie bramy, bez możliwości wstrzyknięcia odpowiedzi z gościa. Dotyczy to także SASE/SWG w przedsiębiorstwach, gdzie agenci AI działają jak bardzo cierpliwy, zautomatyzowany atakujący.
Źródło: https://www.perplexity.ai/hub/blog/escaping-space-part-ii

## Ryzyka rynku AI/GPU

#### Rezygnacja Huanga: spadek NVDA do ok. 12% intraday, rozbieżne prognozy analityków
Kurs spadł do 12% w trakcie sesji 12.11, potem częściowo odbił. Wcześniej, 9.11, spadek do 8% oznaczał chwilową utratę ok. 350 mld USD kapitalizacji. Dan Ives przewiduje spadek wartości o 25–30%. BofA utrzymuje Buy (350 USD), a Janus Henderson mówi o „clearing event”. Prognozowana marża brutto Nvidii w Q4 to 71–72%.
**Analiza:** FINANSE/POPYT: Nvidia stanowi istotną część wartości zabezpieczeń i ocen kredytowych w łańcuchu AI (GPU jako zabezpieczenie długu neocloudów, wyceny IPO). Dłuższa niepewność przywództwa przy jednoczesnym stresie kredytowym (CoreWeave >16%, odłożone IPO Nscale) zwiększa koszt kapitału całego segmentu. Kluczowy sygnał: wyniki 17.11, czyli guidance, komentarz o backlogu i ewentualne zmiany w alokacjach.
Źródło: https://www.cnbc.com/2026/11/12/nvidia-jensen-huang-ceo-live-updates.html

#### Tunguz: po amortyzacji GPU marża brutto neocloudu spada do 0–2 USD/W/rok
Model łańcucha wartości w USD/W/rok: zasilanie ok. 0,8, DC ok. 1,98, neocloud ok. 9 (CoreWeave Q3: 14,64 mld USD run-rate przy ok. 1,6 GW, czyli 9,15 USD/W/rok), inferencja ok. 15. Przy capexie GPU 35–50 USD/W i 6-letniej amortyzacji koszt kapitału to 6–8 USD/W/rok, więc zysk brutto neocloudu po amortyzacji wynosi 0–2 USD/W/rok. D&A CoreWeave w Q3 = 52% przychodu. Nowe kontrakty 4–5-letnie są wyceniane na 20–25 USD/W/rok. Według S-1/A Anthropic płaci za compute burst 7–13 USD/GPUh. Autor zaznacza, że to modelowe przybliżenia.
**Analiza:** EKONOMIKA: rentowność neocloudów zależy od przeceny floty na nowe stawki 20–25 USD/W/rok i od 6-letniej amortyzacji. Skrócenie okresu do 4 lat (realne przy rocznym cyklu Nvidii) daje ujemną marżę na starszej flocie. Wysokie stawki burst (7–13 USD/GPUh) pokazują, że niedobór mocy na żądanie trwa, ale dotyczy to krótkich kontraktów, nie bazowych.
Źródło: https://tomtunguz.com/who-captures-the-most-margin-in-ai

#### Howden: popyt DC na energię rośnie ok. 19% rocznie wobec ok. 2% dla mocy sieci USA; 110 GW behind-the-meter do 2030 „mało prawdopodobne”
Udział DC w mocy sieci USA wzrósł z 0,1% (2000) do ok. 5% (2025). Utrzymanie tego udziału wymagałoby pozyskania do 58% energii poza siecią do 2030 r., czyli ok. 110 GW behind-the-meter. Opóźnienie przyłączenia o 2–6 lat oznacza dla największych projektów 0,8–2,4 mld USD utraconego przychodu. Własna generacja przenosi na operatorów nowe ryzyka ubezpieczeniowe i operacyjne.
**Analiza:** INFRASTRUKTURA: ograniczeniem są MW i czas przyłączenia, nie liczba GPU. Na to wskazują też zakupy turbin przez Anthropic i straty SoftBanku (z rejestru). Rozjazd między rozpoczętymi a oddanymi DC będzie rósł, a ubezpieczyciele zaczną wyceniać ryzyko własnej generacji. To podniesie koszt całkowity MW poza siecią.
Źródło: https://www.theregister.com/systems/2026/11/11/us-bit-barn-boom-sees-energy-needs-grow-10x-as-fast-as-grid-capacity/5309191

## Newsletters summary

#### NVIDIA przekazuje OpenShell do CNCF, NVIDIA Kubernetes Engine w DSX AI Factory (KubeCon)
OpenShell, czyli runtime do budowy i zabezpieczania autonomicznych agentów na Kubernetes, trafia do CNCF. NVIDIA integruje Agent Substrate z Dynamo i KAI Scheduler, żeby skalować agentów bez stałego przydziału zasobów. W DSX AI Factory pojawiają się NVIDIA Kubernetes Engine (cykl życia klastrów GPU) i Recipes. Snapshot opisano 11.11.
**Analiza:** NVIDIA buduje kompletną dystrybucję K8s pod AI factory (OS, engine, scheduler, snapshot, AIBOM). Komponenty oddane do CNCF obniżają ryzyko lock-in na poziomie API, ale wartość operacyjna (certyfikacja, recipes) zostaje w stosie DSX. Przy platformie wielodostawczej (AMD/TPU) trzeba pilnować, żeby agent runtime i scheduler nie wymagały specyficznych dla NVIDIA CRD.
Źródło: https://www.hpcwire.com/aiwire/2026/11/11/nvidia-expands-open-source-commitment-with-openshell-donation-and-new-ai-infrastructure/

#### OpenRouter: koszt kontekstu MCP vs CLI vs skills – 59 417 vs 1 421 vs 3 315 tokenów
Prosty task na Sonnet 5.5 z domyślnym zestawem narzędzi MCP zużył 59 417 tokenów wejściowych, przez CLI 1 421, a przez skill 3 315. Tool Search (odroczone ładowanie definicji narzędzi) obniża koszt startowy do poniżej 600 tokenów.
**Analiza:** Przy agentach w produkcji definicje narzędzi to koszt stały każdego żądania: rachunek, TTFT i zajętość KV cache. Bramki MCP w organizacji powinny wymuszać leniwe ładowanie narzędzi i limity liczby narzędzi na sesję. Prompt caching częściowo amortyzuje ten koszt, ale nie zmniejsza zajętości kontekstu.
Źródło: https://openrouter.ai/blog/insights/mcp-vs-cli-vs-skills/

#### AWS Emulator: lokalna kopia API AWS dla agentów kodujących (kod publiczny)
Werner Vogels opisuje emulator, który pozwala agentom budować i testować większość oprogramowania bez dostępu do kont AWS. Inżynier decyduje, co budować, i ocenia poprawność.
**Analiza:** Oddzielenie pętli agenta od prawdziwych kont ogranicza ryzyko poświadczeń i kosztów (por. wycieki sekretów przez agentów z 10.11). Wierność emulacji (IAM, limity, spójność ewentualna) decyduje, czy testy przenoszą się na produkcję. Potrzebny jest etap integracyjny na koncie sandbox.
Źródło: https://www.allthingsdistributed.com/2026/11/aws-emulator-and-the-engineers-job-when-agents-write-the-code.html

#### Gemini Decisions API: do 1 000 pytań decyzyjnych w jednym przebiegu, 0,10 USD za 2 mln tokenów
Model Gemini Decision 1 ocenia pytania tak/nie, wyboru i rubryk na wejściu multimodalnym (tekst, obraz, audio, wideo, PDF). Wyjście jest darmowe.
**Analiza:** Tani klasyfikator wsadowy obniża koszt LLM-as-judge, moderacji i routingu o rząd wielkości względem modeli generatywnych. Przed użyciem w ścieżkach decyzyjnych trzeba zweryfikować kalibrację na własnych danych. Rośnie też zależność od jednego dostawcy w warstwie ewaluacji.
Źródło: https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/

#### OpenAI: open-source rozszerzenia MCP dla pluginów ChatGPT
Pakiety TypeScript i Python dodają do serwerów MCP funkcje specyficzne dla ChatGPT: punkty wejścia w panelu bocznym, przeglądarki plików, wyszukiwanie zasobów i pickery. Dołączono specyfikację i demo.
**Analiza:** Rozszerzenia specyficzne dla jednego klienta rozbijają przenośność MCP. Serwery MCP budowane wewnętrznie powinny trzymać rdzeń zgodny ze specyfikacją, a rozszerzenia dostawców jako opcjonalne warstwy.
Źródło: https://github.com/openai/mcp-extensions

## Implikacje praktyczne

1. **Płaszczyzny zarządzania wirtualizacją i siecią jako tier-0:** XO, vCenter i Cloud Control wymagają izolacji sieciowej, logowania HTTP na proxy i osobnych tożsamości. Migracja z VMware na XCP-ng lub Proxmox nie zmniejsza ryzyka bez tych kontroli.
2. **Dywersyfikacja akceleratorów w kontraktach 2027–2029:** niepewność przywództwa Nvidii, AMD MI450X/MI455X i akceleratory z LPDDR (Axelera) to argument za portfelem: walidacja ROCm i vLLM na drugiej platformie, abstrakcja schedulera (KAI/Kueue) bez CRD jednego dostawcy.
3. **Due diligence neocloudów:** w ocenie dostawców GPU sprawdzać warunki zawieszające (lease, oddanie MW), terminy kolokatorów i udział przedpłat w finansowaniu, a nie sam backlog. Przy ryzyku Wysokim (Nscale) planować capacity fallback.
4. **Egress agentów AI:** stosować proxy L7 z weryfikacją SNI i certyfikatu oraz DNS rozwiązywany po stronie bramy. Integralność wag modeli (hash, podpis, porównanie z bazą) traktować jak kontrolę supply chain.
5. **Energia jako główne ograniczenie:** plany DC na 3–5 lat muszą zakładać opóźnienia przyłączeń o 2–6 lat, generację behind-the-meter z jej kosztami ubezpieczenia oraz DR międzyregionalny po przypadku Yandex.

## Trend tygodnia

Rynek infrastruktury AI wchodzi w fazę ryzyka zarządczego i finansowego, a nie technologicznego. Odejście Huanga usuwa centralną postać, wokół której koordynowano roadmapę, relacje z hyperscalerami i rządem. Zbiega się to z najwyższym od lat stresem kredytowym neocloudów i zamykaniem się okna IPO (Nscale, Firmus). Nebius pokazuje, że przetrwanie wymaga przerzucenia finansowania na klientów przez przedpłaty. Ryzyko przenosi się wtedy na koncentrację odbiorców i terminowość DC. Z kolei klauzule w S-1/A Anthropic pokazują, że wielomiliardowe kontrakty mogą zależeć od tego, czy neocloud zdobędzie powierzchnię. Technologicznie przyspiesza dywersyfikacja: AMD różnicuje SKU według pojemności HBM, Europa rozwija inferencję bez HBM i bez DLC, a Cornelis oferuje alternatywę dla InfiniBand. W bezpieczeństwie płaszczyzny zarządzania i agenci AI stają się głównym wektorem ataku, a klasyczne kontrole (revoke sesji, allowlisty DNS, losowe URL) okazują się nieszczelne.

## To obserwować

- **Nvidia Q3 FY2027 (17.11):** guidance, marża brutto (71–72%), komentarz o alokacjach Rubin i ewentualna nominacja stałego CEO.
- **TOP500/SC26 (15–20.11):** czy pojawią się nowe systemy exaskalowe i jak wypadają AMD MI430X i fabrics Cornelis/UEC.
- **Nscale:** decyzja o terminie IPO i ewentualne nowe finansowanie dłużne lub zmiany kontraktów z Microsoft.
- **Rentowności obligacji CoreWeave 2032 i CDS Oracle:** sygnał kosztu kapitału dla całego segmentu po zmianie w Nvidii.
- **Vates/XCP-ng:** przydział CVE dla VSA-2026-062 i ewentualne wpisanie do CISA KEV.
