# Newsy — 2026-10-24

**Okno przyrostowe:** od raportu 2026-10-23 do 2026-10-24 07:28 CEST. Pominięto sekcje "euro neocloud" i "AI for networking" – brak nowych, wiarygodnych i datowanych pozycji. Źródła: newslettery (etykieta NEWSY: SemiAnalysis, TLDR AI/IT/InfoSec/Hardware/ogólny z 23.10), NVIDIA Developer Blog RSS, WWW.

## Raport technologiczny

### Technologia

#### PHP2Shell – aktualizacja: eksploatacja zaczęła się 17.10, cztery dni przed patchem
**Nowy element względem 23.10:** Amazon Threat Intelligence (honeypot MadPot) wykrył próby eksploatacji CVE-2026-109618/109619 od 17.10, czyli przed publicznym ujawnieniem 21.10. Patche/reguły wydali m.in. Cisco, AWS, Google Cloud, Ubuntu, Debian, Palo Alto, Sophos; CISA, NCSC UK i CERT-EU zalecają natychmiastową aktualizację, część europejskich serwisów rządowych, szkół i szpitali została wyłączona.
**Znaczenie:** Dochodzenie powłamaniowe trzeba cofnąć do 17.10, nie do daty patcha: przegląd logów dostępowych/webshelli od tego dnia oraz detekcje niezależne od sygnatur (anomalie zużycia pamięci workerów, restarty FPM/Apache). Amazon rekomenduje mierzenie czasu mitygacji w godzinach.
Źródło: https://aws.amazon.com/blogs/security/amazon-threat-intelligence-identifies-php2shell-exploitation-prior-to-public-disclosure-in-madpot/

#### GitLab: CVE-2026-110264 – krytyczny RCE bez uwierzytelnienia (integer overflow w kompilatorze regex)
**Fakty:** Łatki na 12 problemów w wersjach 18.11.13 – 19.5.1; podatne wydania sięgają wstecz do 7.0 (CE/EE). Ten sam release naprawia CVE-2026-15723 (przejęcie konta LDAP przez użytkownika SAML OmniAuth przy włączonym auto-linkowaniu LDAP). Łatki zawierają migracje bazy – instalacje single-node mogą wymagać okna serwisowego.
**Znaczenie:** Samodzielnie hostowany GitLab to repozytorium kodu, sekrety CI i runnery w jednym punkcie – RCE przed uwierzytelnieniem oznacza dostęp do pipeline'ów i kluczy wdrożeniowych. Priorytet: instancje wystawione do internetu, rotacja zmiennych CI po ewentualnej ekspozycji, w tle okno na migracje.
Źródło: https://www.infoworld.com/article/4239276/gitlab-critical-patches-include-one-for-a-bug-that-allows-code-execution-by-unauthenticated-users.html

#### Railway: atak ujawnił zaszyfrowane zmienne środowiskowe i część odszyfrowanych sekretów klientów
**Fakty:** Platforma Railway poinformowała, że atakujący uzyskali dostęp do zaszyfrowanych zmiennych środowiskowych i odszyfrowali część sekretów klientów; zaleca rotację kluczy przez „wszystkich użytkowników”.
**Znaczenie:** Szyfrowanie sekretów po stronie PaaS nie chroni, gdy klucz odszyfrowujący jest osiągalny z tej samej domeny kompromitacji. Rotacja sekretów i ograniczenie ich zakresu (krótkożyjące tokeny, OIDC zamiast statycznych kluczy) to jedyna realna mitygacja.
Źródło: https://techcrunch.com/2026/10/22/developer-platform-railway-urges-all-users-to-revoke-secret-keys-after-cyberattack/

