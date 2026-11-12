# Raport Zbiorczy IT – 2026-11-12

**Okno przyrostowe:** od raportu 2026-11-11. Zdarzenia z 3–10.11 ujęte tylko wtedy, gdy nie było ich w rejestrze. Źródła: newslettery z etykiety NEWSY (TLDR, TLDR AI/IT/InfoSec/Hardware, Own news), RSS NVIDIA Developer Blog i WWW. Problemy ze źródłami: SemiAnalysis RSS zwraca nieaktualny cache z 09/2025 (archiwum newslettera sprawdzone ręcznie, ostatni wpis 7.11 już w rejestrze), a HPCwire/AIwire RSS blokuje Cloudflare. Wyniki Nebius za Q3 wychodzą dziś przed otwarciem rynku w USA, więc nie ma ich w tym wydaniu.

## Raport technologiczny

### Technologia

#### N-able N-central „Able2Shell": ponad 30 zero-dayów z PoC, hosted N-central wyłączony
Badacz Sean Bales opublikował ponad 30 luk z exploitami PoC, testowanych na appliance 2026.4.0.31. Wśród nich są XXE z nadużyciem zaufania NCI, kradzież support-tokenów, wykonanie niepodpisanych NSP, XSS→OGNL, path traversal w uploadzie repozytorium (RCE) i overflow stosu w natywnym GenericTCP, dający root. N-able wyłączył hosted N-central, a klientom on-prem kazał zatrzymać serwery. Wydał 2026.4 HF2 (m.in. CVE-2026-125878…125883, CVE-2026-91000) i zapowiedział kolejny hotfix. Według watchTowr i Previdian próby exploitacji na honeypotach pojawiły się w ciągu kilkunastu godzin (68 prób z jednego IP w ok. 18 h). Censys liczy 1728 instancji wystawionych do Internetu.
**Znaczenie:** RMM to płaszczyzna sterowania z uprawnieniami SYSTEM/root na tysiącach endpointów, więc kompromitacja u MSP od razu przechodzi na wszystkich jego klientów (wzorzec Kaseya 2021). Do zrobienia: logowania i aktywność agentów od 8.11 traktować jako podejrzane, przejrzeć nowe konta admin i zadania skryptowe, a konsolę RMM wystawiać tylko przez ZTNA/VPN. Usługa była niedostępna przez decyzję dostawcy, więc runbooki DR powinny przewidywać wyłączenie platformy zarządzania przez vendora.
Źródło: https://thehackernews.com/2026/11/n-able-takes-hosted-n-central-offline.html

#### Barracuda ESG: łańcuch CVE-2026-116606 + CVE-2026-111560 do roota
Poprawki są z 2.11. Pierwsza luka pozwala bez uwierzytelnienia zmienić stan konfiguracji, np. wstrzyknąć wartość strefy czasowej. Druga wykonuje z tej konfiguracji polecenia jako root. Tylko ESG jest podatny na obie naraz. Fix to firmware 9.4.0.030, ze starszych wersji często trzeba aktualizować wieloetapowo. Barracuda (wg TLDR InfoSec, 11.11) potwierdza aktywną exploitację. Według Barracudy do ataku potrzebny jest dostęp do konsoli administracyjnej. W chwili pisania nie potwierdziliśmy wpisu w CISA KEV.
**Znaczenie:** Po incydencie z 2023 r. (CVE-2023-2868 i wymiana sprzętu) ESG ma historię trwałych implantów, które przetrwały patch. Do zrobienia: zdjąć konsolę admin z Internetu, porównać konfigurację (timezone/NTP, użytkownicy) z kopią sprzed 2.11 i zaplanować ocenę integralności, nie tylko aktualizację.
Źródło: https://fieldeffect.com/blog/barracuda-esg-flaws-root-access

