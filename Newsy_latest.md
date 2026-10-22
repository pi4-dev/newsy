# Newsy — 2026-10-22

**Okno przyrostowe:** od raportu 2026-10-21 (07:28 CEST) do 2026-10-22 07:28 CEST.
**Uwaga:** Sekcje "Raport euro neocloud" i "AI for networking" pominięto — po weryfikacji brak nowych, wiarygodnych i datowanych pozycji wykraczających poza materiał z 21.10 (Nscale/Anthropic, SRv6/gNMI to głównie starsze dokumenty bez nowej treści). Materiał: WWW/WebSearch, NVIDIA Developer Blog RSS, SemiAnalysis, newslettery TLDR (etykieta NEWSY, 21.10).

## Raport technologiczny

### Biznes

#### Anthropic S-1: strata netto >80 mld USD w H1 2026, deklarowane do 750 mld USD na compute
**Fakty:** Z ujawnionego projektu prospektu IPO Anthropic wynika strata netto ponad 80 mld USD w pierwszej połowie 2026 przy szybkim wzroście przychodów; firma sygnalizuje, że w najbliższych latach może wydać do 750 mld USD na dodatkową pojemność obliczeniową, oraz wskazuje niepewny dostęp do compute jako czynnik ryzyka.
**Znaczenie:** Skala deklarowanego CAPEX wielokrotnie przewyższa obecne przychody – bezpośrednio wiąże się z ryzykiem kontraktów typu Nscale (sekcja Ryzyka). Dla architektów: kontrahent tej wielkości, zależny od rynku kapitałowego przed IPO, to koncentracja ryzyka dostawcy przy multi-year compute commitments.
Źródło: https://techcrunch.com/2026/10/19/anthropic-s-1-shows-red-hot-growth-risks-and-massive-computing-costs/

#### DHS wprowadza zakaz zakupów od DeepSeek dla agencji federalnych USA
**Fakty:** Departament Bezpieczeństwa Wewnętrznego USA ogłosił zakaz procurementu produktów DeepSeek dla agencji cywilnych, powołując się na ryzyko przetwarzania/przechowywania danych w Chinach; agencje mają 60 dni na wnioski o wyjątki.
**Znaczenie:** Pierwszy formalny zakaz na poziomie federalnym USA wobec konkretnego chińskiego modelu/dostawcy AI – wzorzec do powtórzenia w innych jurysdykcjach i sektorach regulowanych; firmy z kontraktami rządowymi USA powinny zweryfikować łańcuch dostaw modeli/API.
Źródło: https://cyberscoop.com/dhs-bans-deepseek-federal-agencies/

#### Microsoft: "modele frontier nie mogą być traktowane jako zaufane" – agenci AI jako insider threat
**Fakty:** Microsoft wprowadził Guardian Agents i wykrywanie agentów w Microsoft Defender; CEO Satya Nadella i szefowa bezpieczeństwa Hayete Gallot wprost porównują autonomiczne agenty LLM do insider threat, argumentując za rozdzieleniem "dostawcy inteligencji" od "autorytetu decyzyjnego" poprzez niezależny monitoring.
**Znaczenie:** Duży vendor publicznie kwestionuje zaufanie do własnych modeli frontier w kontekście enterprise – wzmacnia argument za observability/guardrails jako warstwą obowiązkową, nie opcjonalną, przy wdrożeniach agentowych; koresponduje z prognozą Gartnera (40% ograniczeń agentów do 2027, już cytowaną 20.10).
Źródło: https://techcrunch.com/2026/10/19/microsoft-says-ai-agents-are-like-insider-threats-as-frontier-models-cant-be-trusted/

### Technologia