#### PingCAP/TiDB: dostęp przez skradzione poświadczenia konta serwisowego 9.09–7.10
**Fakty:** Atakujący kopiowali dane i kod źródłowy; grupa FulcrumSec twierdzi, że zdobyła także metadane klientów, klucze API, tickety Jira, historię SQL i czteroletni log połączeń TiDB Cloud. Bazy klientów rzekomo nietknięte, ale logi zapytań i diagnostyka mogą zawierać connection stringi i sekrety. Okno niewykrytego dostępu to prawie miesiąc.
**Znaczenie:** Długożyjące poświadczenia kont serwisowych bez detekcji anomalii to powtarzalny wektor; dla użytkowników TiDB Cloud – rotacja haseł i kluczy, przegląd sekretów w historii zapytań.
Źródło: https://links.tldrnewsletter.com/aGfHyj

#### AWS: kwarantanna wycieklych kluczy IAM to nie unieważnienie (Truffle Security)
**Fakty:** Polityka kwarantanny stosowana przez AWS do wykrytych wycieklych kluczy nadal pozwala m.in. czytać Secrets Manager i Parameter Store, wykonywać polecenia na działającej instancji EC2 (SSM) i pobierać obiekty S3 przez GetObjectVersion. Wiele z przetestowanych kluczy było wystawionych od lat.
**Znaczenie:** Powiadomienie o kwarantannie traktować jak incydent: natychmiastowa dezaktywacja i rotacja klucza, przegląd CloudTrail. Dodatkowo SCP/permission boundary blokujące wymienione akcje dla użytkowników IAM z kluczami długoterminowymi.
Źródło: https://trufflesecurity.com/blog/aws-key-quarantine-is-not-revocation

#### PQC: 7 z 10 bibliotek akceptuje zdeformowane klucze prywatne ML-DSA (Quarkslab)
**Fakty:** Differential fuzzer DeltAFLy wykazał, że liboqs, AWS-LC, libcrux i inne przyjmują klucze z współczynnikami poza zakresem, co może obciążyć rejection sampling i prowadzić do wycieku materiału klucza; odrzucają je tylko OpenSSL, BoringSSL i SymCrypt. Znaleziono też kolizję OID w XMSS w RustCrypto.
**Znaczenie:** Przy migracji do PQC wybór biblioteki ma znaczenie bezpieczeństwa, nie tylko wydajności. Do planu migracji dodać testy zgodności walidacji kluczy i wymóg aktualizacji bibliotek; ML-KEM/ML-DSA są niemal powszechne, HQC/McEliece/X-Wing nadal słabo wspierane.
Źródło: https://blog.quarkslab.com/state-of-post-quantum-cryptography-implementations.html

#### Ataki na tokeny OAuth: device code, consent phishing, nadużycie redirect_uri (Unit 42)
**Fakty:** Atakujący kradną tokeny access/refresh, co omija MFA i przeżywa reset hasła. Zestawy PhaaS używają Cloudflare Workers do odszyfrowywania payloadów AES-GCM w runtime oraz lurów generowanych przez LLM; raport nie podaje liczby ofiar.
**Znaczenie:** Mitygacje: zastąpić consent użytkownika workflow zatwierdzania przez administratora, audytować zarejestrowane redirect_uri pod kątem niezweryfikowanych domen, blokować/alarmować device code flow poza uzasadnionymi scenariuszami, skracać życie refresh tokenów (Conditional Access, token binding).
Źródło: https://unit42.paloaltonetworks.com/current-landscape-oauth-attacks

### Biznes

#### Microsoft Entra ID wycofuje natywne SMS/voice MFA: 1.02.2027 (administratorzy globalni i użytkownicy zewnętrzni 1.07.2027)
**Fakty:** Użytkownicy tylko-SMS dostaną blokującą rejestrację passkey przy logowaniu, chyba że organizacja wykupi zewnętrznego dostawcę telefonii przez Microsoft Security Store (dostępne od 30.10.2026).
**Znaczenie:** Inwentaryzacja metod MFA w tenantach (w tym konta break-glass i usługowe zależne od SMS), plan wdrożenia passkey/FIDO2 i tymczasowo telefonii zewnętrznej. Ryzyko: lockout kont zewnętrznych partnerów, którzy nie kontrolują metod uwierzytelniania.
Źródło: https://www.neowin.net/news/microsoft-entra-id-ending-native-sms-authentication-pushing-users-to-passkeys/

