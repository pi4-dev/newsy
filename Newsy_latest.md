# Raport Zbiorczy IT – 2026-11-26

**Okno przyrostowe:** od raportu z 2026-11-25. Obejmuje zdarzenia z 23–26.11, których nie ma w rejestrze z ostatnich 7 dni. Fakty już opisane (powrót Huanga, Shirudo, Claude Ship, Cairn/MONAI, przejęcie Decart, KEV z 23.11, wyniki HPE/Dell, LTV kredytów GPU) pominięto. 26.11 to Święto Dziękczynienia w USA (rynki zamknięte), więc nowych danych z USA za ten dzień nie ma.

---

## Raport technologiczny

### Technologia

#### Idunno.AntiCsrf (CVE-2026-131914, CVSS 9,3): RCE bez uwierzytelnienia w porzuconej bibliotece ASP.NET WebForms, wykorzystywana, poprawki nie będzie (KEV 24.11)
Mandiant potwierdził wykorzystanie w praktyce niebezpiecznej deserializacji w module AntiCsrf (wersje 0.9.1.0 i 0.9.1 Beta, projekt porzucony w 2008 r.). Wystarczy jeden POST bez uwierzytelnienia z łańcuchem gadżetów `ObjectStateFormatter` zakodowanym w Base64, żeby wykonać kod jako `w3wp.exe`. CISA dodała lukę do KEV razem z Drupal Core i Optimizely. Autor nie wyda poprawki.

