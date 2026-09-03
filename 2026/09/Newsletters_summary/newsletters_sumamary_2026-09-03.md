# Newsletters summary — 2026-09-03

**Data podsumowania:** 2026-09-03  
**Źródło:** nieprzeczytane wiadomości w Inboxie z etykietą NEWSY, otrzymane 2026-09-02.

## Abliteracja modeli zniekształca triage podatności

- **Technologia / Zdarzenie:** [Verdict bias w „uncensored” modelach używanych do bug huntingu](https://clearbluejar.github.io/posts/does-abliteration-skew-your-bug-hunting/)
- **Mechanizm działania:** Abliteracja osłabia wyuczone odmowy przez modyfikację reprezentacji aktywacji, ale jednocześnie przesuwa próg decyzji ku potwierdzaniu hipotez. W teście FreeBSD agresywnie abliterationowany model zaakceptował 138 ze 144 kandydatów i nie dotarł do rzeczywistego CVE, podczas gdy model bazowy wskazał je jako pierwszy wynik.
- **Wpływ na architekturę:** W pipeline SAST/RE model i dokładny wariant kwantyzacji muszą być wersjonowane jak reguły skanera; potrzebne są golden sets, precision/recall, budżet kandydatów oraz niezależny walidator.
- **Failure modes i edge cases:** Majority vote nie usuwa skorelowanego yes-bias, a większa liczba kandydatów może wyczerpać context/token budget przed analizą prawdziwego błędu. Deterministyczny fallback to kompilator, sanitizer, PoC/reproducer i ręczna akceptacja przed publikacją.

## AI pomaga przenieść exploit RCE między sterownikami PLC

- **Technologia / Zdarzenie:** [Forescout: port exploit CVE-2021-31886 na WAGO 750-831](https://www.forescout.com/blog/can-ai-create-plc-attacks-yes-but-it%E2%80%99s-not-that-easy-yet/)
- **Mechanizm działania:** Agent korzystał z Ghidry, generowanych skryptów Python i testów sieciowych, aby przenieść pre-auth buffer overflow w serwerze FTP Nucleus na inny model ARM. Osiągnięcie RCE wymagało 8 godzin 32 minut, 1,3 mln tokenów wyjściowych i stałej korekty przez badacza; po uzyskaniu ścieżki wykonania payloady ICMP/UDP powstały w minuty.
- **Wpływ na architekturę:** Ocena ryzyka OT nie może zakładać, że złożony exploit pozostanie ekonomicznie niedostępny. Priorytetem są segmentacja, brokerowany dostęp, wyłączenie FTP/Telnet, detekcja crash-loop i nietypowego egress oraz laboratorium z fizyczną separacją od procesu.
- **Failure modes i edge cases:** Agent generował fałszywe tropy, a próba rozszerzenia payloadu do C2 trwale uszkodziła PLC przez zapis do flash. Fallback musi obejmować snapshoty/emulatory, limit działań na urządzeniu i zatwierdzenie człowieka przed operacją mającą skutek fizyczny.

## Azure i AWS standaryzują prywatny interconnect multicloud

- **Technologia / Zdarzenie:** [Azure Multicloud Interconnect for AWS](https://azure.microsoft.com/en-us/blog/introducing-azure-multicloud-interconnect-for-aws/)
- **Mechanizm działania:** Nie zastosowano modelu AI; control plane łączy Azure Multicloud Interconnect z AWS Interconnect-multicloud przez otwartą specyfikację API. Usługa zapewnia prywatną ścieżkę do 100 Gb/s, integrację z Azure Private Link, MACsec oraz deklarowane cztery dziewiątki dostępności.
- **Wpływ na architekturę:** Automatyzacja skraca provisioning między chmurami i może uprościć przepływ danych treningowych, RAG i inference, ale routing, DNS, MTU, observability i rozliczanie egress pozostają odpowiedzialnością klienta.
- **Failure modes i edge cases:** Wspólny workflow nie eliminuje awarii współdzielonego operatora, asymetrii tras ani konfliktów CIDR. Potrzebne są niezależne testy failover, limity prefiksów, syntetyczne pomiary P95/P99 i zapasowa ścieżka VPN/transit.
