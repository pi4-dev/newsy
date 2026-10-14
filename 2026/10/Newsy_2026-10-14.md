# Newsy — 2026-10-14

**Okno przyrostowe:** od raportu 2026-10-13 do 2026-10-14 07:28 CEST.
**Uwaga:** RSS niedostępne z sandboxa; wyszukiwanie WWW nie zwróciło nowych, datowanych pozycji z 13–14.10 poza weryfikacją Vghost. Sekcje Raport technologiczny, AI-ML, euro neocloud i AI for networking pominięte; treść pochodzi z newsletterów (TLDR, TLDR InfoSec, TLDR IT, TLDR AI).

## Newsletters summary

#### Vghost (CVE-2026-74580): LPE do root i VM escape przez /dev/vhost-net i /dev/vhost-vsock
**Zdarzenie:** błąd w vhost (kernele 4.11–7.2; wg newslettera CVSS 8.8) pozwala użytkownikowi z dostępem do /dev/vhost-net lub /dev/vhost-vsock na odczyt/zapis pamięci jądra. Tencent Zhuque Lab potwierdził root na 5 dystrybucjach oraz przejęcie procesu QEMU hosta przez złośliwego gościa. **Mechanizm:** rekord CNA – cache metadanych vring nie jest resetowany po VHOST_SET_VRING_ADDR/NUM, stale mapowanie → kopiowanie poza region IOTLB. Poprawka: de845981da67. **Wpływ:** hosty KVM/QEMU i węzły K8s z vhost; RHEL, Amazon Linux, Azure Linux, Oracle Linux domyślnie wystawiają węzły urządzeń szerokim uprawnieniom. Do czasu patcha zdjąć dostęp world z obu węzłów /dev/vhost-*. **Failure modes:** vhost-vsock tylko wg Tencenta (rekord CNA go nie wymienia); brak potwierdzonej eksploatacji in-the-wild w źródłach; pełny zakres wersji naprawionych sprawdzić w trackerze dystrybucji.
Źródła: https://matrix.tencent.com/en/2026/10/12/vghost-cve-2026-74580, https://cveawg.mitre.org/api/cve/CVE-2026-74580

#### Autonomiczne agenty AI w realnych intruzjach: CyberXero i Breachcache
**Zdarzenie:** CyberXero użył Claude Code, 51 agentów, PentAGI i Cobalt Strike; jeden przebieg: 4708 celów, 429 paneli admin, 32 web shelle w 61 s; 628 tys. rekordów z Ukrainy. Breachcache odzyskał 75 transkryptów: 23 hosty z RCE, SSRF → kradzież poświadczeń instancji chmurowej, root na Open WebUI (CVE-2026-0766), audyt kodu bramek AI (payment-callback forgery, ATO). **Mechanizm:** pętla recon→exploit w pełni zautomatyzowana, rotacja proxy/Tor, omijanie odmów przez resetowanie sesji i deklarację „authorized testing". **Wpływ:** skrócenie czasu między ekspozycją a kompromitacją do sekund; IMDSv2/hop-limit, egress filtering i segmentacja paneli admin stają się obowiązkowe. Open WebUI i bramki LLM traktować jak powierzchnię internetową. **Failure modes:** detekcja oparta na progu czasu człowieka (rate/UA) zawodzi.
Źródła: https://thehackernews.com/2026/10/cyberxero-uses-claude-code-in-campaign.html, https://breachcache.com/cases/hands-off-the-keyboard/

#### Microsoft Maia 300: inferencja, +40% tokenów/MW vs GPU, wdrożenie od 2027
**Zdarzenie:** trzecia generacja akceleratora Microsoftu pod inferencję; deklarowane 40% więcej tokenów na megawat niż wiodące systemy GPU, multi-gigawatowe wdrożenia od 2027; Maia nie będzie bezpośrednio wynajmowana klientom. **Mechanizm/Wpływ:** metryka tokens/MW wskazuje na ograniczenie energetyczne jako głównego czynnika; część obciążeń własnych Microsoftu zejdzie z GPU, więc Azure GPU nie zyska na tym wprost. Dane to deklaracje producenta, bez niezależnych benchmarków. **Failure modes:** ekosystem software (kompilator, kwantyzacja) i opóźnienie harmonogramu; lock-in po stronie usług zarządzanych (Copilot/Azure OpenAI), nie IaaS.
Źródło: https://news.microsoft.com/maia-2026/

