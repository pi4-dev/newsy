# Newsy — 2026-10-12

**Okno przyrostowe:** od raportu 2026-10-11 do 2026-10-12 07:28 CEST.
**Uwaga:** RSS niedostępne z sandboxa; wyszukiwanie WWW nie zwróciło nowych, datowanych pozycji dla AI-ML, euro neocloud i AI for networking — sekcje pominięte.

## Raport technologiczny

### Technologia

#### Cisco APIC: batch „hardening" — CVE-2026-76499 / -76500 (CVSS 9.8, unauth, network)
Opublikowane 07.10.2026 (zmiana 08.10). Wektor AV:N/AC:L/PR:N/UI:N — wynik wewnętrznego przeglądu bezpieczeństwa zespołu APIC; w tym samym batchu m.in. CVE-2026-76464 (9.6) oraz CVE-2026-76483 (Cisco License On-Prem, 9.1, CWE-522 niedostatecznie chronione poświadczenia). Brak publicznego PoC i wpisu w CISA KEV (stan wg agregatora). Wersje podatne/naprawcze nie są podane w agregatorze — tylko w biuletynie Cisco.
**Znaczenie:** APIC jest kontrolerem fabryki ACI — kompromitacja daje kontrolę nad politykami całej fabryki. Zweryfikować, czy interfejs APIC jest osiągalny spoza OOB/mgmt; ograniczyć ACL-ami do czasu patcha. Failure mode: upgrade klastra APIC wymaga zachowania kworum i zgodności wersji z leaf/spine; zaplanować okno, backup konfiguracji przed upgrade'em. Dane z agregatora — zweryfikować w advisory.
**Źródło:** https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-apic-UOXWtfh (wtórnie: https://www.thehackerwire.com/vulnerability/CVE-2026-76500/)

## Newsletters summary

#### The Business Engineer — „Cheap Tokens, Expensive Consequences" (2026-10-11)
**Zdarzenie:** spadek cen wywołań modeli: OpenAI Decisions API $0,10/mln tokenów wejściowych (bez opłaty za wyjście), Perplexity −50% do $0,02/mln, Claude Haiku 5.5 ok. 75% ceny Haiku 4.5. Teza autora: koszt przesuwa się do warstw niżej (wynajemcy GPU, pamięć, dostawcy chipów), jest niesiony w bilansach jako zobowiązania, gwarancje i dług, a odpowiedzialność za działania agentów spada na wdrażających. Dane z newslettera (wtórne), w odbiorze dostępny był tylko wstęp.
**Mechanizm:** cena tokenu spada szybciej niż koszt infrastruktury; marża utrzymywana przez finansowanie dłużne/zobowiązania zakupowe, nie przez przychód per token.
**Wpływ:** cenniki API nie są stabilną podstawą planowania kosztów na 3-5 lat; kontrakty z dostawcami powinny mieć zabezpieczenie ceny i przejrzystość zmian modeli. Ekspozycja kontrahentów neocloud/GPU na dług podnosi ryzyko ciągłości usług.
**Failure modes:** gwałtowna zmiana cen lub wycofanie taryfy, wzrost zużycia (efekt odbicia) niwelujący oszczędność, niejasna odpowiedzialność kontraktowa za akcje agentów.
**Źródło:** https://businessengineer.ai/p/this-week-in-the-map-of-ai-cheap

#### Pominięte (IGNORED)
- „How WisprFlow Built Its Paid Ads Program…" (Venture Street Playbooks) — marketing, poza profilem.
