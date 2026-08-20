# Raport technologiczny — 2026-08-20

**Data podsumowania:** 2026-08-20  
**Okres analizy:** 2026-08-19 – 2026-08-20 (Europe/Warsaw)

## Biznes

Brak istotnych nowych informacji w analizowanym okresie.

## Technologia

### Cisco Crosswork: krytyczny hardening CVSS 10.0 bez obejścia

19 sierpnia 2026 r. Cisco opublikowało pakiet naprawczy obejmujący CVE-2026-20030, CVE-2026-20357, CVE-2026-20358 i CVE-2026-20359. Podatne są Crosswork Data Gateway, Crosswork Network Controller i Crosswork Planning niezależnie od konfiguracji; Cisco nie zna przypadków aktywnego wykorzystania, ale nie udostępniło obejścia. Wersje 7.2.1 i starsze należy podnieść do 7.2.1-SP.

**Znaczenie:** Crosswork może posiadać szeroki dostęp do telemetrii, poświadczeń i funkcji kontrolnych sieci, dlatego podatności obejmujące brak uwierzytelniania, SQL injection, kontrolę systemu plików oraz niedostatecznie chronione poświadczenia tworzą ryzyko przejęcia płaszczyzny zarządzania. Aktualizację należy potraktować jako pilną zmianę bezpieczeństwa, poprzedzoną kopią konfiguracji i testem integracji z gatewayami oraz kolektorami.

**Źródło:** [Cisco Crosswork Security Hardening Release: August 2026](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-crosswork-UzDTU9Vh)

---

### Cisco Secure Workload: krytyczne błędy uwierzytelniania i kontroli dostępu

Cisco opublikowało równolegle pięć CVE dla Secure Workload, z maksymalnym CVSS 10.0; podatne są wdrożenia SaaS i on-premises niezależnie od konfiguracji. Brak obejścia, a pełna naprawa wymaga wersji 3.10.9.1 lub 4.0.4.16 oraz aktualizacji komponentów Cluster, Agent i Connector; dla SaaS Cisco zaktualizowało Cluster, lecz klient nadal musi zaktualizować Agent i Connector. Cisco nie zna przypadków aktywnego wykorzystania.

**Znaczenie:** niepełna aktualizacja pozostawia niespójny poziom ochrony między kontrolerem a punktami egzekwowania mikrosegmentacji. Należy zinwentaryzować wersje wszystkich agentów i connectorów, monitorować odrzucenia polityk po zmianie oraz utrzymać możliwość wycofania aktualizacji bez utraty segmentacji.

**Źródło:** [Cisco Secure Workload Software Security Hardening Release: August 2026](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-csw1-shSvndWP)
