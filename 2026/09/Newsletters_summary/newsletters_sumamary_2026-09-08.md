# Newsletters summary — 2026-09-08

Podsumowanie nowych nieprzeczytanych wiadomości oznaczonych etykietą `NEWSY`, dopasowane do profilu infrastruktury, sieci i AI/ML.

## Random Attention — bezsygnałowa redukcja KV cache

- **Technologia / Zdarzenie:** [Salesforce AI Research — Random Attention](https://github.com/SalesforceAIResearch/Random-Attention)
- **Mechanizm działania:** Polityka `random_pp` zachowuje prompt, krótki recency window i losowo wybraną, równomierną próbkę wygenerowanych wpisów osobno dla każdej głowy KV. Nie odczytuje attention scores, statystyk wartości ani danych kalibracyjnych; koszt rundy eviction sprowadza się do kompakcji. Autorzy raportują porównywalną lub wyższą jakość niż SnapKV, R-KV, VaSE i TriAttention na kilku benchmarkach reasoning oraz port do vLLM.
- **Wpływ na architekturę:** Brak modelu selektora usuwa dodatkowy inference path i upraszcza scaling przy presji HBM. Metryki produkcyjne muszą jednak obejmować jakość przy konkretnym budżecie KV, czas kompakcji, throughput i stabilność między seedami; repozytorium referencyjne testowano na 8×H200.
- **Failure modes i edge cases:** Losowe usunięcie rzadkiego, ale krytycznego tokenu może powodować skokową degradację na długich zależnościach. Fallbackiem jest brak eviction dla promptu i minimalnego okna świeżości, deterministyczny seed do reprodukcji oraz powrót do pełnego KV cache po wykryciu spadku jakości.

## LLM-as-a-Verifier — probabilistyczna kontrola trajektorii agentów

- **Technologia / Zdarzenie:** [LLM-as-a-Verifier 0.2.0](https://github.com/llm-as-a-verifier/llm-as-a-verifier)
- **Mechanizm działania:** Framework rozkłada ocenę na kryteria, powtarza weryfikację i wykorzystuje pełny rozkład log-probability nad tokenami oceny zamiast pojedynczego werdyktu. Probabilistic Pivot Tournament redukuje ranking N trajektorii z O(N²) do O(Nk); wersja 0.2.0 dodaje prefix-cache optimization, która w opublikowanym teście zwiększyła hit rate z 5,2% do 78,4% i zmniejszyła liczbę niecache'owanych tokenów wejścia około 3,4×.
- **Wpływ na architekturę:** Verifier może pełnić warstwę admission control przed wykonaniem zmian IaC lub closed-loop remediation, ale sam staje się kosztowną usługą inference zależną od logprobs, cache i stabilności kryteriów. Należy osobno monitorować koszt weryfikacji, odsetek cache hits, rozrzut ocen i false accept/false reject.
- **Failure modes i edge cases:** Ten sam model generujący i oceniający może współdzielić błędy, a kryteria podatne na prompt injection mogą zatwierdzić fałszywą narrację. Wymagane są deterministyczne testy, walidatory składni/semantyki, cyfrowy bliźniak sieci oraz human approval dla operacji o dużym blast radius.

## JetBrains Cadence — niezałatany control plane ujawnił sekrety workloadów

- **Technologia / Zdarzenie:** [JetBrains — incydent Cadence i CVE-2026-63077](https://blog.jetbrains.com/pycharm/2026/08/cadence-security-incident-august-2026/)
- **Mechanizm działania:** Cadence używał TeamCity do orkiestracji zadań i pozostał podatny na unauthenticated RCE przez deserializację niezaufanych danych. Atakujący przejęli środowisko, backup z 2024 r., dane osobowe, część AWS IAM credentials oraz dostęp do plików S3; JetBrains wskazuje okres aktywności 8–24 sierpnia i przyznaje, że serwer nie został załatany.
- **Wpływ na architekturę:** Control plane wykonujący kod użytkownika nie może przechowywać długowiecznych sekretów w backupach ani przekazywać wspólnych credentials do jobów. Potrzebne są krótkotrwałe tokeny scope-bound, izolacja tenantów, inwentaryzacja zależnych kont i możliwość masowej rotacji poza kompromitowanym orchestrator-em.
- **Failure modes i edge cases:** Lista sekretów producenta nie jest kompletna, a brak IOC nie wyklucza kompromitacji. Fallback powinien traktować wszystkie wykonania i ich artefakty jako niezaufane, rotować credentials w systemach docelowych oraz auditować repozytoria, registry, IAM i storage od początku potwierdzonego okna.

## Źródłowe newslettery

- TLDR AI, wydanie z 2026-09-07.
- TLDR Information Security, wydanie z 2026-09-07.