#### BeyondTrust Remote Support: zero-day CVE-2026-102764 (CVSS 9.3) exploitowany od sierpnia przez grupę powiązaną z Chinami
**Fakty:** Auth bypass + arbitrary file upload w BeyondTrust Remote Support ≤26.2.3 pozwala nieuwierzytelnionym atakującym na eskalację do root i dostęp do credential vaults. Grupa Armory Castle (dawniej UNC5221, atrybucja Mandiant) wykorzystuje podatność od końca sierpnia do wdrażania webshelli (BARECLAW). Patch (BT26-05) wydany 8.10 dla SaaS (patched 9.10), self-hosted wymaga manualnej aktualizacji. CISA i Canadian Centre for Cyber Security wydały ostrzeżenia.
**Znaczenie:** Kolejny (po Cisco SEG 17-21.10) przypadek krytycznego zero-day w remote-access/support appliance exploitowanego przez aktora państwowego miesiącami przed patchem – self-hosted PRA/Remote Support traktować jako już potencjalnie skompromitowane do czasu forensic review; ograniczyć dostęp zewnętrzny do appliance'ów.
Źródła: https://www.bleepingcomputer.com/news/security/beyondtrust-patches-zero-day-exploited-in-attacks-since-august , https://www.csoonline.com/article/4236421/beyondtrust-zero-day-tied-to-chinese-spies-puts-remote-support-users-on-alert.html

#### 8 skompromitowanych certyfikatów pośrednich TLS (eMudhra) – alarm w branży PKI
**Fakty:** Odkryto osiem defektywnych certyfikatów pośrednich wydanych przez indyjski CA eMudhra Limited; wg badających defekt był "na poziomie uznawanym za niemożliwy".
**Znaczenie:** Naruszenie łańcucha zaufania CA ma potencjalny wpływ systemowy (MITM, podpisywanie malware); do weryfikacji: czy przeglądarki/root store programs (Chrome, Mozilla) podjęły distrust eMudhra; sprawdzić własne store'y trust anchors.
Źródło: https://arstechnica.com/security/2026/10/the-internet-goes-on-high-alert-after-discovery-of-8-compromised-tls-certs/

#### Google Cloud: awaria zasilania w londyńskiej AZ przerywa Cloud Run, GKE, managed Kafka
**Fakty:** Power failure w londyńskiej availability zone Google Cloud przerwał działanie Cloud Run, GKE i managed Kafka, z efektem na usługi bankowe, dostawcze i nieruchomości.
**Znaczenie:** Kolejny przykład (po wcześniejszych awariach hyperscalerów) kaskadowego wpływu pojedynczej AZ na niepowiązane branże – potwierdza potrzebę multi-AZ/multi-region dla usług krytycznych i runbooków failover niezależnych od jednego dostawcy zasilania.
Źródło: https://www.theregister.com/paas-and-iaas/2026/10/20/google-cloud-says-a-power-event-caused-todays-outage/5304160

#### Cairn: w pełni zautomatyzowane agentowe operacje intrusion – 100+ skompromitowanych celów (John Deere, SharkNinja, Cummins)
**Fakty:** NorForge zidentyfikował >20 publicznie eksponowanych paneli Cairn; jeden chiński klaster uruchomił >1100 projektów ataków (RCE, webshelle, SQLi, auth bypass) na e-commerce/płatności w USA/JP/AU/CA. Odnotowany przypadek: atakujący dostarczył instrukcje w <100 znakach chińskich, autonomiczne agenty przełamały >100 serwisów w ok. 23h. Panele ujawniły cele, credentials, API keys, instrukcje agentów (Trufflehog: 11% zweryfikowanych sekretów aktywnych).
**Znaczenie:** Pierwszy dobrze udokumentowany przypadek masowej, zautomatyzowanej ofensywy z niskim nakładem operatora (natural-language prompt → multi-target breach) – przesuwa próg wejścia dla ofensywnych operacji; segmentacja i monitoring reagujący na anomalny, szybki ruch poprzeczny zyskują priorytet.
Źródło: https://www.theverge.com/ai-artificial-intelligence/1013643/ai-agents-hack-john-deere-shark-ninja-cummins

## Raport AI-ML

### Biznes

