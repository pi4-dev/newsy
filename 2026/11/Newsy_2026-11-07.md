# Newsy — 2026-11-07

**Okno:** od raportu 2026-11-06. Pominięto sekcje: euro neocloud, AI for networking (brak nowych zweryfikowanych informacji w oknie). Źródła: newslettery z etykiety NEWSY, RSS/WWW.

## Raport technologiczny

### Technologia

#### Firecracker: ucieczka z gościa do VMM (CVE-2026-122192) + obejście seccomp przez io_uring (FC-03)
Coinspect: nieudana wymiana pamięci po odrzuceniu RAM z snapshotu zostawia nieaktualny slot pamięci KVM wskazujący na reużytą pamięć VMM; gość z userspace czyta/modyfikuje żywą pamięć VMM i uzyskuje wykonanie kodu w procesie hosta. Drugi problem (FC-03): kod VMM może przez io_uring wykonywać operacje hosta blokowane przez seccomp. Dotyczy Firecracker v0.23.0–v1.16.1; poprawki w v1.16.2 i v1.17.0.
**Znaczenie/Analiza:** Firecracker to podstawa sandboxów dla agentów AI/FaaS. Multi-tenant z restore ze snapshotów (balloon/discard RAM) wymaga pilnej aktualizacji; seccomp bez ograniczenia io_uring nie jest granicą izolacji. Sprawdzić też jailer i uprawnienia procesu VMM.
Źródło: https://www.coinspect.com/blog/firecracker-escape/

#### *sshole: logowanie SSH bez hasła w OpenSSH + musl bez PAM
Ars Technica: podatność pozwala na logowanie po SSH bez znajomości hasła w systemach z OpenSSH na musl i bez PAM. Lokalizacja błędu i lista dotkniętych pakietów nie są jeszcze znane.
**Znaczenie/Analiza:** Dotyczy głównie Alpine/embedded/kontenerów i urządzeń sieciowych na musl. Do czasu poprawek: inwentaryzacja hostów musl z sshd, wymuszenie kluczy/AllowUsers, ograniczenie ekspozycji portu 22.
Źródło: https://arstechnica.com/security/2026/11/vulnerability-allows-attackers-to-log-into-some-linux-devices-sans-password/

#### Claude Dispatch: RCE przez rozszerzenie przeglądarki, brak poprawki
The Register: rozszerzenie z dostępem do claude.ai może przez Dispatch wysyłać prompty do Claude Desktop i uzyskać RCE (współdzielone sesje web/desktop, powtarzane kliknięcia zatwierdzeń, złośliwy pakiet npm); możliwe przejście z sesji prywatnej na komputer służbowy. Anthropic zna problem od maja, bez poprawki.
**Znaczenie/Analiza:** Agenci desktopowi z autoryzacją sesji tworzą nową granicę zaufania przeglądarka→host. Ograniczyć rozszerzenia przeglądarek na stacjach z agentami, rozdzielić profile, wyłączyć Dispatch w polityce, jeśli możliwe.
Źródło: https://www.theregister.com/research/2026/11/05/anthropic-wont-fix-a-claude-dispatch-flaw-that-could-let-attackers-execute-remote-commands-across-corporate-boundaries/5307684

#### Storm-2603: bezplikowy webshell w w3wp.exe (ToolShell CVE-2025-53770)
Securelist/Kaspersky: webshell ładowany bezpośrednio do pamięci procesu IIS, aktywny tylko dla określonego User-Agent, tunel suo5, eskalacja EfsPotato, następnie podatny Velociraptor i sterownik EDR-killer; próba Warlock zablokowana. Wskazówka detekcyjna: ścieżki IIS rozwiązywane w konfiguracji bez pliku na dysku (ETW IIS-Configuration 25 i 47); Kaspersky wydał CLRDeserializationExploitFinder.
**Znaczenie/Analiza:** Detekcja oparta o FIM jest ślepa na ten wariant; potrzebna telemetria ETW z IIS i hunting na anomalie konfiguracji. Reagować też na nadużycie legalnych narzędzi DFIR (Velociraptor).
Źródło: https://securelist.com/storm-2603-fileless-toolshell-attack/121467/

