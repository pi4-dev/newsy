# Raport Zbiorczy IT – 2026-11-15

**Okno przyrostowe:** od raportu z 2026-11-14 (weekend, mało nowych zdarzeń pierwotnych). Zdarzenia z 12–13.11 ujęto tylko wtedy, gdy nie było ich w rejestrze. Źródła: newslettery z etykietą NEWSY (SemiAnalysis, The Business Engineer, Own news), RSS NVIDIA Developer Blog, CISA, LMSYS, Federal Reserve, WWW. Problemy ze źródłami: RSS SemiAnalysis nadal zwraca wpisy z 2025 r.; RSS HPCwire/AIwire zwraca pusty kanał lub 403 (Cloudflare); gartner.com nie ma RSS i nic nowego tam nie znalazłem.

## Raport technologiczny

### Biznes

#### Cohere zamyka przejęcia AISLE i cubic (13.11) – powstaje Cohere Cyber
Cohere ogłosił zamknięcie dwóch przejęć. AISLE (zamknięte 10.11, list intencyjny z 16.10) buduje autonomiczne systemy do wykrywania, walidacji i naprawy nieznanych podatności. Według Cohere w ciągu 14 miesięcy znalazł ponad 650 CVE. cubic robi AI code review z automatyczną naprawą. Warunków nie ujawniono. AISLE działa teraz jako Cohere Cyber, a Praga ma być globalnym hubem tej jednostki.
**Znaczenie:** Dostawcy modeli „suwerennych” dokładają własne narzędzia do wykrywania podatności i bezpiecznego kodowania. Wyszukiwanie podatności przez AI staje się funkcją platformy, nie osobnym produktem. Dla zespołów infrastruktury oznacza to więcej zgłoszeń CVE od narzędzi AI, więc trzeba mieć triage odporny na wolumen (por. FORGE z 14.11). Rośnie też ryzyko lock-in: skaner, model i pipeline naprawczy pochodzą od jednego dostawcy.
Źródło: [Dealroom](https://dealroom.co/news/172615-cohere-buys-aisle-and-cubic-to-add-cyber-defense-to-sovereign-ai)

### Technologia

#### CISA KEV (13.11): Check Point CVE-2026-123039 i UnrealIRCd CVE-2026-114713
CISA dodała do katalogu KEV dwie aktywnie wykorzystywane luki. Pierwsza to CVE-2026-123039 w wielu produktach Check Point: błędne nadanie uprawnień do krytycznego zasobu. Druga to CVE-2026-114713 w UnrealIRCd: niewłaściwa neutralizacja separatorów. Alert CISA nie podaje wektora, wersji ani IoC, a dostępne źródła wtórne nie opisują jeszcze CVE-2026-123039.
**Znaczenie:** To kolejna w 2026 r. aktywnie wykorzystywana luka Check Point, po CVE-2026-50751 (VPN, Qilin), CVE-2026-16232 i CVE-2026-93616 (serwer zarządzający). Klasa błędu (uprawnienia do zasobu) sugeruje eskalację lokalną lub obejście kontroli w warstwie zarządzania, a nie pre-auth RCE. To jednak wniosek z samej klasy CWE. Do czasu publikacji advisory warto ograniczyć dostęp do interfejsów zarządzania (SmartConsole/Management API) do sieci administracyjnej i przejrzeć konta oraz uprawnienia na serwerach SMS/MDS.
Źródło: [CISA](https://www.cisa.gov/news-events/alerts/2026/11/13/cisa-adds-two-known-exploited-vulnerabilities-catalog)

## Raport AI-ML

### Biznes

#### Commerce finalizuje regułę o zdalnym dostępie Chińczyków do GPU w 7 kolejnych krajach; chińskie firmy rezerwują moc za granicą
Według The Information (12.11) Departament Handlu finalizuje regułę BIS. Ma ona zakazać chińskim firmom AI zdalnego korzystania z zaawansowanych chipów w DC w siedmiu kolejnych krajach. Znane są trzy z nich: Malezja, Singapur i Tajlandia. Ogłoszenie jest możliwe jeszcze w listopadzie, trwają konsultacje międzyresortowe. Prawnik Baker McKenzie twierdzi, że obecna ustawa nie daje Commerce takiego umocowania. Dałby je RASA (H.R. 2683), przyjęty przez Izbę 369–22 w styczniu i nadal czekający w senackiej komisji bankowej. Według Reutersa chińskie firmy AI kontaktują operatorów DC za granicą, żeby zarezerwować moc na GPU Nvidii przed wejściem reguły w życie. Wcześniej (FT, 3.11) Nvidia zażądała od partnerów chmurowych, by w nowych kontraktach co najmniej 51% usług sprzedawali firmom spoza Chin.
**Analiza:** Ryzyko dotyczy przede wszystkim popytu w neocloudach z Azji Południowo-Wschodniej, ale SemiAnalysis wskazuje też operatorów z Europy, krajów nordyckich, Kanady i USA, którzy sprzedają moc chińskim klientom. Jeśli reguła obejmie istniejące kontrakty (to nie jest jeszcze wiadome), część zakontraktowanego przychodu zniknie z dnia na dzień. Zwolnione klastry trafią wtedy na rynek spot i obniżą ceny GPUh. Wyścig o rezerwacje przed regułą może chwilowo zawyżyć utylizację i ceny w SEA. Dla planowania pojemności: klauzule end-use, KYC dla końcowego beneficjenta i prawo wypowiedzenia z powodu sankcji muszą być w umowach, zanim reguła wejdzie.
Źródła: [MLQ.ai (The Information, Reuters)](https://mlq.ai/news/chinese-ai-firms-rush-to-lock-in-overseas-nvidia-capacity-as-commerce-readies-a-rule-on-remote-chip-access/), [Congress.gov H.R.2683](https://www.congress.gov/bill/119th-congress/house-bill/2683/all-actions), [Yahoo Finance (FT, limit 51%)](https://finance.yahoo.com/technology/articles/nvidia-tells-cloud-clients-cap-073914242.html)

#### Aktualizacja: Anthropic IPO – wycena 18.11, debiut na Nasdaq (ANT) 19.11
Nowy element względem rejestru: według komunikatu spółki z 12.11 cena ma zostać ustalona w środę 18.11, a handel ma ruszyć w czwartek 19.11. Widełki 70–78 USD i 825 mln akcji bez zmian. Wynik Nvidii (17.11) przypada dzień przed wyceną.
**Analiza:** Zbieżność dat sprawia, że wynik i prognoza Nvidii pod nowym CEO bezpośrednio wpłyną na popyt na ofertę ANT. Wycena Anthropic posłuży rynkowi jako punkt odniesienia dla wycen innych laboratoriów i ich zobowiązań compute (ok. 895 mld USD wg S-1). Słaby debiut utrudni finansowanie kontraktowe neocloudów, których głównym klientem jest Anthropic (Nscale, CoreWeave).
Źródło: [Trading 212 (podsumowanie komunikatu)](https://www.trading212.com/learn/investing-101/anthropic-ipo)

### Technologia

#### SGLang Consortium (12.11): NVIDIA, AMD, Intel, Google, Meta, Cloudflare, Modal, RadixArk
LMSYS Org ogłosił na pierwszym SGLang Summit konsorcjum członków założycieli silnika inferencji SGLang. Decyzje techniczne i akceptacja kodu zostają u maintainerów. Członkowie współtworzą roadmapę i finansują wspólną pracę inżynierską: obsługę modeli frontier, reasoning i multimodal, inferencję rozproszoną oraz optymalizacje sprzętowe. W ogłoszeniu nie ma benchmarków ani podziału zadań.
**Analiza:** SGLang dostaje formalne wsparcie wszystkich trzech dostawców akceleratorów ogólnego przeznaczenia oraz Google (TPU, praca z RadixArk od 07.2026). To zmniejsza ryzyko lock-in w warstwie serwowania: ten sam stos (RadixAttention, prefix caching, disaggregated prefill/decode, wide EP) staje się przenośny między CUDA, ROCm, XPU i TPU. Dla architektów oznacza to wybór SGLang lub vLLM jako warstwy abstrakcji zamiast TensorRT-LLM, jeśli flota ma być heterogeniczna. Ryzyko: SemiAnalysis zauważa, że SGLang i vLLM korzystają z kerneli chińskich laboratoriów (DeepEP, DeepGEMM, MSA, FlashKDA). Zaostrzenie kursu wobec Chin może zatrzymać ten dopływ optymalizacji pod Blackwell.
Źródła: [LMSYS](https://www.lmsys.org/blog/2026-11-12-sglang-consortium), [HPCwire/AIwire](https://www.hpcwire.com/aiwire/2026/11/13/lmsys-org-forms-sglang-consortium-with-nvidia-amd-google-and-meta/)

## euro neocloud

### Europejscy operatorzy sprzedający moc chińskim klientom (nienazwani)

#### Ekspozycja na regułę o zdalnym dostępie i RASA
**Fakty:** SemiAnalysis (12.11) pisze, że zna neocloudy m.in. z Europy i krajów nordyckich, które aktywnie sprzedają moc chińskim firmom. RASA czeka na głosowanie w Senacie. Commerce finalizuje regułę dla 7 krajów (wymienione dotąd tylko kraje SEA). Od 3.11 Nvidia wymaga w nowych kontraktach co najmniej 51% klientów spoza Chin. Wcześniej (rejestr 13.11) Moonshot AI szukał w Europie klastrów Blackwell z ponad 8000 GPU.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* popyt chiński to dla części operatorów marginalny, ale wysokomarżowy przychód z krótkimi kontraktami. Regulacje mogą go odciąć szybciej, niż da się znaleźć zastępczych najemców. Ryzyko compliance (EAR, klauzule end-use Nvidii) przenosi się na finansujących (covenanty kredytów pod GPU).
- *Ocena ryzyka:* **Średnie** dla mniejszych operatorów z wysokim udziałem klientów z Azji. Dla dużych (Nebius, Nscale), których backlog opiera się na Microsoft, Meta i Anthropic, **niskie**.
- Linki: [MLQ.ai](https://mlq.ai/news/chinese-ai-firms-rush-to-lock-in-overseas-nvidia-capacity-as-commerce-readies-a-rule-on-remote-chip-access/), [SemiAnalysis](https://newsletter.semianalysis.com/p/nvidia-and-the-world-without-jensen)

### Nscale / Nebius (zależność od wsparcia bilansowego Nvidii)

#### Gwarancje Nvidii pod neocloudy a zmiana CEO
**Fakty:** Według SemiAnalysis (na podstawie raportów Nvidii) bruttowe gwarancje pozabilansowe Nvidii wyniosły na koniec II kw. fiskalnego 530 mld USD, wobec 184 mld USD kwartał wcześniej. Obejmują 36 mld USD minimalnych progów take-or-pay pod mocą neocloudów. Portfel udziałów kapitałowych Nvidii to 99 mld USD. Nscale jest wspierany przez Nvidię i w F-1/A (rejestr 14.11) wskazuje potrzebę dodatkowego kapitału na zobowiązania 2027. Nebius dostał od Nvidii 2 mld USD inwestycji.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* SemiAnalysis i The Business Engineer zwracają uwagę, że nowe gwarancje i alokacje zależały od osobistych decyzji Huanga. Istniejące zobowiązania pozostają, ale kolejne transze (np. oczekiwana dalsza transza Nvidii dla Nscale) wymagają decyzji nowego zarządu. Pierwszym testem będzie komentarz do wyników 17.11.
- *Ocena ryzyka:* **Wysokie** dla Nscale: IPO bez widełek, luka finansowania ok. 30 mld USD, kurs rówieśników −17% 12.11. **Średnie** dla Nebius, który ma szerszy dostęp do rynku długu i akcji.
- Linki: [SemiAnalysis](https://newsletter.semianalysis.com/p/nvidia-and-the-world-without-jensen), [The Business Engineer](https://businessengineer.ai/p/this-week-in-the-ai-supercycle-ai)

### Program UE AI Gigafactories (dotyczy wszystkich operatorów)

#### Zamknięcie składania ofert 12.11
**Fakty:** Według specyfikacji przetargu EuroHPC (EUROHPC-2026-CEI-AIGF-01) termin składania ofert na „do siedmiu” AI Gigafactories (do ok. 150 MW na lokalizację, co najmniej 7 państw) minął 12.11 o 16:00. Jedno źródło wskazywało na stronie wezwania termin 3.12, więc data wymaga potwierdzenia. Rozstrzygnięcia spodziewane są na początku 2027 r., budowa później w 2027 r. Nie ma publicznych informacji o liczbie ofert ani konsorcjach.
**Ocena analityczna:**
- *Sygnały ostrzegawcze:* finansowanie publiczne (do ok. 10 mld EUR) ma zostać dopełnione co najmniej 20 mld EUR kapitału prywatnego. To trudne przy obecnych kosztach długu neocloudów (CoreWeave >16% na niezabezpieczonych obligacjach). Opóźnienie decyzji do 2027 r. nie pomaga płynności operatorów, którzy liczą na ten program.
- *Ocena ryzyka:* **Średnie**: program wydłuża horyzont, ale nie rozwiązuje bieżących luk finansowych.
- Linki: [DC Market Insights](https://www.dcmarketinsights.com/news/the-eus-e30-billion-ai-gigafactory-tender-wants-7-sites-of-up-to-150-mw-bids-are-due-12-november), [Sullivan & Cromwell](https://www.sullcrom.com/insights/memo/2026/October/EuroHPC-Joint-Undertaking-Launches-AI-Gigafactories-Call)

## AI for networking

#### NVIDIA Scale-In: BlueField-4 + DOCA + Spectrum-X jako warstwa bezpieczeństwa i telemetrii dla agentowych AI factories (12.11)
NVIDIA przedstawia Scale-In jako osobny filar sieciowy, obok scale-up i scale-out. Silniki bezpieczeństwa w krzemie BlueField-4 działają out-of-band i zbierają telemetrię z pamięci, storage, GPU i CPU. NVIDIA Sentry (na DOCA) monitoruje zachowanie agentów i poddaje kwarantannie te, które wychodzą poza zdefiniowane granice. OpenShell (od 10.11 w CNCF) zarządza wykonywaniem agentów, dostępem i routingiem inferencji. Brak danych wydajnościowych i dat dostępności.
**Analiza:** Egzekwowanie polityk przenosi się z hosta (eBPF, agent EDR) na DPU. Atakujący z uprawnieniami root na hoście nie wyłączy więc telemetrii, a izolacja tenantów nie obciąża CPU hosta. Koszt: każdy węzeł potrzebuje BlueField-4, a polityki i telemetria są pisane w DOCA, co oznacza silny vendor lock-in w warstwie, która w ekosystemie otwartym była przenośna (eBPF/Cilium). Failure mode: błędna polityka kwarantanny na DPU działa jak wyłącznik sieciowy dla całych pul agentów, więc potrzebne są tryby audit-only i stopniowy rollout. Planując pojemność, trzeba liczyć moc i koszt DPU per węzeł oraz to, czy telemetria out-of-band nie konkuruje o pasmo z ruchem storage na tym samym NIC.
Źródło: [SiliconANGLE](https://siliconangle.com/2026/11/12/ai-networking-nvidia-scalein-security-agentic-aifactoriesdatacenters/)

## Ryzyka rynku AI/GPU

#### Fed Financial Stability Report (13.11): >680 mld USD długu „advanced computing” w 10 miesiącach 2026, >2 bln USD take-or-pay i gwarancji wartości rezydualnej
Fed wylicza, że emisja długu związanego z zaawansowanymi obliczeniami przekroczyła 680 mld USD do października 2026 r. To pięć razy więcej niż 130 mld USD za cały 2023 r. Udział tego długu w emisjach IG spółek niefinansowych wzrósł z poniżej 6% w 2023 r. do ok. 31%. Publicznie wyemitowany dług w obiegu przekracza 1,1 bln USD. Prywatny kredyt utrudnia pełny pomiar, więc to dolna granica. Leasingi jeszcze nierozpoczęte mają niezdyskontowaną wartość ponad 1 bln USD. Kontrakty take-or-pay i gwarancje wartości rezydualnej przekraczają 2 bln USD. Największe banki USA mają blisko 1 bln USD ekspozycji na big tech, deweloperów DC, SPV, wehikuły private credit i utility. Raport opisuje „circular financing” jako podatność wymagającą monitoringu.
**Analiza:** To pierwsze oficjalne oszacowanie skali zobowiązań pozabilansowych (leasingi, take-or-pay), które na poziomie systemu przewyższają dług bilansowy. Kanał transmisji: spadek utylizacji GPU lub cen GPUh uderza w najemców (neocloudy, laboratoria), następnie w gwarancje wartości rezydualnej (spadek cen używanych GPU), a potem w SPV i banki. Prawdopodobna reakcja regulacyjna to wyższe wagi ryzyka lub limity koncentracji dla kredytów pod DC. Podniosłoby to koszt kapitału dla projektów bez najemcy investment grade.
Źródło: [CASRAI (omówienie FSR)](https://casrai.org/news/federal-reserve-financial-stability-report-ai-november-2026), [Federal Reserve – Financial Stability Report](https://www.federalreserve.gov/publications/financial-stability-report.htm)

#### CoreWeave: zabezpieczona pożyczka ≥4 mld USD pod kontrakty z Meta, cel ok. 7,5% wobec ok. 16% na obligacjach (13.11)
Według Semafor i Bloomberga CoreWeave syndykuje (Morgan Stanley, MUFG) pożyczkę co najmniej 4 mld USD. Zabezpieczeniem są sprzęt obliczeniowy i kontrakty z Meta (rating AA). Docelowy koszt to ok. 7,5% all-in, podczas gdy rentowność niezabezpieczonych obligacji sięgała ok. 16%. Wstępny marketing przyjęto ciepło. Sprzedaż w transzach może ruszyć w przyszłym tygodniu. Kurs CRWV spadł 13.11 o 6,1% do 53,13 USD.
**Analiza:** Rozjazd ok. 850 pb między długiem zabezpieczonym kontraktem a niezabezpieczonym pokazuje, że rynek wycenia ryzyko kredytowe najemcy, a nie operatora. Neocloud staje się w praktyce SPV przenoszącym kredyt hyperscalera. Skutki: (1) dług zabezpieczony podporządkowuje dotychczasowych obligatariuszy, więc spread niezabezpieczonych obligacji może dalej rosnąć; (2) moc bez kontraktu z najemcą IG jest praktycznie niefinansowalna; (3) koncentracja na Meta i Microsoft zwiększa ryzyko kaskady przy nieodnowieniu kontraktu, jak w przypadku Microsoft i Oracle oraz Microsoft i Nscale.
Źródła: [Semafor](https://www.semafor.com/article/11/13/2026/coreweave-tunes-out-nvidia-noise-looks-to-raise-4b), [Motley Fool](https://www.fool.com/coverage/stock-market-today/2026/11/13/stock-market-today-nov-13-coreweave-falls-over-6-on-ai-cloud-selling-and-reported-usd4-billion-loan-talks/)

#### „Jensen put”: 530 mld USD gwarancji i 279 mld USD zobowiązań zakupowych Nvidii bez założyciela
SemiAnalysis zestawia dane z raportów Nvidii. Gwarancje pozabilansowe brutto wzrosły ze 184 do 530 mld USD w jednym kwartale (w tym 36 mld USD progów take-or-pay pod neocloudy). Zobowiązania zakupowe i pojemnościowe wzrosły ze 119 do 279 mld USD, głównie na pamięć. Do tego dochodzi 99 mld USD w portfelu udziałów. The Business Engineer dodaje, że Nvidia podpisała gwarancje części płatności za leasing i energię wdrożenia OpenAI w kampusie SB Energy (Ohio). W równoległej ofercie Anthropic dla dostawców (26 mld USD) Nvidia zobowiązała się kupić akcje za 10 mld USD.
**Analiza:** Nvidia pełni funkcję prywatnego „banku centralnego” ekosystemu: sprzedaje GPU z pełną marżą, a potem gwarantuje przychód lub czynsz kupującego. Operatorzy bez ratingu IG pożyczają dzięki temu na warunkach IG. Ryzyko polega na tym, że menedżerski zarząd ograniczy nowe gwarancje. Wtedy mniejsze neocloudy tracą dostęp do taniego długu dokładnie wtedy, gdy Fed sygnalizuje koncentrację ryzyka. Odwrotny scenariusz: 279 mld USD zobowiązań na pamięć przy spadku popytu staje się nadwyżką zapasów (write-down) i presją na ceny HBM i DRAM w 2027–2028.
Źródła: [SemiAnalysis](https://newsletter.semianalysis.com/p/nvidia-and-the-world-without-jensen), [The Business Engineer](https://businessengineer.ai/p/this-week-in-the-ai-supercycle-ai)

## Newsletters summary

#### SemiAnalysis: „Nvidia and the World Without Jensen” (12.11)
Teza autorów: odejście Huanga wiąże się z przemytem GPU. Model akceleratorów SemiAnalysis i Departament Handlu szacują ponad 1,5 GW zastrzeżonych chipów Nvidii w Chinach. Odejście ma też usunąć głównego przeciwnika RASA przed głosowaniem w Senacie. Kluczowe dane:
- HBM4 SK Hynix na 2027 r.: ok. 3,0–3,3 USD/Gb dla Nvidii wobec 3,7–4,1 USD/Gb dla AMD, Broadcom, Amazon i Google (szacunek SemiAnalysis).
- Prognoza: w 2028 r. ponad 70% całej mocy obliczeniowej AI może być poza Nvidią, bo dostawcy przestaną ją priorytetyzować.
- Brak następcy: ok. 60 bezpośrednich podwładnych Huanga. Wewnętrznie wymieniani są Ian Buck i Jonah Alben, a SemiAnalysis ocenia, że żaden z nich nie przejmie roli negocjatora łańcucha dostaw.
- Sprawy przemytu: Supermicro (US v. Liaw, 2,5 mld USD serwerów, w tym 510 mln do Chin; proces 29.03.2027) oraz Keelung (130 serwerów B300, 74 dotarły do Chin; 9 oskarżonych, w tym menedżer Nvidia Taiwan).
- Open source: kernele chińskich laboratoriów (DeepEP, DeepGEMM, MiniMax MSA, Moonshot FlashKDA) używane w SGLang i vLLM mogą przestać być publikowane.
**Analiza:** Dla capacity planning najważniejsza jest prognoza poluzowania alokacji. Jeśli SK Hynix i TSMC przestaną priorytetyzować Nvidię, czas dostaw TPU, Trainium i MI4xx się skróci, a różnica cen HBM się zmniejszy. To podnosi realność floty wielodostawczej w 2027–2028. Ryzyko odwrotne: utrata optymalizacji z chińskich kerneli spowolni poprawę kosztu tokena na Blackwell w otwartych silnikach inferencji.
Źródło: [SemiAnalysis](https://newsletter.semianalysis.com/p/nvidia-and-the-world-without-jensen)

#### The Business Engineer: „AI Lost Its Central Banker” (14.11)
Nowe elementy względem rejestru: 9.11 senatorowie Warren (D) i Banks (R) wspólnie wezwali do zaostrzenia kontroli chipów i zamknięcia kanałów obchodzenia restrykcji. To dowód na ponadpartyjny kurs wobec Chin. Nvidia gwarantuje część płatności za leasing i energię wdrożenia OpenAI w kampusie SB Energy (Ohio). W równoległej ofercie dla dostawców przy IPO Anthropic Nvidia kupuje akcje za 10 mld USD. OpenAI miało 6.11 pierwsze zamknięcie rundy na 11 mld USD (Reuters).
**Analiza:** Finansowanie rozbudowy rozkłada się na więcej bilansów (laboratoria sprzedają udziały, hyperscalerzy emitują dług, dostawcy inwestują w klientów). Ekspozycja dotyczy jednak tych samych założeń popytowych, więc dywersyfikacja inwestorów nie dywersyfikuje ryzyka. Dla architektów: kontrakty compute zawierane z laboratoriami finansowanymi przez dostawców sprzętu mają skorelowane ryzyko kontrahenta.
Źródło: [The Business Engineer](https://businessengineer.ai/p/this-week-in-the-ai-supercycle-ai)

#### Own news (n8n, 14.11): cuBLAS FP64, cuPhoton, cuDLA, SGLang Consortium
cuBLAS FP64 (Ozaki-II) opisano już 14.11. SGLang Consortium jest w sekcji AI-ML. Pozostałe dwie pozycje są niszowe: cuPhoton (interferometria radiowa czarnych dziur na GPU) oraz natywna inferencja DLA w Pythonie na Jetson AGX Orin przez cuDLA. Ta druga ma znaczenie dla edge: odciąża GPU i poprawia perf/W przy wielu równoległych sieciach, ale DLA obsługuje ograniczony zestaw warstw i reszta wraca na GPU.
Źródło: [NVIDIA Technical Blog – cuDLA](https://developer.nvidia.com/blog/run-native-dla-inference-in-python-on-nvidia-jetson-agx-orin-using-cudla/)

## Implikacje praktyczne

1. **Kontrakty compute powinny zakładać regulacyjny „kill switch”.** Reguła Commerce i RASA mogą unieważnić część popytu w chmurach poza USA. W umowach z neocloudami trzeba wymagać deklaracji struktury klientów (limit 51% Nvidii), klauzul o zmianie prawa i prawa do relokacji obciążeń. Przez 3–5 lat ryzyko eksportowe będzie stałym elementem SLA, nie zdarzeniem jednorazowym.
2. **Stos inferencji powinien być przenośny między akceleratorami.** SGLang Consortium i prognoza ponad 70% mocy poza Nvidią w 2028 r. uzasadniają standaryzację na SGLang lub vLLM z backendami CUDA, ROCm i TPU zamiast silników jednego dostawcy. Trzeba utrzymywać testy regresji wydajności per backend, bo optymalizacje kerneli rozjadą się między platformami.
3. **Ryzyko kredytowe najemcy jest ryzykiem dostawcy.** Pożyczka CoreWeave pod kontrakty Meta i dane Fed (ponad 2 bln USD take-or-pay) pokazują, że stabilność neocloudu zależy od ratingu jego największego klienta. Przy wyborze dostawcy trzeba analizować koncentrację klientów i strukturę długu (zabezpieczony vs niezabezpieczony), nie tylko cenę za GPUh.
4. **Telemetria i polityki bezpieczeństwa przechodzą na DPU.** Scale-In (BlueField-4/DOCA) daje izolację odporną na kompromitację hosta kosztem lock-in. Warto projektować polityki w abstrakcji (np. OPA lub Cilium jako źródło prawdy), którą da się kompilować zarówno do eBPF, jak i do DPU, żeby nie wiązać modelu bezpieczeństwa z jednym krzemem.
5. **Wolumen zgłoszeń CVE od narzędzi AI będzie rósł** (Cohere Cyber, FORGE). Proces zarządzania podatnościami trzeba przeskalować: automatyczna deduplikacja, ocena exploitowalności i SLA zależne od KEV, a nie od samego CVSS.

## Trend tygodnia

Odejście Huanga zmienia ryzyko rynku AI z technologicznego na instytucjonalno-finansowe. Fed po raz pierwszy oficjalnie wycenił skalę zadłużenia i zobowiązań pozabilansowych związanych z AI: ponad 1,1 bln USD długu publicznego i ponad 2 bln USD take-or-pay. Te liczby wpisują się w obraz Nvidii jako gwaranta ekosystemu z 530 mld USD gwarancji. Jednocześnie rynek długu ostro rozróżnia ryzyko: kontrakt z hyperscalerem obniża koszt finansowania CoreWeave z ok. 16% do ok. 7,5%. Moc bez najemcy IG staje się więc trudna do sfinansowania, co uderza zwłaszcza w europejskie neocloudy. Polityka eksportowa USA się zaostrza (reguła Commerce, RASA, ponadpartyjny nacisk w Senacie) i zaczyna dotyczyć nie tylko sprzedaży chipów, ale też chmurowego dostępu do mocy. W odpowiedzi branża wzmacnia otwarte, wielodostawcze warstwy oprogramowania (SGLang Consortium), które ograniczają zależność od jednego dostawcy krzemu. Najbliższy tydzień skupia trzy testy: wynik Nvidii 17.11, wycenę i debiut Anthropic 18–19.11 oraz możliwą publikację reguły o zdalnym dostępie.

## To obserwować

- **Nvidia Q3 FY2027 (17.11):** komentarz tymczasowego CEO do gwarancji pozabilansowych, zobowiązań zakupowych (279 mld USD) i alokacji. Każde ograniczenie nowych backstopów to sygnał dla Nscale i mniejszych neocloudów.
- **Anthropic ANT (wycena 18.11, debiut 19.11):** cena względem widełek 70–78 USD i zachowanie kursu w pierwszych dniach.
- **Reguła BIS o zdalnym dostępie i RASA w Senacie:** lista 7 krajów i to, czy reguła obejmie istniejące kontrakty.
- **Pożyczka CoreWeave ≥4 mld USD:** finalny koszt (cel ok. 7,5%) i reakcja spreadów obligacji niezabezpieczonych 2032.
- **SC26 (15–20.11) i lista TOP500:** systemy z emulacją FP64, Ultra Ethernet BoF (18.11) oraz pierwsze konkretne dane Scale-In/BlueField-4.
