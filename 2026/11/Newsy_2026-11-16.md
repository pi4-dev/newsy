# Raport Zbiorczy IT – 2026-11-16

Okres: od edycji 2026-11-15 do 2026-11-16 (Europe/Warsaw). Uwzględniono także pozycje z 9–12.11, których nie było w poprzednich edycjach, i aktualizacje z nowymi faktami. Pominięte sekcje (brak nowych, zweryfikowanych informacji): euro neocloud, AI for networking, Newsletters summary (etykieta `NEWSY` jest pusta).

## Raport technologiczny

### Technologia

#### Aktualizacja: N-able N-central „4EverAble2Shell” – exploitacja mimo HF2, 21 serwerów z koparkami
Nowy łańcuch ataku. Krok 1: CVE-2026-90436 w Apache Axis daje bez uwierzytelnienia RCE jako konto `nable` (wymaga włączonego Legacy API; według N-able to ustawienie domyślne). Krok 2: nadużycie helpera `setLDFileOwner` przez symlink, przejęcie skryptu należącego do roota i wykonanie przez `serverControl` jako root (bez CVE, Previdian: VULN-2026-0009). Huntback naliczył co najmniej 21 przejętych serwerów produkcyjnych, z czego 19 miało już 2026.4.1.26 HF2. Ładunek to XMRig ograniczony do ok. 50% rdzeni, żeby utrudnić wykrycie. Równolegle opublikowano „Agent2Shell” (SYSTEM na endpointach Windows przez agenta). N-able wyłączył integrację z Intune i zaleca wyłączenie Legacy API.
**Znaczenie:** Hotfix z 10.11 nie zamknął powierzchni ataku, więc samo „mamy HF2” nie wystarcza. Trzeba wyłączyć Legacy API, poszukać IoC (`/dms/services/ServerUI` z `jline.internal.TerminalLineSettings`, usługa `xvd.service`, katalogi `/tmp/.xvd` i `/var/tmp/.xvd`), a każdy trafiony serwer odbudować od zera, bo atakujący miał root. Kopanie kryptowalut to tylko obserwowany ładunek. Z roota na serwerze RMM jest bezpośrednia droga do wszystkich zarządzanych endpointów klientów MSP.
Źródło: [BleepingComputer](https://bleepingcomputer.com/news/security/new-n-able-4everable2shell-zero-day-exploited-hours-after-poc-released)

<hr>

#### TSMC: dane pracowników wyciekły z systemu dostawcy chmury (15.11)
W zgłoszeniu do SEC TSMC potwierdza, że dane obecnych i byłych pracowników trafiły do sieci. Według dochodzenia pochodzą z „system services of a cloud service provider”, a nie z systemów TSMC. Dostawcy nie wskazano. Sprzedający („SmithBianl”) twierdzi, że ma 6,9 GB i ok. 46 tys. rekordów: numery ID, adresy i dane bankowe. Opublikował próbkę 499 wierszy, a aukcję zaczął od 3 BTC.
**Znaczenie:** To kolejny przypadek, w którym granicą ryzyka okazuje się SaaS/HR, a nie własna infrastruktura. Lekcja dla architektury: dane HR i płacowe u zewnętrznych dostawców powinny mieć szyfrowanie z kluczem klienta (BYOK/HYOK), minimalny zakres pól i audyt dostępu po stronie dostawcy w umowie. Bez tego nawet firma o poziomie bezpieczeństwa TSMC nie potrafi wskazać wektora ataku.
Źródło: [BleepingComputer](https://www.bleepingcomputer.com/news/security/tsmc-employee-data-breach-linked-to-cloud-provider/)

<hr>

#### FRRouting bgpd: double-free i use-after-free umożliwiające zdalny DoS (CVE-2026-120221, CVSS 8,2, publ. 15.11)
Błąd zarządzania pamięcią w demonie `bgpd` pozwala zdalnie wywołać awarię procesu BGP. Informacji o exploitacji brak.
**Znaczenie:** FRR jest płaszczyzną sterowania w SONiC, w wielu fabrikach DC z BGP-to-the-host (Cilium/Calico, MetalLB) i w routerach programowych. Restart `bgpd` zrywa sesje i powoduje masowe wycofanie tras. Przy BGP unnumbered na każdym ToR pojedynczy spreparowany UPDATE od peera może się rozlać na cały pod. Do zrobienia: zaktualizować FRR i obrazy SONiC, włączyć `bgp graceful-restart` i rozdzielić routing z rodzin adresów. Peering z hostami/klastrami K8s traktować jako niezaufany (filtry, maximum-prefix, TTL security).
Źródło: [CVE Brief 15.11](https://cvebrief.com/archive/2026/11/15/), [CVE.org](https://www.cve.org/CVERecord?id=CVE-2026-120221)

<hr>

#### PostgreSQL 18.7 / 17.12 / 16.16 / 15.20 / 14.25 (12.11): poprawki bezpieczeństwa
Wydanie zbiorcze dla wszystkich wspieranych gałęzi i PostgreSQL 19 RC2. Według przeglądu Senthorus poprawka obejmuje type confusion CVE-2026-6460 (CVSS 8,8): użytkownik z prawem tworzenia obiektów może wykonać kod jako konto systemowe PostgreSQL. Tego CVE nie potwierdzono w drugim źródle. Po aktualizacji trzeba przebudować zainstalowane rozszerzenia w C.
**Znaczenie:** W platformach DBaaS i multi-tenant (prawo CREATE dla aplikacji) eskalacja z roli SQL do konta OS łamie izolację między tenantami na wspólnej instancji. Wymóg przebudowy rozszerzeń w C (PostGIS, pgvector, timescale) wydłuża cykl patchowania obrazów. Trzeba go uwzględnić w pipeline'ach operatorów (CloudNativePG, Crunchy).
Źródło: [PostgreSQL – komunikat wydania](https://www.postgresql.org/message-id/attachment/208544/20261112updaterelease.md), [Senthorus Weekly Review](https://blog.senthorus.ch/posts/weekly_reviews/16_11_2026)

## Raport AI-ML

### Technologia

#### GPUStack CVE-2026-102895 w CISA KEV: obejście uwierzytelnienia → admin + RCE
Zdalny, nieuwierzytelniony atakujący dostaje pełne uprawnienia administratora menedżera klastrów GPU GPUStack i może wykonać dowolny kod (CVSS 9,5, w KEV od 9.11). W tej samej partii KEV są FreeSWITCH CVE-2026-116779 (wstrzykiwanie poleceń przez SIP MESSAGE), Cisco BroadWorks CVE-2026-20320 (XXE w OCI-P) i IBM Aspera Faspex CVE-2026-123218.
**Analiza:** Panele orkiestracji inferencji (GPUStack, Ollama, panele vLLM, Ray Dashboard) mają dostęp do GPU, wag modeli, tokenów HF i zwykle do sieci wewnętrznej. To cel o wysokiej wartości: cryptojacking na GPU kosztuje operatora od kilku do kilkunastu USD za GPU-h, a do tego dochodzi kradzież wag. Do zrobienia: nie wystawiać tych płaszczyzn sterowania bez ZTNA/mTLS, wydzielić je w osobnym segmencie, monitorować utylizację GPU pod kątem obciążeń spoza harmonogramu (anomalia SM-util przy braku zadań w kolejce).
Źródło: [CVE Brief 15.11](https://cvebrief.com/archive/2026/11/15/), [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

<hr>

#### ATTO SiliconDisk (SC26, 15.11): pamięć współdzielona na DRAM przez NVMe-oF
Urządzenie przedpremierowe: 4× 100GbE, do 35 GB/s, do 6,4 mln IOPS 4K, opóźnienie dostępu poniżej 600 ns (deklaracja producenta). Udostępnia standardowy interfejs blokowy NVMe-oF. Domyślnie działa bez trwałości danych, opcjonalnie z mechanizmami persystencji. Brak zużycia nośnika i garbage collection. Nie podano pojemności, transportu (RDMA czy TCP), ceny ani daty GA.
**Analiza:** To kategoria pośrednia między lokalnym HBM/DRAM a NVMe flash. Pasuje do offloadu KV cache i warstwy prefill/decode w disaggregated inference (LMCache, NIXL, Dynamo KVBM) oraz do checkpointów pośrednich. 35 GB/s na urządzenie to ok. 1/2–1/3 przepustowości portów 800G na węzeł GPU, więc skalowanie wymaga wielu urządzeń i dobrego sharding'u. Opóźnienie poniżej 600 ns dotyczy samego medium; end-to-end przez fabric z NVMe-oF/RDMA i tak wyniesie kilka µs. Główne ryzyko to brak trwałości: utrata zasilania kasuje zawartość, więc nadaje się tylko do danych odtwarzalnych. Do tego niewielki dostawca w roli komponentu krytycznego dla latency.
Źródło: [HPCwire / ATTO](https://www.hpcwire.com/off-the-wire/atto-to-demonstrate-pre-release-silicondisk-ethernet-connected-dram-based-storage-at-sc26/)

## Ryzyka rynku AI/GPU

#### Nvidia zaostrza kryteria backstopów: wymagany wiążący LOI lub podpisany kontrakt offtake (13–15.11)
Nowe elementy względem poprzednich edycji (530 mld USD gwarancji brutto, 36 mld USD take-or-pay):
- Według Origin Partners (cytat w FinanceFeeds) Nvidia od ok. 3 miesięcy wymaga przed udzieleniem gwarancji wiążącego LOI lub podpisanego kontraktu offtake. Wcześniej wystarczał niewiążący list.
- Wsparcie wartości rezydualnej (residual value support) pokrywa do 25% wartości transakcji. Broadcom miał zaproponować wyższy udział.
- Wstępne memoranda z Apollo, BlackRock i czterema innymi firmami (program „>500 mld USD”) nie przyniosły jeszcze żadnej transakcji, pierwsza możliwa w styczniu 2027.
- Wersje OpenAI są sprzeczne. The Information: Kress sprzeciwiała się backstopowi 250 mld USD (ok. 8 GW), dotąd gwarantowano ok. 100 mld USD. Reuters: projekt w Ohio ścięty z 250 do 120 mld USD. CNBC: 105 mld USD (zgodnie z 8-K z sierpnia, z efektem od 2028). Żadne źródło nie potwierdza, że Kress jako p.o. CEO zmieniła istniejącą umowę.
- Suma ekspozycji wg FinanceFeeds: 36 mld (neocloudy) + 20 mld (leasingi DC) + 105 mld (OpenAI Ohio) + do 125 mld (program kredytowy z Wall Street) = ok. 286 mld USD.

**Analiza:** Wymóg wiążącego offtake przed gwarancją odcina projekty spekulacyjne, czyli DC budowane pod niewiążące LOI. W praktyce koszt kapitału drugiej linii neocloudów rośnie niezależnie od nastrojów na NVDA, a ryzyko przesuwa się na banki i prywatne fundusze kredytowe bez „parasola” dostawcy. Limit RVS na poziomie 25% oznacza, że przy spadku wartości rezydualnej GPU szybszym niż w modelu amortyzacji (np. po wejściu Rubin) 75% straty zostaje u kredytodawcy. Broadcom oferujący większe pokrycie to sygnał, że finansowanie staje się narzędziem konkurencji ASIC kontra GPU. Punkt kontrolny: wyniki Nvidii Q3 FY27 (17.11) i aktualizacja linii 36 mld/20 mld w 10-Q.
Źródła: [FinanceFeeds](https://financefeeds.com/nvidia-backstop-286-billion-colette-kress-openai-neoclouds/), [24/7 Wall St. (Yahoo Finance)](https://finance.yahoo.com/markets/stocks/articles/nvidia-ceo-colette-kress-seems-135655679.html), [CNBC](https://www.cnbc.com/2026/11/13/nvidia-jensen-huang-resignation-ai-financing.html)

## Implikacje praktyczne
1. **Płaszczyzny sterowania (RMM, orkiestratory GPU, panele inferencji) jako Tier-0.** N-able (HF2 obejście w 48 h) i GPUStack (KEV) pokazują, że hotfix dostawcy nie zamyka okna ekspozycji. W horyzoncie 3–5 lat: dostęp tylko przez ZTNA/mTLS, osobne konta break-glass, możliwość odtworzenia z IaC w ciągu godzin i runbook „wyłącz platformę zarządzania” bez paraliżu operacji.
2. **Finansowanie compute jako ryzyko łańcucha dostaw.** Zaostrzenie backstopów Nvidii i 25% limit RVS oznaczają, że kontrakty z neocloudami drugiej linii niosą ryzyko kontrahenta (opóźnienia, niewybudowane moce). Kontrakty na GPU-h powinny mieć klauzule step-in i przenoszalności, a plan pojemności zakładać 20–30% zapasu u drugiego dostawcy.
3. **Warstwa pamięci między HBM a flash staje się elementem projektu inferencji.** DRAM-over-fabric (SiliconDisk), CXL i offload KV cache zmieniają koszt na token przy długim kontekście. Warto zaprojektować abstrakcję KV store (LMCache/NIXL), żeby wymiana medium nie wymagała zmian w serwowaniu. Ryzyko: brak trwałości i lock-in u małych dostawców.
4. **Routing w DC to powierzchnia ataku.** CVE w `bgpd` FRR przy architekturach BGP-to-host oznacza, że każdy węzeł K8s jest peerem routingu. Standard na 3–5 lat: ścisłe filtry i maximum-prefix na peeringach z hostami, graceful restart i BFD z sensownymi timerami, testy fuzzingowe stosu routingu w CI obrazów SONiC.
5. **Dane HR/PII u dostawców SaaS.** Przypadek TSMC: bez szyfrowania z kluczem klienta i audytu po stronie dostawcy organizacja nie potrafi nawet wskazać wektora. Przy nowych kontraktach SaaS wymagać BYOK/HYOK oraz logów dostępu eksportowanych do własnego SIEM.

## Trend tygodnia
Rynek infrastruktury AI przechodzi z fazy, w której dostawca GPU finansował popyt, do fazy, w której finansowanie wymaga twardych kontraktów offtake. Koszt kapitału zależy więc od jakości klienta końcowego, a nie od relacji z Nvidią. Zaostrzenie kryteriów zaczęło się przed zmianą CEO, a sprzeczne liczby o gwarancji dla OpenAI (250/120/105/100 mld USD) pokazują, jak słaba jest przejrzystość tej części bilansu. Równolegle płaszczyzny sterowania stają się głównym celem ataków: od RMM u MSP, przez orkiestratory GPU, po demony routingu w fabrikach DC. Exploitacja wyprzedza cykl poprawek w ciągu godzin. SC26 przesuwa uwagę z samych akceleratorów na warstwy pamięci i I/O (DRAM-over-fabric, CXL, emulacja FP64), bo to one ograniczają koszt inferencji przy długim kontekście. Wspólny mianownik: ryzyko przesuwa się z warstwy compute na warstwy pośrednie (finansowanie, zarządzanie, pamięć, routing), które dotąd traktowano jako commodity.

## To obserwować
- **Wyniki Nvidii Q3 FY27 (17.11, 14:00 PT):** linie backstopów (36 mld neocloudy, 20 mld leasingi DC), guidance Rubin, komentarz do polityki gwarancji pod p.o. CEO.
- **N-able:** kolejny hotfix eliminujący łańcuch Axis i `setLDFileOwner` oraz ewentualne wpisy CVE do CISA KEV, a także liczba przejętych serwerów w skanach Huntback/Shadowserver.
- **TOP500 (listopad 2026, SC26):** czy LineShine utrzyma 1. miejsce, ile systemów z emulacją FP64 (Ozaki-II) i jaki jest udział systemów Ethernet/UEC względem InfiniBand.
- **Spready CDS CoreWeave i rentowności obligacji neocloudów** po wynikach Nvidii jako wskaźnik dostępu drugiej linii do długu.
- **Wydania FRR i obrazów SONiC** z poprawką CVE-2026-120221 oraz ewentualne pojawienie się PoC.
