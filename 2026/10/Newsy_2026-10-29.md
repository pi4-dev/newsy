# Newsy — 2026-10-29

**Okno przyrostowe:** od raportu 2026-10-28 do 2026-10-29. Pominięte sekcje: AI for networking – brak nowych wiarygodnych pozycji (wątek Cisco PIA ujęto w Newsletters). Ograniczenia: dane Q3 Alphabet/Microsoft pochodzą z newslettera (obliczenia autora), niezweryfikowane w raportach pierwotnych; wyszukiwarka nie zwróciła prymarnych źródeł Q3.

## Raport technologiczny

### Technologia

#### Microsoft Authenticator (Android): eskalacja lokalna – złośliwa aplikacja uzyskuje tokeny kont służbowych (poprawka 6.2606.4246)
Authenticator walidował tylko to, czy deklarowana tożsamość wygląda poprawnie, a nie czy zgadza się z wywołującym. Aplikacja bez uprawnień mogła podszyć się pod zaufaną aplikację Microsoft i otrzymać token do poczty, plików i kalendarza (profil służbowy). Microsoft: exploitation wymaga ukończenia przez użytkownika przepływu logowania, brak znanych ataków.
**Analiza:** Wymusić minimalną wersję aplikacji w MDM/Intune (compliance), ograniczyć czas życia tokenów i włączyć token protection/CAE; przejrzeć logi sign-in pod kątem nietypowych aplikacji klienckich.
Źródło: https://thehackernews.com/2026/10/microsoft-authenticator-flaw-let.html

#### Fortinet FortiSIEM: krytyczna luka CVE-2026-106173 (CVSS 9.8) z publicznym PoC (alert CSA, 22.10)
Dotyczy FortiSIEM 7.5.0; zalecana aktualizacja do 7.5.1+. Publiczny PoC skraca czas do masowej eksploatacji. SIEM trzyma poświadczenia do źródeł logów i często ma szerokie uprawnienia sieciowe.
**Analiza:** Kompromitacja SIEM = dostęp do kolektorów i kont serwisowych oraz możliwość ślepoty detekcyjnej; ograniczyć ekspozycję interfejsów zarządzania, po patchu rotować poświadczenia kolektorów.
Źródło: https://www.csa.gov.sg/alerts-and-advisories/alerts/al-2026-153/

#### GMO Internet: włamanie do Onamae Desktop Cloud (potencjalnie 27 tys. serwerów), żądanie ok. 50 mln USD
Atakujący uzyskał dostęp do usługi pulpitów dla traderów; GMO wyłączyło usługę 21.10, klienci zgłaszają nieautoryzowaną aktywność na kontach tradingowych, domniemane dane klientów pojawiły się w sieci (GMO weryfikuje). Wektor wejścia nieujawniony.
**Analiza:** Scenariusz kompromitacji dostawcy VDI/DaaS: pojedynczy tenant control plane kompromituje tysiące hostów. Wymagać od dostawców segmentacji tenantów, niezależnego logowania i planu odtworzenia bez reuse poświadczeń.
Źródło: https://techcrunch.com/2026/10/27/japan-internet-giant-gmo-says-hackers-posted-stolen-customer-data/

## Raport AI-ML

### Technologia

#### Oracle Cloud pierwszym hyperscalerem poza Google oferującym TPU (od 2027)
TPU w liquid-cooled DC OCI, spięte przez SmartNIC fabric Acceleron. Google deklaruje dla TPU 8. generacji do 12,6 PFLOPS FP4 na chip, do 2x perf/W, >97% goodput z optycznymi przełącznikami (OCS), >50% mniej hopów i do 80% lepszy koszt low-latency serving. Parametry vendorowe, bez niezależnych benchmarków.
**Analiza:** Rozszerza wybór poza CUDA (JAX/XLA, wymaga portowania stosu inferencyjnego); w capacity planningu pojawia się alternatywa dla GPU, ale dostępność dopiero 2027, więc nie rozwiązuje deficytu 2026–27. Lock-in przesuwa się do kompilatora/runtime.
Źródło: https://cloud.google.com/blog/products/infrastructure/bringing-google-tpus-to-oracle-cloud-infrastructure

