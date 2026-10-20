# Newsy — 2026-10-20

**Okno przyrostowe:** od raportu 2026-10-19 do 2026-10-20 07:28 CEST.
**Uwaga:** WWW (neocloud, AI-networking) nie zwróciło nowych, datowanych pozycji z okna – sekcje Raport technologiczny, AI-ML, euro neocloud, AI for networking pominięte. Treść pochodzi z newsletterów (etykieta NEWSY). Pominięto pozycje już ujęte 19.10 (m.in. Meta Muse Spark Open, Gemini 4 Argon, dług CoreWeave/Anthropic).

## Newsletters summary

#### Snowflake-managed PostgreSQL: łańcuch CREATEROLE → RCE (Varonis)
**Zdarzenie:** rola `snowflake_admin` (nie-superuser) eskalowana do OS-level RCE przez pięć udokumentowanych funkcji PG. **Mechanizm:** CREATEROLE → self-grant `pg_write_all_data`, przejęcie joba pg_cron (wykonanie jako postgres), obejście blokad przez `LANGUAGE internal`, nadpisanie `archive_command`; ujawnione klucze mTLS tenanta i AWS STS. **Wpływ:** Snowflake naprawił po stronie serwera; w self-hosted PG traktować CREATEROLE jak superusera, blokować LANGUAGE internal/C, audytować pg_cron i SECURITY DEFINER. **Failure modes:** mitygacje oparte na wyłączaniu pojedynczych funkcji są nieskuteczne wobec łańcuchów.
Źródło: https://www.varonis.com/blog/remote-code-execution-in-snowflake-managed-postgresql

#### OpenBSD: 37-letni błąd 16-bitowego licznika shm → LPE do root (Calif)
**Mechanizm:** podpisany 16-bitowy licznik attach SysV shm przepełnia się po wielokrotnym forkowaniu (wartość ujemna) → UAF w wyścigu; drugi OOB write daje write-what-where. Exploit działa przy KARL, SMAP, SMEP. **Wpływ:** lokalna eskalacja na hostach OpenBSD (firewalle, bramy); patch z errata. **Failure modes:** mitygacje wykonawcze nie chronią przed logicznymi błędami licznika.
Źródło: https://calif.io/research/big-fish

#### OpenCode: wyciek kluczy API zewnętrznych dostawców (~1,4% użytkowników konsoli)
**Zdarzenie:** w ramach „distillation attack" na legacy system ujawniono klucze third-party; użytkownicy proszeni o rotację. **Wpływ:** klucze LLM w SaaS-owych konsolach agentów to ryzyko łańcucha dostaw; stosować klucze o ograniczonym scope, limity budżetowe, rotację. Powiązane: Any.Run – 1 258 incydentów z fałszywymi narzędziami AI (60% podszywanie pod Claude, 45% ClickFix) – celem klucze CI/CD, SSH, chmura.
Źródło: https://blockonomi.com/opencode-security-breach-exposes-api-keys-of-1-4-of-console-users/

#### Phishing przez zaproszenia kalendarza (.ics) przeżywa kwarantannę maila (Sublime / Check Point)
**Mechanizm:** zaproszenia auto-tworzą zdarzenia w Google Workspace / M365; filtry kwarantannują mail, ale nie usuwają zdarzeń. Check Point: 302 tys. unikalnych maili, 40 tys.+ organizacji. **Wpływ:** w playbookach IR dodać czyszczenie zdarzeń kalendarza i wyłączenie auto-add dla zewnętrznych nadawców.
Źródło: https://firethering.com/calendar-invite-phishing-attacks/

#### Heartbreaker: metamorficzny ransomware dla macOS (Objective-See)
**Mechanizm:** Rust, AES-256-GCM na ~/Downloads, LaunchAgent, przebudowa przez mutację kodu i Cargo (zmiana hasha). Jeden zaszyty klucz → darmowe odszyfrowanie. **Wpływ:** detekcja behawioralna (nieufny proces + szybki zapis plików o wysokiej entropii) działa, sygnatury hashy nie.
Źródło: https://objective-see.org/blog/blog_0x8A.html

