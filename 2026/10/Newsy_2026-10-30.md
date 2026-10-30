# Newsy — 2026-10-30

**Okno przyrostowe:** od raportu 2026-10-29 do 2026-10-30. Pominięte sekcje: euro neocloud, AI for networking – brak nowych wiarygodnych pozycji (wyszukiwarka nie zwróciła świeżych źródeł pierwotnych; Acceleron ujęto 29.10). Ograniczenia: większość pozycji pochodzi z newsletterów TLDR (z linkami do źródeł pierwotnych), niezweryfikowana niezależnie.

## Raport technologiczny

### Technologia

#### UEFI/EDK2 PXE Base Code: CVE-2026-97631 (CVSS 9.5), 16-letni buffer overflow, RCE w ring 0 przed Secure Boot
Przepełnienie bufora w sterowniku PXE Base Code (TianoCore EDK2, bazowy kod firmware większości serwerów): zbyt duży komunikat ICMP error podczas network boot daje wykonanie kodu przed załadowaniem Secure Boot i łańcucha zaufania TPM. Poprawka wyciekła przez PR AMI na miesiące przed koordynowanym ujawnieniem w styczniu; eksploatacja wymaga dostępu do segmentu L2/L3 oraz strojenia adresów per model.
**Znaczenie:** Persistencja poniżej OS/hypervisora, niewidoczna dla EDR; w farmach bare-metal (provisioning PXE/iPXE, neocloudy) segment boot to powierzchnia ataku. Wymagać od vendorów firmware (Dell/HPE/Lenovo/Supermicro/AMI), wyłączać PXE tam, gdzie możliwe, izolować VLAN-y provisioningu, monitorować anomalie ICMP, planować okno masowej aktualizacji BIOS + atestację (measured boot).
Źródło: https://arstechnica.com/security/2026/10/code-execution-flaw-in-uefi-firmware-threatens-most-servers-on-planet-earth/

#### Skyw4lker: device code phishing na tokeny Microsoft 365 (produkcja, technologia, ochrona zdrowia w USA)
Ofiara dostaje „zabezpieczoną hasłem” umowę/ofertę przez Google Sites i jest kierowana do legalnego przepływu uwierzytelniania Microsoft; zatwierdzenie kodu urządzenia przekazuje atakującemu tokeny M365. Backend PHP odpytuje o ukończenie autoryzacji, loguje IP i geolokalizację.
**Znaczenie:** Atak omija MFA, bo użytkownik sam autoryzuje sesję. Mitigacja: Conditional Access blokujące device code flow dla użytkowników bez uzasadnionej potrzeby, token protection, alerty na sign-in z device code. Wiąże się z lukami Authenticator z poprzedniej edycji.
Źródło: https://thehackernews.com/2026/10/american-organizations-under-attack.html

#### NIST IR 8547 (final): harmonogram wycofania algorytmów klucza publicznego podatnych na kwanty
Finalna mapa przejścia na standardy PQC; zgodna z czerwcowym rozporządzeniem wykonawczym USA: wymiana key establishment w systemach federalnych o wysokiej wartości do 2030, podpisów do 2031.
**Znaczenie:** Twardy punkt odniesienia dla inwentaryzacji kryptografii (CBOM), kosztu handshake’ów hybrydowych w TLS/IPsec/SSH oraz wymagań wobec dostawców sprzętu sieciowego (HSM, VPN, PKI) w horyzoncie 3–5 lat.
Źródło: https://csrc.nist.gov/pubs/ir/8547/final

#### Kradzieże sprzętu DC w transporcie: >100 mln USD od czerwca, spoofing GPS i przejęte konta spedycyjne
Przejmowanie kont frachtowych przez złośliwe linki, licytowanie przewozów jako przejęta tożsamość, podstawianie kierowców; jeden ładunek ok. 10 mln USD zniknął po prawdopodobnym spoofingu współrzędnych trackera.
**Znaczenie:** Ryzyko łańcucha dostaw dla serwerów GPU (wartość jednostkowa rzędu setek tys. USD za węzeł): wymagać od integratorów weryfikacji przewoźnika, plomb z niezależną telemetrią, łączenia lokalizacji z danymi niepodatnymi na GPS spoofing, ubezpieczenia transportu. Dodatkowo ryzyko firmware’owe sprzętu o niepewnym chain of custody.
Źródło: https://arstechnica.com/tech-policy/2026/10/highway-thieves-are-spoofing-gps-to-steal-millions-in-data-center-bound-cargo/