#### Cloudflare: cross-request KV cache exposure w GLM-5.3 (Workers AI) – dezagregowany SGLang
Prompty >681 280 tokenów zwróciły dane innych klientów w 155 odpowiedziach: dezagregowany SGLang oznaczał żądanie jako odrzucone, ale decode dostawał 1-tokenowy KV cache z nienaruszonymi stronami poprzednich sesji. Pula KV prefill była mniejsza niż reklamowane 1M tokenów, a walidacja przed startem sprawdzała tylko kody HTTP. Okno 27.08–9.09, brak dowodów celowej eksploatacji. Remediacja: SGLang ≥ 0.5.22, weryfikacja rozmiaru puli KV vs kontekst na każdym węźle, flaga --disaggregation-enable-kv-checksum.
**Analiza:** Multi-tenant inferencja z dezagregacją prefill/decode i współdzielonym KV cache to nowa klasa wycieku między tenantami; testy akceptacyjne muszą obejmować graniczne długości kontekstu i zawartość odpowiedzi, nie tylko status.
Źródło: https://blog.cloudflare.com/kv-cache-exposure-remediation

#### NVIDIA Mission Control 2.5: autonomiczne odtwarzanie w Kubernetes oraz Minerva (telemetria → GPU yield) (26.10)
Mission Control 2.5 automatyzuje monitoring i recovery w klastrach AI; Minerva koreluje telemetrię sieci i obliczeń AI Factory dla poprawy uzysku GPU. Treść blogowa bez liczb o MTTR i narzucie agentów telemetrycznych.
**Analiza:** Wpływa na goodput dużych jobów (krótszy czas niedostępności węzłów), ale pogłębia zależność od stosu NVIDIA (Mission Control/Run:ai/KAI); weryfikować na własnych failure modes (NVLink, XID, awarie NIC).
Źródło: https://developer.nvidia.com/blog/how-nvidia-mission-control-2-5-brings-autonomous-recovery-to-kubernetes/

## euro neocloud

### Nebius

#### Nebius: publiczne ceny spot GPU na żywo, AI Builder Program w Europie, Lappeenranta (do 310 MW, 2027) (21.10)
Nebius opublikował żywe ceny spot GPU i rozszerzył program z NVIDIA na Europę. Planuje DC w Lappeenranta w Finlandii do 310 MW (dostawy od 2027); kontraktowana moc EMEA >750 MW, w tym Lille (240 MW docelowo). Wcześniej ujęto emisję 5 mld USD convertible (21.10).
**Ocena analityczna:** Sygnały ostrzegawcze: jawne ceny spot oznaczają presję cenową/ komodyfikację GPU-h; spadek akcji Nebius o ok. 5% 20.10 w parze z CoreWeave (przegląd długu neocloudów). Ocena ryzyka: ŚREDNIE – duża kontraktowana moc i dostęp do rynku kapitałowego, ale wysoka wrażliwość na koszt kapitału i harmonogram energii; brak sygnałów o problemach SLA.
Źródło: https://nebius.com/newsroom/nebius-brings-compute-pricing-into-the-open-as-europe-s-ai-builders-scale-on-its-cloud

## Ryzyka rynku AI/GPU

### Wpisy

#### Q3 2026: Alphabet i Microsoft – chmura przyspiesza, ale Alphabet finansuje capex zewnętrznie
Kwartał do 30.09: Google Cloud >2x r/r do 30,7 mld USD, Azure +48% do 33,2 mld USD. Gotówkowy capex = 40% przychodów (Alphabet) i 43% (Microsoft; 49% z finance leases). Microsoft: ok. 63 USD operacyjnego cash na 100 USD przychodu, FCF 18,6 mld USD; Alphabet: 33 USD, wydatki przekroczyły OCF o 9,1 mld USD, w kwartale emisja not 28,7 mld USD i akcji 12,4 mld USD. Wyliczenia kwartalne autora newslettera (nie prymarne), do weryfikacji w raportach.
**Analiza:** (FINANSE) Nawet hyperscaler o najsilniejszej pozycji przechodzi na finansowanie zewnętrzne – razem z emisją Meta (95 mld USD) wzmacnia tezę, że tempo budowy DC zależy od okna rynku kapitałowego. Dla klientów: ryzyko podwyżek cen mocy lub rygorystyczniejszych kontraktów. Mniejsze podmioty (neocloudy) konkurują o ten sam kapitał.
Źródło: https://businessengineer.ai/p/alphabet-and-microsoft-on-the-map

