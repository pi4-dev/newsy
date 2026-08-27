# AI for networking — 2026-08-27

**Data podsumowania:** 2026-08-27  
**Analizowany okres:** ostatnie 7 dni, z deduplikacją względem zapisanych raportów (Europe/Warsaw)

## IETF 126: autonomiczne sterowanie siecią i admission control dla agentów AI

Materiały IETF 126 w obszarze ANIMA obejmują autonomiczne zarządzanie ruchem w centrach AI, AI-ASA jako warstwę autonomicznego sterowania, agentic AI dla Intent-Based Networking oraz wymagania dotyczące dopuszczania instancji agentów do sieci i ograniczania ich zakresu dostępu. Są to propozycje i materiały robocze, a nie stabilne standardy; ich znaczenie polega na przesunięciu dyskusji z interfejsu konwersacyjnego w stronę jawnego control plane, tożsamości agenta i ograniczonych capability.

**Mechanizm:** Agent otrzymuje intent i telemetrię, przygotowuje zmianę, a warstwa autonomiczna mapuje ją na politykę i działania sieciowe. Bezpieczny model wymaga rozdzielenia ról observe–plan–validate–execute, kryptograficznej tożsamości instancji, scope-down authorization oraz walidacji stanu przed i po zmianie.

**Wpływ na routing, fabric i observability:** Telemetria musi zachowywać kontekst topologii, czasu i jakości danych; same próbki gNMI nie wystarczą bez korelacji z BGP/EVPN, kolejkami, ECN, utratą i stanem kontrolerów. Dla fabric AI decyzje optymalizacyjne powinny być oceniane według job completion time i tail latency, ale fallback musi pozostać deterministyczny w BGP/IGP/ECMP.

**Failure modes i edge cases:** opóźniona telemetria może wywołać oscylację pętli sterowania; dwa agenty mogą wygenerować sprzeczne intenty; model może zoptymalizować lokalną metrykę kosztem stabilności całej fabric. Wymagane są change budgets, rate limits, commit-confirm/rollback, blokady konfliktów, walidacja w digital twin oraz automatyczne przejście do ostatniej znanej dobrej polityki.

**Źródła:** [IETF 126 — materiały ANIMA](https://datatracker.ietf.org/meeting/126/materials) · [APNIC — AI, IPv6 and the future Internet at IETF 126](https://blog.apnic.net/2026/08/25/ai-ipv6-and-the-future-internet-at-ietf-126/)

---

## IPv6, DNS i SRv6 jako warstwa identyfikacji i sterowania dla rozproszonych agentów

Podsumowanie APNIC wskazuje na rosnącą rolę IPv6 i DNS przy dużej liczbie rozproszonych agentów oraz węzłów compute. NAT i translacje utrudniają jednoznaczną identyfikację, service discovery i audyt, natomiast omawiany rekord DNS AMR ma przenosić statyczne mapowanie IPv4/IPv6 w sieciach IPv6-only; równolegle rozwijane są SRv6 i Compressed SRv6 dla środowisk compute interconnect.

**Mechanizm:** IPv6 zapewnia skalowalną przestrzeń adresową, DNS wiąże tożsamość usługi z aktualnym endpointem, a SRv6 może kodować ścieżkę lub funkcję sieciową potrzebną dla konkretnego workloadu. Dla agentów nie może to jednak oznaczać utożsamienia adresu z tożsamością — autoryzacja musi opierać się na workload identity i polityce, a adres pozostaje sygnałem routingu i obserwowalności.

**Wpływ na architekturę:** Warto projektować telemetry pipeline tak, aby zachowywał IPv6 flow labels, segment lists, DNS provenance i identyfikator workloadu. Ułatwia to korelację decyzji agenta z przepływem, ale zwiększa cardinality TSDB i wymaga kontrolowanego sampling, retencji oraz normalizacji danych.

**Failure modes i edge cases:** dynamiczne adresy i split-horizon DNS mogą rozjechać inventory z rzeczywistym stanem; zbyt długa lista segmentów zwiększa narzut i MTU risk; błędnie wygenerowana polityka SRv6 może ominąć wymagany punkt bezpieczeństwa. Fallback powinien używać klasycznego shortest-path/ECMP, a każda polityka generowana przez AI powinna przechodzić formalne sprawdzenie reachability, loop freedom, MTU i obowiązkowych service chains.

**Źródła:** [APNIC — analiza IETF 126](https://blog.apnic.net/2026/08/25/ai-ipv6-and-the-future-internet-at-ietf-126/) · [IETF 126 — materiały DNSOP, ANIMA i SRv6](https://datatracker.ietf.org/meeting/126/materials)

## Wnioski operacyjne

1. Traktować agentów jako osobne workload identities z krótkotrwałymi poświadczeniami i minimalnym zakresem uprawnień.
2. Rozdzielić generowanie planu od wykonania; produkcyjny executor powinien przyjmować wyłącznie typowane, walidowane operacje.
3. Testować zmiany BGP/EVPN/SRv6 w digital twin i wymagać automatycznego rollbacku przy naruszeniu SLO.
4. Mierzyć stabilność closed loop: czas detekcji, czas decyzji, skuteczność remediacji, false positive, rollback rate i liczbę oscylacji.
5. Zachować deterministyczny tryb pracy sieci bez modelu, z ostatnią znaną dobrą konfiguracją i tradycyjną konwergencją protokołów.
