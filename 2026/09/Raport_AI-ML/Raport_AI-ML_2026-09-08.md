# Raport AI-ML — 2026-09-08

Zakres: od ostatniej edycji z 2026-09-07 do 2026-09-08 (Europe/Warsaw).

## Biznes

### Mistral pozyskuje 3 mld EUR przy wycenie 21 mld EUR

8 września Mistral zamknął rundę współprowadzoną przez PSG Equity, Samsung Electronics i unijny Scaleup Europe Fund. Spółka deklaruje przeznaczenie kapitału na rozwój modeli i frontier research; według CFO jest na ścieżce do 1 mld USD ARR na koniec roku, a ponad 125 klientów może pobierać i dostosowywać modele on-premises. Runda zwiększa zdolność finansowania compute i zespołów systemowych, ale nie zawiera publicznego harmonogramu GPU, mocy MW ani kontraktów na energię, więc nie należy utożsamiać wartości finansowania z dostępną pojemnością.

**Znaczenie architektoniczne:** dla europejskich organizacji rośnie alternatywa dla zamkniętych endpointów amerykańskich dostawców, szczególnie tam, gdzie wymagane są lokalne wagi i kontrola danych. Ryzyko pozostaje w warstwie infrastruktury: zgodność modeli z wieloma akceleratorami, przepustowość dystrybucji artefaktów, niezależność od hostingu Microsoftu oraz koszt utrzymywania wersji on-prem.

**Źródło:** [Reuters — 8 września 2026](https://www.reuters.com/world/europe/french-ai-company-mistral-hits-24-billion-valuation-funding-round-2026-09-08/)

## Technologia

### TPUv7 Ironwood wychodzi poza własne workloady Google przez InferenceX

SemiAnalysis opublikował 7 września pierwsze zewnętrzne wyniki InferenceX dla TPUv7 Ironwood z Qwen3.5 397B FP8 i natywnym stosem TorchTPU dla vLLM. W porównaniu modelowanym przy 100 tokenach/s/użytkownika koszt wyniósł około 0,181 USD za milion tokenów wobec 0,222 USD dla B200 i 0,276 USD dla B300; przy 20 tokenach/s/użytkownika Ironwood osiągnął 9364 tokeny/s/chip, około 5% powyżej obu GPU w tych testach. Wyniki nie obejmują jeszcze pełnej parytetowej optymalizacji speculative decoding i disaggregated serving, a TPUv7 nie ma natywnego FP4, dlatego przewaga nie jest uniwersalna.

Stos wykorzystuje data-parallel attention, expert parallelism, przenoszenie nieregularnych collectives i permutacji MoE na SparseCore oraz kompresję stanu hybrydowego. Połączenie dwóch małych all-gatherów oszczędziło około 80 μs na warstwę, kompaktowa alokacja stanu odzyskała około 76 GiB HBM, a tryb sequence-on-lane podwoił liczbę użytecznych stron KV kosztem około 3% wyższego per-token latency przy niskiej współbieżności. Zapowiedziane otwarcie TorchTPU około października będzie ważniejszym testem przenośności niż pojedyncze benchmarki.

**Znaczenie architektoniczne:** TPU staje się realnym backendem inference dla otwartych modeli, ale wymaga osobnego strojenia buckets, page size, DP/EP i offloadu SparseCore zależnie od concurrency oraz długości wejścia/wyjścia. SLO powinno rozdzielać TTFT, TPOT, throughput/chip, koszt/token i zużycie HBM; nie wolno przenosić wyników 8k/1k na agentowe, wieloturowe workloady bez pomiaru prefix-cache hit rate i stanu GDN.

**Źródło:** [SemiAnalysis — TPU InferenceX Full Steam Ahead, 7 września 2026](https://newsletter.semianalysis.com/p/tpu-inferencex-full-steam)

## Implikacje praktyczne

1. Dodać TPU do macierzy kwalifikacyjnej inference, ale wymagać reprodukowalnych testów dla własnych rozkładów promptów, concurrency, TTFT i TPOT.
2. Oddzielić przenośność modelu od przenośności runtime: testować operator coverage, paged attention, prefix caching, MoE collectives, profilery i ścieżkę rollbacku na GPU.
3. Przy ofertach Mistral wymagać konkretnej lokalizacji wykonania, backendu akceleratora, limitów egress i prawa do mirrorowania wag, tokenizerów oraz kontenerów.
4. Nie kontraktować kosztu/token wyłącznie z benchmarku FP8; porównywać jakość i ekonomię z FP4/NVFP4 oraz koszt operacyjny kompilacji i utrzymania osobnych obrazów.
5. Monitorować HBM jako zasób SLO: fragmentację stron KV, recurrent-state footprint, cache hit rate i presję na eviction, nie tylko procent użycia pamięci.

## Trend tygodnia

Rynek inference przestaje być jednobiegunowy: kapitał europejski finansuje lokalne modele, a Google wystawia własny akcelerator przez coraz bardziej standardowy PyTorch/vLLM. Przewaga sprzętu powstaje dziś z koordynacji runtime, pamięci i fabric, a nie z samej wartości FLOPS. W efekcie vendor portability trzeba mierzyć jako koszt utrzymania zoptymalizowanych ścieżek dla różnych backendów.

## To obserwować

- otwarcie TorchTPU i zakres upstreamowego wsparcia w vLLM/SGLang;
- wyniki Ironwood dla speculative decoding, disaggregated prefill i agentowych workloadów wieloturowych;
- TPUv8i z natywnym FP4 względem Rubin NVL72;
- konkretne inwestycje Mistral w compute, lokalizacje i umowy energetyczne;
- możliwość niezależnego mirrorowania i uruchamiania artefaktów Mistral bez zależności od hostingu producenta.

## Kontrola źródeł

Sprawdzono wskazane kanały SemiAnalysis, NVIDIA Developer Blog, AIwire i HPCwire. Do raportu włączono tylko informacje nowe względem rejestru poprzednich edycji.
