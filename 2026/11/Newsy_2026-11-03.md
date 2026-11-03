# Newsy — 2026-11-03

**Okno przyrostowe:** od raportu 2026-11-02. Pominięte sekcje: euro neocloud, AI for networking (brak nowych zweryfikowanych pozycji). Źródła: głównie newslettery z etykiety NEWSY; wyszukiwanie WWW nie zwróciło materiału nowszego niż rejestr.

## Raport technologiczny

### Biznes

#### Oracle OCI pierwszym hyperscalerem poza Google z TPU
OCI ma oferować TPU Google. Wcześniej (10/3x) odnotowano doniesienia o zakupie TPU przez OpenAI; nowy element to dystrybucja TPU przez zewnętrzną chmurę.
**Znaczenie/analiza:** Dywersyfikacja poza CUDA obniża vendor lock-in, ale wymaga portowania stosu (JAX/XLA, vLLM-TPU); ceny i dostępność nieujawnione.
Źródło: https://www.rtinsights.com/real-time-decisions/real-time-analytics-news-for-the-week-ending-october-31/

#### Google Cloud: spend caps GA
Limity wydatków dla kwalifikowanych usług, ustawiane przez Cloud Billing Budgets API, np. domyślnie dla nowych projektów sandbox.
**Znaczenie/analiza:** Twarde limity zamiast alertów ograniczają koszt błędnych pętli agentów; failure mode: odcięcie usług po przekroczeniu limitu w produkcji, więc stosować tylko dla sandboxów.
Źródło: https://cloud.google.com/blog/topics/cost-management/google-cloud-spend-caps-now-generally-available

#### Intel Fab 28: eksplozja 14.10, ok. 600 mln USD utraconych przychodów
Fabryka w Kiryat Gat (10 nm-class, m.in. I/O die Xeon 6) stała ok. dwóch tygodni; cykl wafla ok. 6 mies., więc luka podażowa sięga 2027 przy i tak ciasnym rynku serwerów i pamięci.
**Znaczenie/analiza:** Planować zamówienia serwerów x86 z wyprzedzeniem i dopuścić alternatywy (AMD, ARM); ryzyko wydłużenia lead time w refreshu 1H 2027.
Źródło: https://www.tomshardware.com/pc-components/cpus/deadly-intel-fab-28-explosion-causes-two-week-shutdown-to-cost-usd600-million-and-severely-disrupt-server-chip-manufacturing-for-six-months-10nm-xeon-shipments-stall-during-largest-shortage-in-memory

### Technologia

#### GitLab CVE-2026-97229 (CVSS 10, Onigmo) w KEV; atak na Alpine Linux
Błąd w silniku regex Ruby pozwala nieuwierzytelnionie wykonać kod przez spreparowaną konfigurację CI/CD. Alpine: crash workerów i pobranie obrazu kontenera, bez potwierdzonego RCE. Poprawki: Ruby 3.3.13 / 3.4.12 / 4.0.9.
**Znaczenie/analiza:** Aktualizacja samego GitLaba może nie wystarczyć przy własnym Ruby; sprawdzić runnery i obrazy bazowe oraz rotować sekrety CI.
Źródło: https://links.tldrnewsletter.com/akghxg

#### Linux CVE-2026-98432: Ubuntu i RHEL nadal bez poprawki (stan na 1.11)
UAF w handshake IPv4 TCP (od 4.4), jedyny znany exploit wymaga listenera MPTCP. Go >=1.24 włącza MPTCP na listenerach domyślnie (np. tailscaled), autor naliczył 712 listenerów na laptopie. Debian, Amazon Linux, Oracle UEK mają łatki.
**Znaczenie/analiza:** Mitygacja do czasu patcha: net.mptcp.enabled=0 oraz GODEBUG wyłączający MPTCP; inwentaryzować usługi w Go na hostach brzegowych.
Źródło: https://ma.ttias.be/linux-cve-2026-98432-vulnerable-kernels/