#### CoreWeave: HSBC podtrzymuje bearish call (20.10), obligacje 2032 ok. 15% rentowności, 4,4 mld USD do spłaty/refinansowania do końca 2026
Nowe elementy względem wcześniejszych wpisów (CDS, yield 2030): HSBC podtrzymał niedźwiedzią rekomendację z powodu ryzyka długu, notowania neocloudów (CoreWeave -4%, Nebius -5%) spadły 20.10; raportowany backlog ok. 104 mld USD przy ujemnym FCF H1 2026 ok. -10,45 mld USD (obliczenia z OCF-capex, źródła agregatorowe, rozbieżne dane o długu).
**Analiza:** (FINANSE/POPYT) Backlog rośnie szybciej niż zdolność finansowania; test refinansowania 4,4 mld USD i kosztu długu rozstrzygnie, czy neocloudy utrzymają harmonogram oddawania DC. Dla klientów: ryzyko kontrahenta przy kontraktach zarezerwowanych.
Źródło: https://247wallst.com/investing/2026/10/20/coreweave-falls-4-as-hsbc-reiterates-bearish-call-on-debt-risk-nebius-drops-5-oracle-holds-steady/

## Newsletters summary

### Wpisy

#### Cisco Privileged Infrastructure Access: krótkożyjące, identity-bound uprawnienia do urządzeń sieciowych
Zastępuje długożyciowe poświadczenia i statyczne ścieżki dostępu uprawnieniami per zadanie wygasającymi po zakończeniu pracy, także dla agentów automatyzujących. Materiał vendorowy, brak danych o integracji z TACACS+/RADIUS i narzucie latencji sesji.
**Analiza:** Zgodne z kierunkiem JIT access dla NetOps/AIOps; ocenić zależność od vendora, zachowanie przy awarii kontrolera (break-glass) i audyt sesji.
Źródło: https://blogs.cisco.com/security/cisco-rethinks-privileged-infrastructure-access

#### IBM: tożsamość i kontrola dla agentów AI (Vault ze short-lived credentials, integracja z NVIDIA OpenShell, watsonx Orchestrate)
Vault wydaje krótkożyciowe poświadczenia dla tożsamości nie-ludzkich; watsonx Orchestrate wykrywa, izoluje i ocenia agentów na platformach zewnętrznych.
**Analiza:** Rozwiązuje klasę problemu z poprzednich edycji (nadmierne zgody OAuth, Gheist); skuteczność zależy od pokrycia agentów spoza platformy IBM.
Źródło: https://newsroom.ibm.com/2026-10-27-ibm-introduces-identity-and-control-for-ai-agents,-securing-frontier-intelligence-for-the-enterprise

#### Tracebit: „context bombs” – deception przeciw agentom atakującym (773 oceny w AWS)
Połączone techniki deception zmniejszyły średnią liczbę osiągniętych celów z 2,48 do 0,23 na run, wydłużyły czas 2,8–8,9x i podniosły koszt modelu 1,6–3,9x; fałszywy raport pentestu oznaczający prawdziwe cele jako kanarki obniżył skuteczne runy z 39/40 do 2/40 (4 modele).
**Analiza:** Canary tokens i decoy w IAM/S3 mają większy zwrot przeciw atakom LLM niż przeciw ludziom; wyniki z jednego laboratorium – ryzyko adaptacji atakujących i fałszywych alarmów.
Źródło: https://agentic.tracebit.com/factory/

#### Wiz: anatomia ataku na chmurę prowadzonego przez agentów AI
Ataki hałaśliwe, uporczywe, naprzemienne między fazami łańcucha; zalecenia: ujednolicić logi, zakładać pełną ekspozycję przejętych zasobów, ograniczać zaufanie między systemami.
**Analiza:** Detekcja oparta na wolumenie i powtarzalności (rate, sekwencje API) jest skuteczna; skrócić czas reakcji (automatyczne odcięcie sesji).
Źródło: https://www.wiz.io/blog/anatomy-of-ai-driven-cloud-attacks