#### Turla „Cereal": Exchange jako pasywny C2 i hook w transport agentach
ESET opisuje pasywny backdoor, który odbiera szyfrowane komendy przez zwykłe ścieżki OWA, Autodiscover i ActiveSync, więc nie ma beaconingu wychodzącego. Moduł wstrzyknięty do EdgeTransport.exe podpina się pod istniejące transport agenty i kopiuje pocztę z adresami NATO/UE. Ofiary to agencje rządowe w Kazachstanie i Rumunii. Atrybucja do Turli ma wysoką pewność. Wskaźniki: DLL-e udające komponenty Windows w System32\wbem, nieoczekiwane Microsoft.Exchange.*.ni.dll w cache natywnych obrazów .NET, EdgeTransport.exe ładujący niezarejestrowane assembly.
**Znaczenie:** Ruch C2 wtopiony w legalne endpointy Exchange omija detekcję opartą o egress i reputację. Potrzebne są kontrola integralności binariów i cache NGEN oraz inwentarz transport agentów (Get-TransportAgent) jako baseline. Kolejny argument za migracją z on-prem Exchange albo przynajmniej za separacją ról Edge.
Źródło: https://www.welivesecurity.com/en/eset-research/exchange-for-breakfast-turlas-cereal-tricks-are-not-for-kids/

#### Aktualizacja: ataki na kontrolery chłodnicze we Francji przypisane IRGC
Nowy element względem wpisów z 6.11 i 11.11: Francja formalnie przypisała październikowe i listopadowe ataki na mroźnie sklepowe i banki krwi Korpusowi Strażników Rewolucji (Iran).
**Znaczenie:** Atrybucja państwowa zmienia model zagrożenia dla OT chłodniczego i BMS z cyberprzestępczości na sabotaż. Dla DC i edge oznacza to priorytet segmentacji sieci BMS, monitoringu zmian nastaw i ręcznych procedur fail-safe.
Źródło: https://techcrunch.com/2026/11/09/france-blames-iran-for-mass-hacks-targeting-freezers-in-grocery-stores-and-blood-banks/

### Biznes

#### Kubecost/Apptio (IBM): raportowanie kosztów sieci K8s (NAT Gateway, cross-AZ, cross-region)
IBM rozszerza Apptio o konektory zużycia AI i planowanie finansowe usług AI. Kubecost dostaje szczegółowy podział kosztów AWS NAT Gateway, ruchu cross-zone i cross-region na workloady. MCP serwery i scenario planning są dopiero zapowiedziane.
**Znaczenie:** Koszty egress i cross-AZ to często największa ukryta pozycja w klastrach inferencyjnych z RAG (pobieranie kontekstu, repliki w kilku AZ). Atrybucja do namespace/workloadu pozwala uzasadnić topology-aware routing i lokalne endpointy VPC.
Źródło: https://newsroom.ibm.com/2026-11-09-apptio-expands-portfolio-to-help-enterprises-connect-ai-spending-to-business-value

## Raport AI-ML

### Biznes

#### TSMC: 40,9 mld USD zatwierdzonych środków kapitałowych oraz udziały w Anthropic i OpenAI (10.11)
Zarząd zatwierdził ok. 40,93 mld USD na moce w zaawansowanych węzłach, zaawansowane pakowanie, budowę fabów i R&D 2027. Przychód za Q3 wyniósł 1494 mld TWD, zysk netto 763 mld TWD. Do tego wehikuł inwestycyjny do 500 mln USD oraz zakupy do 500 mln USD akcji Anthropic (po cenie IPO) i do 500 mln USD akcji uprzywilejowanych OpenAI.
**Analiza:** Skala jednej transzy środków potwierdza, że wąskim gardłem pozostają CoWoS i węzły N2/A16. Dla capacity planning 2027–2028 oznacza to, że podaż akceleratorów zależy od tempa pakowania, nie od waferów logiki. Udziały w obu głównych laboratoriach sprawiają, że fab ma bezpośrednią ekspozycję na popyt końcowy. To kolejny element sieci wzajemnych zależności finansowych (vendor financing) w łańcuchu AI.
Źródło: https://pr.tsmc.com/english/news/3352