#### GhostMap (Android): boot loop system_server z aplikacji bez uprawnień; Google bez łatki
Bitdefender: wpis APK z niestandardową metodą kompresji omija kontrolę rozmiaru w libziparchive, a libandroidfw mapuje go w pełnym zadeklarowanym rozmiarze; odczyt zasobu daje SIGBUS. Aplikacja może wprowadzić system_server w pętlę rozruchu usuwalną tylko factory resetem. Android 10–17; Google odmówił poprawki bezpieczeństwa.
**Znaczenie/Analiza:** Dla flot MDM/BYOD: blokować APK z metodami kompresji innymi niż stored/deflate na bramce (skan przed instalacją); ryzyko DoS urządzeń kontrolowanych przez aplikację spoza sklepu.
Źródło: https://www.bitdefender.com/en-us/blog/labs/ghostmap-android

#### Chainalysis: wyciek danych wszystkich użytkowników po ataku z „agentic techniques”
The Record: skradzione nazwy, loginy, hashe haseł i aktywne tokeny wszystkich użytkowników; część usług (m.in. KYT) czasowo wyłączona.
**Znaczenie/Analiza:** Aktywne tokeny w wycieku oznaczają konieczność unieważnienia sesji, nie tylko rotacji haseł. Kolejny przykład ataku automatyzowanego agentami przeciw dostawcy SaaS.
Źródło: https://therecord.media/chainalysis-all-users-hit-by-agentic-cyberattack

#### Google: postmortem awarii w Londynie – generatory nie wystartowały
Wada czujnika napięcia i błędna konfiguracja kontrolerów uniemożliwiły start generatorów; jedna linia zasilania była w naprawie, gdy padła druga, a baterie UPS wyczerpały się przed przywróceniem zasilania.
**Znaczenie/Analiza:** Klasyczny failure mode N+1 przy konserwacji: okno prac na jednej ścieżce zasilania obniża redundancję do N. Weryfikować testy ATS/generatorów pod obciążeniem i politykę zakazu prac przy zdegradowanej redundancji.
Źródło: https://assets.theregister.com/2026/11/05/20265/

#### AWS Security Hub: inwentarz SBOM dla EC2, ECR i Lambda
Organizacyjny inwentarz pakietów budowany z SBOM Amazon Inspector, dostępny przez API bez dodatkowych kosztów tam, gdzie włączony jest Inspector.
**Znaczenie/Analiza:** Skraca czas odpowiedzi „gdzie działa pakiet X” przy zero-dayach typu musl/OpenSSH; wymaga włączonego Inspectora (koszt skanowania).
Źródło: https://aws.amazon.com/about-aws/whats-new/2026/11/security-hub-sbom-inventory/

### Biznes

#### Qualcomm dostarczy radia MIMO i baseband dla naziemnej sieci 5G Starlink Mobile (SpaceX)
Wieloletnia umowa na Dragonwing QRU100/QDU100 (64T64R massive MIMO, oprogramowanie warstwy fizycznej); dziesiątki tysięcy lokalizacji w USA od 2027.
**Znaczenie/Analiza:** SpaceX wchodzi jako czwarty operator infrastrukturalny w USA z architekturą hybrydową sat+naziemna; presja na Ericsson/Nokia w RAN open-ish i na modele wholesale.
Źródło: https://www.cnbc.com/2026/11/04/spacex-strikes-deal-with-qualcomm-for-mobile-network-infrastructure.html

#### CXMT: deklaracja DRAM 4F² (niezweryfikowana), spadek Samsung/SK Hynix o ok. 13% w dzień
Pionowy gate surrounding transistor i hybrid bonding wafli komórek oraz układów peryferyjnych (analityk: ok. +30% gęstości); uzyski nieznane, produkt serwerowy obiecany do końca roku. CXMT ma ok. 10% rynku DRAM.
**Znaczenie/Analiza:** Jeśli potwierdzone, presja cenowa na DRAM serwerowy/HBM-adjacent w 2027; do czasu danych o uzyskach traktować jako sygnał rynkowy, nie dostawczy.
Źródło: https://links.tldrnewsletter.com/Fn9clj

