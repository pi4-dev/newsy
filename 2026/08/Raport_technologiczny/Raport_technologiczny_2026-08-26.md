# Raport technologiczny — 2026-08-26

**Data podsumowania:** 2026-08-26  
**Analizowany okres:** od edycji z 2026-08-25 do 2026-08-26 (Europe/Warsaw)

## Biznes

Brak istotnych nowych informacji biznesowych w analizowanym okresie.

## Technologia

### CISA potwierdza aktywne wykorzystanie krytycznego RCE w Gitea

25 sierpnia 2026 r. CISA dodała CVE-2026-60004 do katalogu Known Exploited Vulnerabilities. Podatność w API `diffpatch` pozwala spreparowanej zawartości repozytorium zainstalować hook Git i wykonać polecenia z uprawnieniami procesu Gitea; podatne są wersje 1.17–1.27.0, a poprawka znajduje się w 1.27.1. Przy otwartej samorejestracji atakujący może sam utworzyć konto i zapisywalne repozytorium, więc praktyczna ścieżka ataku może być zdalna i bez wcześniejszych poświadczeń.

**Znaczenie:** Kompromitacja hosta Gitea może objąć sekrety aplikacji, repozytoria, poświadczenia baz danych oraz tokeny CI/CD, dlatego incydent ma potencjał supply-chain. Należy pilnie zaktualizować do 1.27.1 lub nowszej, wyłączyć otwartą rejestrację do czasu aktualizacji, zinwentaryzować publicznie dostępne instancje i potraktować wcześniej wystawione systemy jako kandydatów do analizy powłamaniowej oraz rotacji sekretów.

**Źródła:** [CISA — dodanie do KEV, 2026-08-25](https://www.cisa.gov/news-events/alerts/2026/08/25/cisa-adds-one-known-exploited-vulnerability-catalog) · [Gitea Security Advisory GHSA-rcr6-4jqh-j84m](https://github.com/go-gitea/gitea/security/advisories/GHSA-rcr6-4jqh-j84m) · [Gitea 1.27.1](https://blog.gitea.com/release-of-1.27.1/)
