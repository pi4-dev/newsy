# Newsy — 2026-10-31

**Okno przyrostowe:** od raportu 2026-10-30 do 2026-10-31. Pominięte sekcje: euro neocloud (brak nowych wiarygodnych pozycji). Ograniczenia: wyszukiwarka WWW zwróciła głównie materiały nieaktualne lub bez daty; pozycje pochodzą z newsletterów TLDR (źródła wtórne, linki do pierwotnych tam, gdzie podano) i wymagają weryfikacji. Dzień wolny (sobota) – niewielki przyrost.

## Raport technologiczny

### Technologia

#### Linux kernel: use-after-free w handshake TCP (CVSS 9.8), znaleziony przez badaczy Anthropic
Race condition w obsłudze opcji gniazda podczas handshake'u TCP: opcje mogą zostać zwolnione, zanim inne CPU skończą je czytać. Skutek: korupcja pamięci, odczyt pamięci jądra, crash. Poprawka wymusza zakończenie odczytów przez wszystkie CPU przed zwolnieniem. Red Hat i inne dystrybucje oceniają wagę niżej niż CVSS 9.8 ze względu na złożoność wyścigu.
**Znaczenie:** Ścieżka sieciowa dostępna zdalnie, więc priorytet dla hostów brzegowych, load balancerów i węzłów K8s; realne ryzyko to DoS (panic) szybciej niż stabilny RCE. Zweryfikować backport do używanych kerneli LTS, nie polegać na samym numerze wersji.
Źródło: https://thehackernews.com/2026/10/linux-kernel-tcp-flaw-with-98-cvss.html

#### Skaner podszywający się pod 32 boty AI szuka kluczy i poświadczeń; 26 400 poświadczeń w repozytoriach Gitee
Od końca września skaner podszywa się pod boty OpenAI, Anthropic, Perplexity, Meta itd. (serie 50–190 żądań, ok. 90 ścieżek: hasła, klucze API, konfiguracje, klucze SSH); 48% organizacji chronionych przez Cequence dostało taki ruch w jednym dniu. Osobno: skan 7,25 mln repozytoriów Gitee dał 26 400 działających poświadczeń (9 473 pary AccessKey Alibaba Cloud, 3 170 kluczy DeepSeek, 1 348 kluczy OpenAI).
**Znaczenie:** Allowlisty oparte na User-Agent botów AI są nieskuteczne – weryfikować po zakresach IP/reverse DNS publikowanych przez dostawców. Wyciek kluczy do chmur i API LLM to bezpośredni koszt (nadużycie kwot) – wdrożyć secret scanning i krótkie TTL.
Źródło: https://thehackernews.com/2026/10/26400-valid-cloud-and-ai-credentials.html

#### GitHub package firewall (public preview) dla Enterprise Cloud
Warstwa między agentami kodującymi/CI a rejestrami upstream: inspekcja metadanych npm i pobrań, okresy karencji (cooldown) dla nowych wersji, sprawdzanie GitHub Advisory Database, blokada downgrade'u atestacji; workflow wyjątków.
**Znaczenie:** Wprost adresuje ataki supply chain przez świeżo opublikowane paczki pobierane automatycznie przez agentów. Zależność od GitHub jako punktu kontrolnego (vendor lock-in) i przyszłe rozliczanie metered po GA; sprawdzić false positives przy cooldownie.
Źródło: https://github.blog/changelog/2026-10-20-github-package-firewall-in-public-preview/

### Biznes

#### Oracle: opłaty za zadania AI (usage-based) obok subskrypcji Fusion
Oracle planuje „AI units” rozliczane za ukończone zadania automatyczne, niezależnie od liczby licencji użytkowników.
**Znaczenie:** Przewidywalność kosztów (budżetowanie zużycia agentów) staje się zmienną FinOps także dla SaaS; ryzyko skokowych rachunków przy pętlach agentowych. Negocjować limity i alerty zużycia przed migracją procesów.
Źródło: https://www.theregister.com/applications/2026/10/29/oracle-wants-to-bill-you-for-every-ai-task-not-just-every-seat/5306403

## Raport AI-ML

### Biznes

