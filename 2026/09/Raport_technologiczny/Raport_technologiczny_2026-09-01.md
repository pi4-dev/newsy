# Raport technologiczny — 2026-09-01

## Biznes

W analizowanym okresie nie odnotowano odrębnej informacji biznesowej spełniającej próg istotności raportu.

## Technologia

### PaperCut NG/MF: dwie podatności aktywnie wykorzystywane w atakach

CISA dodała 31 sierpnia CVE-2026-81578 i CVE-2026-82078 do katalogu Known Exploited Vulnerabilities, wyznaczając agencjom federalnym termin remediacji na 14 września 2026. Pierwsza luka pozwala nieuwierzytelnionemu napastnikowi zdalnie zmienić wybrane ustawienia administracyjne przed zakończeniem kontroli dostępu; druga umożliwia wykonanie kodu Java z classpath procesu PaperCut po uzyskaniu możliwości manipulowania konfiguracją sterownika bazy danych. Producent udostępnił Emergency Patch Release 2 i zaleca natychmiastową aktualizację.

**Znaczenie:** serwer druku jest uprzywilejowanym punktem integracji z katalogiem, urządzeniami, pocztą i bazą danych. Potwierdzona eksploatacja wymaga nie tylko aktualizacji, lecz także przeglądu zmian konfiguracji, kont administracyjnych, połączeń JDBC i ruchu wychodzącego z serwera; sama blokada panelu od Internetu nie wyklucza nadużycia z przejętej sieci wewnętrznej.

**Źródła:** [CISA — dodanie podatności do KEV](https://www.cisa.gov/news-events/alerts/2026/08/31/cisa-adds-two-known-exploited-vulnerabilities-catalog), [PaperCut — pilny biuletyn z 27 sierpnia 2026](https://www.papercut.com/kb/Main/security-bulletin-27-aug-2026-urgent-security-advisory/), [PaperCut — rejestr podatności](https://www.papercut.com/kb/Main/security-vulnerability-log/)
