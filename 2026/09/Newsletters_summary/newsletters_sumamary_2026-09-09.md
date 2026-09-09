# Newsletters summary — 2026-09-09

**Data podsumowania:** 2026-09-09

## 1. Accelerator Agents: agenci do migracji PyTorch→JAX i optymalizacji kerneli TPU

* **Technologia / Zdarzenie:** [Google AI Hypercomputer — Accelerator Agents](https://github.com/AI-Hypercomputer/accelerator-agents)
* **Mechanizm działania:** MaxCode prowadzi wspomaganą przez człowieka migrację kodu PyTorch do JAX/MaxText, a MaxKernel generuje i stroi kernele Pallas, wykorzystując profilowanie oraz harness testowy do porównania poprawności i wydajności. Projekt używa agentów Gemini i jest rozwijany jako repozytorium Apache 2.0, lecz nie jest oficjalnie wspieranym produktem Google.
* **Wpływ na architekturę:** Automatyzuje kosztowną warstwę portowania na TPU i może skrócić iterację optymalizacyjną, ale zwiększa zależność od JAX, Pallas i specyficznych profili sprzętowych. Pipeline powinien przechowywać wyniki benchmarków, wersje kompilatora i regresje numeryczne jako artefakty.
* **Failure modes i edge cases:** Agent może wygenerować kod poprawny dla testowego kształtu tensora, lecz niestabilny numerycznie lub wolniejszy w produkcyjnym rozkładzie wejść. Wymagane są deterministyczne testy referencyjne, tolerancje błędu, benchmark wielu kształtów i ręczne zatwierdzenie przed wdrożeniem.

## 2. RAPTOR: eksperymentalny agentowy pipeline analizy, eksploatacji i łatania

* **Technologia / Zdarzenie:** [RAPTOR — Autonomous Software Vulnerability Management](https://github.com/gadievron/raptor)
* **Mechanizm działania:** Framework łączy analizę statyczną i binarną z walidacją LLM, generowaniem exploita, testem reprodukcji i propozycją poprawki. Domyślnie korzysta z Claude Code, ale dopuszcza inne modele; wyniki są weryfikowane testami mechanicznymi i analizą statyczną.
* **Wpływ na architekturę:** Może zintegrować discovery, triage i remediation w jednym pipeline AppSec, jednak uruchamianie generowanych exploitów wymaga odseparowanego execution plane, kontrolowanej sieci i pełnego audytu artefaktów. Projekt jest eksperymentalny i nie powinien sterować produkcyjną remediacją bez bram CI/CD.
* **Failure modes i edge cases:** Błędna walidacja LLM może oznaczyć false positive jako exploitable, wygenerowany payload może wyjść poza testowany cel, a automatyczna poprawka może wprowadzić regresję. Fallback to izolowany kontener/VM bez sekretów, egress deny-by-default, limity czasu i zasobów, testy regresyjne oraz obowiązkowy human review.

**Źródła newsletterów:** TLDR AI oraz TLDR Information Security, wiadomości z 8 września 2026.