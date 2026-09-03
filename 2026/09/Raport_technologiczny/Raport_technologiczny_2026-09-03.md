# Raport technologiczny — 2026-09-03

**Data podsumowania:** 2026-09-03  
**Okres analizy:** od edycji 2026-09-02 do 2026-09-03 (Europe/Warsaw)

## Biznes

W badanym oknie nie odnotowano odrębnej informacji biznesowej spełniającej próg istotności raportu.

## Technologia

### CISA dodaje siedem aktywnie wykorzystywanych podatności w warstwie dostępu i łańcuchu dostaw

2 września CISA dodała do KEV: CVE-2026-9586 (Sangoma Switchvox, SQL injection), CVE-2026-48710 (Kludex Starlette, request smuggling), CVE-2026-49869 (Kestra OSS, command injection), CVE-2026-59822 (BerriAI LiteLLM, improper authentication), CVE-2026-82329 (JFrog Artifactory, authentication bypass) oraz CVE-2026-83548 i CVE-2026-83549 (SonicWall SMA1000, SSRF i command injection). Wspólny wzorzec to przejęcie control plane albo zaufanego elementu ścieżki dostaw: bramy zdalnego dostępu, repozytorium artefaktów, orchestrator workflow lub proxy modeli.

**Znaczenie:** Zespoły powinny traktować te wpisy jako jeden priorytet operacyjny, nie siedem niezależnych ticketów: zinwentaryzować instancje internet-facing i self-hosted, zaktualizować je, ograniczyć dostęp sieciowy, zrotować tokeny administracyjne oraz przejrzeć tworzenie kont, tokenów i artefaktów. Dla Artifactory JFrog wskazuje poprawione gałęzie 7.111.21, 7.117.28, 7.125.20, 7.133.29, 7.146.38 i 7.161.20; środowiska chmurowe zostały utwardzone przez dostawcę. SonicWall usuwa błędy SMA1000 w hotfixach 12.4.3-03526 i 12.5.0-02952.

**Źródła:** [CISA, 2026-09-02](https://www.cisa.gov/news-events/alerts/2026/09/02/cisa-adds-seven-known-exploited-vulnerabilities-catalog), [JFrog — CVE-2026-82329](https://docs.jfrog.com/releases/docs/jfrog-security-advisories), [SonicWall — SNWLID-2026-0016](https://psirt.global.sonicwall.com/vuln-detail/SNWLID-2026-0016)
