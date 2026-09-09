# AI for networking — 2026-09-09

**Zakres:** 2026-09-02–2026-09-09 (Europe/Warsaw)

## Tool-output injection i luka precedensu w agentowych systemach sieciowych

ARMO opisuje klasę ataku, w której złośliwa instrukcja trafia do agenta jako wynik narzędzia, a wykonanie następuje dopiero w następnym wywołaniu. Kontrola wejścia widzi treść bez działania, a kontrola wykonania widzi poprawne syntaktycznie wywołanie bez pełnego kontekstu; luką jest pierwsza nowa kombinacja narzędzia, argumentu, celu lub ścieżki po odebraniu wyniku.

**Mechanizm działania:** Ochrona wymaga korelacji sekwencji result → następne tool call z profilem zachowania konkretnego agenta. Profil powinien obejmować narzędzie, argumenty i kolejność, a także telemetrię procesów, plików i połączeń sieciowych zbieraną m.in. przez eBPF.

**Wpływ na architekturę sieciową:** W agentowym NetOps sama walidacja konfiguracji nie wystarcza. Control plane powinien rozdzielać planowanie od egzekucji, wiązać każde wywołanie z intentem i zatwierdzonym zakresem urządzeń oraz rejestrować gNMI/API, DNS, sockety i procesy w jednym śladzie audytowym. Nowe cele, prefiksy, komendy lub sekwencje powinny najpierw działać w trybie obserwacji albo wymagać zatwierdzenia.

**Failure modes i edge cases:** Baseline może zostać stopniowo zatruty, legalne operacje awaryjne będą generować false positives, a retry może wielokrotnie wykonać szkodliwy intent. Deterministyczny fallback powinien obejmować allowlistę urządzeń i operacji, limit blast radius, dry-run/digital twin, dwuosobowe zatwierdzenie zmian krytycznych oraz automatyczne zatrzymanie pętli po odchyleniu od SLO.

**Źródło:** [ARMO — Untrusted Tool Output Prompt Injection, 6 września 2026](https://www.armosec.io/blog/untrusted-tool-output-prompt-injection/)