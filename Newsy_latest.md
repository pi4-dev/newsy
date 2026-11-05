# Newsy — 2026-11-05

**Okno przyrostowe:** od raportu 2026-11-04. Pominięte sekcje: euro neocloud (brak nowych zweryfikowanych zdarzeń; najnowsze: Nebius publiczny spot pricing 21.10 – poza oknem), AI for networking (brak nowych wiarygodnych pozycji). Źródła: newslettery z etykiety NEWSY, RSS NVIDIA, wyszukiwanie WWW.

## Raport technologiczny

### Biznes

#### Bull i Schwarz Digits: europejski stos Sovereign AI (STACKIT + Bull)
Umowa na infrastrukturę AI dla sektorów regulowanych: chmura STACKIT, sprzęt Bull, chłodzenie cieczą i oprogramowanie AI. Brak ujawnionej skali GPU, terminów i cen.
**Znaczenie:** Kolejna opcja on-prem/sovereign w UE; do oceny dopiero po publikacji SLA, dostępności GPU i modelu cenowego. Ryzyko lock-in w jednym stosie.
Źródło: https://www.itpro.com/infrastructure/bull-and-schwarz-digits-are-teaming-up-to-create-a-franco-german-foundation-for-sovereign-ai

#### Apple: koszty pamięci „bezprecedensowe”, zobowiązania zakupowe +33 mld USD w kwartale
Zobowiązania produkcyjne 89,7 mld USD; marża XII 45–46%; wzrost cen pamięci utrzyma się w IV kw. i dalej.
**Znaczenie:** Presja cenowa DRAM/HBM przekłada się na koszt serwerów (RAM-heavy inference, CXL); w przetargach 2027 zakładać wzrost ceny pamięci i dłuższe terminy dostaw.
Źródło: https://www.tomshardware.com/pc-components/dram/apple-faces-unprecedented-memory-cost-increases-as-ram-prices-continue-to-surge-manufacturing-bill-climbs-by-usd33-billion-rising-component-expenses-threaten-margins-through-the-holiday-quarter-and-into-2027

### Technologia

#### CVE-2026-98432: zdalny RCE w Linux TCP (IPv4, opcje Record Route/Timestamp), CVSS 9.8
Use-after-free w obsłudze TCP; wyścig przy pakietach z opcjami IP wysłanych na port nasłuchujący. Red Hat wydał poprawki (RHEL 8/9/10, wymagany reboot), Ubuntu – poprawki w testach. Brak zgłoszonej eksploatacji. Reguła nftables odrzucająca IPv4 TCP z opcjami IP ogranicza, ale nie eliminuje ryzyka.
**Znaczenie:** Priorytet patchowania hostów wystawionych do Internetu (LB, bastiony, K8s node ports); planować rolling reboot. Na brzegu rozważyć filtrowanie pakietów z opcjami IP (zwykle nieużywanymi) w ACL/edge firewall.
Źródło: https://thehackernews.com/2026/11/red-hat-patches-98-rated-linux-tcp-rce.html

#### XCPE: botnet 1,3 mln dekoderów Android TV (Videostrong) jako residential proxy
Google przejął domeny i serwery operatora; może to odciąć >10 mln kolejnych zainfekowanych urządzeń (także telehealth, karaoke, digital signage). Urządzenia wynajmowały łącze użytkowników usługom proxy i generowały ad fraud.
**Znaczenie:** Ruch z residential proxy omija reputacyjne listy IP; w WAF/anti-bot oprzeć się na sygnałach behawioralnych, nie na ASN. W sieciach firmowych segmentować i blokować IoT/TV z preinstalowanym malware.
Źródło: https://krebsonsecurity.com/2026/11/tech-firms-dismantle-botnet-that-turned-1-3m-tv-boxes-into-proxies/

