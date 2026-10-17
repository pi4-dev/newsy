# Newsy — 2026-10-17

**Okno przyrostowe:** od raportu 2026-10-16 do 2026-10-17 07:28 CEST.
**Uwaga:** WWW/RSS nie zwróciły nowych, datowanych pozycji (euro neocloud: brak materiału z X 2026) – sekcje Raport technologiczny, AI-ML, euro neocloud, AI for networking pominięte. Treść z newsletterów TLDR z 16.10 (poz. już ujęte w poprzedniej edycji pominięto: Ivanti, XBOW, Discourse, UEC 1.1, Cisco/Nvidia 12,8T, SolarWinds Avior, vLLM NIXL, Netflix GPU, ClickHouse, ASML hybrid bonding, przychód Anthropic, OTel Claude).

## Newsletters summary

#### Cisco Secure Email Gateway: CVE-2026-76519 (CVSS 9.8), 0-day wykorzystywany aktywnie
**Zdarzenie:** krytyczna luka eskalująca do root na SEG, exploitowana przez nieznaną grupę przed patchem. **Wpływ:** SEG na styku = punkt pivotu do sieci wewnętrznej i dostęp do całej poczty; priorytet patchowania/ograniczenie ekspozycji interfejsów zarządzania. **Failure modes:** brak patcha w dniu 0 – kompromitacja możliwa mimo hardeningu; wymagany threat hunting wstecz.
Źródło: https://www.theregister.com/security/2026/10/14/cisco-make-me-root-email-bug-exploited-as-a-0-day-by-unknown-crims/5303168

#### BeyondTrust Remote Support: ostrzeżenie Google o masowej eksploatacji (BT26-05)
**Zdarzenie:** luka używana w ukierunkowanych atakach od sierpnia; Google ostrzega przed szerokim wykorzystaniem (podejrzewane grupy powiązane z Chinami). Hosting BeyondTrust załatany; self-hosted – patche BT26-05 oraz prewencyjny dla Privileged Remote Access. **Wpływ:** narzędzia PAM/RS mają uprzywilejowany dostęp do hostów – kompromitacja = lateral movement. **Failure modes:** instancje self-hosted wystawione do internetu.
Źródło: https://links.tldrnewsletter.com/Qyxngq

#### curl 8.23.0: CVE-2026-92392 – HTTP/2 UAF → RCE po stronie klienta
**Mechanizm:** reset na wstrzymanym strumieniu uploadu zostawia dangling pointer do zwolnionego easy handle; przy cleanupie multi-handle dane serwera nadpisują pamięć (w tym wskaźnik funkcji), wyciek adresów omija ASLR. Wykryte przez Anthropic OSS Scanner. **Wpływ:** złośliwy serwer może atakować klientów libcurl (agenty, CI, proxy); maintainerzy dali < CRITICAL (wymaga ~135 KB uploadu i zależy od układu alokacji w distro/libc). **Zalecenie:** aktualizacja libcurl w obrazach bazowych i kontenerach.
Źródło: https://daniel.haxx.se/blog/2026/10/14/a-uaf-turned-rce/

#### Claude Code Desktop „SneakPeek”: zero-click przez podgląd HTML (Noma)
**Mechanizm:** złośliwy HTML w publicznym repo uruchamiał się w panelu podglądu przy review/edycji; łańcuch słabości omijał sandbox i kradł poświadczenia AWS/GitHub/SSH/OAuth; Noma znalazła obejście pierwszej poprawki, załatane. **Wpływ:** stacje deweloperskie z długożyciowymi sekretami = wysoki blast radius przy narzędziach agentowych. **Zalecenie:** aktualizacja, krótkożyciowe poświadczenia, izolacja środowiska pracy z niezaufanym repo.
Źródło: https://links.tldrnewsletter.com/qezkOs