#### Marvell w zaawansowanych rozmowach o przejęciu Lumentum (ok. 91 mld USD) – doniesienia, niepotwierdzone
Lumentum produkuje lasery i moduły optyczne dla interkonektów AI; ogłoszenie możliwe w przyszłym tygodniu. Lumentum +11% po sesji, Marvell −8%; Marvell miał ok. 3,9 mld USD gotówki w ostatnim kwartale.
**Analiza:** Konsolidacja optyki (lasery CW/EML dla CPO i 800G/1,6T) w jednym dostawcy ASIC/DSP skraca łańcuch dostaw interkonektów, ale koncentruje ryzyko dostaw laserów (wąskie gardło dla skalowania klastrów). Skala transakcji względem gotówki oznacza finansowanie akcjami/długiem. Źródło wtórne.
Źródło: https://links.tldrnewsletter.com/U15PP1

#### FuriosaAI: 620 mln USD przy wycenie 2,9 mld USD na skalowanie RNGD (180 W)
Czterokrotny wzrost wyceny; deklarowana przepustowość inferencji na poziomie GPU przy połowie mocy lub mniej.
**Analiza:** Deklaracja vendora bez niezależnych benchmarków. Dla capacity planningu istotny jest limit mocy racka (180 W/układ ułatwia chłodzenie powietrzem), ale koszt portowania stosu serwującego (vLLM/kompilator) i dostępność ekosystemu to realna bariera adopcji.
Źródło: https://app.dealroom.co/news/note/furiosaai-raises-620m-series-d-quadruples-valuation-to-2-9b-in-nvidia-challenge

#### DOJ bada przejęcie Hugging Face przez NVIDIA (12,9 mld USD) – wg Reutera
Obawy o kontrolę nad główną dystrybucją modeli open i możliwość pogorszenia pozycji rywali (chipy, firmy AI).
**Analiza:** Ryzyko vendor lock-in na poziomie łańcucha narzędzi (hub modeli, formaty, optymalizacje pod CUDA/TensorRT). Planować mirrory artefaktów modeli i niezależność od jednego huba. Źródło wtórne.
Źródło: https://www.tomshardware.com/tech-industry/artificial-intelligence/dept-of-justice-reportedly-probes-nvidia-over-usd13-billion-hugging-face-ai-platform-acquisition-concerns-raised-over-competition-in-ai-and-pressure-on-customers

### Technologia

#### Eclypsium: modyfikacja wag modelu w pamięci po przejściu kontroli integralności
Plik modelu, kontener, hash i podpisy na dysku pozostają poprawne, a wagi w pamięci są zmieniane przez ścieżki: firmware hosta, BMC, DPU. Raport: systemy Nvidia, AMD, Intel i Arm są podatne. Proponowana ochrona: DPU egzekwujące politykę niezależnie od systemu AI oraz pełna inwentaryzacja komponentów.
**Analiza:** Atestacja na poziomie artefaktu (podpisy obrazów) nie obejmuje stanu runtime; wymagany measured boot, atestacja GPU/CC i izolacja BMC. W połączeniu z UEFI CVE-2026-97631 (edycja 30.10) firmware serwerów inferencyjnych to główna powierzchnia ataku na integralność modeli.
Źródło: https://www.cybersecuritydive.com/news/ai-hardware-model-tampering-eclypsium/873177/

#### Mistral Large 4: otwarte wagi, ok. 1 bln parametrów, pod self-hosting
Deklarowana czołówka open-weight w agentowym kodowaniu, zdolności cyber porównywalne z GLM-5.3.
**Analiza:** Trylionowy model wymaga wielu węzłów (parallelizm tensor/expert, FP8/FP4) – kluczowe są liczba aktywnych parametrów (MoE?) i wymagania pamięci, których newsletter nie podaje. Wzmacnia alternatywę dla API zamkniętych (kontrola danych i kosztu tokena); dojrzałość serwowania w vLLM do sprawdzenia.
Źródło: https://mistral.ai/news/mistral-large-4-oss/