#### Microsoft: nieujawnione limity Copilot Chat bez licencji M365 Copilot
Rozszerzanie limitów od 2 listopada (małe organizacje) do 9 listopada (>2000 stanowisk); wartości limitów nie są publikowane ani konfigurowalne przez administratorów.
**Znaczenie/Analiza:** Brak przewidywalności kosztu/SLA po stronie administratora; w planach wdrożeniowych nie zakładać gwarantowanej pojemności.
Źródło: https://www.theregister.com/ai-and-ml/2026/11/05/copilot-chat-has-usage-caps-and-microsoft-would-rather-you-found-them-by-hitting-them/5307814

## Raport AI-ML

### Technologia

#### Supermicro + IBM + NVIDIA: KV Cache w trwałym, współdzielonym magazynie
Serwery Petascale (AMD EPYC 9005), IBM Storage Scale, NVIDIA Dynamo i Spectrum-X zachowują stany KV poza HBM/DRAM. Deklarowane do 56x krótszy TTFT dla dużych modeli przy długim kontekście i przepływach agentowych (testy producenta).
**Znaczenie/Analiza:** Wynosi pamięć KV do tieru storage: poprawia wykorzystanie GPU przy długich kontekstach, ale przenosi wąskie gardło na sieć/IO (Spectrum-X, RDMA) i zwiększa lock-in do stosu Dynamo/Scale. Do capacity planningu: pojemność i latencja tieru KV, polityki eviction, izolacja tenantów.
Źródło: https://www.hpcwire.com/off-the-wire/supermicro-launches-kv-cache-solution-with-ibm-and-nvidia/

#### Inkling 2 Small (Thinking Machines): MoE 594B/17B aktywnych, wagi Apache 2.0 w przyszłym tygodniu
Multimodalny (obraz) model MoE ze zmiennym „effort”; na start zamknięte wagi przez Tinker, pełne wagi po testach bezpieczeństwa i red-teamingu.
**Znaczenie/Analiza:** 17B aktywnych przy 594B całkowitych: koszt obliczeń jak model średni, ale pamięć na wagi jak duży (tensor/expert parallel, min. wiele GPU 80–192 GB przy FP8/FP4). Warto planować test self-hostingu po publikacji wag.
Źródło: https://thinkingmachines.ai/news/inkling-2-small/

#### NVIDIA Cosmos 3 (4 otwarte modele, po 16B) i VFlash
Cosmos 3: modele do interaktywnej symulacji świata i scen jazdy z generacją wielosensorową; VFlash generuje predykcje geometryczne (trajektorie, ruch 3D) równolegle, a nie token po tokenie.
**Znaczenie/Analiza:** Ścieżka robotyka/AV: równoległe dekodowanie zmniejsza latencję pętli sterowania; po stronie infrastruktury to obciążenia o innym profilu niż LLM (batch symulacji, wysokie wymagania pamięci wideo).
Źródło: https://huggingface.co/blog/nvidia/cosmos-interactive-world-simulation

#### GitHub: nowa architektura storage Git – 5000 równoległych agentów na repo
Tysiące zapisów/s i skalowalne odczyty z niską latencją, zaprojektowane pod ruch generowany przez agentów.
**Znaczenie/Analiza:** Sygnał, że ruch agentowy (zapisy, klony) zmienia profil obciążenia platform deweloperskich; self-hosted Git/CI wymaga przeglądu limitów współbieżności.
Źródło: https://github.blog/engineering/architecture-optimization/inside-githubs-new-git-storage-architecture/

### Biznes

#### Apple: limity wydatków pracowników na AI (100 USD/dzień inżynierowie, 25 USD pozostali), preferencja modeli lokalnych
Doniesienia nieoficjalne (ITPro).
**Znaczenie/Analiza:** Koszt tokenów agentowych staje się pozycją budżetową; wzrost zainteresowania modelami lokalnymi/self-hosted i bramkami z kwotami.
Źródło: https://www.itpro.com/technology/artificial-intelligence/apple-set-to-cap-employee-ai-use-amid-rising-costs