#### Gemini 4 Carbon: GA w API, cena wprowadzająca 2/10 USD za 1M tokenów do 30.11, potem 4/20 USD
Najmocniejszy model Google jest dostępny w płatnym Gemini API i Gemini for business. Google podaje, że Carbon zużywa więcej tokenów niż Argon na rozumowanie i weryfikację, dlatego Argon zostaje domyślny dla większości zadań. /swarm w Antigravity (preview) dzieli zadanie na role: Carbon koordynuje i audytuje, lżejsze modele pracują równolegle, a weryfikatory testują zmiany. W programie Fairwind model trafił najpierw do zespołów obrony cyber.
**Analiza:** Od 1.12 cena za token się podwaja, a model zużywa więcej tokenów na zadanie, więc deklarowany „2× niższy koszt na zadanie" trzeba zweryfikować na własnych trace'ach. Swarm mnoży zużycie limitów, bo każdy worker to osobny agent. Potrzebne są budżety per sesja i routing modeli (Argon domyślnie, Carbon tylko na kroki wysokiego ryzyka).
Źródło: https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-4-carbon-now-available/

### Technologia

#### NVIDIA AICR + k8s-aibom: runtime AIBOM (CycloneDX 1.6) dla workloadów inferencyjnych na K8s
Kontroler k8s-aibom (projekt Google, open source) obserwuje API Kubernetes przez informery. Obsługuje grafy Dynamo, NIMService, LeaderWorkerSet, DisaggregatedSet i CronJob. Dla każdego workloadu generuje AIBOM w formacie CycloneDX 1.6: kategoria workloadu, runtime (np. vLLM), model, obraz kontenera. Każdy atrybut ma poziom pewności (declared/inferred/unresolved/verified) i lokalizator dowodu. Rekord jest zasobem CRD z hashem SHA-256. Weryfikacja Sigstore istnieje, ale w AICR jest domyślnie wyłączona. Narzut na GKE: ok. 4 żądania/min do API servera, 2,2 mCPU, 28–34 MiB.
**Analiza:** Rozwiązuje to realny rozjazd między „przepisem" platformy a tym, co faktycznie działa na węzłach, co ma znaczenie dla audytów EU AI Act i inwentarza modeli. Ograniczenia: kontroler czyta specyfikacje w całym klastrze, łącznie z inline env, więc sekrety wpisane w env trafiają do cache kontrolera. Nie widzi wersji sterowników ani kernela, a Slurm i Kubeflow nie są natywnie wspierane. Bez włączonej weryfikacji Sigstore pole „model" jest deklaratywne, nie kryptograficzne.
Źródło: https://developer.nvidia.com/blog/validating-gpu-platforms-with-runtime-ai-inventory-via-the-nvidia-dsx-ai-factory/

#### AMD Helios w Azure: zapowiedź ND MI455X v7 dla produkcyjnej inferencji
Przy okazji GA EPYC 6. generacji (opisanego 11.11) AMD podał, że Azure planuje instancje ND MI455X v7 oparte na racku Helios. Wcześniej (3.11) AMD informował, że pierwsze racki Helios z MI450 obsługują już workloady m.in. Anthropic, Meta, Microsoft, OpenAI i Oracle.
**Analiza:** To pierwszy hyperscaler z publicznie nazwaną instancją rack-scale AMD w klasie NVL72. Dla architektów to realna druga ścieżka zakupowa, ale portowalność zależy od dojrzałości ROCm w vLLM/SGLang i od wydajności fabricu scale-up (UALink/Ethernet). Benchmarki trzeba robić na własnym miksie prefill/decode, nie na peak FLOPS.
Źródło: https://www.globenewswire.com/news-release/2026/11/10/3400032/0/en/amd-showcases-6th-gen-amd-epyc-processors-to-lead-next-era-of-ai-and-enterprise-computing.html