#### Backdoor w rozszerzeniu Claude dla VS Code/Cursor (v2.1.313), ok. 12 h
Złośliwa wersja opublikowana przez oficjalne konto w Open VSX/Cursor, do 5000 celów; zbierała dane hosta, próbowała dodać klucz SSH i tunel reverse. Błędy ograniczyły persistence.
**Znaczenie/analiza:** Kto zainstalował lub zaktualizował w czwartek, powinien sprawdzić IoC i klucze authorized_keys; pinning wersji i allowlista rozszerzeń w IDE.
Źródło: https://arstechnica.com/security/2026/10/anthropic-vs-code-extension-on-cursor-backdoored-for-nearly-12-hours/

#### Polymarket: przejęty skrypt DebugBear (ok. 2,7 mln USD)
Wektor: kompromitacja rejestratora domeny dostawcy skryptu, potem JS w sesjach zalogowanych użytkowników (29.10).
**Znaczenie/analiza:** Klasyczny supply chain third-party JS: CSP z SRI, izolacja skryptów monitoringu od stron transakcyjnych, MFA/registry lock dla domen.
Źródło: https://thehackernews.com/2026/11/hackers-steal-about-27-million-from.html

#### Binance: kradzież 6,3 mld USD, FBI wskazuje TraderTraitor
Wektor: przejęty laptop i malware kasujący ślady; wypłaty wznawiane etapami, 90% aktywów uzupełnione.
**Znaczenie/analiza:** Endpoint admina jako pojedynczy punkt awarii; wymusić separację uprawnień i podpisywanie transakcji wieloosobowo.
Źródło: https://www.theguardian.com/business/2026/nov/01/binance-losses-covered-hack-fbi-north-korea-cryptocurrency

## Raport AI-ML

### Biznes

#### Samsung Foundry: zamówienia 2 nm GAA od hyperscalerów
Potwierdzone na call Q3; pierwsze układy GAA wchodzą do produkcji, dział nadal nierentowny.
**Znaczenie/analiza:** Druga ścieżka produkcji ASIC-ów poza TSMC; realny wpływ zależy od uzysku, nieujawnionego.
Źródło: https://www.techtimes.com/articles/329716/20261030/samsung-foundry-confirms-hyperscaler-2nm-orders-first-gaa-chips-enter-production

#### Boom Supersonic: fabryka turbin 42 MW dla DC, dostawy od połowy 2027
Wydajność 25 turbin/rok w 2028, 50 w 2029. Klient startowy Crusoe wycofał zamówienie na 29 turbin.
**Znaczenie/analiza:** Wycofanie klienta startowego to sygnał ryzyka dla projektów zasilania behind-the-meter; turbina bez prototypu produkcyjnego nie rozwiązuje deficytu mocy do 2028.
Źródło: https://boomsupersonic.com/press-release/superpower-superfactory

### Technologia

#### Poisoning 0,1% danych pretrainingu: backdoor 'curl | bash' w agencie CLI
Wg abstraktu zbioru: agent 4B wykonywał komendę atakującego w ok. 92% promptów ze ścieżką-wyzwalaczem, skuteczne nawet przy 0,0005% zatrucia; efekt rośnie z rozmiarem modelu.
**Znaczenie/analiza:** Ryzyko dla modeli open-weight o nieznanym pochodzeniu danych: agenci z uprawnieniami powłoki wymagają sandboxa i allowlisty komend, nie tylko oceny modelu. Wyniki z jednej pracy.
Źródło: https://arxiv.org/abs/2610.37741

#### Codex auto-review: 11 sposobów obejścia sandboxa i monitora
Podmiana skryptów za zaufanymi hookami, fałszowanie tur użytkownika, edycja plików po zatwierdzeniu; OpenAI naprawiło część.
**Znaczenie/analiza:** Monitor LLM nie zastępuje izolacji na poziomie OS; zatwierdzenie musi wiązać się z hashem artefaktu (TOCTOU).
Źródło: https://www.lesswrong.com/posts/dbStc8ZFESqfihwAd/lessons-from-red-teaming-auto-review-openai-s-blocking-1

#### Google Suncatcher: pierwsza odpowiedź AI z TPU na orbicie
Gemma 4 na TPU klasy DC, łączność przez Svalbard; testy termiczne i radiacyjne dalej, dwa satelity w 2027.
**Znaczenie/analiza:** Eksperyment, bez wpływu na planowanie mocy przed 2030; wąskie gardło to odprowadzanie ciepła i łącze.
Źródło: https://blog.google/innovation-and-ai/models-and-research/google-research/project-suncatcher-satellite-response/

