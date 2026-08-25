# Raport technologiczny — 2026-08-25

**Data podsumowania:** 2026-08-25  
**Analizowany okres:** od edycji z 2026-08-24 do 2026-08-25 (Europe/Warsaw)

## Biznes

Brak istotnych nowych informacji biznesowych w analizowanym okresie.

## Technologia

### CISA potwierdza aktywne wykorzystanie CVE-2026-21962 w Oracle HTTP Server i WebLogic Proxy Plug-in

24 sierpnia 2026 r. CISA dodała CVE-2026-21962 do katalogu Known Exploited Vulnerabilities na podstawie dowodów aktywnego wykorzystania. Błąd improper access control o CVSS 10.0 pozwala nieuwierzytelnionemu napastnikowi z dostępem HTTP naruszyć Oracle HTTP Server lub WebLogic Server Proxy Plug-in; Oracle wskazuje jako podatne wersje 12.2.1.4.0, 14.1.1.0.0 i 14.1.2.0.0 (dla plug-inu IIS: 12.2.1.4.0).

**Znaczenie:** Komponent pełni funkcję bramy między serwerem WWW a WebLogic, dlatego ekspozycja może objąć dane i systemy zaplecza, a sam serwer pośredniczący powinien być traktowany jako element strefy brzegowej. Priorytetem jest zastosowanie styczniowego CPU Oracle, identyfikacja publicznie dostępnych instancji oraz analiza logów HTTP i WebLogic pod kątem nieautoryzowanych odczytów lub modyfikacji; samo załatanie nie wyklucza wcześniejszej kompromitacji.

**Źródła:** [CISA — dodanie do KEV, 2026-08-24](https://www.cisa.gov/news-events/alerts/2026/08/24/cisa-adds-one-known-exploited-vulnerability-catalog) · [Oracle Critical Patch Update — January 2026](https://www.oracle.com/security-alerts/cpujan2026.html) · [CVE Record CVE-2026-21962](https://www.cve.org/CVERecord?id=CVE-2026-21962)
