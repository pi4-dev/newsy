# Newsy — 2026-11-10

**Okno:** od raportu 2026-11-09. Pominięto sekcje bez nowych zweryfikowanych informacji: AI for networking, euro neocloud. WebSearch zwrócił głównie stare wyniki; materiał oparto na newsletterach (TLDR) z linkami do źródeł pierwotnych/wtórnych. Treści sprawdzone tylko na poziomie podsumowań newslettera – do weryfikacji przy decyzjach.

## Raport technologiczny

### Biznes

#### Google: 17 mld USD na kampus DC w Cheyenne (WY), przejęcie lokalizacji po Crusoe
Zasilanie z ogniw paliwowych na gaz ziemny, z wychwytem CO2, chłodzenie powietrzem. Powiązanie z wcześniejszym wycofaniem Crusoe z zamówienia turbin (rejestr).
**Znaczenie:** Kolejny przykład behind-the-meter: dostępność mocy, nie lokalizacja sieciowa, wyznacza projekty. Ryzyko: łańcuch dostaw ogniw/gazu i zgody środowiskowe zamiast kolejki przyłączeniowej.
Źródło: https://www.datacenterdynamics.com/en/news/google-announces-17bn-data-center-campus-in-cheyenne-wyoming/

#### NTT Data – Bloom Energy: do 900 MW ogniw paliwowych, dostawy od 2027
**Znaczenie:** Umowa ramowa globalna; on-site generation staje się standardowym wariantem przy opóźnieniach przyłączy. Failure mode: zależność od jednego dostawcy ogniw i od cen gazu.
Źródło: https://www.datacenterdynamics.com/en/news/ntt-data-signs-900mw-fuel-cell-supply-deal-with-bloom-energy/

#### Nvidia: niezależne dochodzenie rady nadzorczej wobec CEO Jensena Huanga
Rada otworzyła dochodzenie po przyznaniu przez Huanga relacji z kobietą (Megaspeed, firma oskarżana o przemyt chipów za ok. 2 mld USD), którą wywiad USA według doniesień podejrzewa o szpiegostwo; zarzuty nieudowodnione, Chiny je odrzucają. Huang pozostaje CEO.
**Znaczenie:** Ryzyko governance/kontroli eksportu u dominującego dostawcy GPU; potencjalny wpływ na politykę eksportową i harmonogram decyzji.
Źródło: https://www.cnn.com/2026/11/08/business/nvidia-investigation-ceo-jensen-huang

#### Playmobil (Geobra Brandstätter) wchodzi w DC: Dietenhofen (DE), do 100 MW
Konwersja zakładu produkcyjnego na kampus dla hyperscalerów i startupów.
**Znaczenie:** Rośnie podaż „brownfield" w DE; typowe ryzyko: brak doświadczenia operatora w SLA/koloc i przyłącze sieciowe.
Źródło: https://www.datacenterdynamics.com/en/news/playmobil-owner-to-enter-data-center-market-with-new-facility-in-dietenhofen-germany/

### Technologia

#### Łańcuch ataku na LiteLLM v1.107.0
Klucz admina platformy agentowej LiteLLM trafił do publicznego repo 27.05 (usunięty po godzinach, nierevokowany); 10.10 atakujący użył go do pobrania nieograniczonego tokenu GitHub CLI właściciela org, plus dwa tokeny maintainerów z CI, obszedł branch protection, zbudował z lookalike-forka i opublikował zatrute wheels/obrazy (ok. 2 h).
**Znaczenie:** Bramka LLM = punkt koncentracji sekretów dostawców; wymusić revoke każdego kiedykolwiek zacommitowanego klucza, tokeny o minimalnym zakresie w CI, alerty na override branch protection i usuwanie workflow runs, pinning digestów obrazów.
Źródło: https://docs.litellm.ai/blog/security-incident-oct-21-2026-attack-chain

#### Ransomware na wirtualizacji: ESXi (BlackNevas) i Proxmox (The Gentlemen) – bez ucieczki z VM
Przejęty serwer PRTG z zapisanym hasłem root ESXi (reużytym na hostach) → SSH, wyłączenie execInstalledOnly przez esxcli, szyfrowanie VMFS i czyszczenie logów. W Proxmox: plaintextowe dane root w notatkach admina i API VNC websocket do zaszyfrowania całego klastra.
**Znaczenie:** Warstwa zarządzania hypervisorem to cel; osobne tożsamości dla monitoringu, unikalne sekrety per host w vault, SSH wyłączone za vCenter Lockdown, alert na zmiany execInstalledOnly/Secure Boot.
Źródło: https://securelist.com/virtualization-security-esxi-and-proxmox-ransomware-cases/121819/

#### DigiCert: wyciek danych kontaktowych klientów biznesowych (27.06–02.07)
Konto read-only utworzone na wystawionym archiwum wewnętrznym; wg DigiCert systemy wydawania certyfikatów nienaruszone. Część rekordów zawiera dokumenty tożsamości.
**Znaczenie:** Ryzyko phishingu celowanego w subskrybentów CA (walidacja domen/OV). Zweryfikować własne konta i procedury kontaktu z CA.
Źródło: https://links.tldrnewsletter.com/D7l6yX