**Znaczenie:** To klasyczny problem zależności „zamrożonej” w aplikacjach WebForms, których nikt nie inwentaryzuje: biblioteka siedzi w `bin/` i `Web.config`, a SCA jej nie widzi, bo nie pochodzi z NuGet. Do zrobienia: przeszukać `Web.config` pod kątem `AntiCsrfModule`, zastąpić bibliotekę `System.Web.Helpers.AntiForgery` lub `IAntiforgery` (ASP.NET Core), a jeśli w logach IIS są nietypowe POST-y do `.aspx` z odpowiedzią 302/500, zrotować `machineKey`. Po kompromitacji klucz `machineKey` pozwala też fałszować ViewState, więc sama wymiana biblioteki nie wystarczy.
Źródło: [The Hacker News](https://thehackernews.com/2026/11/hackers-exploit-critical-aspnet-anti.html), [Security Affairs – KEV](https://securityaffairs.com/203965/uncategorized/u-s-cisa-adds-drupal-core-idunno-anticsrf-and-optimizely-web-experimentation-flaws-to-its-known-exploited-vulnerabilities-catalog.html)

#### CISA KEV (25.11): dziesięć wpisów, w tym Juniper Session Smart Conductor/Router, pięć luk Mitel MiCollab i Xen Orchestra
Dodano m.in. CVE-2026-103406 (OS command injection w HPE Juniper Session Smart Conductor i Router), pięć CVE dla Mitel MiCollab (CVE-2026-133275/-133303/-133311/-133323/-133324: upload pliku, XML injection, słabe uwierzytelnianie, command injection; advisory MISA-2026-0012), CVE-2026-133469 w Vates Xen Orchestra oraz trzy luki sterowników Danfoss AK-SM 800A (path traversal, walidacja wejścia, obejście uwierzytelnienia).

**Znaczenie:** Session Smart Conductor (dawny 128T) to centralny kontroler SD-WAN. Przejęcie go daje kontrolę nad politykami routingu wszystkich podłączonych routerów, czyli tę samą klasę ryzyka co wcześniejsze ataki na kontrolery SD-WAN. Xen Orchestra ma poświadczenia do wszystkich hostów XCP-ng, więc luka w niej działa jak luka w vCenter. Pięć jednoczesnych CVE w MiCollab wskazuje na łańcuch exploitów: trzeba je łatać razem. Danfoss AK-SM steruje chłodnictwem (także w obiektach DC). Interfejsy zarządzania tych systemów nie powinny być dostępne z sieci płaskiej. Terminy federalne należy zweryfikować w samym katalogu KEV.
Źródło: [Shattered.io – zestawienie KEV z 25.11](https://shattered.io/cisa-kev-10-exploited-flaws-mitel-micollab-danfoss-juniper-xen-orchestra-nov-26/), [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

---

## Raport AI-ML

### Biznes

#### TSMC składa w Teksasie wniosek o ulgi podatkowe dla kampusu 2 nm za 22,6 mld USD w hrabstwie Brazos (budowa 2027–2031)
Wniosek obejmuje teren o powierzchni 3 232 akrów, co najmniej jeden fab logiczny (2 nm i węższe procesy) dla AI, HPC i mobile oraz do 2 099 bezpośrednich miejsc pracy. Budowa ma trwać od 2027 do 2031 r. To lokalizacja odrębna od kampusu w Arizonie.

**Analiza:** Przy harmonogramie 2027–2031 i typowym 2–3-letnim rampie pierwsze wafle N2 z Teksasu trafią na rynek najwcześniej w latach 2030–2031. W horyzoncie obecnego cyklu GPU (Rubin, Rubin Ultra, Feynman) niczego to nie zmienia: produkcja zaawansowanych węzłów i pakowania CoWoS pozostaje skoncentrowana na Tajwanie. Dla planowania zdolności oznacza to, że ryzyko geopolityczne łańcucha dostaw akceleratorów utrzyma się co najmniej do końca dekady. Druga lokalizacja w USA wzmacnia też argument polityczny za ograniczeniem eksportu (TSMC już wstrzymuje część dostaw dla chińskich projektantów, zob. rejestr z 24.11).
Źródło: [DCD](https://www.datacenterdynamics.com/en/news/tsmc-files-application-for-proposed-2nm-chip-fab-in-brazos-county-texas/)

### Technologia

#### NVIDIA: NVFP4 attention na RTX Blackwell (SM120) z „register recycling” – kernel 1,17–1,64× szybszy, zmiana scalona do FlashInfer (25.11)
Zoptymalizowany kernel SageAttention3 (QK i PV w NVFP4) dzieli kafel wyników N128 na dwie połówki N64. Gdy jedna połówka przejdzie eksponentę i kwantyzację do NVFP4, jej rejestry są od razu używane do QK następnego kafla. Matmul kolejnego kafla nakłada się więc na softmax, kwantyzację i PV bieżącego, bez alokacji drugiego pełnego bufora wyników. Konfiguracja warp się nie zmienia (4 producentów, 8 konsumentów). Na RTX PRO 6000 Blackwell Server Edition kernel jest 1,17–1,64× szybszy (maksimum przy 8 192 tokenach z maską przyczynową), a latencja end-to-end modeli wideo (OmniDreams, LongLive, Cosmos 3 Nano) spada o 1,1–15,3%.

**Analiza:** Według NVIDIA RTX Blackwell w attention BF16 jest ograniczony przepustowością matmul, a Blackwell DC (B200/B300) przepustowością softmaxu. Technika nie przenosi się więc wprost na GPU centrów danych, gdzie tę rolę pełnią FA3/FA4. Ma znaczenie dla flot inferencyjnych na RTX PRO 6000 (tańsze, chłodzone powietrzem serwery PCIe do modeli wideo i world models): kilkanaście procent latencji przy tym samym sprzęcie przekłada się bezpośrednio na koszt klatki lub zapytania. Ryzyko: NVFP4 w attention obniża dokładność, więc przed wdrożeniem produkcyjnym trzeba walidować jakość na własnym zbiorze. Kod jest w głównej gałęzi FlashInfer, więc pojawi się w vLLM i SGLang przy aktualizacji zależności.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/how-to-optimize-nvfp4-attention-on-nvidia-rtx-blackwell-through-register-recycling/)

#### vLLM-GR: inferencja generatywnych modeli rekomendacyjnych z beam search na paged KV cache (24.11)
Rozszerzenie vLLM (bazuje na 0.30.0) dla modeli rekomendacji generujących semantyczne ID (np. OneRec-1.7B). Sesja beam search jest własnością EngineCore. Po każdym etapie zapisywane są wyniki i rodzice ścieżek, a kolejny decode czyta KV przez indeksy rodzica. Wszystkie kandydaty współdzielą zbuforowaną historię użytkownika zamiast ją kopiować. Host przygotowuje kolejny krok, gdy akcelerator liczy bieżący. Przestrzeń wyszukiwania można zawęzić do katalogu. Przy beam width 256 i batchu 8 P99 wynosi 369 ms na H20 (SID-GR 428 ms, SGLang 1 280 ms) i 275 ms na A100 (SID-GR 452 ms). Rozwiązanie działa na GPU NVIDIA i NPU Ascend (A2, A5). Współtwórcy: m.in. ByteDance i TikTok E-commerce, openPangu.

**Analiza:** Rekomendacje generatywne to obciążenie o innym profilu niż chat: krótkie wyjście (kilka tokenów) i bardzo szeroki beam, więc wąskim gardłem są pamięć KV i narzut CPU między krokami, a nie przepustowość dekodowania. Współdzielenie prefiksu i nakładanie pracy hosta odpowiadają dokładnie tym dwóm problemom. Wsparcie Ascend obok CUDA w jednym silniku zmniejsza koszt przejścia chińskich operatorów na krajowe akceleratory. Zastrzeżenie: porównania pochodzą z osobnych przebiegów z różnymi definicjami pomiaru czasu (autorzy sami to przyznają), a przy batchu 1 na RTX PRO 6000 SID-GR jest minimalnie szybszy.
Źródło: [vLLM Blog](https://vllm.ai/blog/2026-11-24-vllm-gr-announcement) (via TLDR AI)

---

## euro neocloud

### Nebius

#### HPE Financial Services nie finansuje neocloudów; kurs −12% (24.11) i ok. −8,7% przed sesją 25.11
**Fakty:**
- Według relacji z konferencji wynikowej HPE (24.11) CEO Antonio Neri powiedział, że HPE Financial Services nie udzielało finansowania neocloudom ani klientom o podwyższonym ryzyku (to parafraza, nie cytat).
- Akcje Nebius spadły o 12% do 79,33 EUR, co zniwelowało wzrost z poprzedniej sesji. 25.11 przed otwarciem kurs tracił ok. 8,7%, przy jednoczesnych spadkach CoreWeave (ok. −9,8%) i IREN (ok. −8%).
- Konsensus FCF dla Nebius: −18,6 mld USD w 2026 i −24,4 mld USD w 2027 r. Capex: ok. 27 mld USD w 2026 i 40–45 mld USD w 2027 (ten fakt jest już w rejestrze, podany dla kontekstu).

**Ocena analityczna:**
- **Wykryte sygnały ostrzegawcze:**
  - Po Dellu (weryfikacja finansowania każdego zamówienia, rejestr 25.11) drugi duży OEM publicznie dystansuje się od kredytowania neocloudów. Finansowanie sprzętu przez producentów (vendor financing) zamyka się dla tego segmentu.
  - Ujemny FCF rzędu 40+ mld USD w dwa lata trzeba pokryć długiem lub kapitałem, a w tym samym tygodniu rentowność obligacji CoreWeave 2031 zbliżyła się do 18% (zob. Ryzyka). To wyznacza koszt marginalnego długu dla całego segmentu.
  - Wysoka beta kursu: ruchy ±10% dziennie utrudniają emisje kapitałowe i konwertowalne.
- **Ocena ryzyka: Wysokie.** Nebius ma przychody kontraktowe (Microsoft, Meta) i mniejszy dług niż CoreWeave (ok. 10 mld USD wobec 35,6 mld), ale skala capexu 2027 przy zamykającym się vendor financingu i rosnącym koszcie długu robi z refinansowania główne ryzyko. Krótkoterminowe ryzyko SLA dla klientów jest niskie, bo dotyczy ono nowych mocy (opóźnienia), a nie działających klastrów.
- **Linki:** [ad-hoc-news (za AP)](https://www.ad-hoc-news.de/boerse/news/unternehmensnachrichten/nebius-stock-falls-12-prozent-as-hpe-cools-the-neocloud-financing-well/70551265), [Invezz/TradingView](https://www.tradingview.com/news/invezz:4e1975427094b:0-why-coreweave-nebius-and-iren-shares-are-tumbling-on-wednesday/)

---

## AI for networking

#### AWS WAF: automatyczne wykrywanie endpointów LLM, MCP i A2A oraz zarządzana grupa reguł „AI Security” z Bedrock Guardrails (GA, 23.11)
API Portal wykrywa w całej AWS Organization endpointy LLM, MCP i A2A za API Gateway, ALB i CloudFront na podstawie logów ruchu i konfiguracji usług, bez agentów. Publiczne, nieudokumentowane serwery MCP oznacza jako shadow API. Grupa reguł AI Security blokuje komunikaty MCP i A2A niezgodne z protokołem. Po włączeniu inspekcji treści wywołuje Bedrock Guardrails (`InvokeGuardrailsCheck`), które wykrywają prompt injection i jailbreak w żądaniach oraz dane wrażliwe w odpowiedziach. Discovery działa w 32 regionach, AI Security w 7 (w Europie: Londyn i Sztokholm). Inspekcja Bedrock jest płatna osobno.

**Analiza:** To pierwsze u hyperscalera przeniesienie inwentaryzacji i filtrowania ruchu agentowego (MCP i A2A) na poziom brzegowego WAF. Ma to sens jako kontrola zakresu: shadow MCP to dziś realny wektor wycieku danych. Ograniczenia:
- **Latencja i koszt:** synchroniczne wywołanie Guardrails dokłada opóźnienie (setki ms przy dłuższych payloadach) i koszt za każde żądanie. Przy strumieniowanych, wieloetapowych sesjach agentowych opłacalna będzie inspekcja selektywna (tylko narzędzia z efektami ubocznymi).
- **Zasięg:** ruch spoza API Gateway, ALB i CloudFront (np. NLB, PrivateLink, sieć mesh wewnątrz EKS, połączenia agent–agent po stronie klienta) pozostaje niewidoczny.
- **Dostępność:** brak regionu Frankfurt i innych regionów UE poza Londynem i Sztokholmem komplikuje rezydencję danych.
- **Vendor lock-in:** polityki oparte na Guardrails nie przenoszą się do innych chmur. Przy architekturze wielochmurowej lepiej trzymać polityki MCP w neutralnej bramie (np. własny proxy MCP lub OPA), a AWS WAF traktować jako dodatkową warstwę.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/waf-ai-endpoint-discovery-protection/)

---

## Ryzyka rynku AI/GPU

#### S&P stawia rating AA Nvidii na CreditWatch negatywny z powodu ładu korporacyjnego; możliwe obniżenie o jeden stopień w ciągu 90 dni (25.11)
S&P Global Ratings wskazuje na rosnące ryzyko zarządcze: niezależność rady (długi staż części dyrektorów), planowanie sukcesji, ryzyko kluczowej osoby oraz kontrole wewnętrzne dotyczące ekspozycji na Chiny. Dodaje, że nie widzi żadnych działań naprawczych rady. Ewentualne obniżenie (do AA−) ma być ograniczone do jednego stopnia, chyba że dochodzenia wykażą szkodę dla perspektyw biznesowych. Ewentualne naruszenia kontroli eksportu lub zaniżone ujawnienia dotyczące Chin S&P potraktuje jako negatywne. Moody's (Aa1) 18.11 zmienił perspektywę na stabilną i utrzymał ją po powrocie Huanga. Nvidia ma ok. 32,5 mld USD obligacji w indeksie high-grade, w tym emisję 25 mld USD z czerwca 2026.

**Analiza:** Bezpośredni wpływ na koszt długu Nvidii jest niewielki (AA− to nadal bardzo wysoki rating, a firma ma dodatni FCF). Istotny jest mechanizm transmisji. Nvidia jest w ekosystemie de facto kredytodawcą i gwarantem ostatniej instancji: udziały w neocloudach, gwarancje rezerwacji mocy, backstopy dla projektów DC. Agencja, która kwestionuje kontrole wewnętrzne Nvidii, osłabia wartość tych gwarancji w oczach banków finansujących leasing GPU. Do tego dochodzi ryzyko regulacyjne, gdy w Senacie zaczynają się przesłuchania w sprawie eksportu. Nvidia jest w tej chwili źródłem ryzyka dla rynku, a nie tylko jego wskaźnikiem.
Źródło: [CNBC](https://www.cnbc.com/2026/11/26/nvidia-market-governance-risk.html), [Bloomberg](https://www.bloomberg.com/news/articles/2026-11-25/s-p-says-it-may-downgrade-nvidia-s-ratings-on-governance-worries)

#### Kolejna fala wyprzedaży 25.11: rentowność obligacji CoreWeave 2031 blisko 18%, S&P GMI szacuje 5-letnie prawdopodobieństwo niewypłacalności CoreWeave na 68%, ANT chwilowo poniżej ceny z IPO
- Nvidia traciła w trakcie sesji do 9,1%, zamknęła ok. −4%. Oracle −5%, Micron −4%, Amazon i Alphabet ok. −2%. Nasdaq zakończył dzień na −0,6% po znacznie głębszym spadku rano. Bezpośrednie wyzwalacze: gorące dane inflacyjne (ryzyko wyższych stóp) i wątpliwości co do zwrotu z capexu na AI.
- Anthropic (ANT) spadł chwilowo poniżej ceny z IPO (83 USD), po czym zamknął się na plusie (ok. +4%). Jeszcze 19.11 akcje otwierały się po 111,50 USD.
- Obligacje CoreWeave 2031: cena spadła o ponad 1,6 centa za dolara, do poziomu bliskiego rekordowego minimum (Reuters za Tradeweb), rentowność blisko 18%. Obligacje 9% 2032 notowane po ok. 75–76 centów. CDS CoreWeave, Oracle, Alphabet i SpaceX na rekordowych poziomach. S&P Global Market Intelligence wycenia 5-letnie prawdopodobieństwo niewypłacalności CoreWeave na 68%.

**Analiza:** Rentowność ~18% dla senior unsecured to poziom distressed: nowa emisja niezabezpieczona jest praktycznie niemożliwa, a refinansowanie przechodzi na dług zabezpieczony GPU i kontraktami, gdzie LTV jest już obcięte do 50–60% (rejestr 25.11). Dla ekonomiki GPUh: przy koszcie długu 15–18% i 5-letniej amortyzacji B200/B300 próg rentowności wynajmu rośnie o kilkadziesiąt centów na GPUh względem założeń z 2025 r. Neocloudy bez take-or-pay od hyperscalerów nie wytrzymają tego przy obecnych spotowych stawkach H100/H200. Spadek ANT poniżej ceny z IPO pokazuje, że rynek nie odróżnia już twórców modeli od infrastruktury: obie grupy wyceniane są jako jedno ryzyko capexowe. To podnosi koszt kapitału także dla odbiorców mocy, którzy podpisują ~900 mld USD zobowiązań obliczeniowych (rejestr 21.11). Interakcja z inflacją: wyższe stopy uderzają w aktywa o długim okresie zwrotu, czyli w DC i GPU, podwójnie (wycena i koszt rolowania długu).
Źródło: [CNBC](https://www.cnbc.com/amp/2026/11/25/nvidia-ai-stock-tech-selloff.html), [Invezz/TradingView](https://www.tradingview.com/news/invezz:4e1975427094b:0-why-coreweave-nebius-and-iren-shares-are-tumbling-on-wednesday/), [Forbes](https://www.forbes.com/sites/zacharyfolk/2026/11/25/nvidia-down-8-as-tech-stocks-extend-selloff/)

#### Banki przeglądają klientów DC w ASEAN po regule BIS; wstrzymany kredyt 480 mln USD dla operatora, który miał dostarczać moc ByteDance (25.11)
Citigroup i Standard Chartered (ten ostatni informuje regulatorów) oraz DBS, OCBC, UOB i Maybank przeglądają portfele klientów DC w Azji Południowo-Wschodniej pod kątem reguły BIS z 16.11 (obowiązuje od 19.11). Reguła wymaga licencji, gdy centrum danych dostarcza podmiotom chińskim moc powyżej 12 mln TPP łącznie (ok. 750 H200). Wstrzymano proponowany kredyt 480 mln USD dla nienazwanego dostawcy, który miał obsługiwać ByteDance. OCBC nie spodziewa się zakłóceń, bo jego ekspozycja dotyczy głównie hyperscalerów z USA. Część malezyjskich spółek DC i łańcucha dostaw straciła po wprowadzeniu reguły blisko 20%.

**Analiza:** To nowy element w stosunku do rejestru (reguła BIS z 15–16.11 i wpis Megaspeed na Entity List z 24.11): ryzyko eksportowe przechodzi z poziomu dostawców chipów do underwritingu bankowego. Mechanizm jest prosty: bank nie sfinansuje aktywa, którego głównego najemcę może zablokować licencja. Koszt KYC klienta końcowego przy najmie GPU spada na operatora. W ASEAN rośnie ryzyko osieroconych mocy: budynków przygotowanych pod chińskich najemców, dla których nie będzie zastępczego popytu przy obecnej nadpodaży w USA.
Źródło: [Crypto Briefing](https://cryptobriefing.com/banks-scrutinize-southeast-asia-data-centers-chip-curbs/)

---

## Newsletters summary

#### SemiAnalysis GDPx: trafność prompt cache różnicuje koszt inferencji o 18–32%; Fable 5.2 kosztuje 88 USD i 1,23 mln tokenów wyjściowych na zadanie (26.11)
Przebieg Gemini 4 Neon na benchmarku GDPx (206 zadań) kosztował 15 286 USD. Przy trafności cache Opus 5.5 kosztowałby 12 536 USD (−18%), a przy trafności GPT-6.2 Astra 10 403 USD (−32%). Google wypuścił 25.11 aktualizację cache, która według SA zmniejsza ten koszt o ok. 17–18%. Fable 5.2 zużywa 3,9× więcej tokenów wyjściowych i jest 1,9× droższy od 6.2 Astra, głównie przez uruchamianie subagentów do testów i przeglądu. W SWE prowadzi jednak wyraźnie (24,4% wobec 8,9% Opus 5.5). Ranking ogólny: 6.2 Astra 19,3%, Fable 5.2 16,4%, Gemini 4 Carbon 10,9%.

**Analiza:** Dla capacity planningu inferencji agentowej główną dźwignią kosztu staje się trafność prefix cache, a nie cena za token: różnica 30% wynika wyłącznie z warstwy serwującej. Konsekwencje: budżety tokenów dla agentów trzeba liczyć z rzeczywistym rozkładem wejścia cached i uncached (tę metrykę trzeba wymagać od dostawcy API), a przy self-hostingu priorytetem są rozproszone warstwy KV cache (LMCache, Dynamo i podobne) i routing świadomy cache. Agentowe rozmnażanie subagentów zwielokrotnia zużycie tokenów wyjściowych, więc limity wydatków na poziomie harnessu są obowiązkowe.
Źródło: [SemiAnalysis](https://newsletter.semianalysis.com/p/evaluating-recent-model-releases)

#### Paradigm3: roje agentów przy równym budżecie tokenów są szybsze, ale mniej efektywne niż pojedynczy agent z resetem kontekstu (TLDR AI)
Według badania kara za koordynację rośnie z liczbą agentów i obniża jakość. Wcześniej raportowana „premia roju” wynikała z degradacji kontekstu pojedynczego agenta. Pojedynczy agent ze świeżym resetem kontekstu wypada lepiej od roju.

**Analiza:** Architektury wieloagentowe kupują skrócenie czasu wykonania kosztem tokenów, a nie wyższą jakość. Dla planowania mocy oznacza to, że ruch od rojów jest bardziej „burstowy” (wiele równoległych sesji) przy tej samej pracy. Tańszy wzorzec to jeden agent z kompakcją lub resetem kontekstu, a równoległość tylko tam, gdzie liczy się czas.
Źródło: [Paradigm3](https://www.paradigm3.org/research/swarms)

#### Agenci w EDA mnożą obciążenie systemów danych projektowych 10–20× (SemiEngineering, TLDR Hardware)
Każdy inżynier sterujący 10–20 agentami skaluje w tym samym stopniu obciążenie kontroli wersji, NFS i liczbę licencji narzędzi EDA. Bazy przebiegów (waveform) o rozmiarze terabajtów trzeba konwertować do tekstu, co pochłania tokeny, dlatego potrzebne są odpytywalne abstrakcje. Klienci wracają z odizolowanych (air-gapped) modeli otwartych do modeli frontier.

**Analiza:** To konkretny przykład ogólnego zjawiska: agenci zamieniają się w syntetycznych użytkowników infrastruktury współdzielonej. Storage plikowy, licencjonowanie per seat lub per checkout i serwery VCS zostały zwymiarowane pod ludzkie tempo pracy. Przed wdrożeniem agentów trzeba przeprowadzić capacity planning NFS (metadane i IOPS), serwerów licencji i Git/Perforce, a także renegocjować modele licencyjne EDA.
Źródło: [SemiEngineering](https://semiengineering.com/ai-can-read-the-code-but-can-it-understand-the-chip/)

#### Databricks: analiza ok. 5 400 złośliwych pakietów npm – 7% przejętych legalnych pakietów odpowiada za ponad 99% pobrań; 85% publikacji przez OIDC trusted publishing to kompromitacje (23.11, TLDR InfoSec)
Pakiety tworzone przez atakujących stanowią 89% zbioru, ale tylko ok. 0,2% pobrań. Po wycofaniu przez npm długożyjących tokenów atakujący przeszli na przejmowanie CI i publikację przez trusted publishing. 38% pakietów ukrywa C2 w legalnych usługach (Cloudflare Workers, webhooki Discord, Telegram, Slack). Mediana podmiany pakietu na placeholder przez npm wynosi 7,1 dnia, a 16% pakietów nadal było instalowalnych w chwili analizy. Dla kampanii DPRK mediana to 19,5 dnia.

**Analiza:** Proweniencja (trusted publishing) potwierdza pochodzenie pakietu, a nie czystość pipeline'u. Kontrole oparte na „podpisany z CI = bezpieczny” trzeba uzupełnić o minimalny wiek wersji w proxy rejestru, monitoring zmian skryptów cyklu życia i maintainerów oraz egress allowlist (reputacja domen nie działa, skoro C2 siedzi w Workers). Rekomendacja Databricks, by pliki konfiguracyjne MCP i asystentów AI chronić jak klucze SSH, jest istotna dla stacji deweloperskich.
Źródło: [Databricks Blog](https://www.databricks.com/blog/what-5400-malicious-npm-packages-reveal-about-software-supply-chain-risk-2026)

#### Microsoft „Code Red” po awarii Microsoft 365: cel globalnego odtworzenia usług w ciągu 1 godziny (TLDR IT)
Według wewnętrznych notatek, po 24-godzinnej awarii Exchange Online z 12–13.11 Microsoft przebudowuje praktyki inżynieryjne i reagowania na incydenty. Cel to globalne odtworzenie usług w ciągu godziny, a środki to lepsza izolacja ruchu, inny proces wdrażania poprawek i lepsza diagnostyka. (Raport końcowy z awarii jest już w rejestrze z 25.11; nowy element to cel RTO i program naprawczy.)

**Analiza:** Cel 1 godziny przy defekcie latentnym od lipca wymaga szybkiego wycofania zmian w warstwie platformy (Windows i gateway), a nie tylko usługi. Wymusza to wolniejsze, segmentowane wdrożenia, co w krótkim terminie może spowolnić tempo nowych funkcji M365. Dla klientów: plany ciągłości działania muszą zakładać wielogodzinną niedostępność poczty i Teams. TLDR odsyła też do tekstu InfoWorld „SaaS resiliency bill”, który zwraca uwagę na ukryte zależności aplikacji on-prem od chmury.
Źródło: [TLDR IT (link do relacji)](https://links.tldrnewsletter.com/pJz7F7), [InfoWorld](https://www.infoworld.com/article/4252431/the-saas-resiliency-bill-is-coming-due.html)

#### Cloudflare Flamingo: ciągłe sondy DNS z każdego DC jako SLI bramkujące wdrożenia (23.11, TLDR IT)
Rekord TXT ze znacznikiem czasu jest stale aktualizowany. Sondy z każdego DC odpytują autorytatywne serwery nazw (3 adresy IP) i resolver 1.1.1.1. Kryterium świeżości: 90 s dla serwerów autorytatywnych i 600 s dla resolvera, plus asercje NOERROR i flagi AA. Wyniki zasilają SLI, SLO i budżety błędów, a systemy wydań na ich podstawie promują zmianę albo ją wycofują. Planowane jest automatyczne zatrzymanie rolloutu przy skoku latencji propagacji w kanarkowym DC.

**Analiza:** To wzorzec wart skopiowania dla własnych platform DNS i GSLB: pomiar opóźnienia propagacji (a nie tylko dostępności) jako bramka wdrożeniowa wychwytuje klasę awarii „serwer odpowiada, ale treść jest nieaktualna”, której zwykłe health-checki nie widzą.
Źródło: [Cloudflare Blog](https://blog.cloudflare.com/probing-dns-at-cloudflare)

#### Inception (prawdopodobnie odnowiony ShinyHunters): siedem ofiar w tydzień, w tym AMD, Cigna i PepsiCo (TLDR InfoSec)
Nowa marka wymuszeń opublikowała na stronie wycieków m.in. AMD, Cigna, PepsiCo, AutoZone i Lucid. Zakresu skradzionych danych nie potwierdzono.

**Analiza:** Profil ShinyHunters to kradzież przez przejęte tokeny SaaS i OAuth (Salesforce, Snowflake), a nie szyfrowanie. Warto sprawdzić integracje OAuth firm trzecich i tokeny serwisowe w CRM i hurtowniach danych. Przy AMD ewentualny wyciek dokumentacji projektowej lub klientów DC byłby istotny dla łańcucha dostaw.
Źródło: [The Register](https://www.theregister.com/security/2026/11/23/inception-possibly-a-shinyhunters-reboot-claims-seven-victims-in-a-week/5311768)

#### Ternary Bonsai 2.5 27B: model do agentowego kodowania w 5,9 GB (TLDR AI)
Model 27B z wagami ternarnymi zachowuje rozmiar 5,9 GB poprzednika, a według producenta wyraźnie poprawia wieloetapowe użycie narzędzi i kodowanie agentowe na urządzeniu.

**Analiza:** Model 27B w ok. 6 GB mieści się w pamięci laptopa lub stacji z iGPU, co pozwala uruchamiać lokalnych agentów na prywatnych danych bez wysyłania kontekstu do API. Deklaracje jakości pochodzą od producenta i wymagają niezależnej weryfikacji. Kernele ternarne nie są jeszcze standardem w głównych silnikach serwujących.
Źródło: [PrismML](https://prismml.com/news/bonsai-2-5-27b)

#### Antithesis „nightmare mode”: wstrzykiwanie awarii dysku – rozerwane zapisy i sekwencje, błędy I/O, utrata zasilania, latencja (TLDR)
Antithesis opisuje model błędów dysku w swoim deterministycznym symulatorze: utrata zasilania, rozerwane zapisy (torn writes) i rozerwane sekwencje zapisów, błędy I/O oraz wstrzykiwanie latencji ujawniające błędy współbieżności.

**Analiza:** Torn writes i zmiana kolejności zapisów po `fsync` to klasy błędów, które ujawniają się w produkcji dopiero przy awarii zasilania lub kontrolera. Dla zespołów budujących storage, bazy danych czy kolejki to gotowa lista kontrolna scenariuszy do testów odporności.
Źródło: [Antithesis](https://antithesis.com/blog/2026/nightmare-mode/)

---

## Implikacje praktyczne

1. **Kontrakty mocy AI z klauzulami kredytowymi kontrahenta.** Rentowność długu CoreWeave blisko 18%, zamknięcie vendor financingu przez HPE i Della oraz CreditWatch Nvidii oznaczają, że w horyzoncie 3–5 lat ryzyko kredytowe dostawcy mocy jest ryzykiem operacyjnym. W umowach z neocloudami potrzebne są: escrow lub step-in rights do sprzętu, prawo przeniesienia obciążeń (portability obrazów, danych i sieci), krótkie okresy prepay oraz monitorowanie wskaźników kredytowych dostawcy (CDS, rentowność obligacji) jako KPI dostawcy.
2. **Trafność KV i prefix cache jako parametr architektoniczny, nie optymalizacja.** Różnica kosztu 18–32% wynikająca wyłącznie z cache (GDPx) uzasadnia: routing świadomy cache, rozproszone warstwy KV (LMCache, Dynamo, CXL), stabilne prefiksy promptów w harnessach agentów oraz wymaganie od dostawców API raportowania udziału cached tokens. W modelu TCO na 3–5 lat zakładaj, że więcej niż 50% tokenów wejściowych agentów może pochodzić z cache.
3. **Agenci jako syntetyczni użytkownicy infrastruktury współdzielonej.** Obciążenie 10–20× na NFS, VCS i licencjach (EDA) oraz burstowy ruch rojów agentów wymagają przeliczenia pojemności metadanych storage'u, limitów API wewnętrznych i modeli licencyjnych. Ryzyko: limity licencyjne i rate limity staną się wąskim gardłem szybciej niż GPU.
4. **Brzegowe kontrole dla MCP i A2A, ale neutralne względem chmury.** AWS WAF AI Security pokazuje kierunek. Przy architekturze wielochmurowej polityki dla ruchu agentowego (allowlist narzędzi, inspekcja selektywna, logowanie) powinny jednak żyć w neutralnej bramie MCP, a natywne WAF-y to dodatkowa warstwa. Inaczej powstaje lock-in polityk bezpieczeństwa.
5. **Inwentaryzacja „zamrożonych” zależności i płaszczyzn zarządzania.** Idunno.AntiCsrf (brak poprawki), kontrolery SD-WAN Juniper SSR, Xen Orchestra i BMS (Danfoss) w KEV potwierdzają, że największe ryzyko leży w komponentach, których nikt nie posiada. Potrzebne jest SBOM obejmujące binaria spoza menedżerów pakietów oraz segmentacja płaszczyzn zarządzania (OOB, bez dostępu z sieci użytkowników).

## Trend tygodnia

Rynek infrastruktury AI przeszedł w ciągu tygodnia od kapitulacji (20.11), przez rekordowe odbicie (23.11), do ponownej wyprzedaży (24–25.11). Sama zmienność jest już sygnałem: kapitał nie ma stabilnej wyceny dla aktywów GPU. Ważniejsze od kursów akcji jest to, co dzieje się w kredycie. Rentowność obligacji CoreWeave blisko 18%, rekordowe CDS kilku emitentów AI, obcięte LTV kredytów pod GPU, OEM-y (Dell, HPE), które odmawiają finansowania sprzętu neocloudom, oraz banki w ASEAN wstrzymujące kredyty z powodu ryzyka eksportowego składają się na jeden obraz: koszt i dostępność długu stają się zmienną decydującą o tym, które moce powstaną. Ryzyko przestało też omijać „bezpieczne” ogniwa łańcucha: CreditWatch S&P dla Nvidii przenosi problem ładu korporacyjnego i eksportu na samego dostawcę, a spadek ANT poniżej ceny z IPO pokazuje, że rynek wycenia twórców modeli i infrastrukturę jako jedno ryzyko capexowe. Równocześnie po stronie technologii przewaga przesuwa się z surowej mocy na efektywność serwowania: trafność cache, kernele NVFP4 i silniki wyspecjalizowane pod konkretny typ obciążenia (vLLM-GR) dają 15–30% oszczędności na tym samym sprzęcie. Dla architektów wniosek jest taki, że w 2027 r. ważniejsza będzie zdolność do przenoszenia obciążeń między dostawcami i optymalizacji warstwy serwującej niż dostęp do najnowszego GPU.

## To obserwować

- **Rentowność obligacji CoreWeave 2031/2032 i CDS CoreWeave, Oracle i SpaceX.** Rentowność stale powyżej 18% albo spadek ceny 2032 poniżej 70 centów oznaczałby, że rynek zakłada restrukturyzację.
- **Decyzja S&P w sprawie Nvidii (do ok. 23.02.2027) i przesłuchanie w komisji bankowej Senatu w sprawie eksportu.** Ewentualne ustalenia o naruszeniach eksportu to scenariusz wykraczający poza obniżkę o jeden stopień.
- **Raportowanie trafności prompt cache przez dostawców API** (Google po aktualizacji 25.11, OpenAI, Anthropic) jako składnik efektywnej ceny tokena.
- **AWS re:Invent (przełom listopada i grudnia):** nowe instancje Trainium, ceny GPU w EC2 oraz to, czy AWS rozszerzy AI Security w WAF na regiony UE (Frankfurt, Irlandia).
- **Underwriting DC w ASEAN:** kolejne wstrzymane kredyty, licencje BIS dla operatorów w Malezji i Singapurze, wyceny malezyjskich spółek DC.