#### Ornn / TLDR: struktura rynku pierwotnego mocy GPU – rezerwacje, on-demand, spot
Rezerwacje służą finansowaniu przez przewidywalny przychód, on-demand daje elastyczność przy wyższej cenie, spot monetyzuje wolną moc. Indeksy Ornn śledzą wykonane transakcje on-demand, bez rezerwacji i spotu.
**Znaczenie/Analiza:** Przy planowaniu pojemności rozdzielić bazę (rezerwacja) od piku (on-demand); indeks on-demand jest wskaźnikiem krańcowej ceny mocy.
Źródło: https://links.tldrnewsletter.com/Fs66ME

## Ryzyka rynku AI/GPU

#### Teksas: zapowiedź moratorium na zatwierdzanie data center – 13,3 GW zapowiedzianych projektów zagrożone
Gubernator-elekt Hinojosa zapowiada wstrzymanie zatwierdzeń do czasu nowych ustaw (sesja w styczniu); odchodzący gubernator Abbott wstrzymał nowe przyłączenia do sieci i zlecił audyt. Teksas to ok. 25% planowanej mocy DC w USA.
**Znaczenie/Analiza:** Ryzyko infrastrukturalne: opóźnienia energizacji i PPA, przesunięcia z „rozpoczętych” do „oddanych” DC. Wpływa na kontrakty z terminami dostaw mocy i na wyceny dewelperów opartych na Teksasie.
Źródło: https://links.tldrnewsletter.com/AF2VNc

#### Anthropic IPO: inwestorzy oczekują obniżenia wyceny z 1,5 bln do ok. 1 bln USD, raise ok. 80 mld zamiast 100 mld (WSJ)
Termin: tydzień przed Thanksgiving. WSJ wiąże nastroje z wynikami wyborów śródokresowych, które wzmocniły sceptyków budowy DC w USA.
**Znaczenie/Analiza:** Obniżona wycena lidera zmniejsza apetyt na finansowanie długiem kontraktów compute pod laboratoria; kanał ryzyka: neocloudy z koncentracją klientów (labs) i refinansowanie. Źródło płatne (WSJ), brak niezależnego potwierdzenia liczb.
Źródło: https://www.wsj.com/finance/major-investors-are-skeptical-anthropic-ipo-can-achieve-1-5-trillion-valuation-ca965d8b

#### WSJ: „Election day revolt” przeciw budowie mocy obliczeniowych
Wyniki wyborów wskazują na silny sprzeciw wyborców wobec rozbudowy compute; wstrzymanie budowy pogłębia deficyt mocy u firm już mających problem z jej pozyskaniem.
**Znaczenie/Analiza:** Rośnie ryzyko regulacyjne lokalizacji (moratoria, taryfy dla DC); w planie capex uwzględnić lokalizacje poza USA i alternatywy energetyczne.
Źródło: https://www.wsj.com/tech/ai/the-election-day-revolt-that-threatens-americas-ai-boom-03a53b52

#### CoreWeave: dług ok. 50 mld USD, wciąż bez zysku (LSEG); wcześniej nieudane finansowanie Blue Owl (luty) i pożyczka DDTL 8,5 mld USD (kwiecień)
Tło z wyszukiwania WWW; w oknie nie znaleziono nowego zdarzenia z listopada. CDS dla CoreWeave i Oracle wskazywane na rekordowych poziomach (Seeking Alpha, data niepewna).
**Znaczenie/Analiza:** Pozycja monitorowana: koszt długu vs. przychód i koncentracja klientów; brak nowego zdarzenia – informacja kontekstowa, nie alert.
Źródło: https://www.macrobusiness.com.au/2026/10/coreweave-a-loose-thread-in-the-fabric-of-ai/

## Newsletters summary

### Podsumowanie