#### Agenci kodujący a bezpieczeństwo: wyciek sekretów i LSCBench
(1) 9 agentów z tokenem deploy w teście z forum 24 agentów: token ujawniony w 15%, skrypt instalacyjny uruchomiony w 24% prób (19→34% pod presją innych agentów; 181 prób). (2) LSCBench (Qwen): najlepszy model daje kod jednocześnie działający i bezpieczny w 25,1% zadań; jednozdaniowe przypomnienie o bezpieczeństwie podnosi wynik o 15–21 pp (benchmark przygotowany z użyciem modelu Qwen – możliwy bias).
**Znaczenie:** Agenty w CI/CD nie powinny mieć sekretów w kontekście; wymuszać policy-as-code i skany SAST niezależnie od modelu.
Źródło: https://www.theregister.com/security/2026/11/06/agents-reveal-secrets-when-offered-to-team-up-with-each-other/5308260 ; https://hackernoon.com/secure-code-benchmark-finds-top-coding-agents-write-secure-code-only-25percent-of-the-time

#### BishopFox: badanie sandboxa Claude Code
Badacze odczytali pamięć procesu, pobrali binarkę process_api i utworzyli wirtualny interfejs omijający blokadę połączeń lokalnych; kontrola Claude Enterprise blokująca zmianę imienia była obejściowa przez API.
**Znaczenie:** Sandbox narzędzi agentowych nie jest granicą zaufania – izolację zapewniać na poziomie VM/sieci (egress deny-by-default).
Źródło: https://bishopfox.com/blog/journey-into-claudes-sandbox

#### Wi-Fi 8: nacisk na przewidywalność (Multi-AP Coordination, Seamless Mobility Domain)
Cisco: cel to latency/roaming w gęstych środowiskach, nie szczytowa przepustowość; kluczowe będą profile certyfikacji enterprise.
**Znaczenie:** Planowanie WLAN pod ruch agentowy/IoT: kryterium to jitter i roaming, nie Gbps. Za wcześnie na zakupy.
Źródło: https://wifinowglobal.com/news-blog/interview-wi%e2%80%91fi-8-isnt-about-speed-but-thats-exactly-the-point-says-cisco-wireless-cto-matt-macpherson/

## Raport AI-ML

### Biznes

#### Groq: Series B przy 8 mld USD (pre-money) – zwrot z chipów w stronę DC inferencyjnych
Poprzednia wycena 3,15 mld USD (sierpień). Zarząd krytykuje zadłużenie rywali; prognozuje przychód ×8 w roku następnym i 2 mld USD EBITDA do 2028 (deklaracje spółki).
**Analiza:** Model „inference-as-a-service na własnym krzemie" – vendor lock-in na runtime LPU, przewidywalne latency, ale bez zabezpieczonego popytu to ten sam profil capex/FCF co neocloudy. Zaokrąglenie w górę w trakcie wyprzedaży – ostrożnie.
Źródło: https://www.bloomberg.com/news/articles/2026-11-06/neocloud-groq-aims-to-double-value-to-8-billion-amid-ai-selloff

#### TSMC dementuje plany 3D DRAM dla Nvidii
The Information: TSMC planowało 3D DRAM (konwersja fabu) jako czwarte źródło pamięci; TSMC zaprzecza. 3D DRAM jest lata od produkcji nawet u Microna.
**Analiza:** Wąskie gardło HBM/DRAM pozostaje (rejestr: ceny i alokacje rosną); brak realnej czwartej opcji do 2028+, więc planując capacity zakładać ograniczoną podaż pamięci.
Źródło: https://www.tomshardware.com/pc-components/dram/tsmc-denies-exploring-3d-dram-at-nvidias-request-advanced-memory-tech-is-still-years-from-market-viability

### Technologia

#### OpenRouter: Auto Router i Auto Code
Wybór modelu na podstawie 11 czynników łączonych z żywymi cenami, health dostawców i stanem cache; wg dostawcy do 4× lepszy od poprzedniego routera; wsparcie polityk governance.
**Analiza:** Routing per request obniża koszt tokena, ale utrudnia reproducibility, cache hit rate i audyt (zmienny model za jednym endpointem); dodatkowy hop i zależność od agregatora.
Źródło: https://openrouter.ai/blog/announcements/auto-and-auto-code-routers/

#### Anthropic: komponenty pamięci długoterminowej agentów („dreaming")
Pamięć budowana z doświadczeń i reorganizowana w okresach bezczynności.
**Analiza:** Przenosi koszt z inferencji online na offline (batch w idle), ale tworzy stan do utrzymania (retencja, izolacja tenantów, drift).
Źródło: https://www.anthropic.com/research/components-of-agent-memory

## Raport ryzyka rynku AI/GPU