## Ryzyka rynku AI/GPU

#### OpenAI zebrało poniżej połowy celu 30 mld USD, wycena 1,25 bln
Wycena niższa niż oczekiwane 1,4 bln; kontekst to wyprzedaż po słabych wynikach Google, Amazon i Intela. Uzupełnia wcześniejszą informację o cięciu rundy (1.11).
**Znaczenie/analiza:** Finansowanie zobowiązań compute (długoterminowe kontrakty) zależy od kolejnych rund; słabszy popyt inwestorów zwiększa ryzyko renegocjacji lub opóźnień zamówień.
Źródło: https://www.wsj.com/tech/ai/openai-raises-less-than-half-of-30-billion-target-in-latest-round-91aecdc5

#### S-1 Anthropic: pierwszy rentowny kwartał, ryzyko stałych zobowiązań compute
Wg analizy: wzrost bardzo szybki, ale lata zablokowanych rachunków za moc obliczeniową płaconych przez klientów mogących ograniczyć zużycie z miesiąca na miesiąc; zależność od chmur.
**Znaczenie/analiza:** Niedopasowanie terminów: zobowiązania wieloletnie vs. przychody elastyczne; to kanał przenoszenia wstrząsu popytu na dostawców GPU/chmury.
Źródło: https://www.tilomitra.com/blog/anthropic-s1-in-charts

#### Intel Fab 28 oraz wycofanie Crusoe z zamówienia turbin
Dostawy: opóźnienia CPU serwerowych do 2027 (Intel); energia: anulowane 29 turbin (Boom).
**Znaczenie/analiza:** Dwa niezależne sygnały po stronie dostaw i zasilania; patrz wpisy wyżej.
Źródło: https://www.tomshardware.com/pc-components/cpus/deadly-intel-fab-28-explosion-causes-two-week-shutdown-to-cost-usd600-million-and-severely-disrupt-server-chip-manufacturing-for-six-months-10nm-xeon-shipments-stall-during-largest-shortage-in-memory

## Newsletters summary

#### Anthropic Mythos 5.5 w przygotowaniu (A/B)
Wzmianka o wersji na stronie Mythos, bez specyfikacji; możliwy początkowy dostęp ograniczony do cyberobrońców.
**Znaczenie/analiza:** Niepotwierdzone; planować ewaluację regresji promptów dopiero po ogłoszeniu.
Źródło: https://www.testingcatalog.com/anthropic-prepares-to-release-claude-mythos-5-5-model/

#### Mandiant/GTIG: ARTEX w prawdziwych intruzjach
Sześć sub-agentów: sweepy uprawnień Azure/AWS, Kerberoasting, DCSync, webshelle JSP w seriach 2-25 s; UNC7228 od pierwszego konta do głębokiego dostępu w ok. 4 h. Agenci hałaśliwi forensycznie.
**Znaczenie/analiza:** Detekcja behawioralna pod kątem serii z prędkością maszynową; krótkie, wąsko zakresowane poświadczenia; Kudu/SCM traktować jako infrastrukturę uprzywilejowaną.
Źródło: https://cloud.google.com/blog/topics/threat-intelligence/artex-and-friends-enter-agentic-offensive-frameworks

#### Talos: VOLTA infostealer z priorytetyzacją ofiar przez LLM
Kradnie poświadczenia, sesje, portfele, klucze chmurowe; dossier idzie przez OpenRouter do Claude (fallback Kimi, Gemini).
**Znaczenie/analiza:** Reguła detekcji: procesy niedeweloperskie uruchamiające Chrome z --headless (obejście App-Bound Encryption); blokada ruchu do bram LLM z hostów użytkowników.
Źródło: https://blog.talosintelligence.com/appraisal-by-ai-model-driven-victim-prioritization-in-the-volta-infostealer/