#### Microsoft zakłóca 3 zestawy phishingowe (CoGUI, Haozi, Iron Man System)
**Zdarzenie:** >1 mld wiadomości od stycznia, 75 tys. organizacji, ~17 tys. domen, przejęte środowiska chmurowe MS/Google, WebSocket do kradzieży danych. **Wpływ:** takedown domen; infrastruktura PhaaS szybko się odtwarza.
Źródło: https://thehackernews.com/2026/10/microsoft-disrupts-3-phishing-kits.html

#### Datacenters 2027: +50 GW w jednym roku (SemiAnalysis)
**Zdarzenie:** >24 GW w zaawansowanej budowie na dostawy 2027 (40 kampusów); ~1 bln USD wiążących leasingów podpisanych YTD 2026 (TCV 30–50 mld USD/GW, 15–20 lat). **Mechanizm:** modularyzacja (prefab elektryka Google, AWS „Project Houdini", czas budowy <10 mies., DayOne ~9 mies. vs >18), power behind-the-meter, ekspansja poza USA. **Wpływ:** po stronie podaży DC zwiększa się; wąskie gardło może przejść na pamięć i logikę w 2027–28; presja na stopy. **Failure modes:** braki pracowników, opóźnienia dostawców, finansowanie.
Źródło: https://newsletter.semianalysis.com/p/datacenters-2027-the-year-of-acceleration

#### DeepSeek-V4.1-Flash-1017: KV cache 890 B/token
**Mechanizm:** cross-layer KV reuse w Compressed Sparse Attention 2 + FP4 KV; trwały KV ok. 1/8 poprzedniej wersji. **Wpływ:** większa współbieżność na GPU przy tym samym HBM, niższy koszt serwowania długiego kontekstu; ryzyko jakości przy FP4 KV na długich kontekstach. Dodatkowo API Search/Fetch (JSON, bearer).
Źródło: https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017

#### Intel i Qualcomm ofertują do 10 mld USD za Tenstorrent; CXMT zysk ~20 mld USD za 9 mies.
**Zdarzenie:** doniesienia o przejęciu Tenstorrent (Blackhole ~1 000 USD, natywne BlockFloat8); koszty to dane vendora, niezweryfikowane, transakcji brak. CXMT: zysk 131–132,5 mld CNY przy przychodzie 265–270 mld CNY, napędzany niedoborem DRAM. **Wpływ:** alternatywy dla GPU w inferencji; ceny DRAM zostaną wysoko do czasu nowej podaży, cykl pamięci zwykle się odwraca.
Źródło: https://theinference.org/article/intel-and-qualcomm-are-reported-in-talks-for-tenstorrent-worth-up-to-10-billion

#### Microsoft: >2 mld USD rocznie dla Anthropic w Copilot; ServiceNow AI Passport (Q4 2026)
**Zdarzenie:** Microsoft płaci za modele Claude w Copilot, router kieruje część do OpenAI. ServiceNow AI Passport kontroluje uprawnienia agentów w runtime (Nvidia partnerem); krytyka AI Control Tower: brak uniwersalnego kill switcha, egzekucja runtime po stronie narzędzi zewnętrznych. Replit zmienia domyślnego dostawcę na OpenAI. **Wpływ:** multi-model routing jako standard; unikać lock-inu w warstwie agent governance.
Źródło: https://cryptobriefing.com/microsoft-anthropic-2b-copilot-deal/

#### Akamai: migracja ~1 500 mikroserwisów z hyperscalera dla redukcji kosztów
**Wpływ:** sygnał egress/cost-driven repatriacji do alternatywnych chmur; w TCO uwzględnić koszty egress i migracji.
Źródło: https://www.globenewswire.com/news-release/2026/10/15/3381456/0/en/ai-enterprise-saas-workloads-drive-akamai-cloud-momentum.html

#### Pominięte (IGNORED)
Venture Street (rynki vertical AI) – poza profilem infrastrukturalnym. Z pozostałych: konsumenckie/naukowe/HR (Apple HomeView, kolejna dyskusja o matematyce, Tesla Roadster, NASA Roman).