#### Epoch AI: modele open-weight zamieniają opublikowane poprawki w exploity przed wydaniem do użytkowników
GLM 5.3 potrafi znajdować i wykorzystywać luki w niewydanych jeszcze wersjach Firefoksa na podstawie publicznych commitów z poprawkami.
**Analiza:** Okno patch gap się kurczy, szczególnie dla open source z publicznym kodem. Skracać czas od merge do wdrożenia, rozważyć embargo na commity bezpieczeństwa, a w ocenie ryzyka zakładać automatyczną weaponizację diffów.
Źródło: https://epoch.ai/publications/open-weight-models-patch-gap

## Raport AI for networking

#### PsiQuantum: optyczny przełącznik obwodowy 8×8, 1,2 dB strat, <3 ms rekonfiguracji
Platforma krzemowo-fotonowa; typowa strata wtrąceniowa 1,2 dB, przesłuch 40 dB, rekonfiguracja poniżej 3 ms bez wzmacniania. Próbki u wybranych klientów, produkty komercyjne „w ciągu miesięcy”.
**Analiza:** OCS w fabrykach AI zmniejszają liczbę warstw przełączania elektrycznego i moc na bit, ale 8×8 to port count daleko od skali klastra (rząd tysięcy portów); rekonfiguracja ms pasuje do kolektywów o przewidywalnym wzorcu (rekonfiguracja między fazami treningu), nie do ruchu per-flow. Wymaga integracji sterowania z schedulerem/topologią (Digital Twin). Brak danych o kosztach i niezawodności w skali.
Źródło: https://www.psiquantum.com/news-import/psiquantum-unveils-optical-circuit-switch-technology-built-on-advanced-low-loss-silicon-photonics-platform

## Ryzyka rynku AI/GPU

#### Koncentracja i finansowanie dostaw optyki: Marvell/Lumentum przy ~3,9 mld USD gotówki
(DOSTAWY/FINANSE) Transakcja ok. 91 mld USD względem kasy rzędu 3,9 mld USD wymaga dużego finansowania akcjami lub długiem; spadek Marvella o ponad 8% pokazuje sceptycyzm rynku. Jeśli dojdzie do skutku, integracja może opóźnić roadmapy optyki dla klastrów. Doniesienia, niepotwierdzone przez strony.
Źródło: https://links.tldrnewsletter.com/U15PP1

#### Ryzyko regulacyjne i łańcuch eksportowy: kadra wyższego szczebla singapurskiej firmy DC jako agent chińskiego wywiadu (NYT)
(DOSTAWY) Amerykańscy urzędnicy uznali, że Alice Huang z singapurskiej spółki DC, podejrzewanej o kierowanie chipów Nvidii do Chin, była aktywem chińskiego wywiadu. Zaostrzenie kontroli eksportowych i weryfikacji odbiorców (KYC dla DC w Azji PD) może ograniczyć dostępność GPU u pośredników i neocloudów oraz wydłużyć terminy dostaw. Źródło wtórne (NYT, paywall).
Źródło: https://www.nytimes.com/2026/10/29/us/politics/nvidia-china-jensen-huang-trump.html

#### GitHub: 7,4 mld commitów we wrześniu (5x r/r), +4 mln rdzeni
(POPYT/INFRASTRUKTURA) Wzrost obciążenia napędzany agentami kodującymi wymusza dołożenie mocy obliczeniowej, by uniknąć przestojów; sygnał rosnącego zużycia inferencji/CI wokół agentów, ale też presji na niezawodność usług zależnych od jednego dostawcy.
Źródło: https://www.theregister.com/ai-and-ml/2026/10/28/github-powering-up-four-million-extra-cores-to-stop-its-service-going-down/5305863

## Newsletters summary

#### Cisco IT: –75% wolumenu telemetrii endpointów w Splunk Observability Cloud
Przyjęcie modelu „right-sized telemetry” obniżyło wolumen danych o 75% przy poprawie troubleshootingu.
**Analiza:** Redukcja kosztu ingestu przez filtrowanie/próbkowanie u źródła; ryzyko utraty sygnałów do forensics – zachować pełny zapis dla klas zdarzeń security.
Źródło: https://blogs.cisco.com/cisco-on-cisco/cisco-its-endpoint-observability-strategy-right-sized-telemetry-for-splunk-observability-cloud