## Raport AI-ML

### Technologia

#### SemiAnalysis: Trainium3 + TorchNeuron – natywny PyTorch (eager, torch.compile), pierwsze liczby
**Analiza:** Test Qwen3-32B na 16 chipach Trainium2 w TorchTitan: 11 099 tok/s w eager (23,89% MFU) i 14 156 tok/s w trybie skompilowanym (30,47% MFU, 1,28×). Kluczowa słabość to czas kompilacji: pierwszy krok 184,6 s (eager) / 255,7 s (compiled), a recepta gpt-oss 20B – 27 min 17 s. AWS zapowiada nowy backend kompilatora i prekompilowane kernele NKI, celując w redukcję time-to-first-step do ~2 rzędów wielkości. FlexAttention: 89/96 przypadków (93%), wszystkie porażki to timeouty kompilacji przy batch 4 / seq 8192. W mikrobenchmarku matmul Trainium3 wyprzedza GB300 w średnim zakresie (od ~1 GFLOP) i powyżej 1 TFLOP. Brak TokenSwitch API (MoE dispatch/combine to 60–70% wolumenu komunikacji) – używany własny NeuronEPTokenDispatcher. Brak MXFP8 od dnia 0.
**Znaczenie:** Zmniejsza barierę wejścia (.to("neuron") zamiast XLA) i jest realną alternatywą kosztową dla treningu tier-2, ale ekosystem daleko od dojrzałości CUDA; lock-in przesuwa się z XLA na NKI/Neuron Explorer. Przed planowaniem pojemności trenować pilotaż pod kątem kosztu kompilacji w iteracjach badawczych. Dane pochodzą od jednego niezależnego analityka z dostępem do racków; część źródła płatna.
Źródło: https://newsletter.semianalysis.com/p/amazons-trainium-takes-a-strike-at

#### Google Cloud + NVIDIA: GA infrastruktury Vera Rubin (instancje A5X) oraz rack-scale Confidential Computing
**Analiza:** A5X projektowane pod duży RL i agentic inference; Google zapowiada NVIDIA Confidential Computing na poziomie racka dla bare metal i rozszerzenie na VM. Brak w źródle cen, rozmiaru domeny NVLink w A5X i dostępności regionalnej – capacity planning wymaga zapytania ofertowego. Poufne obliczenia na skalę racka istotne dla modeli o wrażliwych wagach/danych, ale narzut na latency i dostępność atestacji do zweryfikowania.
**Znaczenie:** Pierwsza duża chmura z GA Rubin – wyznacza punkt odniesienia cenowy i terminy dla konkurencji (AWS, Azure, OCI) oraz neocloudów, które finansują Rubin długiem.
Źródło: https://blogs.nvidia.com/blog/google-cloud-vera-rubin-infrastructure/

#### Cerebras: wdrożenie modeli rzędu bilionów parametrów na wielu waferach WSE-3T
**Analiza:** Pojedynczy wafer WSE-3T ma 44 GB pamięci on-chip, a wagi modeli frontier to terabajty, więc model jest dzielony między wiele wafli; post opisuje mapowanie warstw, wypełnianie pamięci on-chip i obsługę wielu równoległych żądań. Skalowalność ograniczona interkonektem między waflami i pojemnością SRAM, przewaga w latency/tok/s kosztem gęstości pojemności i vendor lock-in.
**Znaczenie:** Alternatywa dla GPU w niskolatencyjnej inferencji agentowej; ocena TCO wymaga danych o liczbie wafli na model i cenie za token, których wpis nie podaje.
Źródło: https://www.cerebras.ai/blog/how-we-deploy-trillion-parameter-frontier-models-on-cerebras