#### Huawei–DeepSeek: symbioza sprzętowo-programowa – Ascend 950PR dogania H100 w utylizacji FLOPs
**Analiza (SemiAnalysis):** DeepSeek wydał wsparcie Ascend dla 6 repozytoriów (TileLang, DeepGEMM, DeepEP, TileKernels, FlashMLA, DeepSelect); mikrobenchmarki: Ascend 950PR osiąga mediana 41,5% attainable FLOPs na GEMM compute-bound vs 46,8% dla H100 (częściowo z powodu power/clock limitów H100), a bandwidth utilization HBM przewyższa H100 przy małych transferach. Cytat Liang Wenfenga (V2026): "4 karty Huawei = 1 Nvidia, Huawei ~2 lata za rynkiem sprzętowo" – ale gap softwarowy zamyka się szybciej niż sprzętowy. DeepSeek dostarcza Huawei modele/workloady (dzień-0 optymalizacje dla V4/V4.1), Huawei dostarcza sprzęt i natywny stack (CANN, URMA/HCCL). Ekosystem nadal niedojrzały poza supernode'ami: brak DeepGEMM na x86, zależności od ARM-only bibliotek (memfabric_hybrid), brak CI NPU+x86.
**Znaczenie:** Dla capacity planning poza USA/EU – chiński ekosystem akceleratorów (Huawei Ascend) zyskuje wiarygodny, współdzielony software stack z frontier labem; obniża to efektywny vendor lock-in Nvidii w Chinach i regionach objętych eksport-control, ale pozostaje wąski (ARM/supernode-centric), nie "drop-in" dla ogólnego x86 datacenter.
Źródło: https://newsletter.semianalysis.com/p/huawei-deepseek

### Technologia

#### DeepSeek open-sources DeepInfer: framework produkcyjny – 155 bln tokenów/dzień/klaster, 38 USD przychodu/GPU-h
**Analiza:** DeepInfer to silnik inferencji zasilający produkcję DeepSeek w skali (wersja offline, bez PD-disaggregation/load-balancing z pełnej usługi – wymaga doinstrumentowania przez użytkownika). Działa na Nvidia SM100 i Huawei Ascend 950. Opublikowane metryki ekonomiczne (155T tok/dzień/klaster, 38 USD/GPU-h) to pierwszy publiczny, konkretny benchmark rentowności inferencji dużego dostawcy – użyteczny punkt odniesienia dla kalkulacji $/GPU-h przy capacity planning (patrz też sekcja Ryzyka, "EKONOMIKA").
Źródło: https://github.com/deepseek-ai/DeepInfer

#### TorchTPU: natywny backend PyTorch dla Google TPU (Google + Meta)
**Analiza:** Analogicznie do TorchNeuron (AWS, opisany 21.10), TorchTPU dodaje eager execution i integrację torch.compile dla TPU bez przepisywania kodu pod XLA. Razem z TorchNeuron tworzy wzorzec: duzi cloud-providerzy inwestują w natywne, kompatybilne z PyTorch backendy, by obniżyć bariery migracji z CUDA. Ryzyko identyczne jak przy TorchNeuron: parytet operatorów/kernel performance względem dojrzałego stacku CUDA nieznany na starcie; testować na obciążeniach o stabilnych shape'ach przed produkcją.
Źródło: https://pytorch.org/blog/introducing-torchtpu/

#### NVIDIA Vera: 1024-bit LPDDR5X-9600 (1,228 TB/s), NVLink C2C 780 GB/s, ale cross-socket latency ~660 ns
**Analiza (Chips and Cheese):** Teardown potwierdza silny single-socket memory-bound throughput (ok. 20% przewagi nad Graviton 5 w testach DRAM-bound SPEC2026) przy słabszej wydajności cross-socket/core-to-core względem AMD/Amazon. Cache L3 164 MB, moduły pamięci field-serviceable (SOCAMM2). Wniosek dla capacity planning: Vera dobry fit dla workloadów jednosocketowych memory-bound; dla silnie współbieżnych, cross-socket NUMA workloadów sprawdzić realny impact 660 ns latency przed migracją.
Źródło: https://chipsandcheese.com/p/nvidia-veras-system-architecture

