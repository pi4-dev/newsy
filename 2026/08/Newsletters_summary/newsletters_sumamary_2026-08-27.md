# Newsletters summary — 2026-08-27

**Data podsumowania:** 2026-08-27  
**Źródło:** nieprzeczytane wiadomości w Gmail Inbox oznaczone etykietą `NEWSY`

## d-Matrix Raptor: compute bezpośrednio na 3D DRAM dla fazy decode

**Technologia / Zdarzenie:** [d-Matrix Raptor 3D-DRAM Accelerator at Hot Chips 2026](https://www.servethehome.com/d-matrix-raptor-3d-dram-accelerator-for-generative-inference-at-hot-chips-2026/)

**Mechanizm działania:** Raptor umieszcza logikę TSMC N4 bezpośrednio na 3D DRAM. Stream blocking usuwa około 33% overfetch wynikający z niedopasowania banków do 128-bajtowych flitów, pinless data-bus inversion ogranicza przełączenia, a thermal-aware refresh, ECC i bank chaining mają utrzymać przepustowość blisko 100 TB/s mimo temperatury i wad banków.

**Wpływ na architekturę:** Układ celuje w memory-bound decode, gdzie wysoka lokalna przepustowość może zmniejszyć inter-token latency i zużycie energii względem transferu przez HBM/interposer. 72-kartowy rack ma mieścić model klasy 3T z kontekstem 1M, co wymaga nowego scale-up fabric, schedulerów rozdzielających prefill/decode i telemetryki obejmującej temperaturę, refresh, korekcje ECC oraz efektywną przepustowość banków.

**Failure modes i edge cases:** To nadal zestaw deklaracji z prezentacji, bez niezależnego pomiaru działającego systemu. Krytyczne ryzyka to hotspoty w DRAM, spadek retencji przy wysokiej temperaturze, yield bonded dies, awarie banków powodujące throttling kanału oraz nierównowaga między pojemnością i przepustowością; fallback powinien utrzymywać ścieżkę decode na GPU/HBM i routing oparty na realnym SLO.

---

## Samsung zHBM: pionowe połączenie XPU i DRAM bez interposera

**Technologia / Zdarzenie:** [Samsung Evolving HBM Base Die at Hot Chips 2026](https://www.servethehome.com/samsung-evolving-hbm-base-die-at-hot-chips-2026/)

**Mechanizm działania:** Roadmapa prowadzi od aktywnego base die z memory controllerem i RAS, przez processing elements w cHBM/aHBM, do zHBM łączącego XPU i stos DRAM za pomocą wafer-on-wafer oraz hybrid copper bonding. Usunięcie klasycznego HBM PHY/SERDES i skrócenie ścieżki danych ma zmniejszyć energię I/O; w przykładzie cztery stosy przy GPU 1200 W oszczędzają około 100 W.

**Wpływ na architekturę:** Pamięć przestaje być wymiennym komponentem, a staje się częścią współprojektowanego akceleratora. Zwiększa to potencjalny throughput/W i telemetrię RAS blisko danych, lecz wzmacnia zależność od konkretnego procesu, producenta pamięci, pakietowania i wspólnego cyklu kwalifikacji XPU–DRAM.

**Failure modes i edge cases:** Pionowy stack zwiększa gęstość cieplną, utrudnia testowanie i naprawę oraz może obniżyć yield całego pakietu. Wymagane będą telemetryka per stack, fault containment, spare capacity, degradacja wydajności zamiast całkowitej utraty pakietu i plan awaryjny oparty na bardziej standardowym HBM.

---

## Keycloak CVE-2026-18963: przejęcie konta przez obejście resetu hasła

**Technologia / Zdarzenie:** [Red Hat — CVE-2026-18963](https://access.redhat.com/security/cve/cve-2026-18963)

**Mechanizm działania:** Spreparowane żądanie do `reset-credentials` może przełączyć sesję uwierzytelniania bezpośrednio do fazy ustawienia nowego hasła, z pominięciem action tokenu wysłanego e-mailem. Atak nie wymaga wcześniejszego uwierzytelnienia i może objąć również konta administracyjne.

**Wpływ na architekturę:** Kompromitacja centralnego IdP ma większy blast radius niż pojedynczej aplikacji: dotyczy SSO, tokenów, klientów OIDC/SAML i kont uprzywilejowanych. Oprócz aktualizacji należy skorelować nietypowe wywołania resetu z logami sesji, zmianami credentials, wydaniem tokenów i aktywnością administratorów.

**Failure modes i edge cases:** Samo wyłączenie resetu hasła ogranicza atak, ale tworzy operacyjny DoS dla legalnych użytkowników; sama rotacja haseł nie unieważnia wszystkich aktywnych tokenów i sesji. Fallback powinien obejmować wyłączenie „Forgot password” do czasu patcha, wymuszone unieważnienie sesji po podejrzanym resecie i niezależny kanał odzyskiwania kont uprzywilejowanych.

---

## Supabase MCP: centralne zarządzanie dostępem przez IdP

**Technologia / Zdarzenie:** [Supabase — enterprise-managed auth for MCP](https://supabase.com/blog/enterprise-managed-auth-for-the-supabase-mcp-server)

**Mechanizm działania:** Administrator autoryzuje integrację Supabase w Claude raz, a użytkownik działa przez MCP z własną rolą i dotychczasowymi uprawnieniami projektowymi. Dostęp można ograniczać grupą Okta, a offboarding w IdP odbiera również dostęp z Claude; SCIM pozostaje na roadmapie.

**Wpływ na architekturę:** To właściwy kierunek dla agentów korzystających z narzędzi: tożsamość pozostaje indywidualna, a MCP nie otrzymuje wspólnego konta właściciela organizacji. Audyt powinien korelować użytkownika, sesję Claude, wywołanie MCP, projekt Supabase i wykonane zapytanie lub zmianę administracyjną.

**Failure modes i edge cases:** Błędna mapa grup lub nadmiernie szeroka rola w Supabase nadal daje agentowi zbyt duży zakres, a opóźniona propagacja revocation może pozostawić aktywną sesję. Potrzebne są krótkotrwałe tokeny, deny-by-default, reautoryzacja dla operacji destrukcyjnych i osobny approval dla zmian o wysokim blast radius.

---

## Sygnalizacja SS7/Diameter: behawioralne wykrywanie komercyjnego nadzoru

**Technologia / Zdarzenie:** [Citizen Lab — Bad Connection](https://citizenlab.ca/research/uncovering-global-telecom-exploitation-by-covert-surveillance-actors/)

**Mechanizm działania:** Badacze skorelowali firewall telemetry, PCAP, IR.21, DNS i BGP, a aktorów klastrowali po sekwencyjnych transaction IDs, niestandardowych Session-Id, powtarzalnych parametrach i rozbieżnościach między deklarowanym a obserwowanym routingiem. Kampanie przełączały się między SS7 i Diameter, spoofowały operator identities i używały ukrytych poleceń SIM w SMS.

**Wpływ na architekturę:** Sygnalizacyjny SOC potrzebuje analityki ścieżki i zachowania, nie tylko statycznych allow-list operatorów. Telemetria powinna zachowywać GT/OPC, Origin-Host/Realm, Route-Record, IMSI-targeting pattern i korelację z IR.21, ASN/BGP oraz IPX providerem.

**Failure modes i edge cases:** Legalny roaming i pośrednicy generują anomalie podobne do ataku, a spoofing zaufanych identyfikatorów osłabia klasyfikację źródła. Model detekcyjny musi mieć fallback do reguł protokołowych i ręcznej walidacji, a automatyczne blokowanie powinno uwzględniać ryzyko odcięcia roamingu i usług alarmowych.

---

**Przetworzone wiadomości:**
- TLDR Hardware — „OpenAI claims its chip beats Nvidia…, d-Matrix introduces Raptor…”, 2026-08-26.
- TLDR InfoSec — „Critical Keycloak ATO…, Global Telecom Exploitation…”, 2026-08-26.
- TLDR IT — „OpenAI’s Jalapeño chip…, GPT-5.6 hits AWS GovCloud…”, 2026-08-26.

Nie powtórzono Jalapeño, ponieważ temat został zapisany w raporcie z 2026-08-26.


---

## Vercel Connect: krótkotrwałe poświadczenia dla agentów zamiast stałych sekretów

**Technologia / Zdarzenie:** [Vercel Connect](https://vercel.com/kb/vercel-connect)

**Mechanizm działania:** Vercel Connect centralizuje połączenia agentów z zewnętrznymi usługami i wydaje poświadczenia w runtime zamiast przekazywać stałe tokeny przez kod, zmienne środowiskowe lub prompt. Dla integracji OAuth token może być związany z konkretnym użytkownikiem, a wywołanie runtime jest uwierzytelniane przez Vercel OIDC; model może więc żądać dostępu do narzędzia bez bezpośredniego posiadania długowiecznego sekretu.

**Wpływ na architekturę:** Wzorzec rozdziela execution plane agenta od credential plane i upraszcza rotację, offboarding oraz audyt dostępu. Dla środowisk produkcyjnych warto logować subject, connector, zakres tokenu, identyfikator sesji/agenta i wykonane operacje oraz wymuszać osobne polityki dla read-only i zmian destrukcyjnych.

**Failure modes i edge cases:** Krótkie TTL nie pomaga, jeśli broker poświadczeń pozwala agentowi pozyskać zbyt szeroki zakres albo jeśli prompt injection może wymusić użycie legalnego connectora do nielegalnej operacji. Deterministyczny fallback to deny-by-default, minimalny scope per task, allow-list connectorów, approval dla operacji o wysokim blast radius, natychmiastowa revocation oraz brak ekspozycji tokenu do kontekstu modelu.

**Źródło newslettera:** TLDR AI — „OpenAI Jalapeño 🌶️, Perplexity Portable Computer 💻, Claude combines memory 🧠”, 2026-08-26. Pozostałe tematy z tej wiadomości pominięto jako duplikaty albo materiały o niższej wartości infrastrukturalnej.
