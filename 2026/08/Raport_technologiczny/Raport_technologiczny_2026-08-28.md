# Raport technologiczny — 2026-08-28

Data podsumowania: **2026-08-28**  
Zakres przyrostowy: od edycji z 2026-08-26 do 2026-08-28, Europe/Warsaw.

## Biznes

W analizowanym okresie nie pojawiła się odrębna, materialna informacja biznesowa spełniająca kryteria raportu.

## Technologia

### Ubiquiti usuwa 22 podatności UniFi, w tym trzy o CVSS 10.0

26 sierpnia Ubiquiti opublikowało Security Advisory Bulletin 067 obejmujący 22 podatności w ekosystemie UniFi. Najpoważniejsze problemy to nieuwierzytelnione command injection w UniFi Protect (CVE-2026-77537) i UniFi Talk (CVE-2026-77554) oraz obejście uwierzytelniania przez CRLF injection w UniFi OS (CVE-2026-77550); nie ma potwierdzenia aktywnego wykorzystania. Wymagane poziomy poprawek obejmują co najmniej Protect 7.2.105, Talk 5.3.2, UniFi OS Server 5.1.37 oraz odpowiednie wydania UniFi OS dla urządzeń.

**Znaczenie:** urządzenia UniFi często współdzielą płaszczyznę zarządzania z kamerami, telefonią, gatewayami i storage, więc pojedyncze przejęcie hosta może rozszerzyć domenę awarii na wiele funkcji infrastruktury. Należy traktować aktualizację jako pilną zmianę bezpieczeństwa, ograniczyć dostęp do interfejsów zarządzania do dedykowanych segmentów oraz monitorować nietypowe procesy potomne, żądania z sekwencjami CRLF i zmiany konfiguracji po aktualizacji.

**Źródła:** [Ubiquiti Security Advisory Bulletin 067](https://community.ui.com/releases/Security-Advisory-Bulletin-067/fc4a3488-7c43-4628-8bab-f715e96dbfc9), [CVE-2026-77550](https://www.cve.org/CVERecord?id=CVE-2026-77550), [CVE-2026-77554](https://www.cve.org/CVERecord?id=CVE-2026-77554)