### Biznes

#### Tesco planuje migrację brzegowej wirtualizacji z VMware na Proxmox
Oferta pracy opisuje przejście edge z VMware na Proxmox z powodu zmian licencyjnych Broadcom.
**Znaczenie:** Kolejny sygnał erozji bazy VMware w edge/branch (niskie wymagania, wysoki wolumen węzłów, gdzie koszt licencji per rdzeń dominuje); kosztem są narzędzia zarządzania flotą, DR i wsparcie vendorowe.
Źródło: https://www.itpro.com/software/open-source/tesco-eyes-open-source-vmware-alternative-amid-long-running-spat

## Raport AI-ML

### Biznes

#### OpenAI ma kupować TPU od Google (nie tylko wynajmować) — raport The Information
Poza dotychczasowym wynajmem, OpenAI miałoby nabywać TPU, równolegle do GPU NVIDIA i własnych układów z Broadcom. Informacja z drugiej ręki, niepotwierdzona przez strony.
**Analiza:** Dywersyfikacja poza CUDA przez największego konsumenta mocy; zwiększa wolumen TPU (spójne z TPU na OCI od 2027), ale wymaga portowania stosu serwującego pod XLA/JAX. Dla capacity planningu: rosnąca konkurencja o moce TPU i potencjalne wiązanie dostępności z kontraktami strategicznymi.
Źródło: https://links.tldrnewsletter.com/dcOmE4

#### NVIDIA i Oracle: Vera Rubin NVL72 w OCI Superclusters do skali multi-gigawatowej
Plan rozbudowy Superclusterów OCI o systemy Vera Rubin NVL72 i procesory Vera, do skali wielu GW.
**Analiza:** Zobowiązania mocy po stronie Oracle rosną równolegle do dywersyfikacji na TPU; kluczowe ryzyka to energia/PPAs, harmonogram oddawania DC i koszt finansowania (patrz Ryzyka). Brak podanych terminów i wolumenów w materiale newslettera.
Źródło: https://nvidianews.nvidia.com/news/nvidia-and-oracle-expand-nvidia-vera-rubin-to-multi-gigawatt-scale-and-bring-nvidia-vera-cpu-to-oracle-cloud-infrastructure

### Technologia

#### Decagon: samodoskonalący się stos inferencyjny (AutoResearch), do 2,5x przepustowości w limicie latency
Agenci diagnozują wąskie gardła, testują optymalizacje na replayu ruchu produkcyjnego: kwantyzacja, scheduling, patche silnika, speculative decoding; zmiany produkcyjne zatwierdza człowiek.
**Analiza:** Wartość leży w replayu ruchu i bramce SLO, nie w agencie; liczba 2,5x to deklaracja vendora/klienta bez niezależnej weryfikacji, zależna od profilu obciążenia. W kontekście wycieku KV z poprzedniej edycji zmiany silnika wymagają testów izolacji tenantów.
Źródło: https://links.tldrnewsletter.com/3vHagX

## Ryzyka rynku AI/GPU

### Wpisy

#### Koncentracja dostawców: prospekt IPO Anthropic – prawie połowa sprzedaży przez Amazon i Google
Prospekt pokazuje silną zależność dystrybucyjną od partnerów chmurowych; komentatorzy zalecają traktowanie zakupów AI jako kwestii odporności (przenośność danych i workflow).
**Analiza:** (POPYT) Kontrakty i retencja klientów modeli są pośrednio zależne od marż i polityk hyperscalerów; efekt cieniowania: wzrost zużycia u laboratoriów napędza capex chmur finansowany zewnętrznie (wpis Q3 z 29.10). Źródło wtórne.
Źródło: https://www.itpro.com/business/what-enterprises-can-learn-from-anthropics-ipo-prospectus-about-vendor-dependency

## Newsletters summary

### Wpisy

