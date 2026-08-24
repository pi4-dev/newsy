# Raport technologiczny — 2026-08-24

**Data podsumowania:** 2026-08-24  
**Zakres:** od poprzedniej edycji z 2026-08-21 do 2026-08-24, Europe/Warsaw.

## Biznes

Brak istotnych nowych informacji biznesowych spełniających kryteria raportu.

## Technologia

### Zimbra: aktywnie wykorzystywany RCE w komponencie SNMP

CISA dodała CVE-2026-73570 do katalogu Known Exploited Vulnerabilities 21 sierpnia. Błąd w Zimbra Collaboration przed 10.1.20 pozwala nieuwierzytelnionemu napastnikowi wysłać spreparowane żądanie SMTP i wykonać polecenia systemowe jako użytkownik Zimbra, jeśli zainstalowano opcjonalny pakiet `zimbra-snmp` i włączono powiadomienia SNMP. Poprawka jest dostępna w Zimbra 10.1.20.

**Znaczenie:** To podatność na styku publicznego SMTP i warstwy monitoringu, więc samo ograniczenie panelu administracyjnego nie zamyka ścieżki ataku. Administratorzy powinni pilnie zinwentaryzować instalacje `zimbra-snmp`, zaktualizować systemy i przejrzeć logi SMTP, procesy potomne oraz nietypowe polecenia uruchamiane przez konto Zimbra.

**Źródła:** [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search_api_fulltext=CVE-2026-73570), [Zimbra Security Advisories](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories), [NVD](https://nvd.nist.gov/vuln/detail/CVE-2026-73570)

### Microsoft skorygował informację o eksploatacji CVE-2026-69836 w Entra ID

CVE-2026-69836 to podatność deserializacji niezaufanych danych w hostowanej usłudze Entra ID, oceniona na CVSS 10.0 i umożliwiająca nieuwierzytelnione zdalne wykonanie kodu. Microsoft usunął błąd po stronie usługi i nie wymaga działań naprawczych od klientów; po początkowym oznaczeniu „Exploited: Yes” producent skorygował status na brak potwierdzonej eksploatacji.

**Znaczenie:** Zmiana statusu jest istotna operacyjnie: zespoły SOC powinny skorygować wewnętrzne alerty i raporty ryzyka, ale zachować przegląd zdarzeń tożsamości z okresu ekspozycji. Przypadek pokazuje też ograniczenie modelu SaaS — klient nie może sam załatać warstwy sterującej i musi polegać na telemetrii oraz komunikacji dostawcy.

**Źródła:** [Microsoft MSRC](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-69836), [CVE Record](https://www.cve.org/CVERecord?id=CVE-2026-69836), [Cybersecurity Dive](https://www.cybersecuritydive.com/news/microsoft-maximum-severity-flaw-entra-id-exploitation/828501/)