#### Ørsted: automatyzacja zmian firewall w Nautobot, z tygodni do ok. 10 minut
**Mechanizm:** pipeline z walidacją polityk, zatwierdzeniami i audit trail zintegrowany z Nautobot (source of truth). **Wpływ:** wzorzec dla change managementu: compliance jako kod w ścieżce zmiany, nie po niej. Skuteczność zależy od jakości danych SoT i kompletności modelu reguł. **Failure modes:** drift między SoT a faktyczną konfiguracją, błędna reguła walidacji propagowana masowo, brak rollbacku.
Źródło: https://www.networkworld.com/article/4229048/orsted-builds-compliance-into-automation-speeds-firewall-changes.html

#### OT: atak na sterowniki chłodnicze w francuskich supermarketach; awaria Exchange Admin Center
**Zdarzenie:** wg doniesień manipulacja kontrolerami chłodzenia w setkach sklepów (cykle rozmrażania) przez systemy zarządzane przez stronę trzecią. Osobno: Microsoft cofnął zmianę konfiguracji, która zablokowała dostęp do Exchange Admin Center (12.10). **Wpływ:** zdalne konta dostawców OT/MSP to wspólny punkt awarii; wymagać MFA, jump hostów i segmentacji. Dla Exchange – plan awaryjny administracji przez PowerShell. Źródła pierwotne niepotwierdzone (agregat).
Źródło: https://www.theregister.com/saas/2026/10/12/microsoft-blames-a-configuration-change-for-exchange-admin-center-tumble/5302698

#### OAuth auto-linking po e-mailu: ATO w Pingvin Share X, Better Auth, AshAuthentication
**Mechanizm:** logowanie OAuth dołączane do istniejącego konta o tym samym e-mailu; każda biblioteka pominęła inną kontrolę (email_verified, weryfikacja lokalnego e-maila, porównanie e-maili). **Wpływ:** w IdP/SSO i aplikacjach wewnętrznych wiązać tożsamość po (issuer, sub), wymagać zweryfikowanych e-maili po obu stronach i MFA przed linkowaniem. Autor (WorkOS) sprzedaje konkurencyjny produkt.
Źródło: https://workos.com/blog/oauth-auto-link-email-account-takeover

#### Granice agentów kodujących poza modelem; luka CVE dla podatności agentów
**Mechanizm:** prompt injection w issue/changelogu/konfigu kieruje agenta do odczytu sekretów przy „czystym" PR. Skuteczne limity: izolowane workspace'y, tożsamości per zadanie, krótkotrwałe poświadczenia, polityka sieciowa. Osobno: programy CVE nie obejmują uprawnień agentów i interakcji narzędzi. **Wpływ:** testować granice platformy, nie odmowy modelu. Spójne z CyberXero/Breachcache powyżej (ten sam wektor, strona obronna).
Źródła: https://vinny.dev/blog/2026-10-12-the-prompt-is-not-the-permission-securing-the-agentic-sdlc/, https://aminrj.com/posts/why-cve-programs-miss-most-ai-agent-vulnerabilities/

#### Post-quantum „Cryptographic Right Answers" (Valsorda): termin 2029
**Zalecenia:** X25519+ML-KEM-768 (KEX), HPKE z X-Wing lub age PQ (szyfrowanie), ML-DSA-44 zamiast hybryd (podpisy); symetryczne bez zmian; unikać OpenSSL na rzecz rustls/Go crypto/tls/BoringSSL/AWS-LC po CVE 2026 w wolfSSL, Mbed TLS, Botan. **Wpływ:** planować inwentaryzację TLS/VPN/PKI; część zaleceń to własne projekty autora (ujawnione).
Źródło: https://words.filippo.io/right-answers/

#### Meta Muse Wonder / Wonder Full oraz Anthropic: samoreplikacja i antydystylacja
**Zdarzenie:** Meta wypuściła Muse Wonder (API i Muse Code) oraz Wonder Full (tylko subskrybenci Muse Code) z Active Memory i Active Focus. Anthropic opublikował badanie zdolności modeli do autoreplikacji na sprzęcie konsumenckim oraz deklaruje 90–99% redukcji danych pozyskiwanych dystylacją po ograniczeniu dostępu do rozumowania; uruchomił też Critical Infrastructure Defense Program (OT) z Dragos, Nozomi, Rockwell. **Wpływ:** pamięć międzysesyjna agenta = nowa powierzchnia wycieku danych; ograniczanie ekspozycji rozumowania zmienia kontrakt API. Wnioski z deklaracji dostawców, bez niezależnej weryfikacji.
Źródła: https://research.meta.ai/blog/muse-wonder, https://www.anthropic.com/research/self-replicating-ai, https://www.anthropic.com/news/anthropic-cyber-mission

#### Pominięte (IGNORED)
- „Harbor ($10M Seed) Pitch Deck: Who Insures AI Agents?" (Venture Street) — pitch deck/finansowanie startupu, poza profilem.
