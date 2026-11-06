# Newsy — 2026-11-06

**Okno:** od raportu 2026-11-05. Pominięte sekcje: euro neocloud, AI for networking, Ryzyka rynku AI/GPU (brak nowych zweryfikowanych informacji; wyszukiwanie WWW nie zwróciło materiałów z okna). Źródła: newslettery z etykiety NEWSY.

## Raport technologiczny

### Technologia

#### Greeter: chiński framework eksploatuje Flowise (CVE-2024-58351) – infrastruktura AI jako cel
Era Security śledziła od lipca do października framework wykorzystujący code injection w overrideConfig/javascriptFunction we Flowise; każdy request „hello” niósł inną komendę, bez modyfikacji zapisanego chatflow (utrudnia detekcję po konfiguracji). Po wejściu: rekonesans hosta, kradzież .env, kluczy SSH i kluczy szyfrujących Flowise, odpytywanie metadata AWS/Azure/GCP, sprawdzanie dostępu do Docker socket, capabilities i ścieżek persistence. Z 3277 widocznych instancji 703 bez uwierzytelniania, 46 spełniało warunki podatności.
**Znaczenie/Analiza:** Narzędzia low-code do agentów traktować jak usługi wystawiające RCE: nie wystawiać do Internetu, wymusić auth, blokować IMDS (IMDSv2, hop limit 1), nie montować Docker socket, rotować sekrety po ekspozycji.
Źródło: https://era.security/blog/greeter-flowise-ai-infrastructure-attack

#### Atak na dostawcę chłodnictwa: zmiana nastaw w magazynach osocza EFS (Francja), kontrolery Danfoss AK-SM/AK-SC
Przez kompromitację dostawcy zmieniono nastawy komór chłodniczych z -30°C na 110°C; monitoring EFS wykrył zmianę, produkty przeniesiono, bez strat. Wcześniej podobne ataki na chłodnictwo supermarketów (zmiany cykli odszraniania). CERT-FR wzywał do łatania Danfoss AK-SM/AK-SC.
**Znaczenie/Analiza:** Kanał dostawcy zdalnego serwisu OT/BMS to wektor wysokiego wpływu; dotyczy też chłodzenia i BMS w DC. Wymagane: segmentacja dostępu serwisowego, niezależny monitoring temperatury z alarmem poza systemem sterowania, brak bezpośredniej ekspozycji kontrolerów.
Źródło: https://www.theregister.com/security/2026/11/04/cyberattack-on-french-plasma-stores-is-enough-to-make-your-blood-boil/5307519

#### CrowdStrike AI Kill Switch
Blokuje dostęp agentów do modeli AI przez podłączone gatewaye; szersze kontrole (zabijanie procesów na endpointach, odwoływanie uprawnień agentów) dopiero w rozwoju.
**Znaczenie/Analiza:** Kontrola na warstwie gateway nie zatrzyma agenta z lokalnymi poświadczeniami lub bezpośrednim egress; traktować jako uzupełnienie egress allowlist i limitów tokenów, nie zamiennik. Ryzyko pojedynczego punktu kontroli (fail-open vs fail-closed).
Źródło: https://www.crowdstrike.com/en-us/blog/crowdstrike-introduces-ai-kill-switch/

#### Chainstack: włamanie do produkcji przez komponent zewnętrzny (26.10), wyciek hashy haseł
Atakujący wszedł przez komponent third-party i dotarł do systemów produkcyjnych; dostęp do nazw, danych organizacji, e-maili i hashy haseł klientów.
**Znaczenie/Analiza:** Dostawca RPC/node dla blockchain – klienci powinni rotować hasła i klucze API; ogólnie: izolować komponenty third-party od produkcji.
Źródło: https://thehackernews.com/2026/11/chainstack-reports-production-systems.html

### Biznes

