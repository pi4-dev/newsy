# Raport technologiczny — 2026-08-21

**Data podsumowania:** 2026-08-21  
**Okres analizy:** 2026-08-20 – 2026-08-21 (Europe/Warsaw)

## Biznes

Brak istotnych nowych informacji w analizowanym okresie.

## Technologia

### TrueConf Server: dwie krytyczne podatności trafiły do CISA KEV

20 sierpnia 2026 r. CISA dodała do katalogu Known Exploited Vulnerabilities podatności CVE-2026-72529 i CVE-2026-72530 po potwierdzeniu aktywnej eksploatacji. Pierwsza pozwala nieuwierzytelnionemu napastnikowi wywołać przez 4307/TCP nieudokumentowaną funkcję i wykonać skrypt na serwerze (CVSS 9.8); druga umożliwia wyjście z izolowanego środowiska i wykonanie poleceń w systemie hosta (CVSS 9.0). TrueConf wskazuje jako wersje naprawione 5.3.9, 5.4.9 i 5.5.5.

**Znaczenie:** podatności tworzą kompletny łańcuch od wejścia bez uwierzytelnienia do wykonania kodu poza sandboxem, dlatego publicznie osiągalne wdrożenia należy traktować jako potencjalnie skompromitowane, a nie tylko „wymagające patcha”. Oprócz aktualizacji trzeba ograniczyć dostęp do 4307/TCP, sprawdzić procesy potomne i trwałość na hoście, przeanalizować połączenia wychodzące oraz zresetować sekrety dostępne dla usługi.

**Źródła:** [CISA — KEV update z 20 sierpnia 2026](https://www.cisa.gov/news-events/alerts/2026/08/20/cisa-adds-two-known-exploited-vulnerabilities-catalog), [TrueConf — podatności i wersje naprawione](https://trueconf.com/blog/news/security-fixes-updates-and-advisories)
