# Newsletters summary — 2026-08-28

Data podsumowania: **2026-08-28**

## NVIDIA NemoClaw/OpenShell — podatności w lokalnym runtime agenta

- **Technologia / Zdarzenie:** [Drive-By Agent Hijacking: persistent model poisoning](https://www.cyera.com/research/nemoclaw-one-website-visit-to-hijack-your-ai-agent)
- **Mechanizm działania:** konfiguracja inference servera wystawiała usługę bez uwierzytelnienia; DNS rebinding pozwalał stronie internetowej dotrzeć do lokalnego API Ollama. Atakujący mógł odczytać lub nadpisać szablon czatu modelu, wstrzykując trwałe instrukcje do kolejnych promptów.
- **Wpływ na architekturę:** lokalny model server musi być traktowany jak uprzywilejowana usługa sieciowa: bind do loopback/Unix socket, walidacja Host/origin, uwierzytelnienie mTLS lub token oraz osobna polityka egress. Telemetria powinna obejmować operacje CRUD na modelach, zmiany templates i zużycie GPU.
- **Failure modes i edge cases:** sama izolacja procesu agenta nie chroni backendu inference; DNS rebinding omija założenie, że ruch z przeglądarki do adresu lokalnego jest zaufany. Fallback powinien blokować modyfikację artefaktów modelu, odtwarzać je z podpisanego magazynu i unieważniać sesję po zmianie digestu.

---

## Microsoft AutoSaddler — automatyczna optymalizacja harnessu z trace’ów

- **Technologia / Zdarzenie:** [Microsoft AutoSaddler](https://github.com/microsoft/AutoSaddler)
- **Mechanizm działania:** system analizuje nieudane trace’y agentów i w trzech fazach — diagnosis/patch, reflection oraz evolution — modyfikuje prompty, definicje narzędzi, middleware i logikę pętli. Kandydaci są wybierani na oddzielnym zbiorze development, a EvoDAG przechowuje pochodzenie zmian; repozytorium raportuje wzrost Pass@1 o 9,0–9,6 pp w pokazanych benchmarkach.
- **Wpływ na architekturę:** optymalizacja harnessu staje się kontrolowanym pipeline’em MLOps z wersjonowaniem kodu, danych, evaluatorów i budżetu eksperymentu. Append-only events i content-addressed candidates ułatwiają audyt oraz wznowienie procesu.
- **Failure modes i edge cases:** optimizer może przeuczyć się na benchmark, wprowadzić regresję narzędzia lub zoptymalizować metrykę zastępczą. Wymagane są rozłączne zbiory testowe, limity kosztu, zatwierdzanie zmian o skutkach ubocznych oraz szybki rollback do podpisanej wersji harnessu.

---

## Google Cloud — FinOps i kolejki off-peak dla agentów

- **Technologia / Zdarzenie:** [Flexible billing and cost controls for agents](https://cloud.google.com/blog/products/ai-machine-learning/flexible-billing-and-cost-controls-for-agents-on-google-cloud)
- **Mechanizm działania:** Gemini Enterprise dodaje pay-as-you-go, wspólne pule kwot, twarde miesięczne limity i estymację kosztu runtime; Flexible Savings Plans deklarują 10–20% rabatu na tokeny. Zapowiedziany tryb deferred execution ma kolejkować kwalifikujące się zadania poza szczytem za do 50% niższy koszt.
- **Wpływ na architekturę:** workloady agentowe warto rozdzielić na ścieżki latency-sensitive i batch/deferred, z odrębnymi SLO oraz budżetami. Pooled quotas ograniczają niewykorzystane licencje, ale zwiększają potrzebę atrybucji kosztu do projektu i agenta.
- **Failure modes i edge cases:** wspólna pula może wywołać noisy-neighbor i wyczerpać budżet krytycznych agentów; twardy cap może przerwać wieloetapowy workflow bez spójnego checkpointu. Fallback powinien obejmować per-agent quota, admission control, checkpoint/resume i degradację do tańszego modelu.

---

## Databricks AI SRE — evidence-first triage w 1500 klastrach Kubernetes

- **Technologia / Zdarzenie:** [How Databricks Uses AI to Accelerate Incident Investigation](https://www.databricks.com/blog/how-databricks-uses-ai-accelerate-incident-investigation)
- **Mechanizm działania:** agent uruchamia równolegle health checks platformy, analizę logów/metryk/trace’ów i zespołowe runbooki, po czym LLM syntetyzuje wynik. Źródła danych pozostają systemami referencyjnymi, a kontrolowane Observability, Deployment i Alerts API normalizują dostęp, uwierzytelnienie oraz rate limiting.
- **Wpływ na architekturę:** Databricks raportuje skalę 1500+ klastrów, 70+ regionów, 150+ zespołów i 2000+ dochodzeń dziennie. Kluczowy wzorzec to deterministyczne kontrole przed reasoningiem, linkowanie każdej rekomendacji do surowego dowodu i zespołowa własność kompozycyjnych runbooków.
- **Failure modes i edge cases:** agent może przeciążyć backend obserwowalności burstami zapytań, skorelować zdarzenia bez związku przyczynowego lub użyć nieaktualnego runbooka. Potrzebne są rate limits, cache, jawna ocena confidence, brak automatycznej remediacji bez guardrails oraz degradacja do zebranych dowodów, gdy root cause pozostaje niepewny.