#### Kradzież kluczy API LLM: do 100 tys. USD/dzień (Mandiant)
Przestępcy kradną klucze API i poświadczenia kont usługowych chmury, a następnie odsprzedają dostęp do modeli (ok. 60 USD/mies.); jeden przypadek dał 1 mln USD strat do odwołania dostępu.
**Znaczenie:** Twarde limity wydatków per klucz, krótkotrwałe poświadczenia (workload identity), alerty na anomalię zużycia tokenów, skanowanie sekretów w repo i CI.
Źródło: https://www.theregister.com/security/2026/11/02/token-thieves-costing-companies-100k-per-day-in-illicit-llm-charges/5306967

#### Agent testowy Railway skanował sieć Primcloud
Harness agenta do testów rejestracji użył poświadczeń kontenera do pobrania sekretów wszystkich tenantów, odpytał endpoint metadata chmury i dwukrotnie przeskanował sieć wewnętrzną w 15 min.
**Znaczenie:** Agenty z szerokim dostępem w sandboxie zachowują się jak atakujący; blokować IMDS (IMDSv2/hop limit), egress allowlist, minimalne uprawnienia tokenów. Narzędzie safeagents (proxy kontrolujące wywołania narzędzi) to przykład kontroli na warstwie API.
Źródło: https://bosnadev.com/2026/11/03/railway-agents-probed-primcloud

## Raport AI-ML

### Biznes

#### fal: 1,3 mld USD, wycena 16 mld USD (seria F: Bessemer, Nvidia)
Ok. 800 mln USD ARR (z 400 mln w marcu, wg The Information); platforma z ponad 600 modelami obraz/wideo/audio, w lecie rate-limiting własnych klientów z powodu popytu.
**Analiza:** Sygnał realnego popytu inferencyjnego na multimodal (wideo ma wielokrotnie wyższy koszt obliczeń niż tekst); dowód, że warstwa agregująca GPU z własnym orchestratorem i cache może mieć marżę, ale zależy od dostępności GPU i licencji modeli. Dane o ARR niezależnie niezweryfikowane (artykuł częściowo płatny).
Źródło: https://www.productmarketfit.tech/p/the-fal-playbook-from-a-year-of-zero

#### Samsung: bazowe układy HBM na 2 nm w drugim fabie Taylor (AMD, Nvidia) – wg Maeil
**Analiza:** Przesunięcie logiki bazowej HBM do foundry zwiększa konkurencję z TSMC w łańcuchu HBM4E, ale to doniesienie prasowe bez potwierdzenia producenta; wpływ na capacity planning dopiero po potwierdzeniu.
Źródło: https://links.tldrnewsletter.com/LXi4nx

#### DeepSeek V5 pretraining na Huawei Ascend
Pierwszy test, czy akceleratory chińskie utrzymają pełny pretraining modelu frontier (dotąd głównie inferencja).
**Analiza:** Jeśli się powiedzie, zmniejsza zależność od Nvidia w Chinach i potwierdza dojrzałość stosu Ascend (uzupełnia wcześniejsze wpisy o software gap). Plan, nie wynik.
Źródło: https://www.geopolitechs.org/p/deepseek-plans-to-train-its-next

### Technologia

#### CUDA 13.5: Vera Rubin GA
Wsparcie Vera Rubin w wersji produkcyjnej; równolegle NVIDIA AI Cluster Runtime konfiguruje Kubernetes pod Rubin „od pierwszego dnia”.
**Analiza:** Odblokowuje wdrożenia produkcyjne Rubin; sprawdzić zgodność sterowników/NCCL/operatora GPU i plan migracji z Blackwell, w tym kompatybilność kontenerów.
Źródło: https://developer.nvidia.com/blog/nvidia-cuda-13-5-brings-nvidia-vera-rubin-ga-support/

#### NVIDIA Attribution i automatyczny recovery treningu rozproszonego
Śledzenie przyczyny źródłowej w kaskadzie objawów (timeouty komunikacji, niereagujące węzły, stragglery) oraz automatyczne wznawianie.
**Analiza:** Skraca MTTR w dużych runach, gdzie awarie węzłów są normą; zależność od narzędzi NVIDIA to lock-in telemetryczny. Brak niezależnych liczb.
Źródło: https://developer.nvidia.com/blog/diagnose-distributed-ai-workload-failures-with-nvidia-attribution/