#### Drupal: CVE-2026-108097 – krytyczny Field UI, ścieżka do RCE bez uwierzytelnienia (przez moduły contrib)
**Wpływ:** exploitowalność zależy od zainstalowanych modułów contrib; inwentaryzacja i patch.
Źródło: https://thehackernews.com/2026/10/drupal-patches-critical-core-rce-flaw.html

#### Jev jako sędzia: „counterfeit policy” obniża recall wykrywania prompt injection z 82,9% do 3,9% (Datadog)
**Mechanizm:** polityka prepended do niezaufanej treści redefiniuje kategorie ataków jako nieszkodliwe; osadzona w edytowalnym polu profilu obniżyła ocenę ryzyka nieautoryzowanej edycji (LangChain tool-risk gating) z 0,893 do 0,290 (< progu 0,5). **Wpływ:** klasyfikator LLM jako bramka bezpieczeństwa jest podatny na wstrzyknięcie kontekstu. **Zalecenie:** twarda autoryzacja poza modelem; pola użytkownika dochodzące do klasyfikatora traktować jako powierzchnię ataku.
Źródło: https://securitylabs.datadoghq.com/articles/jev-as-a-judge-how-counterfeit-policy-steers-jev/

#### Atlassian: OCSF security lakehouse na Databricks (44 mld zdarzeń/dzień)
**Mechanizm:** schemat OCSF zaszyty w strukturach Go (walidacja w compile-time), agentowy workflow mapowania nie może wyemitować nieprawidłowych pól; pipeline canary-gated, samoobsługowy onboarding logów (2/3 danych w 2 miesiące vs 1/3 w rok). **Wpływ:** znormalizowane encje pozwoliły zastąpić alerty 1:1 klastrowanymi sygnałami ryzyka (500+ detekcji).
Źródło: https://www.databricks.com/blog/reducing-noise-how-atlassian-built-ocsf-security-lakehouse-databricks

#### Cloudflare Gateway: akcja „Forward” (custom code na ruchu) i walidacja DLP
**Zdarzenie:** Forward (public beta) uruchamia własny kod na ruchu webowym (strony uzasadnienia dostępu, modyfikacja żądań, alerty); custom functions dla polityk przed końcem roku; walidacja DLP sprawdza, czy wykryty poświadczenie jest aktywne. **Wpływ:** mniej własnych proxy; vendor lock-in w logice polityk SASE; redukcja false positive DLP.
Źródło: https://blog.cloudflare.com/programmable-gateway-dlp/

#### ASML: podaż narzędzi 2027–2028 poniżej popytu; Q3 – 113 systemów, 11,7 mld EUR
**Zdarzenie:** plan +30% mocy na bazie 2026 (~65 low-NA EUV, ~130 DUV immersion), rozważane więcej na 2028; klienci chcą zamawiać na 2029–2030; guidance Q4 14,2–15,2 mld EUR. **Wpływ:** limit podaży litografii = sufit dla podaży akceleratorów/HBM do 2028; planowanie capacity (GPU lead times, ceny) pod stałą niedobór.
Źródło: https://www.cnbc.com/2026/10/14/asml-3q-earnings-ai-demand-orders-capacity.html

#### AWS: laboratorium sieciowe (ServeTheHome)
**Fakty:** przełączniki 3,2–102,4 Tbps (porty 1,6T OSFP jeszcze nie w wolumenie), 12,8 Tbps z 16 FPGA szyfrującymi każdy port, dedykowana sieć czasu referencjonowana satelitarnie, DWDM 64 λ, światłowód hollow-core ~33% niższa latencja. **Wpływ:** wskazuje kierunek hyperscalerów (szyfrowanie na porcie, precyzyjny czas, niska latencja) – benchmark dla własnych wymagań fabric.
Źródło: https://www.servethehome.com/inside-the-secretive-aws-networking-lab/