#### Broadcom: VMware vSphere Express (do 128 rdzeni)
Oferta dla małych wdrożeń: live migration, automatyczny recovery, distributed switching, brak centralnego zarządzania instalacjami.
**Znaczenie/Analiza:** Zmiana polityki po fali odejść z VMware na niskim końcu rynku; limit 128 rdzeni i brak centralnego zarządzania ograniczają zastosowanie w wielu lokalizacjach. Nie zmienia kalkulacji migracji dla większych środowisk.
Źródło: https://www.theregister.com/virtualization/2026/11/04/vmware-returns-to-low-end-server-virtualization-with-vsphere-express/5307466

#### Google: 15 mld USD na DC w Malezji w ciągu 2 lat
Inwestycja w centra danych i infrastrukturę pod popyt AI w regionie APAC.
**Znaczenie/Analiza:** Dywersyfikacja geograficzna mocy pod kątem energii i chłodzenia; potencjalnie nowe regiony chmurowe w SEA (latencja do klientów regionalnych).
Źródło: https://techxplore.com/news/2026-11-google-unveils-billion-center-investment.html

## Raport AI-ML

### Technologia

#### SemiAnalysis: TPUv7 (Ironwood) vs GB300/Rubin w InferenceX AgentX
Wyniki długiego kontekstu, multi-turn, z dużym prefix reuse. GLM-5.2 FP8: TPUv7 1,2–2,7x tokenów/USD względem B200/B300/MI355X (60–300 tok/s/user). Kimi K3 (MXFP4): 1,15–2,5x GB200/B200/B300, ale GB300 FP4 po czterech refreshach prowadzi 1,5–2,6x, a VR200 daje 4,3–5,6x TPUv7. Qwen3.5 397B FP8 z llm-d (disaggregated serving, hosty po 4 chipy, 24–32 chipów): TPUv7 > GB300 FP8 przy każdej interaktywności, ~1,67x względem GB300 NVL72 FP4 wg autorów (kontrargument NVIDIA FP4 przestrojony pod decode kosztem prefill). Cache-aware routing podniósł hit rate decode z 56% do 76%. Brak natywnego FP4 w TPUv7 – przy 100 tok/s/user GB300/GB200 FP4 mają 3,5–3,6x tokenów/USD; TPUv8 z FP4 oczekiwany w Q1 2027.
**Znaczenie/Analiza:** Wnioski: przewaga TPU zależy od precyzji, kształtu modelu (head_dim 64 wypełnia 25% systolic array paddingiem) i metryki (decode vs end-to-end, prefill). Źródło ma interes komercyjny (sprzedaż modeli TCO), benchmark zgłoszony częściowo przez dostawców. Megakernele (AMD MonoKernel: 2459 → 210 launchy, 2,31x przy batch 1; NVIDIA dogoniła w tydzień) eliminują narzut launchy w decode. Dla capacity planning: dostępność TPU poza GCP (Meta, Anthropic, OpenAI, Oracle jako klienci wg autorów) i stos vLLM/SGLang/llm-d na TorchTPU zmniejszają lock-in CUDA, ale wymagają kodesignu modelu pod TPU.
Źródło: https://newsletter.semianalysis.com/p/tpu-strikes-the-king

#### NVIDIA DSX OS: konfiguracja, certyfikacja i operacje klastrów GPU na Kubernetes
Pojedynczy wolny GPU spowalnia cały job rozproszony i może przechodzić podstawowe health checki; klaster zmienia się po wymianach węzłów i aktualizacjach sterowników, więc walidacja gotowości przy przekazaniu nie wystarcza – potrzebna ciągła weryfikacja.
**Znaczenie/Analiza:** Przesuwa weryfikację z jednorazowego acceptance do continuous; warto ją odtworzyć w własnym procesie (testy NCCL/GEMM per węzeł po każdej zmianie). Zależność od narzędzi NVIDIA = lock-in operacyjny.
Źródło: https://developer.nvidia.com/blog/configure-certify-and-operate-kubernetes-gpu-clusters-with-nvidia-dsx-os/