## euro neocloud

### Europejscy operatorzy neocloud (nienazwani) – Moonshot AI

#### Moonshot AI szukał klastrów Blackwell (8000+ GPU) u europejskich neocloudów
**Fakty:** CNBC (5.11, cztery anonimowe źródła) podaje, że chińskie Moonshot AI prowadziło w tym roku rozmowy o wynajmie mocy Nvidia Blackwell u europejskich neocloudów. Chodziło o klastry 8000+ GPU, głównie do inferencji, możliwe że także do treningu. Pojedyncze źródło wspominało o premii 30–50% za płatność z góry. Nie zgłoszono podpisanej umowy, a firmy odmówiły komentarza. Wcześniej (lipiec) Biały Dom wskazywał, że Moonshot korzystał z Blackwell przez obiekt w Tajlandii. Zdalny dostęp przez chmurę poza USA nie jest dziś szeroko objęty kontrolą eksportu. Projekt Remote Access Security Act jest w toku.

**Ocena analityczna**
- *Wykryte sygnały ostrzegawcze:* popyt z Chin płacący premię 30–50% to dla neocloudów z niską utylizacją atrakcyjny przychód, ale z ryzykiem regulacyjnym. Na pewno ryzyko KYC/compliance (EAR, przyszłe przepisy o remote access). Do tego warunki umów z Nvidią i hyperscalerami (klauzule end-use) oraz ryzyko reputacyjne przy kontraktach suwerennych (UE, sektor publiczny).
- *Ocena ryzyka:* **Średnie**. Nie ma potwierdzonego kontraktu ani nazwanego dostawcy. Jeśli jednak USA rozszerzy kontrolę na zdalny dostęp, operator z takim klientem może stracić alokacje GPU od Nvidii albo kontrakty z Microsoft/Anthropic. Dla klientów korporacyjnych neocloudu oznacza to ryzyko nagłej zmiany warunków lub dostępności.
- *Linki:* https://dealroom.co/news/170092-moonshot-sought-8-000-gpu-blackwell-clusters-in-europe-to-skirt-us-chip/ ; https://www.unite.ai/?p=504678

## AI for networking

#### Arista: brak materialnych przychodów ze scale-up Ethernet w 2027, AI fabrics ok. 7 mld USD
Q3 2026 (3.11): przychód 3,66 mld USD (+58,5% r/r), marża brutto 64% (rok wcześniej 65,2%). Prognoza 2027: +38–40% (ok. 18,5 mld USD), w tym AI fabrics ok. 3,5 → 7 mld USD. CEO zapowiada triale scale-up Ethernet w 2027, produkcję w 2028 i żadnej „materialnej liczby" w 2027. Arista szacuje ponad 10 GW wdrożeń AI (scale-up/out/across) do końca 2027. Microsoft i Meta to każdy co najmniej 10% przychodu.
**Analiza:** Scale-up (domena NVLink) pozostanie w latach 2027–2028 domeną własnościową Nvidii albo UALink/ESUN w fazie pilotażu. Projekty zakładające Ethernet w scale-up dla racków 2027 powinny mieć plan B. Scale-out Ethernet (UEC) jest dojrzały, a spadek marży świadczy o presji cenowej Spectrum-X i Silicon One. Koncentracja przychodów na dwóch hyperscalerach przekłada się na ryzyko roadmapy pod ich wymagania.
Źródło: https://www.techtimes.com/articles/330161/20261106/arista-q3-earnings-beat-estimates-ceo-sees-no-material-scale-ethernet-revenue-next-year.htm