#### Ayar Labs + GUC: symulacja CPO 3 kW – tylko chłodzenie cieczą trzyma spec
**Mechanizm:** pakiet 105×140 mm: 2 XPU, 8 HBM4, 8 silników TeraPHY, >100 Tbps optycznie, >3 kW; radiator powietrzny (5–6 m/s) nie spełnia spec, cold plate z impingementem 4,5 L/min utrzymuje silniki optyczne <100°C. **Ograniczenie:** statyczne założenia mocy; symulacja dynamiczna dopiero planowana. **Wpływ:** CPO wymusza DLC w racku.
Źródło: https://semiengineering.com/thermal-co-simulation-testing-of-co-packaged-optics/

#### Meta Muse Spark Open: 816B MoE (42B aktywnych), kontekst 1M, open-weight
**Fakty:** wejście tekst/obraz/wideo/audio, wyjście tekst; benchmark Meta na dwóch Mac Studio M5 Ultra (MLX), konfiguracja pamięci nieznana. **Wpływ:** lokalny inference modelu klasy frontier bez GPU DC; ograniczenia: throughput i współbieżność na unified memory. Vercel AI Gateway Index: open-weight 71,5% tokenów, 23% wydatków; prawie 1/3 zespołów zmieniła główny lab we wrześniu – niski koszt zmiany dostawcy.
Źródła: https://research.meta.ai/blog/muse-spark-open ; https://vercel.com/blog/ai-gateway-production-index-october-2026

#### Microsoft MAI-Base-2 (2,8T MoE, 80B aktywnych) i Nvidia Nemotron 3.5 Super VL
**Fakty:** Microsoft opisuje stack pretrainingu i plan udostępnienia modeli trenowanych wewnętrznie przez Foundry; Nvidia: Nemotron 3.5 Super VL (mniej tokenów wyjściowych niż poprzednik), Bolt 74B-A9B oraz wagi NVFP4 (near-lossless). **Wpływ:** NVFP4 obniża koszt pamięci/inference na Blackwell (lock-in sprzętowy); Microsoft ogranicza zależność od zewnętrznych labów.
Źródła: https://microsoft.ai/news/frontier-pre-training-at-microsoft-ai/ ; https://developer.nvidia.com/blog/nvidia-nemotron-3-5-super-vl-powers-specialized-agents-with-multimodal-reasoning/

#### CoreWeave: CDS na poziomie distress, ponad 51 mld USD zobowiązań
**Fakty:** rosnące rentowności długu i spread 5-letnich CDS; popyt niekwestionowany, ale cash flow musi nadążyć za capex. **Wpływ:** ryzyko kontrahenta dla klientów z długoterminowymi rezerwacjami GPU; wzorzec do oceny neocloudów (także europejskich) – klauzule SLA/exit i dywersyfikacja.
Źródło: https://www.cnbc.com/2026/10/15/coreweave-credit-default-swaps-hit-distress-levels-as-investors-sour-on-ai-debt.html

#### Drobne (krótko)
Perplexity open-source SPACE (sandbox agentów): https://www.perplexity.ai/hub/blog/open-sourcing-space · Apple „Minions” – pula wolnych Maców jako inference (tylko wewnętrznie): https://www.computerworld.com/article/4235901/apples-minions-tech-turns-idle-macs-into-an-ai-data-center.html · Passkey Index 2026: 36% udanych logowań, 93% sukcesu vs 78% MFA: https://fidoalliance.org/passkey-index-2026/ · Arm vs Qualcomm: naruszenie licencji, odszkodowanie 2 USD (licencja do 2033): https://links.tldrnewsletter.com/DCTT1P · Solidigm/SK Hynix Dalian – zgoda na eksport narzędzi w zamian za fabrykę NAND w USA: https://techcrunch.com/2026/10/15/sk-hynix-bargains-for-chip-equipment-exports-to-its-china-facility-in-return-for-building-a-factory-in-us · ITRES Labs: odtworzenie badań nad exploitacją jądra Windows z pomocą LLM w ~24 h: https://labs.itresit.es/2026/10/15/copycats-at-machine-speed-reproducing-frontier-windows-kernel-exploitation-research/
