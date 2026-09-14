# Newsletters summary — 2026-09-14

Źródło wejściowe: nieprzeczytane newslettery TLDR AI i TLDR z 11.09.2026 z etykietą NEWSY; zweryfikowano pełne teksty źródłowe. Nie powtarza tematów pozostałych dzisiejszych raportów.

## 1. OpenAI Agents API — zarządzany harness agentów

* **Technologia / Zdarzenie:** [Agents API w public beta, 10.09.2026](https://openai.com/index/introducing-the-agents-api/).
* **Mechanizm działania:** Udostępniono warstwę orkiestracji Codex: sesje długotrwałe, zarządzanie kontekstem, wywołania narzędzi i podagentów oraz środowiska uruchomieniowe hostowane lub własne. Producent nie deklaruje odrębnego algorytmu ML odpowiedzialnego za harmonogram; w tym ogłoszeniu istotny jest harness i izolacja wykonania, nie nowy model.
* **Wpływ na architekturę:** Upraszcza utrzymanie trwałych sesji i burstowej równoległości, ale przenosi część control plane, telemetrii i rozliczania do dostawcy; porównać end-to-end latency, koszt sesji, odzyskiwanie po przerwaniu i eksport śladów z własnym stosem.
* **Failure modes i edge cases:** Złośliwa odpowiedź narzędzia, nieograniczona delegacja i niezamierzone działania po wznowieniu sesji wymagają limitów uprawnień, zatwierdzania zmian, idempotentnych operacji i deterministycznego fallbacku do ręcznej procedury.

## 2. PlanetScale Neki — cykl zapytania na shardowanym PostgreSQL

* **Technologia / Zdarzenie:** [Opis ścieżki zapytania w Neki, 10.09.2026](https://planetscale.com/blog/the-lifecycle-of-a-sharded-postgres-query).
* **Mechanizm działania:** Router implementuje protokół i uwierzytelnianie PostgreSQL, mapuje shard key przez hash, planuje scatter-gather i wykonuje hash join albo zleca lokalny JOIN shardom. To deterministyczny planer SQL, nie model AI; zmiana klucza shardingowego z `orders.id` na `orders.customer_id` pozwala ulokować powiązane dane razem.
* **Wpływ na architekturę:** Colocation zmniejsza liczbę zapytań do shardów i presję na RAM routera; dla wysokokardynalnej telemetrii oznacza to konieczność dobrania klucza do typowych JOIN/filtrów oraz pomiaru fan-out, p99 latency, spill i przeciążenia pooli.
* **Failure modes i edge cases:** Nierównomierny rozkład kluczy, międzyshardowe JOIN-y i awaria sidecara/poola mogą odwrócić zysk; wymagać testów planner fallback, retry z ograniczeniem, budżetów pamięci i izolacji zapytań ad hoc.