#### NVIDIA: pamięć HBM jako zasób – „użyteczna praca na bajt”
Bajty zajęte przez zduplikowane wagi, nieużywany KV cache i przeliczane prefiksy nie generują tokenów; kluczowa jest efektywność, nie pojemność.
**Znaczenie/Analiza:** Zgodne z trendem tiered KV cache i prefix caching (por. offload do DRAM hosta w wynikach TPU). Przy drożejącej pamięci optymalizacja wykorzystania HBM obniża koszt tokena szybciej niż zakup kolejnych GPU.
Źródło: https://developer.nvidia.com/blog/five-ways-nvidia-co-design-maximizes-the-value-of-every-byte-in-the-ai-factory/

#### OpenAI: autonomiczny pipeline, 948 załatanych podatności LPE w Linuksie w jednym przebiegu
Model do cyberbezpieczeństwa wykrywał, walidował (exploitem) i generował patche. Opis pochodzi od dostawcy, bez niezależnej weryfikacji.
**Znaczenie/Analiza:** Skala automatycznego odkrywania luk w jądrze rośnie – oczekiwać zwiększonej liczby advisories i skrócenia okna między ujawnieniem a exploitem; proces patchowania i reboot musi nadążać. Por. OpenEMR: 42 advisories z przeglądu wspomaganego AI (wersja 8.5.0).
Źródło: https://links.tldrnewsletter.com/KGHH6y

#### Ripple-65B Preview – otwarty model, trening za 32 mln USD
Deklarowany SOTA na TerminalBench 2.1 wśród modeli open-weight, przewyższa większe modele (Muse Spark Open 816B, Nemotron 3 Ultra 550B); koszt treningu ok. 1/10 poprzedniego lidera; dostępny m.in. na Together AI, GMI Cloud, OpenRouter.
**Znaczenie/Analiza:** Jeśli wynik się potwierdzi, spada koszt inferencji (65B vs 500–800B): mniej HBM i węzłów na token. Wynik od autorów modelu – wymaga niezależnej ewaluacji.
Źródło: https://www.ricursive.com/ripple

#### Together Compiler – ciągła kompilacja/optymalizacja inferencji
Pętla optymalizacji kerneli prowadzona przez agentów dla różnych obciążeń.
**Znaczenie/Analiza:** Zbieżne z megakernelami i autotuningiem w SemiAnalysis; skraca czas wsparcia nowych modeli/akceleratorów, ale wynik zależy od jakości ewaluatora (ryzyko przetuningu pod benchmark).
Źródło: https://www.together.ai/blog/together-compiler-continual-compilation-for-ai-inference

## Newsletters summary

### Podsumowanie

#### Mozilla AI Otari – open-source gateway governance dla AI
Egzekwuje ograniczenia narzędzi, budżety użytkowników, tracing sesji i przełączanie dostawców jedną zmianą konfiguracji.
**Znaczenie/Analiza:** Alternatywa dla komercyjnych bram (Snowflake Cortex AI Gateway, CrowdStrike): redukuje vendor lock-in; do oceny skalowalność i opóźnienie dodane przez proxy.
Źródło: https://blog.mozilla.ai/otari-by-mozilla-ai-your-operating-system-for-ai-governance/

#### IBM Cloud Secrets Manager Vault Dedicated
Single-tenant HashiCorp Vault Enterprise zarządzany przez IBM: dynamiczne poświadczenia, rotacja, polityki.
**Znaczenie/Analiza:** Sensowne wobec kradzieży kluczy LLM i sekretów agentów; weryfikować model odpowiedzialności za HA/DR i możliwość wyjścia (lock-in).
Źródło: https://www.ibm.com/new/announcements/introducing-ibm-cloud-secrets-manager-vault-dedicated-hashicorp-vault-enterprise-managed-by-ibm

