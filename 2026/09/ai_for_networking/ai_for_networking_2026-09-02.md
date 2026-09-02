# AI for networking — 2026-09-02

**Data podsumowania:** 2026-09-02  
**Okres analizy:** od edycji 2026-08-27 do 2026-09-02 (Europe/Warsaw)

## Cloudflare Adaptive Intelligence: ciągłe uczenie detekcji botów

Cloudflare uruchomił pierwszy etap Adaptive Intelligence w Bot Management. Silnik agreguje JA4/TLS fingerprints, strukturę żądań, wyniki challenge, zachowanie sesji, reputację sieciową i telemetrię kliencką, po czym stale przeucza model stojący za istniejącym bot score. Nowe wagi są najpierw uruchamiane w shadow mode i porównywane z wersją produkcyjną m.in. na solve rate, precision i recall.

**Mechanizm działania:** Klasyfikacja jest probabilistyczna i wielosygnałowa, a ocena obejmuje równolegle krótkie i długie okna czasowe. Pozwala to korelować wolny credential stuffing rozproszony po residential proxies, mimo że żaden adres nie przekracza klasycznego rate limitu. Zapowiedziane, lecz jeszcze niewdrożone elementy obejmują automatyczne generowanie i losową rotację krótkotrwałych reguł.

**Wpływ na architekturę:** Bot score pozostaje stabilnym interfejsem dla WAF, natomiast semantyka modelu może zmieniać się bez wersjonowania po stronie klienta. SLO musi więc obejmować nie tylko catch rate, ale również false-positive rate, challenge solve rate, dry-run/shadow telemetry i możliwość szybkiego rollbacku polityki egzekwowania.

**Failure modes i edge cases:** Drift ruchu legalnego może pogorszyć recall lub blokować rzeczywistych użytkowników; opóźniona reakcja i niedeterministyczne ważenie sygnałów utrudniają reprodukcję incydentu. Automatyczna pętla obronna może też oscylować z adaptującym się botem. Należy zachować deterministyczne limity, allowlisty krytycznych integracji, niezależny sampling logów oraz kill switch dla automatycznych zmian.

**Źródło:** [Cloudflare, 2026-08-31](https://blog.cloudflare.com/introducing-adaptive-intelligence/)