#### Linux EDR: zielony dashboard bez telemetrii (niezgodności jądra, tryb zdegradowany)
Czujniki EDR na serwerach Linux potrafią działać w degradated mode po aktualizacji kernela. Rekomendacja: weryfikować zdrowie sensora per host, pilotować zmiany przez aktualizacje jądra, pokryć kontenery i alerty poza godzinami.
**Analiza:** Dodać do compliance monitoring metrykę „sensor w pełnym trybie” i test regresji po każdym upgrade kernela.
Źródło: https://technologymatch.com/blog/extending-edr-to-linux-servers

## Implikacje praktyczne
1. Multi-tenant inferencja (dezagregowany SGLang, współdzielony KV cache) wymaga testów izolacji tenantów na granicach kontekstu i kontroli rozmiaru puli KV; incydent Cloudflare pokazuje, że walidacja statusów HTTP nie wystarcza.
2. Kapitał stał się ograniczeniem budowy DC: Alphabet wydaje więcej niż generuje (emisje 28,7 mld USD not i 12,4 mld USD akcji), Meta planuje 95 mld USD; w kontraktach capacity zawierać klauzule wyjścia, step-in i dywersyfikować dostawców.
3. Ryzyko kontrahenta neocloudów (CoreWeave ~15% yield, 4,4 mld USD do refinansowania do końca 2026): nie koncentrować treningu na jednym dostawcy bez przenośnej warstwy (K8s + vLLM/llm-d) i bez kopii checkpointów poza dostawcą.
4. TPU na OCI od 2027 otwiera drugą ścieżkę wobec CUDA; planować przenośność (JAX/XLA vs PyTorch/CUDA) już teraz, jeśli horyzont 3–5 lat zakłada negocjacje cenowe z NVIDIA (licencje per GPU).
5. Tożsamości i dostęp uprzywilejowany (SIEM, urządzenia sieciowe, DaaS, agenci) przechodzą na model krótkożyjących uprawnień JIT; zakładać kompromitację narzędzi bezpieczeństwa (FortiSIEM) i projektować segmentację oraz rotację poświadczeń kolektorów.

## Trend tygodnia
Tydzień pokazuje, że ograniczeniem rozwoju infrastruktury AI przesuwa się z dostępności GPU na koszt i dostępność kapitału. Alphabet i Microsoft osiągają przyspieszenie chmury (Google Cloud 30,7 mld USD, Azure +48%), ale tylko Microsoft pokrywa capex z operacyjnego cash flow; Alphabet i Meta sięgają po emisje, a CoreWeave płaci około 15% rentowności za dług. Równolegle Google otwiera TPU poza własną chmurą (Oracle 2027), co osłabia monopol CUDA, lecz dopiero w kolejnych latach. Na froncie bezpieczeństwa błędy przesuwają się do warstw współdzielonych: KV cache w inferencji multi-tenant, tokeny aplikacji mobilnych i systemy zarządzające (SIEM, DaaS) dają jeden punkt kompromitacji wielu tenantów. Obrona także się automatyzuje: deception przeciw agentom (context bombs) i dostęp JIT dla agentów to odpowiedź na tanie ataki prowadzone przez LLM. Wspólny mianownik: skala i współdzielenie zasobów zwiększają zarówno efektywność, jak i promień rażenia pojedynczej awarii lub decyzji finansowej.

## To obserwować
1. Refinansowanie 4,4 mld USD CoreWeave i yield obligacji 2030/2032.
2. Raporty Amazon Q3 oraz guidance capex Alphabet/Microsoft z call'ów (potwierdzenie wyliczeń z newslettera).
3. Dostępność i ceny TPU na OCI oraz niezależne benchmarki TPU 8. generacji.
4. Poprawki FortiSIEM 7.5.1+ i dowody eksploatacji CVE-2026-106173 (CISA KEV).
5. Ceny spot Nebius względem CoreWeave/Lambda i zmiany w publicznym cenniku GPU-h.
