# Newsy — 2026-10-05

**Data podsumowania:** 2026-10-05  
**Okno przyrostowe:** od raportu 2026-10-02 do 2026-10-05 08:01 CEST.

## Raport technologiczny

### Technologia

#### FortiMail CVE-2026-104286: aktywna eksploatacja unauthenticated arbitrary file write

Fortinet ujawnił CVE-2026-104286 (CVSS 9.8), aktywnie wykorzystywany przeciw FortiMail. Path traversal + NULL-byte handling pozwalają nieuwierzytelnionemu atakującemu zapisywać pliki w systemie przez management HTTP/HTTPS; CISA dodała podatność do KEV.

**Znaczenie:** internet-facing management plane FortiMail należy traktować jako incydent P1: odciąć ekspozycję, zastosować vendor mitigation, przeprowadzić forensic triage i nie zakładać, że późniejsze załatanie usuwa persistence. Failure mode to pozostawienie web management dostępnego z Internetu lub uznanie braku IOC za dowód braku kompromitacji.

**Źródła:** https://fortiguard.fortinet.com/psirt/FG-IR-26-175 ; https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-104286

#### Cloudflare scala logs, traces i analytics w jeden observability plane

Cloudflare uruchomił wspólny Logs home, request-level Traces w open beta, unified SQL API, custom alerts/dashboards oraz eksport przez Logpush. Tracing obejmuje security rules, transforms, cache, routing, Workers i origin, wspiera W3C trace context oraz eksport OpenTelemetry.

**Znaczenie:** korelacja edge→origin może skrócić MTTR przy 5xx/latency i ograniczyć ręczne sklejanie telemetryki z wielu produktów. Ryzyka to koszt/cardinality telemetryki, sampling ukrywający rzadkie p99/p999 zdarzenia oraz vendor lock-in zapytań i retencji; OTel export powinien pozostać ścieżką niezależności.

**Źródło:** https://blog.cloudflare.com/one-observability-platform/

#### Cloudflare K2: durable event log na object storage zamiast klasycznego Kafka substrate

K2 przechowuje partycjonowany ordered log na R2, wykorzystując atomic operations do offsetów bez osobnego coordination service. Compute i storage skalują się niezależnie; batching segmentów redukuje koszt storage, ale pierwsza wersja ma około 1 s p99 produce latency.

**Znaczenie:** wzorzec object-storage-backed log jest atrakcyjny dla telemetryki i wysokiego fan-out, ale nie dla latency-sensitive event processing. Brak message-level retries i wyższa produce latency wymagają rozdzielenia workloadów queue/stream; awaria konsumenta jest tolerowana dzięki długiej retencji.

**Źródło:** https://blog.cloudflare.com/cloudflare-k2-streams/

## AI for networking

### Clef-flash: mały decision model jako element hot path automatyzacji

- **Technologia / Zdarzenie:** Cloudflare Clef/Clef-flash — https://blog.cloudflare.com/clef-decision-models/
- **Mechanizm działania:** model zwraca ograniczone structured decisions bez generowania tekstu pośredniego; Clef-flash osiąga w opublikowanych testach medianę 38.8 ms i p95 122.4 ms. Pipeline RL obejmuje AI Gateway dataset, rollouts, sandbox i ponowne wdrożenie modelu.
- **Wpływ:** decision model może wykonywać klasyfikację/policy selection w hot path taniej i szybciej niż pełny LLM, co jest interesujące dla AIOps i policy automation.
- **Failure modes:** benchmark producenta nie zastępuje testu domenowego; błędna decyzja jest szybsza, ale nadal błędna. Wymagane confidence thresholds, deterministic fallback, shadow mode i audyt dataset drift.

### Implikacje praktyczne

1. Dla AI-driven network automation rozdzielać reasoning LLM od szybkiego, ograniczonego decision plane.
2. Wymagać shadow-mode i replay testów przed wpuszczeniem modelu do ścieżki zmian sieciowych.
3. Telemetrię edge/fabric eksportować w otwartym formacie (OTel/gNMI), nawet jeśli analiza odbywa się w vendor platform.
4. Nie używać object-storage-backed streamów do pętli sterowania wymagających sub-second deterministic latency.

### Trend tygodnia

Warstwa observability i automation przesuwa się w stronę wspólnego data plane, nad którym działają wyspecjalizowane modele decyzyjne. Jednocześnie trwa rozdzielanie storage od compute w stream processing. Efektem jest łatwiejsze skalowanie telemetryki, ale większa zależność od jakości danych, sampling policy i poprawności automatycznych decyzji.

### To obserwować

- p95/p99 Clef-flash na rzeczywistych policy workloads;
- Cloudflare Traces sampling i koszt eksportu OTel;
- K2 p99 produce latency po kolejnych iteracjach;
- możliwość niezależnego replay/audytu decyzji modeli;
- interoperacyjność telemetryki poza platformą dostawcy.

## Newsletters summary

### FortiMail zero-day: management plane jako aktywna ścieżka kompromitacji

- **Technologia / Zdarzenie:** CVE-2026-104286, aktywna eksploatacja FortiMail.
- **Mechanizm działania:** unauthenticated path traversal/NULL-byte flaw umożliwia zapis plików przez HTTP/HTTPS management interface.
- **Wpływ:** appliance pocztowy może stać się persistent foothold na granicy sieci; wymagane mitigation i forensic triage, nie tylko późniejszy patch.
- **Failure modes:** kompromitacja sprzed mitigacji, persistence poza artefaktem naprawianym przez patch, management wystawiony publicznie.

### Pi Durable: checkpointowany runtime dla długo działających agentów

- **Technologia / Zdarzenie:** Pi Durable — https://earendil.com/posts/pi-durable/
- **Mechanizm działania:** każdy krok jest durable taskiem z checkpointem; po crashu runtime odtwarza niedokończone zadania. requestId zapewnia exactly-once submission, a application state jest commitowany atomowo razem z transcript.
- **Wpływ:** agent może przeżyć restart procesu/hosta bez utraty workflow; model operacyjny zaczyna przypominać durable workflow engine, a nie sesję chat.
- **Failure modes:** tool call musi jawnie deklarować bezpieczeństwo ponowienia; źle oznaczona operacja side-effect może zostać wykonana ponownie. Exactly-once na wejściu nie oznacza exactly-once dla zewnętrznego systemu.

### Clef/Clef-flash: bounded decisions zamiast pełnego LLM

- **Technologia / Zdarzenie:** wyspecjalizowane decision models z structured output.
- **Mechanizm działania:** klasyfikacja/decyzja bez generowania długiego chainu tekstowego obniża latency i koszt.
- **Wpływ:** potencjalny komponent szybkich policy gates dla agentów i AIOps.
- **Failure modes:** domain shift, benchmark overfitting i brak deterministic fallback mogą propagować błędne decyzje do automatyki.