#### NVIDIA DSX POD i OpenShell: standaryzacja wdrożeń AI factory i governance agentów w runtime
**Analiza:** NVIDIA DSX POD – pre-zwalidowana architektura referencyjna celująca w szybsze i bardziej przewidywalne uzyskanie "productive AI capacity" (odpowiedź na problem lead-time, patrz Lenovo AI Express z 21.10). OpenShell Reference Architecture i Sentry – runtime governance/security dla agentów AI (podobny motyw co Microsoft Guardian Agents, sekcja Raport technologiczny) – trend: dostawcy infrastruktury AI dodają natywne warstwy kontroli uprawnień agentów jako standardowy element stosu, nie dodatek.
Źródła: https://developer.nvidia.com/blog/achieve-productive-ai-capacity-faster-and-more-predictably-with-nvidia-dsx-pod/ , https://developer.nvidia.com/blog/how-nvidia-openshell-reference-architecture-secures-and-governs-ai-agent-workloads-at-scale/ , https://developer.nvidia.com/blog/inside-nvidia-sentry-secure-ai-agents-at-runtime/

## Raport Ryzyka rynku AI/GPU

#### Anthropic S-1: strata >80 mld USD (H1 2026) vs deklaracja 750 mld USD CAPEX na compute
**Analiza:** FINANSE – pierwszy publicznie znany, konkretny zestaw liczb (z projektu prospektu IPO) łączący ogromną stratę operacyjną z deklaracją dalszej, znacznie większej ekspansji CAPEX. Ryzyko: skala przyszłych zobowiązań compute (w tym wobec Nscale, ~90 mld USD łącznie w dwóch kontraktach od sierpnia) zależy od sukcesu IPO i rynku kapitałowego – pokrywa się z ryzykiem opisanym 21.10 (pętla finansowania Nscale↔Anthropic).
Źródło: https://techcrunch.com/2026/10/19/anthropic-s-1-shows-red-hot-growth-risks-and-massive-computing-costs/

#### Ekonomika inferencji: DeepSeek publikuje 38 USD przychodu/GPU-h jako punkt odniesienia rynkowy
**Analiza:** EKONOMIKA – jawne ujawnienie metryki przychód/GPU-h przez duży operator inferencji ustanawia punkt odniesienia, wobec którego moża oceniać deklaracje neocloudów o utylizacji i marżach; niska przejrzystość innych dostawców w tym zakresie utrudnia porównania, ale brak porównywalnego publicznego ujawnienia przez zachodnich neocloudów to sam w sobie sygnał asymetrii informacyjnej dla inwestorów.
Źródło: https://github.com/deepseek-ai/DeepInfer

#### "The Gap Between GPUs and Compute": finansowanie, moc i hardware jako realne ograniczenia podaży, nie dostępność chipów
**Analiza:** INFRASTRUKTURA/FINANSE – operatorzy AI infra mają praktycznie nieograniczony popyt, ale finansowanie, dostępność energii, wdrożenie sprzętu i opór społeczny (permitting) ograniczają realną podaż; starsze GPU zachowują wartość dłużej niż oczekiwano (sygnał niedoboru, nie przesytu), a zdolność do zabezpieczenia kapitału i niezawodnej eksploatacji klastrów staje się przewagą konkurencyjną odrębną od samego dostępu do krzemu. Koresponduje z tematem Microsoft/Loughton (21.10) i +50GW SemiAnalysis (20.10).
Źródło: (TLDR AI, link skrócony – oryginał niezweryfikowany bezpośrednio, traktować jako wskazanie tematu, nie cytat źródła pierwotnego)