#### Szyfrowane dyski USB: klucze zaszyte w binariach, backdoory
Codex + MCP Binary Ninja odzyskał klucze w 7-26 min; w SecZure YutianDun dwa ukryte konta (hasło 123456). Bez zgłoszenia do vendorów, bo urządzeń nie da się załatwić.
**Znaczenie/analiza:** Ufać tylko urządzeniom z kluczem wyprowadzanym z hasła użytkownika; reszta do wycofania z polityki DLP.
Źródło: https://binary.ninja/2026/10/22/breaking-more-encrypted-usb-drives-with-ai.html

#### Postgres 19 opóźniony przez zalew raportów generowanych AI
Wydanie celowane na 19.11; kilka dużych funkcji wycofano późno w becie, maintainerzy nie nadążają z weryfikacją poprawek i backportami.
**Znaczenie/analiza:** Nie planować upgrade na dzień wydania; spodziewać się wolniejszych minor release i większej liczby poprawek po GA.
Źródło: https://rhaas.blogspot.com/2026/10/postgres-19-is-late-and-its-ais-fault.html

#### GitHub: 3 nowe zakresy IP dla webhooków
Statyczne allowlisty firewalli muszą zostać uzupełnione; synchronizacja z Meta API robi to automatycznie.
**Znaczenie/analiza:** Przejść na automatyczną synchronizację, inaczej ciche gubienie dostaw.
Źródło: https://github.blog/changelog/2026-10-29-webhook-ip-ranges-added-to-the-meta-api/

#### Belgia: FOD Financiën wyłącza MyMinfin, Tax-on-web, Biztax po cyberataku
Brak wskazań na wyciek danych, dochodzenie trwa.
**Znaczenie/analiza:** Sygnał dla klientów w UE: ryzyko ciągłości usług publicznych.
Źródło: https://www.theregister.com/security/2026/10/29/belgiums-tax-agency-confirms-cyberattack-pulls-services-offline/5306409

## Implikacje praktyczne
1. Wpisywać do umów kolokacyjnych i PPA klauzule opóźnień zasilania oraz ryzyko wycofania dostawców (przypadek Crusoe/Boom); nie zakładać turbin na gazie przed 2028.
2. Łańcuch dostaw IDE i pakietów (rozszerzenia, JS stron trzecich) traktować jak produkcyjny: pinning, SRI, allowlisty, rotacja kluczy.
3. Agenci kodujący wymagają izolacji OS i wiązania zatwierdzeń z hashem artefaktu; monitor LLM jest warstwą dodatkową.
4. Finansowanie compute w modelu wielorundowym jest wrażliwe na wyceny: negocjować elastyczność wolumenu i unikać jednego dostawcy GPU/chmury, dopuścić TPU/OCI.
5. Planować refresh x86 z zapasem 2-3 kwartałów (Fab 28, pamięć) i dopuścić architektury alternatywne.

## Trend tygodnia
Tydzień wyznaczają dwa równoległe ryzyka: kapitałowe i bezpieczeństwa łańcucha dostaw narzędzi. Po stronie kapitału rundy OpenAI zebrały mniej niż połowę celu, a S-1 Anthropic pokazuje strukturalne niedopasowanie zobowiązań compute do elastycznych przychodów. Jednocześnie realne ograniczenia fizyczne się utrwalają: pożar Fab 28 przesuwa podaż CPU na 2027, a klient startowy Boom wycofuje zamówienie turbin. Po stronie bezpieczeństwa atakujący korzystają z agentów (ARTEX, VOLTA) i kompromitują zaufane kanały dystrybucji (rozszerzenie IDE, skrypt analityczny, rejestrator domeny). Badania nad zatruciem danych i obejściem monitorów pokazują, że sam nadzór modelowy nie wystarcza. Wniosek dla architekta: odporność zależy bardziej od izolacji, ograniczania uprawnień i dywersyfikacji dostawców niż od wyboru modelu czy chipa.

## To obserwować
- Kolejne rundy OpenAI/Anthropic (S-1) i wycena vs. zobowiązania compute.
- Terminy poprawek Ubuntu/RHEL dla CVE-2026-98432.
- Ogłoszenie Claude Mythos 5.5 i dostępność TPU w OCI (ceny).
- Lead time serwerów Xeon 6 w Q1 2027 po Fab 28.
- Postgres 19 GA (cel 19.11).