#### SC26 (15–20.11): warsztat Broadcom/Meta o transporcie scale-up (SUE-T, ESUN) i UEC BoF
Broadcom pokaże Tomahawk 6/Ultra, Jericho 4, NIC Thor Ultra 800G i CPO TH6-Davisson. Warsztat „High Performance Fabrics for AI and HPC" (16.11), prowadzony przez Broadcom i Meta, obejmie load balancing, kontrolę przeciążeń, odporność i telemetrię. BoF Ultra Ethernet jest zaplanowany na 18.11.
**Analiza:** Warto śledzić dane o kontroli przeciążeń i telemetrii in-band dla SUE-T/ESUN. To one rozstrzygną, czy Ethernet scale-up zbliży się do NVLink pod względem tail latency przy all-to-all w MoE. CPO zmienia model serwisowy: awaria lasera oznacza wymianę modułu przełącznika zamiast pluggable.
Źródło: https://www.hpcwire.com/off-the-wire/broadcom-brings-open-high-performance-ai-networking-to-sc26/

#### Cisco AI Defense + Claude Mythos: adaptacyjny red teaming agentów
Mythos ma planować testy, generować pomysły ataków i zamieniać je na payloady w adaptacyjnym red teamingu AI Defense. Kontrole: stałe zadanie, zarejestrowany cel, brak otwartego interfejsu promptowania, działanie ograniczone do autoryzowanego celu, pełna atrybucja do operatora i audyt. Na razie dla wybranych firm, szersza dostępność później („when-and-if available").
**Analiza:** Red teaming agentów przesuwa się z katalogowych testów (OWASP LLM) do adaptacyjnych, wieloetapowych ataków generowanych przez model ofensywny. Daje to lepsze pokrycie prompt injection w łańcuchach narzędzi, ale wyniki są niedeterministyczne i trudne do powtórzenia w regresji. Testy trzeba wersjonować razem z polityką guardrails. Ograniczona dostępność oznacza też lock-in na stos Cisco w części SASE/AI security.
Źródło: https://blogs.cisco.com/ai/cisco-ai-defense-to-add-anthropics-claude-mythos-for-ai-red-teaming

## Ryzyka rynku AI/GPU

#### SoftBank: 3,7 mld USD straty na zmianie planów zakupu turbin/generatorów dla DC w USA; 9,48 mld USD przedpłat
(INFRASTRUKTURA/FINANSE) Fiskalne Q2 (do 30.09): strata netto 1,12 bln JPY (ok. 7 mld USD). Za półrocze strata 597,2 mld JPY (ok. 3,7 mld USD) wynika ze zmiany planów zakupu urządzeń wytwórczych dla amerykańskich DC AI. Skumulowane płatności za te urządzenia wynoszą 9,48 mld USD, przykładem jest kampus PORTS-Pike (Ohio). SoftBank tłumaczy to „bridge investments" wyprzedzającymi niedobory. Projekt w Ohio opiera się na 20-letniej umowie najmu z OpenAI na 8 GW IT. Wycena udziałów w OpenAI: 99,6 mld USD fair value przy 54,6 mld USD zainwestowanych.
**Analiza:** To pierwszy duży, zaksięgowany odpis na rynku sprzętu energetycznego dla DC. Pokazuje, że rezerwacje turbin z wyprzedzeniem stają się kosztem utopionym, gdy zmieniają się lokalizacje albo moratoria (Ohio jest na liście stanów z oporem lokalnym). Bilans SoftBanku opiera się na niezrealizowanym zysku z OpenAI, więc spadek wyceny OpenAI uderza jednocześnie w LTV holdingu i w finansowanie projektów DC. Warto porównać z ruchem Anthropic (przejęcie zamówienia turbin, 11.11): rynek wtórny slotów turbinowych zaczyna działać.
Źródło: https://www.cnbc.com/2026/11/10/softbank-2q-earnings-tech-selloff.html

#### Aktualizacja: Anthropic IPO – 825 mln akcji po 70–78 USD, ticker ANT (Nasdaq)
Nowy element względem 11.11: widełki cenowe i struktura. 801,35 mln akcji nowej emisji i 23,65 mln od dotychczasowych akcjonariuszy, plus opcja greenshoe 123,75 mln akcji. To ok. 58–64 mld USD brutto z oferty. WSJ podaje dodatkowe ok. 26 mld USD od dużych inwestorów (m.in. Google, AMD), wycenę ok. 1,3 bln USD i start notowań w czwartek przyszłego tygodnia. TSMC zadeklarowało do 500 mln USD po cenie IPO.
**Analiza:** Pozyskanie ok. 85–90 mld USD łącznie nie pokrywa zobowiązań compute (ok. 895 mld USD według S-1, opisane 8.11). IPO jest więc etapem w finansowaniu, nie jego zamknięciem. Udział dostawców (Google, AMD, TSMC) w ofercie wzmacnia cyrkularność finansowania: dostawcy compute finansują klienta, który kupuje ich compute. Wskaźnik do obserwacji to wycena na otwarciu i zachowanie po zakończeniu lock-upu, bo od tego zależy koszt kapitału całego segmentu.
Źródło: https://www.anthropic.com/news/roadshow-launch ; https://www.wsj.com/finance/stocks/anthropic-ipo-valuation-8337b669

#### Zakaz chińskich transceiverów optycznych: Microsoft lobbuje przeciw, ryzyko „bezczynnych GPU"
(DOSTAWY) Reuters (10–11.11) podaje, że Microsoft i grupy branżowe (ITI) naciskają na Biały Dom, by porzucił planowany zakaz FCC dla nowych modeli chińskich transceiverów. Według części źródeł zakaz miałby dotyczyć klasy 3,2 Tb/s. Chińscy producenci (Innolight, Eoptolink) mają ok. 2/3 rynku globalnego (Counterpoint). Argument branży: zakupione GPU będą stać niepodłączone, dopóki nie przybędzie transceiverów. Decyzja jest otwarta.
**Analiza:** 3,2T to optyka dla generacji 2027–2028 (Rubin Ultra, przełączniki 204,8T), więc zakaz uderzyłby dokładnie w rampę następnej generacji klastrów. Alternatywni dostawcy (Coherent, Lumentum/Marvell po fuzji, Broadcom CPO) nie mają dziś takiej skali. Należy oczekiwać wzrostu cen i lead time'ów optyki oraz przyspieszenia CPO. Dla projektów 2027+: dywersyfikacja i kwalifikacja drugiego źródła transceiverów, bufor magazynowy, klauzule w umowach z integratorami.
Źródło: https://www.silicon.co.uk/cloud/datacenter/microsoft-optical-transceivers-632284

## Newsletters summary

#### The Token Trap: koszty AI rosną przez architekturę danych, nie przez sam model (HPCwire/AIwire)
Według autora tokeny generuje cały pipeline agentowy: pobieranie, łączenie i walidacja danych, ponowne próby. Największe koszty wynikają z nieefektywnej architektury danych. Rekomendacje: audyt tokenów per krok, ograniczenie ruchu danych, lepsze metadane, optymalizacja kontekstu.
**Analiza:** Zgodne z obserwacjami FinOps. Retry przy błędach narzędzi i nadmiarowy kontekst RAG dają mnożnik kosztu rzędu kilku razy na zadanie. Warto mierzyć tokeny per udane zadanie, nie per wywołanie, i stosować cache promptów oraz deduplikację kontekstu.
Źródło: https://www.hpcwire.com/aiwire/2026/11/11/the-token-trap-why-ai-costs-spiral-at-scale/

#### MAI-Transcribe-2 (Microsoft): ASR z diarizacją i znacznikami słów za 0,10 USD/h audio
Microsoft deklaruje przewagę nad Gemini 3.5 Transcribe, GPT-Transcribe i Whisper V3-Large oraz obsługę szerszego spektrum nagrań.
**Analiza:** Przy 0,10 USD/h API staje się tańsze niż self-hosting Whispera na GPU dla większości wolumenów, chyba że wymagania danych (suwerenność, PII) wymuszają lokalne przetwarzanie. Dla contact center i transkrypcji spotkań zmienia to TCO, ale dodaje zależność od Azure.
Źródło: https://microsoft.ai/news/mai-transcribe-2-is-the-fastest-most-accurate-and-cheapest-speech-recognition-model-in-the-world/

#### AWS: retencja CloudWatch Logs na poziomie konta oraz wiele Route 53 Profiles na VPC
CloudWatch obsługuje teraz polityki retencji dla całego konta lub prefiksu grup logów. Route 53 Profiles pozwala podpiąć kilka profili do jednego VPC z jawnymi priorytetami.
**Analiza:** Retencja na poziomie konta zamyka typowy wyciek kosztów: grupy logów tworzone automatycznie przez Lambdę/EKS z „Never expire". Wiele profili DNS na VPC ułatwia rozdzielenie odpowiedzialności (platforma vs zespół aplikacyjny) w multi-account, ale przy konfliktach reguł priorytety trzeba testować, bo błędna kolejność jest cichym failure mode resolvera.
Źródła: https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-cloudwatch-log-retention/ ; https://aws.amazon.com/about-aws/whats-new/2026/11/route-53-profiles-multiple-per-vpc/

#### Gemini w Gmail domyślnie włączony dla wszystkich użytkowników Workspace (od 10.11, do 15 dni)
Podsumowania wątków i „Help me write" są domyślnie włączone dla użytkowników 18+ z aktywnymi smart features. Administrator może je wyłączyć.
**Analiza:** Domyślne włączenie przetwarzania treści poczty przez LLM wymaga przeglądu DPIA i polityk DLP. Przy wymogach regulacyjnych admini powinni świadomie zdecydować o OU/grupach przed końcem rolloutu.
Źródło: https://workspaceupdates.googleblog.com/2026/11/expanding-gemini-in-gmail-to-more-Google-Workspace-users.html

#### installctl: izolowane uruchamianie instalatorów `curl | sh` z diffem zmian
Narzędzie uruchamia instalator w nieuprzywilejowanych namespace'ach na nakładce FUSE copy-on-write. Pokazuje każdy utworzony lub zmieniony plik, wstrzymuje się na instalacji pakietów, nowych kontach, systemctl i crontab, a na koniec generuje skrypt odinstalowujący.
**Analiza:** Praktyczne dla obrazów bazowych i stacji deweloperskich jako kontrola supply chain dla instalatorów skryptowych. Nie zastępuje weryfikacji podpisów. Mechanizmy oparte na FUSE i namespace'ach mogą zachowywać się inaczej niż docelowy system (np. skrypty sprawdzające uprawnienia root).
Źródło: https://github.com/yamixst/installctl

#### ExecuTorch WebGPU i Meta e-means (FAISS)
ExecuTorch uruchamia modele PyTorch na GPU w przeglądarce bez konwersji formatu. Meta FAIR wydał e-means, zamiennik k-means drop-in, który osiąga niższą stratę przy do 10× mniejszej liczbie epok.
**Analiza:** WebGPU przesuwa część małej inferencji na klienta, co zmniejsza koszt serwerowy i latency, ale komplikuje kontrolę wersji modelu i ochronę wag. e-means może skrócić budowę indeksów IVF w FAISS przy dużych korpusach RAG; trzeba zweryfikować wpływ na recall.
Źródła: https://pytorch.org/blog/executorch-webgpu-pytorch-models-on-the-gpu-in-the-browser/ ; https://github.com/facebookresearch/faiss/tree/main/contrib/e_means

## Implikacje praktyczne
1. **Płaszczyzny zarządzania (RMM, ESG, Exchange) jako tier-0:** po N-able, Barracudzie i Turli konsole administracyjne mają być dostępne tylko przez ZTNA i odizolowane od Internetu. Monitoring integralności binariów i konfiguracji, a w DR scenariusz „vendor wyłącza platformę".
2. **Optyka jako pozycja krytyczna w capacity planning 2027–2028:** możliwy zakaz chińskich transceiverów 3,2T i konsolidacja dostawców (Marvell–Lumentum) wymagają kwalifikacji drugiego źródła i rozważenia CPO z jego modelem serwisowym. Pełne pokrycie optyką przed dostawą GPU, inaczej GPU czeka bezczynnie.
3. **Scale-up Ethernet nie wcześniej niż 2028:** w projektach rack-scale 2027 zakładać NVLink albo pilotaż UALink/ESUN z planem wyjścia. Scale-out na UEC/Ethernet można standaryzować już teraz.
4. **Ekonomia tokenów zamiast cennika:** zmiany cen (Gemini 4 Carbon 2× od 1.12) i modele zużywające więcej tokenów na rozumowanie wymagają routingu modeli, budżetów per sesja agentowa i metryki „koszt per udane zadanie".
5. **Ryzyko kontrahenta w łańcuchu energii i compute:** odpis SoftBanku na przedpłatach turbin pokazuje, że rezerwacje sprzętu energetycznego to aktywa z ryzykiem utraty wartości. W umowach z dostawcami DC i neocloudami trzeba weryfikować status przyłącza i zasilania, nie tylko zamówienia sprzętu.

## Trend tygodnia
Finansowanie AI przechodzi z fazy obietnic do fazy księgowania kosztów. SoftBank pokazuje pierwszy duży odpis na przedpłatach za sprzęt energetyczny, a Anthropic wchodzi na giełdę z kwotą, która pokrywa tylko ułamek zobowiązań compute. Jednocześnie dostawcy (TSMC, Google, AMD) obejmują akcje swoich największych klientów, co pogłębia cyrkularność finansowania. W razie korekty uderzy to w cały łańcuch naraz, nie w pojedyncze ogniwo. Po stronie podaży wąskie gardło przesuwa się z GPU na komponenty towarzyszące: optykę 3,2T (ryzyko polityczne), pakowanie (CoWoS) i energię. Arista potwierdza, że Ethernet w domenie scale-up to perspektywa 2028, więc do tego czasu własnościowe interkonekty Nvidii zachowają przewagę, a AMD Helios na Azure będzie testem realnej alternatywy. W bezpieczeństwie dominuje kompromitacja płaszczyzn zarządzania (RMM, bramki pocztowe, Exchange) i rosnąca rola modeli ofensywnych w red teamingu i obronie (Mythos w Cisco, Carbon w Fairwind). Dla architekta wniosek jest taki, że ryzyko przesuwa się z „czy dostanę GPU" na „czy cały łańcuch (energia, optyka, finansowanie, kontrahent) dowiezie działający klaster".

## To obserwować
- **Nebius Q3 2026 (12.11):** przychód vs konsensus ok. 0,9 mld USD, capex (ok. 6,6–7,1 mld USD), dług netto, komentarz o kontraktach Microsoft i aktualizacja wytycznych ARR 7–9 mld USD.
- **Anthropic (ANT):** cena emisji względem widełek 70–78 USD, zachowanie w pierwszym tygodniu notowań i wpływ na spready CDS neocloudów (CoreWeave).
- **Decyzja FCC/Białego Domu w sprawie transceiverów 3,2T:** zakres (nowe modele vs wszystkie), okres przejściowy, lead time'y Innolight/Eoptolink vs Coherent/Lumentum.
- **SC26 (15–20.11):** wyniki warsztatu SUE-T/ESUN i BoF UEC, lista TOP500 oraz ogłoszenia CPO.
- **Barracuda ESG CVE-2026-116606/111560:** wpis do CISA KEV i ewentualne zalecenie wymiany appliance'ów, jak w 2023 r.