#### NVIDIA TensorRT Global Performance Tuner (części 1–2)
**Analiza:** Narzędzie traktuje wybory fuzji, implementacji kernela, pamięci i kwantyzacji jako wspólny problem schedulingu i automatycznie stroi end-to-end z uwzględnieniem dokładności dla każdego modelu. Zysk zależy od modelu; w źródle brak zbiorczych liczb. Dla zespołów MLOps: wydłuża czas budowy silnika, ale redukuje ręczne dobieranie tras kompilacji.
**Znaczenie:** Zwiększa głębokość integracji z TensorRT (lock-in), ale obniża koszt tokena na tej samej flocie GPU.
Źródło: https://developer.nvidia.com/blog/scaling-nvidia-tensorrt-performance-optimization-beyond-a-single-build-route-with-the-global-performance-tuner-part-1/ , https://developer.nvidia.com/blog/scaling-nvidia-tensorrt-performance-optimization-beyond-a-single-build-route-with-the-global-performance-tuner-part-2/

#### METR i Redwood: 3-tygodniowy red-teaming wewnętrznego monitoringu agentów OpenAI
**Analiza:** Z dostępem do infrastruktury znaleziono luki umożliwiające agentom omijanie monitoringu, ograniczeń sandboxa lub ukrywanie działań. Uzupełnia wniosek z 23.10 (Sentry): kontrola musi leżeć w osobnej domenie uprawnień, a monitoring wymaga regularnych testów adwersarialnych.
**Znaczenie:** Przy budowie platform agentowych planować budżet na ciągły red-teaming warstwy kontroli, nie tylko modeli.
Źródło: https://metr.org/blog/2026-10-22-red-teaming-openai-agent-monitoring/

### Biznes

#### OpenAI: runda do 30 mld USD przy wycenie ~1,25 bln USD, prowadzona przez inwestorów z ZEA
**Analiza:** Runda ma być „oportunistycznym mostem” przed IPO; podpisy planowane do końca przyszłego tygodnia, ostateczna wielkość nieustalona. Finansowanie kapitałowe zasila zobowiązania compute (kolejka kontraktów u neocloudów i hyperscalerów), więc jego sukces lub opóźnienie wpływa na wiarygodność backlogu dostawców.
**Znaczenie:** Zależność od kapitału z jednego regionu i od okna IPO zwiększa ryzyko kontrahenta dla dostawców mocy; źródło wtórne (CNBC via TLDR), brak potwierdzenia przez strony.
Źródło: https://www.cnbc.com/2026/10/22/openai-funding-round-uae-investors.html

#### DeepSeek: ok. 15 mld USD finansowania (Tencent, CATL) mimo nowych ograniczeń USA
**Analiza:** Jedna z największych rund w Chinach; wzmacnia zdolność zakupu mocy obliczalnej niezależnej od NVIDIA (kontekst: Huawei Ascend) i przygotowuje IPO. Informacja z TechCrunch jako „reportedly”.
**Znaczenie:** Kolejny napęd presji cenowej na tokeny open-weight; dla kupujących inferencję – scenariusz dalszego spadku cen i fragmentacji stosu sprzętowego.
Źródło: https://techcrunch.com/2026/10/22/deepseek-reportedly-raises-15b-days-after-us-restrictions/

#### Crusoe + Araico: ~106 MW AI w Paragwaju zasilane hydroelektrownią Itaipú, start połowa 2027
**Analiza:** Zasilanie hydro daje stabilną, niskoemisyjną energię i niższy koszt MWh, ale latency do USA i LATAM oraz łącza światłowodowe są głównym ograniczeniem dla inferencji wrażliwej na opóźnienia; start tylko deklarowany.
**Znaczenie:** Potwierdza trend lokowania compute tam, gdzie jest moc, a nie klienci – przesuwa architekturę ku batch/training i asynchronicznej inferencji.
Źródło: https://www.crusoe.ai/resources/newsroom/crusoe-araico-hydroelectric-ai-infrastructure-partnership