#### TLDR AI/IT/InfoSec/Hardware/Tech – elementy pominięte w sekcjach powyżej
Cloudflare: ramy zaufania dla agentów AI (weryfikacja tożsamości i rozliczalność agentów działających w imieniu użytkowników). Google Cloud: Agentic Data Cloud – Lakehouse na Apache Iceberg, Knowledge Catalog; AlloyDB Express (provisioning <2 s, scale-to-zero). Confluent Lightning Tables GA na AWS (SQL/API nad żywymi danymi Kafka). MariaDB Enterprise Platform 2027 (+56% szczytowej przepustowości vs 11.8, Raft Cluster multi-region active-active). Norad: 27 przejętych kont w portalu grantów, potencjalnie 20 614 użytkowników. Kela: ponad 300 domen phishingowych banków pokazujących „suspended” przy ponownej wizycie (omijanie takedownów). Apple: skan współdzielonych linków iCloud (utrata E2EE dla publicznych linków). Trellix: spadek aktywności państwowej Chin o 41,4% w Q2–Q3 (możliwy artefakt widoczności). METR: ocena przyspieszenia AI R&D w Anthropic (ok. 2x szybsze usprawnianie na dobrze zdefiniowanych zadaniach).
**Znaczenie/Analiza:** Dla architekta: bramki dla agentów (tożsamość/delegacja) oraz skracanie okna exploitacji przez AI (a16z, „the exploit was the bottleneck”) to kierunek dla polityk patchowania; Iceberg/Kafka-SQL redukują duplikację warstwy serwującej, ale zwiększają zależność od vendor-specific katalogów.
Źródło: https://blog.cloudflare.com/trusted-agent-delegation

## Implikacje praktyczne
1. Izolacja agentów/sandboxów: nie traktować mikroVM ani seccomp jako jedynej granicy (Firecracker, Dispatch); projektować defense-in-depth z odseparowanymi hostami per tenant i szybkim kanałem aktualizacji VMM.
2. Okno patchowania skraca się do godzin dzięki AI-assisted exploit dev; budżetować automatyczne wdrażanie poprawek i inwentarz SBOM jako stały element operacji.
3. Lokalizacja i energia DC stają się ryzykiem politycznym (Teksas, wybory); w horyzoncie 3–5 lat dywersyfikować regiony i umowy energetyczne oraz unikać koncentracji w jurysdykcjach z moratoriami.
4. Pamięć KV jako tier storage (Dynamo, Supermicro/IBM) zmienia kalkulację GPU vs storage; planować sieć RDMA i polityki cache z myślą o lock-inie do jednego stosu.
5. Koszt tokenów i limity (Copilot, Apple caps): wprowadzić bramkę AI z kwotami i routingiem do modeli otwartych/self-hosted jako hedge cenowy i dostępnościowy.

## Trend tygodnia
Tydzień zdominowało przesunięcie ryzyka z warstwy modeli na warstwę wykonawczą i fizyczną. Po stronie bezpieczeństwa agenci stają się zarówno celem (Dispatch, Chainalysis, wcześniej Flowise), jak i narzędziem ataku, co skraca czas od ujawnienia do eksploatacji. Granice izolacji, na których opiera się infrastruktura agentowa (Firecracker, seccomp, rozszerzenia przeglądarki), okazują się mniej szczelne, niż zakładano. Równolegle finansowanie i lokalizacja mocy obliczeniowej zaczynają zależeć od polityki: moratorium w Teksasie i oczekiwana niższa wycena Anthropic pokazują, że sceptycyzm rynkowy i wyborczy może zwolnić budowę DC. Na poziomie technologii optymalizacja przenosi się z samych FLOPS na hierarchię pamięci (KV cache w storage) i architektury MoE o małej liczbie aktywnych parametrów, co zmienia planowanie pojemności. Dla operatorów oznacza to, że przewidywalność kosztu, energii i izolacji liczy się teraz bardziej niż surowa wydajność.

## To obserwować
- Wydanie wag Inkling 2 Small (Apache 2.0, ok. przyszły tydzień) i wymagania pamięciowe self-hostingu.
- Poprawki dla *sshole (OpenSSH+musl) oraz wdrożenie Firecracker v1.16.2/1.17.0 w ekosystemie.
- Decyzje legislacyjne w Teksasie (sesja w styczniu): zgody na DC i przyłączenia do sieci; liczba GW wstrzymanych.
- Wycena i skala IPO Anthropic (tydzień przed Thanksgiving) oraz spready CDS CoreWeave/Oracle.
- Dane o uzyskach DRAM 4F² CXMT i ceny DRAM serwerowego w Q1 2027.