#### Hacktron: backdoor w Bash możliwy przez GNU Savannah (26 podatności, wszystkie naprawione)
Niewyescapowany adres e-mail wstrzyknięty do polecenia powłoki dał RCE na backendzie forge; potem przejęcie konta maintainera Bash, dostęp do klucza SSH umożliwiającego push do upstreamu oraz root przez niezałatany kernelowy bug na serwerze backupu GNU.
**Analiza:** Klasyczny łańcuch supply chain: wstrzyknięcie argumentów w skryptach pomocniczych. Wnioski: argumenty procesów jako tablice, local-part e-maila jako dane niezaufane, weryfikacja backportów w LTS.
Źródło: https://www.hacktron.ai/blog/hacking-gnu-savannah

#### Polowanie na złośliwe IP-KVM (Nebulock)
IP-KVM (np. PiKVM) daje wejście indistinguishable od lokalnego użytkownika; używane przez północnokoreańskich pracowników IT w „farmach laptopów”. Heurystyki: telemetria USB, rzadkie kompozytowe urządzenia HID z storage/siecią/wideo, korelacja z terminem onboardingu.
**Analiza:** Sygnały są do triage, nie do atrybucji (VID/PID można podrobić); wdrożyć allowlistę urządzeń USB na endpointach uprzywilejowanych.
Źródło: https://nebulock.io/blog/threat-hunting-for-malicious-ip-kvm-usage

#### Przedsiębiorstwa przechodzą na chińskie modele open-weight dla kontroli kosztu tokena
Red Hat i inni rozdzielają obciążenia między modele zamknięte i otwarte; bariery: integracja, infrastruktura, geopolityka.
**Analiza:** Routing per zadanie (klasyfikacja trudności) obniża koszt, ale wymaga własnej warstwy ewaluacji i polityki pochodzenia modeli (zgodność, łańcuch dostaw wag).
Źródło: https://aibusiness.com/generative-ai/chinese-ai-invasion-gives-enterprises-new-way-control-cost

## Implikacje praktyczne
1. Integralność runtime modeli: budżetować atestację firmware/BMC/DPU i measured boot dla floty inferencyjnej; sam podpis artefaktu nie wystarcza.
2. Łańcuch dostaw optyki jest wąskim gardłem – kontrakty wieloźródłowe na lasery/moduły 800G–1,6T i planowanie na ryzyko konsolidacji (Marvell/Lumentum).
3. Model hubs i lock-in: mirrory artefaktów modeli i niezależność od jednego dystrybutora (wątek DOJ/HF).
4. Skrócić patch gap: zakładać automatyczną weaponizację publicznych poprawek przez modele open-weight; SLA wdrożeń dla krytycznych komponentów w dniach.
5. Rozliczenia usage-based dla agentów (Oracle itd.): FinOps na poziomie zadań, limity i alerty w umowach.

## Trend tygodnia
Obszar bezpieczeństwa i infrastruktury AI zbiega się wokół jednego problemu: warstwy poniżej aplikacji (firmware, BMC, DPU, optyka, łańcuch dostaw sprzętu) stają się głównym polem ryzyka. Eclypsium pokazuje, że integralność modelu da się naruszyć po pozytywnych kontrolach na dysku, a luki w UEFI z poprzedniej edycji zwiększają tę powierzchnię. Równolegle modele open-weight (GLM-5.3, Mistral Large 4) obniżają koszt znajdowania i wykorzystywania podatności, co skraca okno między poprawką a exploitem. Po stronie rynku konsolidacja optyki i finansowanie sprzętu przez dług i akcje (Marvell/Lumentum, FuriosaAI) zwiększają wrażliwość na zmiany nastrojów, a nadzór regulacyjny (DOJ, kontrola eksportu) dotyka dystrybucji modeli i chipów. Dla architekta oznacza to potrzebę dywersyfikacji dostawców, atestacji end-to-end i kosztów rozliczanych za zadanie, a nie za licencję.

## To obserwować
- Oficjalne ogłoszenie (lub dementi) transakcji Marvell/Lumentum i sposób finansowania.
- Wynik dochodzenia DOJ w sprawie NVIDIA/Hugging Face.
- Komercyjna dostępność i skala portów OCS PsiQuantum.
- Benchmarki niezależne FuriosaAI RNGD i obsługa w vLLM.
- Poprawki firmware (UEFI CVE-2026-97631) oraz backporty fixa TCP do kerneli LTS.
