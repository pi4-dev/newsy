# Newsletters summary — 2026-09-01

## Wybrane informacje

* **Technologia / Zdarzenie:** [Amazon Kiro IDE — prompt injection prowadzący do eksfiltracji danych](https://mindgard.ai/blog/amazon-kiro-data-exfiltration)
* **Mechanizm działania:** Treść złośliwego workspace'u jest interpretowana jako instrukcja dla agenta. Agent odczytuje sekret z pliku `.env`, zapisuje go w `kiroAgent.powersRecommendationUrl`, a wywołanie Kiro Powers powoduje żądanie HTTP do domeny napastnika z sekretem w query stringu; Amazon naprawił opisany wariant w Kiro IDE 0.8.140.
* **Wpływ na architekturę:** Granica zaufania musi obejmować przepływ danych repozytorium → model → narzędzie → konfiguracja IDE → egress. Workspace trust nie wystarcza; potrzebne są per-tool capabilities, kontrola źródeł konfiguracji, blokada nieautoryzowanych domen oraz rejestrowanie przepływu sekretów.
* **Failure modes i edge cases:** Model może potraktować nazwy katalogów, Markdown lub konfigurację jako instrukcje i połączyć legalne operacje w ścieżkę eksfiltracji. Deterministyczny fallback to brak dostępu agenta do sekretów, deny-by-default dla egressu i zatwierdzanie zmian ustawień wpływających na sieć.

* **Technologia / Zdarzenie:** [Fire Ant — implanty IOS XR, przejęcie TACACS i fałszowanie telemetrii](https://www.sygnia.co/blog/fire-ant-evolves-from-hypervisors-to-trusted-infrastructure/)
* **Mechanizm działania:** Aktor wdrożył komponenty dostosowane do control plane IOS XR, tworzył niewidoczne w konfiguracji tunele GRE, przechwytywał ruch i wstrzykiwał bibliotekę do `tac_plus` w celu zbierania poświadczeń. Zmienione ścieżki syslog i poleceń CLI ukrywały zdarzenia oraz konfigurację przed operatorem.
* **Wpływ na architekturę:** Router, TACACS i host zarządzający muszą być traktowane jako osobne źródła dowodowe. Telemetrię należy krzyżowo weryfikować z flow, sąsiedztwami, zewnętrznym syslogiem, AAA, obrazem pamięci i stanem fabric, ponieważ lokalny CLI może świadomie kłamać.
* **Failure modes i edge cases:** Jednoczesne przejęcie routingu, uwierzytelniania i logów niszczy wspólną podstawę detekcji. Fallback powinien obejmować out-of-band management, niezależne konta break-glass, podpisane obrazy, kontrolę integralności i porównywanie operacyjnego stanu interfejsów z commit history.

* **Technologia / Zdarzenie:** [Grow Therapy — agentowy threat hunting na AWS poniżej 500 USD miesięcznie](https://engineering.growtherapy.com/post/threat-hunt-ai-how-we-built-an-ai-security-analyst-on-aws-for-under-500-month)
* **Mechanizm działania:** Żądanie YAML uruchamia pięciofazowy pipeline, w którym Claude Sonnet pobiera dane ze Snowflake, CloudTrail i Datadog, a Claude Opus porównuje je z baseline'em, wzbogaca kontekst, nadaje confidence score i wykonuje adversarial validation. Historyczne false positives są zapisywane w Snowflake i podawane do kolejnych walidacji.
* **Wpływ na architekturę:** Rozdzielenie taniego etapu zbierania od droższego rozumowania ogranicza koszt tokenów i pozwala skalować liczbę huntów. Kluczowe są obserwowalność każdej fazy, budżet per hunt, śledzenie wersji promptów i możliwość odtworzenia zapytań źródłowych.
* **Failure modes i edge cases:** Zanieczyszczony baseline lub pamięć false positives może systematycznie tłumić prawdziwe alarmy; model może także nadać zbyt wysoką pewność niepełnym danym. Fallbackiem pozostają deterministyczne reguły SIEM, ręczne zatwierdzenie działań i zachowanie surowych dowodów poza kontekstem modelu.

* **Technologia / Zdarzenie:** [GitHub Agentic Workflows — agenci uruchamiani przez GitHub Actions](https://github.github.com/gh-aw/)
* **Mechanizm działania:** Instrukcje w Markdown z YAML frontmatter są kompilowane przez `gh aw compile` do zablokowanego workflow GitHub Actions. Zdarzenia i harmonogramy mogą uruchamiać Copilot, Claude Code, Gemini lub Codex, a warstwa wykonawcza dodaje sandbox, ograniczenia uprawnień, safe outputs, limity kosztu i bramki review.
* **Wpływ na architekturę:** Agent staje się częścią CI control plane, dlatego jego SLO, koszt i uprawnienia trzeba zarządzać tak samo jak pozostałe automatyzacje. Zadania interpretacyjne mogą uzupełniać deterministyczne testy, lecz build, lint i deployment powinny pozostać odtwarzalne.
* **Failure modes i edge cases:** Prompt injection z issue lub PR może skłonić agenta do niepożądanego użycia narzędzi, a sprzężenie botów wywołać lawinę workflowów. Fallback to read-only default, blokada wyzwalania przez boty, limity współbieżności, timeouts, review przed zapisem oraz deterministyczne Actions dla operacji produkcyjnych.