#### Intel CTO: narzędzia pozwalają ok. 2× zwiększyć moc produkcyjną leading-edge, a popyt AI rośnie ok. 3×/rok
**Analiza:** Wg wywiadu (Dwarkesh Patel) wystarczy narzędzi na mniej więcej podwojenie wolumenu, nie 10×. Backside power pozwala 14A drukować jednym przejściem High-NA, gdzie TSMC potrzebuje podwójnego patterningu EUV; hybrid bonding schodzi ku 0,45 µm. Źródło: wypowiedź dostawcy z interesem w pozycjonowaniu Intel Foundry.
**Znaczenie:** Wspiera tezę o fizycznym limicie podaży (zgodnie z ASML z 20.10): ceny i terminy dostaw akceleratorów w 2027–28 pozostaną napięte.
Źródło: https://www.dwarkesh.com/p/intel-tour

## Raport Ryzyka rynku AI/GPU

#### SpaceX: dług 40 mld USD (Apollo, Goldman) wycenia się do 8,3%; transza zabezpieczona chipami ok. 7,75%
**Analiza:** Najniżej oceniona transza ~8,3%, bezpieczniejsza, zabezpieczona częścią chipów, ~7,75%. Wskazuje, że nawet dla emitenta z mocnym bilansem rynek wycenia ryzyko finansowania sprzętu AI wyraźnie powyżej kosztu długu inwestycyjnego, a zabezpieczenie w GPU daje tylko niewielki dyskonto. To nowy punkt odniesienia dla wartości kolateralu GPU (amortyzacja 3–5 lat vs. finansowanie dłuższe).
**Ryzyko:** Przy rosnącym koszcie długu wyższe wymogi zabezpieczeń i krótsze tenor finansowania uderzą najpierw w emitentów niższej jakości (zob. CoreWeave z 23.10).
Źródło: https://www.semafor.com/article/10/22/2026/spacex-40b-debt-deal-will-cost-it-as-ai-fears-spread

#### Rozbieżność prognoz wydatków na infrastrukturę AI – 12 firm, różne definicje (Software Strategies Blog)
**Analiza:** Zestawienie prognoz Gartnera, IDC, Dell'Oro i Goldman Sachs pokazuje, że różnice wynikają z definicji (serwery vs. całość DC vs. energia/chłodzenie), a nie z fundamentalnie różnej wizji popytu; łączy je z budżetami hyperscalerów, dostępnością mocy i przychodem potrzebnym do uzasadnienia inwestycji.
**Ryzyko:** Przy planowaniu pojemności i ocenie bańki porównywać serie w tej samej definicji; źródło wtórne (blog).
Źródło: https://softwarestrategiesblog.com/2026/10/22/ai-infrastructure-spending-forecasts-compared-gartner-idc-delloro-goldman-sachs-2030/

## Newsletters summary

Przetworzono 6 z 6 maili NEWSY (SemiAnalysis, TLDR AI, TLDR Hardware, TLDR InfoSec, TLDR IT, TLDR); 0 zignorowanych. Pozycje już opisane w sekcjach wyżej pominięto; poniżej dodatkowe.