## Newsletters summary
Przetworzono 6 z 8 maili z etykietą NEWSY (21.10, TLDR IT/AI/InfoSec/Hardware) – kluczowe pozycje ujęte wyżej (Microsoft Guardian Agents, Google Cloud outage, BeyondTrust 0-day, Meta Muse leak, Cairn, DHS/DeepSeek ban, Anthropic S-1, DeepInfer, TorchTPU, Nvidia Vera) oraz 1 mail SemiAnalysis (Huawei-DeepSeek, ujęty w Raporcie AI-ML). Pominięte jako nieistotne dla profilu architekta infra (IGNORED): "The B2B LinkedIn Playbook" (Venture Street, marketing), "Put Some Brain in Your Thinking" (The Algorithmic Bridge, blog osobisty/lifestyle).

## Implikacje praktyczne
1. Appliance'y zdalnego dostępu/supportu (BeyondTrust, Cisco SEG) jako powtarzający się wektor APT – wymusić izolację zarządzania i brak trwałych poświadczeń, traktować self-hosted jako "compromise until proven otherwise" do czasu forensic review po każdym zero-day.
2. Koncentracja ryzyka kontrahenta: ekspozycja na Anthropic/Nscale (łącznie ~90 mld USD w kontraktach od VIII.2026) przy stracie Anthropic >80 mld USD w H1 i IPO obu stron w toku – limity koncentracji, step-in rights przy długoterminowych rezerwacjach GPU.
3. Governance agentów AI jako wymóg architektoniczny, nie opcja: Microsoft (Guardian Agents) i NVIDIA (OpenShell/Sentry) zbieżnie wprowadzają niezależny monitoring agentów – budować observability/guardrails jako warstwę obowiązkową przed produkcyjnym wdrożeniem agentowym.
4. Dywersyfikacja ekosystemu akceleratorów: TorchNeuron + TorchTPU obniżają koszt migracji z CUDA; równolegle Huawei/DeepSeek zamykają software gap dla Ascend – warto utrzymywać warstwę abstrakcji modelu/runtime niezależną od pojedynczego vendora.
5. PKI/trust anchors: incydent eMudhra wymaga przeglądu własnych root/intermediate store'ów i zależności od CA trzecich stron w pipeline'ach CI/CD i mTLS.

## Trend tygodnia
Tydzień 21–22.10 domyka obraz z poprzednich dni: wąskie gardło przesuwa się z samego dostępu do GPU na finansowanie i energię, a nowe dane (S-1 Anthropic, kontrakty Nscale) kwantyfikują skalę tego ryzyka – straty i zobowiązania liczone w setkach miliardów USD, uzależnione od powodzenia IPO. Równolegle rośnie presja na governance autonomicznych agentów AI z dwóch stron: defensywnej (Microsoft, NVIDIA wprowadzają niezależny monitoring agentów jako standard) i ofensywnej (Cairn – udokumentowany przypadek masowej, zautomatyzowanej kompromitacji wielu celów przy minimalnym nakładzie operatora). W warstwie krzemowej Chiny demonstrują realne domykanie przewagi softwarowej (Huawei/DeepSeek), co przy eksport-control zmienia długoterminową kalkulację vendor lock-in. Na brzegu sieci i w zdalnym dostępie utrzymuje się cykl krytycznych zero-day w appliance'ach (BeyondTrust po Cisco SEG), exploitowanych miesiącami przed publicznym ujawnieniem. Dodatkowo incydent PKI (eMudhra) przypomina, że łańcuch zaufania pozostaje krytyczną, rzadko audytowaną zależnością.

## To obserwować
1. Finalizacja/konkretne warunki drugiego kontraktu Nscale–Anthropic (45 mld USD) oraz harmonogram IPO obu spółek.
2. Zakres wpływu zakazu DHS na DeepSeek – ewentualne rozszerzenie na inne agencje/jurysdykcje lub sektor prywatny regulowany.
3. Tempo łatania self-hosted instancji BeyondTrust Remote Support oraz ewentualny wzrost skali exploitacji po publicznym ujawnieniu.
4. Reakcja programów root store (Chrome, Mozilla) na incydent eMudhra – decyzje o distrust/ograniczeniu CA.
5. Dojrzałość ekosystemu Ascend poza supernode'ami (x86 support, CI) jako wskaźnik realnej dostępności alternatywy dla Nvidii w Chinach.
