# Raport technologiczny — 2026-09-14

Okres: od edycji 2026-09-10 do 2026-09-14, 07:30 CEST.

## Biznes

Brak odrębnego nowego zdarzenia spełniającego próg istotności.

## Technologia

### GitLab: CVE-2026-85706, odczyt plików bez uwierzytelnienia

10 września GitLab wydał poprawki 19.1.8, 19.2.6 i 19.3.2 dla self-managed CE/EE. Luka w API commitów repozytorium pozwala w określonych warunkach odczytywać dowolne pliki serwera bez logowania (CVSS 10.0); 11 września watchTowr zaobserwował sondowanie podatności w sieci honeypotów, co nie jest dowodem udanego włamania do konkretnych klientów. Poprawiono również kilka odrębnych problemów uprawnień do zmiennych CI/CD i zatwierdzania wdrożeń.

**Znaczenie:** Pilnie zinwentaryzować publiczne instancje self-managed, wdrożyć odpowiednią gałąź poprawki i sprawdzić logi API oraz ekspozycję tokenów/sekretów przechowywanych na hoście; GitLab.com jest już poprawiony, a GitLab Dedicated nie wymaga akcji. Źródła: [komunikat GitLab](https://docs.gitlab.com/releases/patches/patch-release-gitlab-19-3-2-released/), [obserwacje watchTowr](https://watchtowr.com/resources/rapid-reaction-gitlab-critical-path-traversal-vulnerability-cve-2026-85706/).
