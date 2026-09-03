# Raport AI-ML — 2026-09-03

**Data podsumowania:** 2026-09-03  
**Okres analizy:** od edycji 2026-09-02 do 2026-09-03 (Europe/Warsaw)

## Biznes

### Broadcom: 16,7 mld USD kwartalnego przychodu z półprzewodników AI

Broadcom podał, że w Q3 FY2026 przychód z półprzewodników AI wyniósł 16,7 mld USD, rosnąc o 221% rok do roku i 54% kwartał do kwartału; na Q4 prognozuje 21,7 mld USD. Kategoria obejmuje custom AI accelerators i networking, a więc potwierdza przesunięcie wartości z pojedynczego GPU na projektowany wspólnie XPU, fabric i oprogramowanie.

**Analiza techniczna:** Custom ASIC może obniżyć koszt i energię na token przy stabilnym profilu, ale zwiększa zależność od konkretnego kompilatora, topologii sieci i długiego cyklu tape-out. Capacity planning powinien rozdzielać przepustowość akceleratora od fabric bandwidth, pamięci i dostępności pakowania, a SLO uwzględniać możliwość przeniesienia krytycznego serving na GPU lub drugi ASIC.

**Źródła:** [Broadcom — wyniki Q3 FY2026, 2026-09-02](https://investors.broadcom.com/news-releases/news-release-details/broadcom-inc-announces-third-quarter-fiscal-year-2026-financial), [Broadcom — formularz 8-K](https://investors.broadcom.com/static-files/9d426ddc-fa55-4fa1-8685-fc64642cbdad)

### Vertiv kupuje UtilityInnovation Group za 1,45 mld USD plus earn-out

Vertiv zawarł umowę przejęcia UtilityInnovation Group za około 1,45 mld USD w gotówce oraz do 1,15 mld USD zależnie od EBITDA. Transakcja dodaje sterowanie mikrosiecią, orkiestrację lokalnego wytwarzania i magazynowania energii, switchgear oraz projektowanie architektury behind-the-meter; zamknięcie jest oczekiwane w Q4 2026.

**Analiza techniczna:** Ograniczeniem wdrożeń AI staje się czas do dostępnej i sterowalnej mocy, nie tylko dostawa serwerów. Mikrosieć może skrócić time-to-power i zwiększyć odporność, ale tworzy nową domenę awarii obejmującą automatykę energetyczną, paliwo, magazyny, synchronizację z siecią i cyberbezpieczeństwo OT; SLO klastra trzeba powiązać z telemetryką zasilania, stanem wyspowym i testami black-start.

**Źródła:** [Vertiv, 2026-09-02](https://www.vertiv.com/en-us/about/news-and-events/corporate-news/2026/vertiv-announces-agreement-to-acquire-utilityinnovation-group-to-accelerate-time-to-power-for-ai-data-centers/), [Reuters, 2026-09-02](https://www.reuters.com/legal/transactional/vertiv-strikes-145-billion-deal-microgrid-firm-utility-innovation-group-2026-09-02/)

## Technologia

### Speculative decoding: dobór draftu według acceptance length i kosztu weryfikacji

NVIDIA opisała dobór speculative decoding, w którym mały model lub mechanizm draftujący proponuje kilka tokenów, a model docelowy weryfikuje je równolegle. Porównano m.in. external draft models, EAGLE-3, MTP, DFlash, DSpark i n-gram; korzyść zależy od acceptance length, kosztu draftu, rozmiaru batcha, granic tile 128 oraz presji na KV cache.

**Analiza techniczna:** Technika zachowuje sekwencję modelu docelowego, o ile nie poluzowano kryterium akceptacji, lecz nie daje stałego mnożnika przyspieszenia. Trzeba ją benchmarkować na własnym rozkładzie promptów i P95/P99 TTFT/ITL, mierząc acceptance length per domena, dodatkową pamięć oraz regresję po aktualizacji modelu; draft należy wersjonować i skalować niezależnie od targetu.

**Źródło:** [NVIDIA Developer Blog, 2026-09-02](https://developer.nvidia.com/blog/co-designing-ai-models-using-speculative-decoding-for-faster-llm-inference/)

## Implikacje praktyczne

1. Rozdzielać kontrakty i capacity plan na akcelerator, pamięć, fabric, energię i chłodzenie; wzrost przychodu ASIC nie gwarantuje dostępności kompletnego racka.
2. Utrzymywać ścieżkę fallback z custom ASIC na GPU lub drugi backend, z testem zgodności modeli, kompilatorów i serving API.
3. W speculacji monitorować acceptance length, draft overhead, KV-cache pressure oraz P99 TTFT/ITL per klasa ruchu; automatycznie wyłączać draft przy ujemnym zysku.
4. W projektach z mikrosiecią włączyć OT do threat modelu, observability i ćwiczeń DR, w tym testów pracy wyspowej i black-start.
5. Wiązać płatności za nowe centra AI z osiągniętą mocą IT, testami pod obciążeniem i SLO całej ścieżki od źródła energii do tokenu.

## Trend tygodnia

Wartość i ryzyko przenoszą się z pojedynczego GPU na cały system: custom XPU, sieć, pamięć, serving oraz zasilanie. Wyniki Broadcomu pokazują skalę popytu na projektowane wspólnie akceleratory i fabric, a ruch Vertiv potwierdza, że sterowanie energią staje się częścią produktu infrastrukturalnego. Równolegle optymalizacje takie jak speculative decoding wymagają strojenia do rzeczywistego ruchu, zamiast polegania na jednym wyniku benchmarku. Przewagę daje dziś spójna telemetria kosztu i niezawodności od sieci energetycznej do P99 token latency.

## To obserwować

- realizację prognozy Broadcomu 21,7 mld USD przychodu AI w Q4 i udział networkingu wobec custom accelerators;
- zamknięcie przejęcia UIG oraz integrację telemetryki mikrosieci z platformą Vertiv;
- acceptance length i rzeczywisty speedup EAGLE-3, MTP, DFlash i DSpark na produkcyjnych profilach;
- dostępność alternatywnych backendów dla custom XPU i koszt przeniesienia kerneli;
- awarie i SLO centrów korzystających z bridge-to-grid oraz pracy wyspowej.
