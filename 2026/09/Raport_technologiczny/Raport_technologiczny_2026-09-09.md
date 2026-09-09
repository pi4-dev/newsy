# Raport technologiczny — 2026-09-09

**Zakres:** 2026-09-03–2026-09-09 (Europe/Warsaw)

## Biznes

Brak nowych, wiarygodnych zdarzeń biznesowych spełniających próg istotności.

## Technologia

### N-able N-central: krytyczna podatność pre-auth RCE CVE-2026-86218

N-able opublikował 5 września poprawkę Hot Fix 4 dla N-central 2026.3. Podatność CVE-2026-86218 umożliwia nieuwierzytelnionemu napastnikowi z dostępem sieciowym zdalne wykonanie kodu; producent nie potwierdził aktywnej eksploatacji w chwili publikacji. Instancje hosted zostały zaktualizowane przez N-able, natomiast instalacje on-premises wymagają przejścia do wersji 2026.3.1.14; aktualizacja nie wymaga nowej wersji agenta.

**Znaczenie:** N-central jest uprzywilejowanym systemem zarządzania flotą urządzeń, więc kompromitacja serwera może przełożyć się na szeroki dostęp administracyjny do środowisk klientów. Priorytetem jest ograniczenie ekspozycji interfejsu, natychmiastowe wdrożenie HF4 i przegląd logów serwera oraz aktywności kont uprzywilejowanych.

**Źródła:** [N-able — N-central 2026.3 HF4 Release Notes](https://documentation.n-able.com/N-central/Release_Notes/GA/Content/N-central_2026.3_HF4_Release_Notes.htm), [NVD — CVE-2026-86218](https://nvd.nist.gov/vuln/detail/CVE-2026-86218)