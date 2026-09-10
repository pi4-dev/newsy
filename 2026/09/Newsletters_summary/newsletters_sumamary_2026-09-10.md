# Newsletters summary — 2026-09-10

**Data podsumowania:** 2026-09-10  
**Źródło:** nieprzeczytane wiadomości w Inboxie z etykietą NEWSY  
**Język:** polski

## 1. Megakernel serving engine dla North Mini Code

- **Technologia / Zdarzenie:** [Cohere — Inside the megakernel serving engine for North Mini Code](https://cohere.com/blog/megakernels)
- **Mechanizm działania:** Silnik zamienia cały krok decode modelu 30B MoE w jeden persistent kernel na H100. Host buduje listy kafelkowych zadań dla SM, zależności są egzekwowane licznikami w pamięci globalnej, a statyczny scheduler łączy się z dynamicznym work stealing dla attention i MoE; wspólny ABI obejmuje continuous batching, paged attention i sekwencje ragged.
- **Wpływ na architekturę:** W BF16 na pojedynczym H100 implementacja osiąga 292 tok./s przy batch=1 i 256K kontekstu, czyli 1,58× vLLM w teście Cohere; zysk pochodzi głównie z ograniczenia launch/synchronization overhead, wave quantization i lepszego wykorzystania pasma HBM. To poprawia latency i koszt inference dla małych batchy, lecz optymalizacja jest silnie związana z architekturą Hopper, ABI kernela i konkretnym grafem modelu, więc wymaga osobnych SLO oraz profili po każdej zmianie modelu.
- **Failure modes i edge cases:** Błędne zliczanie barier może prowadzić do cichej korupcji danych lub zakleszczenia długo po źródłowym błędzie; spin-wait i statyczny plan zadań mogą tracić efektywność przy nietypowych rozkładach sekwencji i ekspertów. Deterministyczny fallback powinien przełączać ruch na zweryfikowany silnik vLLM/SGLang, a testy muszą obejmować poprawność numeryczną, długi kontekst, skrajny ragged batching i timeouty work queue.

---

## 2. RoboTok: wyszukiwanie demonstracji z internetowych nagrań dla robotyki

- **Technologia / Zdarzenie:** [Rice University / NVIDIA — RoboTok](https://rice-robotpi-lab.github.io/RoboTok/)
- **Mechanizm działania:** Pipeline filtruje nagrania internetowe, rekonstruuje 3D trajektorie dłoni w układzie odniesienia torsu i uczy zwarty embedding nadzorowany podobieństwem DTW. W zapytaniu wykorzystuje wyszukiwanie cosinusowe zamiast kosztownego DTW po całym korpusie, dzięki czemu może stale indeksować duże zbiory demonstracji i wybierać dane do trenowania polityk manipulacji.
- **Wpływ na architekturę:** Na korpusie 100 tys. klipów RoboTok osiągnął mAP@20 0,3531, nDCG@20 0,7757 i MRR@20 0,8576, a polityki oparte na pobranych demonstracjach miały najlepszy wynik w 5/6 łatwych i 3/3 trudniejszych zadań symulacyjnych. Rozdziela to koszt pozyskiwania danych od fizycznej teleoperacji, ale dodaje warstwę ETL dla video, estymacji pozy, wektorowego indeksu oraz lineage klip → embedding → checkpoint → polityka.
- **Failure modes i edge cases:** Dane webowe dziedziczą bias, problemy licencyjne i błędy estymacji 3D przy okluzjach; metryki retrieval oparte na DTW nie gwarantują bezpieczeństwa ani skuteczności na realnym robocie. Wyniki downstream są obecnie symulacyjne, a strona oznacza rezultaty real-world jako „coming soon”, więc wdrożenie powinno wymagać walidacji na fizycznym sprzęcie, filtrów jakości, deduplikacji i deterministycznych ograniczeń sterowania niezależnych od polityki uczonej.