#### Microsoft Entra ID: wycofanie SMS/voice MFA w lutym 2027
Przypomnienie dla administratorów o wygaszeniu uwierzytelniania SMS i głosowego dostarczanego przez Microsoft; migracja na passkeys.
**Analiza:** Zaplanować inwentaryzację kont zależnych od SMS (konta serwisowe, break-glass, zewnętrzni), politykę rejestracji passkey i procedurę odzyskiwania; ryzyko lockoutów przy masowym wygaszeniu.
Źródło: https://links.tldrnewsletter.com/vmR5pX

#### „AI hacker” znajduje 4 luki w EPICS Base w <4 godziny (CVSS 9.8 integer overflow, RCE)
Cipher (Causal Security) znalazł cztery podatności w EPICS (sterowanie akceleratorami/fuzją); jedna krytyczna, trzy wysokie (crash, memory corruption, info disclosure); wymagany dostęp sieciowy do usług EPICS.
**Analiza:** Skraca czas od kodu do exploita także dla ICS/OT; segmentacja OT i patching stają się krytyczne. Anthropic opublikował równolegle badania nad odkrywaniem i exploitacją luk (przeglądarki, mobilne OS, stos sieciowy jądra) w trybie CVD.
Źródło: https://www.theregister.com/security/2026/10/27/ai-hacker-finds-4-severe-bugs-in-popular-industrial-control-software-in-under-4-hours/5305811

#### Anthropic: odkrywanie i exploitacja podatności w oprogramowaniu krytycznym
Model badawczy znalazł RCE, ucieczki z sandboxów i obejścia blokady ekranu, zgłoszone vendorom w CVD.
**Analiza:** Oczekiwać fali poprawek w komponentach sieciowych jądra i przeglądarkach; skrócić cykle patch management, bo okno między ujawnieniem a wykorzystaniem się kurczy.
Źródło: https://www.anthropic.com/research/discovering-vulnerabilities-with-claude

## Implikacje praktyczne
1. Firmware jako warstwa zaufania: budżetować cykliczne aktualizacje UEFI/BMC i atestację (measured boot) w całej flocie; PXE tylko w izolowanych VLAN-ach provisioningu.
2. Dywersyfikacja akceleratorów (GPU+TPU, JAX/XLA obok CUDA) jako koszt portowania w zamian za negocjacyjność – planować abstrakcję warstwy serwującej.
3. Tożsamość jako główny wektor: wyłączyć device code flow, wymusić passkeys/token protection do 2027.
4. Migracja PQC wg NIST IR 8547 (2030/2031): inwentaryzacja kryptografii w sprzęcie sieciowym i PKI już teraz.
5. Łańcuch dostaw sprzętu DC: ubezpieczenie, weryfikacja przewoźników i chain of custody; ryzyko firmware z nieautoryzowanych źródeł.

## Trend tygodnia
Rynek infrastruktury przesuwa się z pojedynczej zależności od NVIDIA w stronę wielu dostawców mocy: TPU na OCI od 2027, plany zakupu TPU przez OpenAI i równoczesna rozbudowa Vera Rubin w tej samej chmurze. Jednocześnie finansowanie budowy DC staje się zmienną krytyczną, bo nawet silni hyperscalerzy emitują dług i akcje, co przenosi ryzyko na cenę i warunki kontraktów. Po stronie bezpieczeństwa dominują dwa wektory: poniżej systemu (UEFI/PXE) oraz tożsamość (device code phishing, tokeny), przy czym AI skraca czas odkrywania luk, także w OT. Koncentracja dostawców (prospekt Anthropic, zależność od partnerów chmurowych) pokazuje, że odporność trzeba projektować na poziomie zakupowym i architektonicznym. Wreszcie koszty licencyjne (Broadcom/VMware) nadal wypychają edge ku open source, kosztem narzędzi zarządzania flotą.

## To obserwować
- Poprawki firmware dla CVE-2026-97631 od Dell/HPE/Lenovo/Supermicro i ewentualne wpisy w CISA KEV.
- Potwierdzenie (lub dementi) zakupu TPU przez OpenAI oraz terminy TPU na OCI.
- Refinansowanie 4,4 mld USD długu CoreWeave i rentowność obligacji 2032 (~15%).
- Warunki i harmonogram Vera Rubin na OCI (GW, PPAs).
- Wygaszanie SMS MFA w Entra (luty 2027) i adopcja passkeys.