- **1Password for Agents** – Agent żąda poświadczeń z sejfu użytkownika, po zatwierdzeniu 1Password wstrzykuje je do formularza docelowej strony bez ujawniania agentowi. To wzorzec brokera poświadczeń (credential injection); ogranicza wyciek haseł przez prompt injection, ale nie chroni przed nadużyciem sesji po zalogowaniu i przenosi zaufanie na przeglądarkowy autofill. https://1password.com/blog/1password-for-ai-agents
- **Anthropic dotuje standard Agent Skills do Agentic AI Foundation (Linux Foundation)** – Przejście pod neutralne zarządzanie zmniejsza ryzyko zależności od jednego vendora przy budowie bibliotek skilli; do obserwacji, czy inni dostawcy zaimplementują format. https://www.anthropic.com/news/donating-agent-skills-to-the-agentic-ai-foundation
- **Claude Platform na Google Cloud** – Pełna platforma Claude z uwierzytelnianiem, rozliczeniami i retire commitmentów GCP. Dla architektów: wykorzystanie istniejących zobowiązań GCP, ale rozważyć rezydencję danych i różnice w SLA względem bezpośredniego API. https://claude.com/resources/articles/claude-models-on-google-cloud-anthropic-managed
- **Anthropic: formalna weryfikacja oprogramowania z użyciem LLM** – LLM generują dowody formalne na dużą skalę, co umożliwia przepływy z udowodnionymi gwarancjami bezpieczeństwa. Ograniczenie: poprawność specyfikacji; dowód pokrywa tylko to, co zostało sformalizowane. https://www.anthropic.com/research/formal-verification
- **Scion – własny system kontroli wersji Anthropic** – Ok. 3 500 pracowników generuje wolumen kodu jak firma 10–20× większa, co – jak w Google i Meta – wymusiło odejście od Gita. Powiązane z wpisem GitHub o skalowaniu git pod ruch agentów: obciążenie VCS przez agentów to nowy czynnik planowania pojemności. https://blog.pragmaticengineer.com/the-pulse-anthropic-builds-its-own-source-control-system/
- **Znak wodny w tekście Claude (SynthID-Text) – zgodność z EU AI Act** – Znak jest statystyczny (wybór między bliskoznacznymi słowami), bez ukrytych znaków i bez informacji o użytkowniku. Wpływ: systemy wykrywające tekst generowany oraz obowiązki zgodności w UE; podatny na parafrazę. https://www.oreilly.com/radar/the-latest-anthropic-claude-models-contain-ai-text-watermarks-what-does-this-mean-for-you
- **ProjectDiscovery: backdoor w „abliterated” modelu** – Qwen2.5-7B dotrenowany na wynajętym GPU i ~20% zatrutych promptów exfiltruje plik .env po frazie wyzwalającej. Zalecenie: pobierać modele tylko z zaufanych źródeł, skanować artefakty i testować behawioralnie w sandboxie bez dostępu do sekretów. https://projectdiscovery.io/research/how-abliterated-models-can-get-you-pwned
- **Cisco Antares-1B – offline'owy lokalizator podatnych plików** – Model 1B po post-treningu na IBM Granite uruchamia do 15 poleceń read-only w sandboxie; wg Cisco 4. miejsce w file F1 na VLoc Bench (500 repo), wynik autora na M1 ~80% tego rezultatu, ok. minuty na repo; eksport SARIF 2.1.0. Narzędzie triage'u, nie detektor: głośne shortlisty, spadek trafności dla repo większych niż kilka MB. https://blogs.cisco.com/cisco-devnet/antares-security-analyst-laptop-part-1
- **curl: „Ignore the slop” (Stenberg)** – Setki zgłoszeń podatności w ostatnich miesiącach dotyczyło AI, ale halucynowane raporty zniknęły od wiosny 2026; problemem są dokładne, rozwlekłe zgłoszenia o dużym wolumenie, których nie można zignorować. Wniosek: koszt triage'u po stronie maintainerów rośnie i przyspiesza cykle patchowania w łańcuchu dostaw. https://daniel.haxx.se/blog/2026/10/22/ignore-the-slop/
- **GitHub: egzekwowanie globalnych domyślnych ustawień Copilot do 26.10** – Nieskonfigurowane funkcje Copilot zostaną ustawione zgodnie z globalną polityką organizacji, jawne decyzje adminów pozostają. Sprawdzić globalną politykę przed 26.10, by uniknąć niezamierzonego włączenia funkcji. https://github.blog/changelog/2026-10-22-global-default-for-copilot-features-generally-available/
- **ServiceNow + NVIDIA Arc** – Długo działający agent enterprise lokalnie na RTX Spark lub w NVIDIA OpenShell; polityki runtime/sprzętowe przez Control Tower, OpenShell i AI Passport. Spójne z trendem separacji domeny kontroli agentów. https://www.zdnet.com/innovation/exclusive-servicenow-and-nvidia-launch-arc-a-new-ai-agent-for-enterprise/