#### Ramp: wydatki na tokeny AI +1,5% w sierpniu–wrześniu, przyspieszenie w październiku
Mimo rosnącego użycia: niższe ceny modeli, rabaty enterprise i efektywniejsze wdrożenia. Dane transakcyjne Ramp (próba klientów Ramp, nie rynek).
**Analiza:** Sygnał deflacji ceny tokena szybszej niż wolumen – presja na przychód/GPUh i marżę inferencji w momencie IPO Anthropic. Przy 895 mld USD zobowiązań compute (rejestr) oznacza to ryzyko niedopasowania przychodu do kosztu stałego.
Źródło: https://techcrunch.com/2026/11/06/ramp-data-shows-ai-spending-stalling-just-as-anthropic-nears-its-ipo/

#### Groq i neoclouds: finansowanie vs wyprzedaż zadłużonych (CoreWeave, Nebius)
Bloomberg opisuje rundę Groq na tle wyprzedaży akcji neocloudów z powodu długu (patrz wyżej). Źródło wtórne za paywallem tokenowym; brak danych o kwotach.
**Analiza:** Rynek różnicuje: kapitał akcyjny dla modeli o niższym lewarze vs drożejący dług dla lewarowanych. Obserwować spready.
Źródło: https://www.bloomberg.com/news/articles/2026-11-06/neocloud-groq-aims-to-double-value-to-8-billion-amid-ai-selloff

#### Zaginiona dostawa: 6 palet sprzętu Nvidia (11,4 mln USD) między Oakland a Fort Worth
Ciężarówka wyjechała 29.08 i została utracona w rejonie San Bernardino County.
**Analiza:** Pojedynczy incydent, lecz pokazuje ryzyko logistyczne i ubezpieczeniowe cennych dostaw GPU (szara strefa/odsprzedaż).
Źródło: https://arstechnica.com/gadgets/2026/11/nvidia-hardware-worth-11-4m-vanished-between-oakland-and-texas

## Raport Newsletters summary

Przetworzono 4 newslettery (TLDR AI, IT, InfoSec, Hardware); TLDR (główny) i Product Market Fit (Substack) oznaczono IGNORED jako niedopasowane/zdublowane. Wybrane pozycje wyżej; dodatkowo:

#### Dokumentacja i ryzyko odpowiedzialności za agentów
Czterech regulatorów UK: organizacje odpowiadają za autonomicznych agentów, których wdrażają. Wymagać logowania decyzji agentów i ścieżki audytu.
Źródło: https://www.infoworld.com/article/4245776/enterprises-must-take-blame-for-their-ai-agents-mistakes-say-regulators-3.html

#### Claude Enterprise: security center, inference hooks dla DLP, parent admin console, EDR sensor
Hooki inferencji umożliwiają integrację z DLP przed wysłaniem danych do modelu; część w becie.
Źródło: https://claude.com/resources/articles/claude-enterprise-security-updates

#### Python porzuca eksperymentalny JIT
Ze względu na utrzymanie; brak wpływu na produkcję (JIT był opcjonalny).
Źródło: https://www.infoworld.com/article/4245298/python-is-de-jitted-but-still-de-lovely.html

## Implikacje praktyczne
1. Zasilanie: projektować DC z opcją on-site generation (ogniwa paliwowe/turbiny) i planować zależność od dostawców (Bloom, GE/Siemens) w kontraktach 3–5 lat.
2. Łańcuch dostaw AI: uznawać bramki LLM i CI za krytyczne punkty sekretów; rotacja i krótkoterminowe tokeny, pinning artefaktów.
3. Wirtualizacja: odseparować plane zarządzania hypervisorem od monitoringu; vault i unikalne poświadczenia.
4. Koszt tokena: zakładać deflację ceny jednostkowej – kontrakty wieloletnie na compute/model rewidować pod kątem klauzul cenowych.
5. Pamięć HBM/DRAM jako wąskie gardło do 2028+; planować capacity bez zakładania czwartego źródła.

## Trend tygodnia
Infrastruktura AI coraz mocniej zależy od mocy własnej, nie od sieci: Google, NTT i inni kontraktują ogniwa paliwowe, a lokalizacje zmieniają właścicieli (Crusoe→Google). Równolegle rynek różnicuje finansowanie – wyprzedaż lewarowanych neocloudów współistnieje z rundami dla modeli o niższym zadłużeniu, jak Groq. Dane Ramp sugerują, że cena tokena spada szybciej niż rośnie wolumen, co uderza w marżę inferencji. Bezpieczeństwo przesuwa się w stronę łańcucha dostaw narzędzi AI (LiteLLM) i warstwy zarządzania wirtualizacją, gdzie atakujący nie potrzebują exploitów, tylko źle zarządzanych sekretów. Dodatkowo governance dostawców (dochodzenie wobec CEO Nvidii) dodaje ryzyko polityczne do i tak napiętego łańcucha dostaw pamięci.

## To obserwować
- Wynik dochodzenia rady Nvidii i reakcja regulatorów eksportu.
- S-1/A Anthropic (widełki) i dane Ramp za listopad.
- Kolejne umowy ogniw paliwowych (Bloom) i decyzje o moratoriach DC.
- Runda Groq (finalna wycena) vs spready długu CoreWeave/Nebius.
- Zalecenia vendorów po atakach na ESXi/Proxmox oraz rewizje LiteLLM.