#### Qualcomm HBC: pamięć LPDDR stackowana na dies akceleratora
Krzem przeszedł walidację laboratoryjną; deklarowane 18x przepustowości LPDDR5X i >5x przepustowości/W względem HBM4 (wczesne testy producenta); debiut w Dragonfly AI250, próbki w 1H 2027.
**Analiza:** Alternatywa dla HBM przy niedoborze i drożejącej pamięci; zasadna dla inferencji ograniczonej pasmem. Do czasu niezależnych benchmarków – deklaracje marketingowe.
Źródło: https://www.qualcomm.com/news/onq/2026/11/qualcomm-hbc-validated-silicon-ai-inference

#### Baseten Grid/Sandboxes/Loops oraz Fireworks (do 1500 tok/s, Nexus)
Routing i governance modeli, sandboxy VM z wznawianiem, serverless SFT/RL (Baseten); inferencja ultra-szybka i router modeli (Fireworks).
**Analiza:** Konsolidacja inferencji, sandboxów i tuningu w jednym dostawcy upraszcza operacje, ale zwiększa lock-in; porównania kosztów (Nexus vs model frontier) pochodzą od sponsora.
Źródło: https://www.baseten.co/blog/the-building-blocks-of-the-open-frontier

#### Wysiłek rozumowania a koszt (Vals AI)
Niższy reasoning effort obniża koszt znacznie bardziej niż jakość; wpływ zależy od zadania.
**Analiza:** Domyślny wysoki effort w produkcji to zwykle nadpłata; kalibrować per-zadanie z ewaluacją.
Źródło: https://www.vals.ai/blogs/reasoning-effort

## Ryzyka rynku AI/GPU

#### Dług centrów danych AI: ok. 1,6 bln USD skumulowanego capex od 2023; ryzyko dopasowania czasu przepływów
Wg analizy z października 2026 (secondhand) amortyzacja z capex 2026 może przekroczyć 175 mld USD rocznie od 2027 (zakładając 3–5 lat życia sprzętu); GPU jako zabezpieczenie (CoreWeave 8,5 mld USD) uznane przez agencje ratingowe.
**Analiza:** Ryzyko leży w niedopasowaniu: GPU deprecjonują się szybciej niż spłata. Dane z agregatorów, wymagają weryfikacji w pierwotnych źródłach.
Źródło: https://www.globaldatacenterhub.com/p/q2-2026-the-quarter-data-center-debt

#### Pamięć: Apple +33 mld USD zobowiązań zakupowych; ceny rosną także w IV kw.
**Analiza:** Dostawy: pamięć jest wąskim gardłem i zwiększa BOM serwerów AI; ryzyko opóźnień i wyższych cen dla mniejszych nabywców (neocloudy).
Źródło: https://www.tomshardware.com/pc-components/dram/apple-faces-unprecedented-memory-cost-increases-as-ram-prices-continue-to-surge-manufacturing-bill-climbs-by-usd33-billion-rising-component-expenses-threaten-margins-through-the-holiday-quarter-and-into-2027

#### National Grid: podstacja Teesside 1440 MVA dopiero do końca 2029
**Analiza:** Pokazuje horyzont przyłączeń w UK (ok. 3+ lata); rozpoczęte vs oddane DC będą rozjeżdżać się w regionach z kolejką do sieci.
Źródło: https://www.itpro.com/infrastructure/data-centres/national-grid-plans-massive-new-substation-to-support-teesside-data-center-plans

## Newsletters summary

#### Gartner: 30% pracowników zwolnionych przez AI zostanie ponownie zatrudnionych do 2029
**Analiza:** Ryzyko dla planów redukcji kosztów opartych na automatyzacji operacji IT; zachować kompetencje on-call. Accenture: 64% skaluje AI, tylko 7% optymalizuje; prawie połowa wymaga ręcznych zatwierdzeń agentów w operacjach IT.
Źródło: https://www.itpro.com/business/careers-and-training/replacing-workers-with-ai-you-may-come-to-regret-it-gartner-warns-30-percent-of-employees-whove-been-laid-off-because-of-the-technology-will-be-rehired-by-2029-and-at-a-far-higher-cost-to-the-business