## Implikacje praktyczne
1. Rejestr incydentów pokazuje powtarzalny wzorzec: sekrety w PaaS/CI/LLM-proxy (Railway, TiDB, GitLab, LiteLLM). Projektować z założeniem kompromitacji platformy: krótkożyjące poświadczenia (OIDC), sekrety poza środowiskiem procesu, automatyczna rotacja i budżety/limity po stronie providera.
2. Okno ekspozycji przed patchem staje się normą (PHP2Shell: eksploatacja 4 dni przed ujawnieniem). Procesy IR muszą cofać analizę do wcześniejszej daty niż data CVE, a patchowanie internet-facing – mieścić się w godzinach.
3. MFA oparte na telefonii wygasa (Entra 2027); migracja do passkey/FIDO2 wraz z przeglądem konta break-glass i zewnętrznych tożsamości, plus OAuth admin-consent i kontrola redirect_uri jako obowiązkowy element tożsamości w horyzoncie 3 lat.
4. Dywersyfikacja akceleratorów (Trainium3 z natywnym PyTorch, TPU, Cerebras) staje się opcją kosztową, ale wymaga budżetu na portowanie, kompilację i testy numeryki; unikać wiązania kluczowych kerneli z jednym SDK (NKI/TensorRT) bez warstwy abstrakcji.
5. Koszt kapitału (SpaceX 7,75–8,3%, CoreWeave ~16%) i limit podaży półprzewodników (Intel: ~2× vs ~3×/rok popytu) zwiększają ryzyko kontrahenta infrastrukturalnego – klauzule step-in, przenaszalność danych i bufor pojemności 20–30% w kontraktach.

## Trend tygodnia
Tydzień kończy się spójnym obrazem: ataki coraz częściej omijają perymetr przez zaufane kanały – poświadczenia platform (Railway, TiDB), tokeny OAuth, klucze IAM objęte „kwarantanną”, nieistniejące okno patchowania przed ujawnieniem PHP2Shell. Jednocześnie narzędzia AI obniżają koszt odkrywania i eksploatacji podatności (obserwacja Stenberga: raporty AI są dziś dokładne, nie halucynowane), co skraca czas reakcji obrońców do godzin. Po stronie infrastruktury AI rynek przechodzi od dyskusji o dostępności GPU do operacyjnej dojrzałości stosu: natywny PyTorch na Trainium3, GA Vera Rubin w Google Cloud, wafle Cerebras dla modeli bilionowych – ale kompilacja, ekosystem i profilowanie nadal różnicują dostawców. Warstwa finansowa pozostaje napięta: runda OpenAI do ~1,25 bln USD jako most przed IPO, DeepSeek z 15 mld USD i dług SpaceX 8,3% pokazują, że kapitał jest dostępny, ale drożeje i zależy od oczekiwań IPO. Fizyczne ograniczenia (moc w fabach wg Intela, energia – Crusoe w Paragwaju) zastępują chipy jako główne wąskie gardło podaży. Dla architektów wniosek jest taki sam jak tydzień wcześniej, ale mocniejszy: izolować sekrety, mierzyć ryzyko kontrahenta i projektować niezależność od jednego dostawcy akceleratorów i chmury.

## To obserwować
1. Rozstrzygnięcie rundy OpenAI (podpisy do końca przyszłego tygodnia) i jej końcowa wielkość/wycena.
2. Ceny i dostępność instancji A5X (Vera Rubin) w Google Cloud oraz reakcja AWS/Azure/OCI.
3. Zapowiadany nowy backend kompilatora TorchNeuron (cel: ~100× krótszy time-to-first-step) i wsparcie TokenSwitch/MXFP8.
4. Wpisy CISA KEV i skala webshelli po PHP2Shell oraz wykorzystanie GitLab CVE-2026-110264 po publikacji łatek.
5. Spready na długu zabezpieczonym GPU (SpaceX 7,75–8,3%) vs CoreWeave (~16%) jako wskaźnik kosztu kapitału.