#### Microsoft: kończy się CNAME discovery dla MDM-only enrollment (15 stycznia); wygasanie certyfikatów Windows Update w 2027
Zmiany w Intune pozwalają standardowym użytkownikom zmieniać zatwierdzone ustawienia sieci.
**Znaczenie/Analiza:** Sprawdzić rejestrację urządzeń MDM-only i łańcuch certyfikatów przed terminami; ryzyko masowych błędów enrollmentu po 15.01.
Źródło: https://techcommunity.microsoft.com/blog/windows-itpro-blog/windows-news-you-can-use-october-2026/4552396

#### Rozproszone DR w świecie zależnym od AI
Zalecenie zdefiniowania „minimum viable company”: modele, dane RAG i zależności tożsamości, z uwzględnieniem niedeterminizmu modeli i autonomicznych agentów.
**Znaczenie/Analiza:** Wprowadzić modele AI i indeksy RAG do RTO/RPO oraz testów DR; dostawca modelu jako zależność krytyczna.
Źródło: https://www.itpro.com/security/what-does-disaster-recovery-look-like-in-an-ai-centric-world

## Implikacje praktyczne
1. Agenty i platformy low-code AI (Flowise) traktować jak powierzchnię RCE: egress allowlist, blokada IMDS, brak Docker socket, brama kontrolna – obowiązkowe w architekturze referencyjnej.
2. Kanały serwisowe dostawców OT/BMS (chłodzenie DC) wymagają segmentacji i niezależnego monitoringu fizycznego; w horyzoncie 3–5 lat to realne ryzyko ciągłości dla DC o dużej gęstości.
3. Projektować inference stack neutralnie sprzętowo (vLLM/SGLang/llm-d, abstrakcja akceleratora): TPU/AMD/GPU mają różne przewagi zależnie od precyzji (FP4 vs FP8) i kształtu modelu; kontrakty nie powinny wiązać na całą generację.
4. Zakładać skracanie okna ujawnienie–exploit (automatyczne odkrywanie luk): zautomatyzowany rolling reboot i patchowanie jądra jako zdolność operacyjna, nie wyjątek.
5. Ciągła walidacja klastrów GPU (nie jednorazowy acceptance) i optymalizacja wykorzystania HBM/KV cache jako główna dźwignia kosztu tokena przy drożejącej pamięci.

## Trend tygodnia
Dyskusja o inferencji przesuwa się z czystej przepustowości na koszt tokena w obciążeniach agentowych: długi kontekst, wysoki prefix reuse i bursty sub-agentów, gdzie liczy się zarządzanie KV cache, a nie sama moc obliczeniowa. Wyniki InferenceX pokazują, że przewaga akceleratora zależy od precyzji, kształtu modelu i metryki, więc pojedynczy ranking przestaje wystarczać do decyzji zakupowych. Megakernele i ciągła automatyczna kompilacja skracają czas, w którym przewaga sprzętowa się utrzymuje: NVIDIA dogoniła rozwiązania AMD/TPU w tydzień. Jednocześnie operacje klastrów przechodzą na ciągłą weryfikację zdrowia węzłów, bo jeden wolny GPU degraduje cały job. W bezpieczeństwie dominują ataki na infrastrukturę AI (Flowise, klucze LLM) i łańcuch dostawców OT, a automatyzacja po obu stronach (patchowanie 948 luk LPE, kill switche agentów) przyspiesza cykl obrony i ataku. Wspólny mianownik: kontrola przesuwa się do warstw bram i orkiestracji, a wybór otwartych stosów ogranicza lock-in.

## To obserwować
- Kolejne wyniki AgentX: TPUv8i i Rubin NVL72 w InferenceX oraz TPUv8 z FP4 (Q1 2027).
- Niezależna weryfikacja Ripple-65B (TerminalBench 2.1) i koszt treningu.
- Skala eksploatacji Flowise (CVE-2024-58351) i liczba instancji bez uwierzytelniania (703 z 3277).
- Wydania vLLM/SGLang/llm-d z megakernelami i TorchTPU.
- Terminy Microsoft: 15.01 (CNAME MDM) i wygasanie certyfikatów 2027.