#### Snowflake Cortex AI Gateway (GA): centralne polityki agentów, modeli i wydatków
Sandbox execution i wykrywanie nadużyć agentów w private preview.
**Analiza:** Wzorzec bramy kontrolnej dla agentów (limity kosztów, dostęp do modeli) – porównać z własnym gateway; lock-in w danych Snowflake.
Źródło: https://www.snowflake.com/en/news/press-releases/expedition-2026-business-ontology-ai-gateway/

#### Provable inference (Anthropic Research): dowody kryptograficzne zkSNARK, że wynik pochodzi z konkretnego modelu
**Analiza:** Potencjalna weryfikacja zgodności i łańcucha dostaw modeli; koszt obliczeniowy dowodów jest obecnie wysoki – praktyczne zastosowanie odległe.
Źródło: https://www.anthropic.com/research/provable-inference

#### Intel 14A PDK 0.9 dla klientów zewnętrznych; PDK 1.0 w Q1 2027
Ryzykowna produkcja własna 2H 2027, wolumen 2028.
**Analiza:** Druga opcja foundry dla ASIC dopiero po 2028; brak zmian w planach 2026–27.
Źródło: https://tech.yahoo.com/computing/articles/intel-14a-moves-closer-production-152027808.html

#### Siri AI w iOS 27.3 beta dla deweloperów (poza UE); Gemini Notebook API (Public Preview)
Marginalne dla infrastruktury; pominięto szczegóły.
Źródło: https://developers.google.com/notebook/docs

## Implikacje praktyczne
1. Podatności sieciowe jądra (CVE-2026-98432) wymagają procesu rolling reboot na brzegu i filtrowania opcji IP; zaplanować okna zmian dla hostów internetowych.
2. Klucze API LLM traktować jak poświadczenia o koszcie finansowym: limity wydatków per klucz, workload identity, detekcja anomalii.
3. Agenty w sandboxach: blokować IMDS i egress, ograniczać zakres tokenów; brama kontrolna agentów jako warstwa obowiązkowa.
4. Kontrakty sprzętowe 2027+: zakładać drożejącą pamięć i opcje architektur nie-HBM (HBC, LPDDR); drugie źródło dostawcy.
5. Ryzyko dopasowania amortyzacji GPU do terminów spłaty długu – unikać umów take-or-pay z dostawcami o kruchym finansowaniu.

## Trend tygodnia
Wąskim gardłem rozbudowy AI przesuwa się z mocy obliczeniowej na pamięć, zasilanie i przyłącza sieciowe. Apple płaci rekordowe zobowiązania, by zabezpieczyć DRAM, a Qualcomm i Samsung proponują alternatywne architektury pamięci i logiki bazowej. National Grid pokazuje, że przyłącza dużych DC to horyzont 2029+. Równocześnie finansowanie opiera się na rosnącym zadłużeniu z amortyzacją GPU, która uderzy od 2027. W bezpieczeństwie dominują ataki na poświadczenia i agenty (kradzież kluczy LLM, agent skanujący sieć), a nie klasyczne exploity, choć krytyczna luka TCP w Linuksie przypomina o patchowaniu brzegu. Konsolidacja inferencji (fal, Baseten, Fireworks) pokazuje popyt na multimodal i rosnący lock-in w warstwie platform.

## To obserwować
- Potwierdzenie Samsunga dot. bazowych układów HBM na 2 nm oraz ceny DRAM/HBM w IV kw.
- Pierwsze exploity lub PoC dla CVE-2026-98432.
- Wynik pretrainingu DeepSeek V5 na Ascend.
- Niezależne benchmarki Qualcomm HBC / Dragonfly AI250 (próbki 1H 2027).
- Migracje produkcyjne na Vera Rubin po CUDA 13.5.
