# Raport Zbiorczy IT – 2026-11-25

**Okno przyrostowe:** od raportu z 2026-11-24. Obejmuje zdarzenia z 23–25.11, których nie ma w rejestrze z ostatnich 7 dni. Fakty już opisane (CrowdStrike–Zscaler: warunki transakcji, powrót Huanga, reguła BIS 16/14 nm, cięcia w Nscale, odbicie z 23.11, Cairn/MONAI) pominięto albo podano tylko nowy element. Sekcja „euro neocloud” obejmuje wyłącznie Nscale – u pozostałych dostawców nie znaleziono w tym oknie nowych, zweryfikowanych informacji.

## Raport technologiczny

### Biznes

#### Aktualizacja CrowdStrike–Zscaler: 500 mln USD oszczędności rocznie, sprzedaż Zscalera przez Falcon Flex (24.11)
Nowe w stosunku do rejestru (warunki transakcji opisane 24.11):
- CrowdStrike zakłada 500 mln USD rocznych oszczędności kosztowych w ciągu dwóch lat. Mają pochodzić z infrastruktury DC, zakupów u dostawców i operacji.
- Cel dodatkowej sprzedaży krzyżowej to ok. 1 mld USD ARR w ciągu 30 miesięcy od zamknięcia.
- Obie firmy mają ponad 2 000 wspólnych klientów. Red Canary (Zscaler, od 08.2025) konkuruje bezpośrednio z Falcon Complete MDR.
- Produkty Zscalera trafią do licencjonowania Falcon Flex. Nie wiadomo, co z klientami, którzy łączą Zscaler z EDR innego dostawcy.

**Znaczenie:** Oszczędności „z infrastruktury DC” oznaczają konsolidację chmur obu dostawców. To ryzyko migracji węzłów egzekwowania ZIA/ZPA (zmiany PoP, adresacji IP egress, latencji) w latach 2027–2028. Klienci używający list dozwolonych IP egress Zscalera po stronie SaaS powinni to uwzględnić w planach zmian. Dublujące się MDR (Red Canary i Falcon Complete) zostaną najpewniej scalone. Kontrakty MDR na więcej niż 24 miesiące warto zabezpieczyć klauzulą o zmianie kontroli.

Źródło: [The Register](https://www.theregister.com/security/2026/11/24/crowdstrike-splashes-44b-on-zscaler-customers-brace-for-the-bill/5311877)

#### Nutanix: zmiana CEO – Rajiv Ramaswami odejdzie po znalezieniu następcy (8-K, 24.11)
Rada Nutanix zatrudniła zewnętrzną firmę rekrutacyjną. Ramaswami pozostaje CEO do objęcia funkcji przez następcę, a potem zostanie doradcą. Kieruje firmą od 12.2020 i przeprowadził ją przez okres przechodzenia klientów z VMware po przejęciu przez Broadcom.

**Znaczenie:** Nutanix (AHV + NCI) to główna alternatywa on-prem dla klientów uciekających od cennika VCF. Zmiana CEO w trakcie tej fali migracji grozi przesunięciem priorytetów, np. agresywniejszą monetyzacją wobec zdobytej bazy albo zmianami w programie zewnętrznej pamięci masowej (wsparcie Pure/Dell). Dla migracji planowanych na 2027 r. warto zablokować ceny wieloletnie przed zmianą kierownictwa.

Źródło: [SEC 8-K Nutanix (ex. 99.2)](https://www.sec.gov/Archives/edgar/data/0001618732/000117184326007693/exh_992.htm), [The Register](https://www.theregister.com/virtualization/2026/11/25/nutanix-ceo-announces-gradual-departure-intent-to-complete-set-of-storage-links/5312010)

### Technologia

#### VMware VMSA-2026-0011: 23 luki, w tym obejście uwierzytelnienia w vCenter i dwa RCE na hoście ESX (24.11)
Trzecie krytyczne wydanie poprawek Broadcom w ciągu tygodnia (po VMSA-2026-0009 i -0010 z rejestru 20–21.11):
- **CVE-2026-123903 (CVSS 9,8):** obejście uwierzytelnienia w vCenter. To trzecia luka 9,8 tego typu w vCenter od lipca.
- **CVE-2026-123910 (9,8):** zapis poza zakresem sterty w usłudze I/O filter, wykonanie kodu na hoście ESX.
- **CVE-2026-123911 (9,8):** use-after-free, wykonanie kodu na hoście ESX.
- **CVE-2026-123912 (krytyczna):** ucieczka z gościa w Workstation i Fusion. W ESX luka też jest, ale sandbox VMX blokuje ucieczkę. Obejście: usunięcie kontrolera USB z VM.

Dotyczy ESX, vCenter, VCF, vSphere Foundation, Telco Cloud Platform/Infrastructure, VMware Tools i Avi Load Balancer. Brak informacji o aktywnym wykorzystaniu. Część poprawek była już w paczkach z poprzedniego tygodnia, a reszta wymaga nowych buildów.

**Znaczenie:** vCenter ma już czwartą w tym kwartale lukę pre-auth klasy 9,8, a CVE-2026-123890 z 20.11 jest wykorzystywana przez chińską grupę. To przesądza: interfejs zarządzania vCenter musi być w osobnym segmencie z dostępem tylko przez PAM/jump host, a nie w sieci biurowej. RCE w I/O filter jest wykorzystywalne z poziomu VM z filtrem VAIO (replikacja, cache), czyli zwykle wymaga już zdobytego przyczółka. W klastrach multi-tenant (VCF jako IaaS) to jednak realna ścieżka do hosta. Trzy okna serwisowe w tydzień przy rolling upgrade ESX (vMotion, DRS) to istotny koszt operacyjny. Konsolidacja do jednego okna z buildem obejmującym 0009–0011 jest rozsądna tylko wtedy, gdy vCenter jest odizolowany.

Źródło: [The Register](https://www.theregister.com/virtualization/2026/11/25/this-could-mess-with-thanksgiving-says-vmware-as-it-drops-third-critical-patch-bundle-in-a-week/5312017)

#### Microsoft 365: końcowy raport z awarii Exchange Online 12–14.11 – defekt Windows z lipca, ponad 80% bramek w Ameryce Północnej niedostępnych
Raport końcowy (PIR) dla awarii, której przyczyny rejestr podawał wstępnie 19.11:
- **Przyczyna:** zmiana kodu Windows z lipca w konfiguracji specyficznej dla serwerów bramowych M365 (nie w standardowym Windows). Działała globalnie ponad 3 miesiące, zanim wywołała awarię.
- **Przebieg:** awarię pogłębił failover przy zmniejszonej pojemności. Rosła wraz z porannym logowaniem użytkowników w USA i w szczycie ponad 80% bramek w Ameryce Północnej nie odpowiadało.
- **Skutki:** część wiadomości do kont konsumenckich nieaktywnych ponad 30 dni wygasła i wróciła jako NDR, mimo wydłużenia okna ponowień do 48 h. Według Microsoftu poczta firmowa nie została utracona.
- **Luki w testach:** testy obciążeniowe szły na starszym sprzęcie o mniejszej liczbie rdzeni, a defekt nasilał się na nowych maszynach. Failover był testowany niewystarczająco.
- **Zobowiązania:** partycjonowanie i izolacja usług, więcej zapasu pojemności, testy na nowym sprzęcie przy realnym ruchu szczytowym, lepszy audyt zmian w Windows.

**Znaczenie:** To klasyczny latent defect zależny od sprzętu: błąd skalujący się z liczbą rdzeni nie wyjdzie w testach na starszej flocie. Wniosek dla własnych platform: staging i testy obciążeniowe muszą działać na tej samej generacji CPU co produkcja, szczególnie przy wymianie floty na 192–256 rdzeni (Venice, Diamond Rapids). Drugi wniosek: failover do mniejszej pojemności zamienia częściową awarię w kaskadę. Dla systemów N+1 trzeba testować failover przy szczycie dobowym, a nie w oknie nocnym.

Źródło: [The Register](https://www.theregister.com/saas/2026/11/23/microsoft-mega-outage-report-reveals-testing-gaps-and-emails-that-never-arrived/5311656)

#### Komisja Europejska: kradzież danych z EU Academy przy użyciu „narzędzia hakerskiego wspomaganego AI” (potwierdzenie 23.11)
Podejrzaną aktywność wykryto 22.10, atak ujawniono 19.11, a 23.11 KE potwierdziła wyciek imion, nazwisk i adresów e-mail użytkowników spoza instytucji UE. Kilka aplikacji wewnętrznych odłączono, część nadal nie działa. Skala wycieku jest nieznana, sprawca nieustalony, a KE nie odpowiada, czy napastnicy zostali usunięci z sieci. Platforma jest wyłączona do 14.01.2027. Użytkowników nie powiadomiono indywidualnie, sprawę prowadzi EDPS.

**Znaczenie:** To pierwszy przypadek, w którym instytucja UE oficjalnie przypisuje włamanie narzędziu ofensywnemu z AI. Zbiega się to z raportami o Cairn i ARTEX z rejestru. Prawie miesiąc od wykrycia do ujawnienia i brak potwierdzenia eradykacji to typowy obraz, gdy autonomiczne narzędzie rozprasza aktywność po wielu aplikacjach. Platformy szkoleniowe i LMS z kontami zewnętrznymi są zwykle najsłabiej segmentowane i bywają punktem wejścia do sieci wewnętrznej.

Źródło: [The Register](https://www.theregister.com/cyber-crime/2026/11/24/european-commission-confirms-personal-data-stolen-in-ai-assisted-cyberattack/5311834)

#### Aktualizacja: CISA KEV (23.11) – MONAI Label (path traversal) i N-able Mail Assure (SSRF)
CISA dodała CVE-2026-133968 (path traversal w Project MONAI Label) i CVE-2026-134624 (SSRF w N-able Mail Assure/SpamExperts). Obie były już w rejestrze jako incydenty: MONAI jako cel agentów Cairn (21.11), N-able jako włamanie do chmury (21.11). Nowy element to formalny status KEV.

**Znaczenie:** MONAI Label to pierwsze narzędzie MLOps/obrazowania medycznego w KEV z wykorzystaniem przypisanym autonomicznym agentom. W organizacjach podlegających BOD 22-01 lub stosujących KEV jako politykę priorytetyzacji obejmuje to także stacje anotacji i serwery GPU w zespołach badawczych, zwykle poza inwentarzem IT.

Źródło: [CISA](https://www.cisa.gov/news-events/alerts/2026/11/23/cisa-adds-two-known-exploited-vulnerabilities-catalog)

## Raport AI-ML

### Biznes

#### HPE Q4 FY2026: przychód 14,5 mld USD (+50% r/r), serwery +94,7%, wzrost FY2027 napędzany cenami, nie wolumenem (23–24.11)
- **Przychód i segmenty:** 14,5 mld USD (rekord). AI i Cloud 11,3 mld USD (+68,6%), serwery 8,8 mld (+94,7%), sieci 3,1 mld (+12,3%; DC +12,1%, routing +10,1%, security −1,8%), storage 1,5 mld (+33,8%).
- **Zamówienia i marża:** zamówienia ponad +80% r/r. Marża brutto 38% wobec 40,1% kwartał wcześniej. Klienci opierają się podwyżkom.
- **Podaż:** ograniczeniem jest podaż. CEO spodziewa się niedoboru DRAM i NAND co najmniej do II połowy 2028 r.
- **Prognoza:** FY2027 61,25 mld USD ±1,85 mld (ok. +31%), wzrost głównie z ASP, nie z liczby jednostek. Q1 FY27 14,3 mld ±0,3 mld.
- **Gotówka i dług:** FCF 1,9 mld USD, dług netto spadł do 12,9 mld.

**Analiza:** Spadek marży o ok. 2 pp kw/kw przy wzroście przychodu o 50% oznacza, że HPE nie przenosi w pełni kosztów pamięci na klientów. To spójne z doniesieniem FT o podwyżkach do 45% po podpisaniu umów (rejestr 23.11). Prognoza oparta na ASP oznacza, że w 2027 r. za ten sam budżet kupisz mniej węzłów. Capacity planning trzeba liczyć w jednostkach (GPU, TB), a nie w USD. Horyzont niedoboru pamięci do 2H 2028 obejmuje cały cykl Rubin/Rubin Ultra. Kontrakty ramowe z ceną pamięci indeksowaną (zamiast „cena w dniu wysyłki”) stają się kluczową dźwignią negocjacyjną.

Źródło: [Blocks & Files](https://www.blocksandfiles.com/ai-ml/2026/11/24/hpe-customers-ravenous-for-ai-servers-in-its-latest-quarter/5311861), [The Register](https://www.theregister.com/systems/2026/11/25/dell-and-hpe-both-say-theyve-become-different-companies/5312003)

#### Dell Q3 FY2027: ISG 42 mld USD (+198% r/r), backlog AI 32 mld, prognoza roczna 202 mld; każde zamówienie AI weryfikowane pod kątem finansowania i zasilania (24.11)
- **Wyniki:** ISG 42 mld USD (+198%), CSG 12,9 mld (+3%), backlog serwerów AI na koniec Q3 32 mld USD. Prognoza FY27 podniesiona o 10 mld do 202 mld USD (+78% r/r).
- **Kryteria zamówień:** Dell deklaruje, że jest „wysoce selektywny” wobec partnerów. Każde zamówienie AI przechodzi weryfikację finansowania, jakości klienta, zgodności eksportowej oraz dostępu do gruntu i energii.
- **Komponenty:** część klientów odkłada zakupy PC na rzecz sprzętu AI, a Dell przesuwa komponenty zamówione dla PC do serwerów.

Dane pochodzą z relacji The Register. Pełny komunikat Dell nie był dostępny w momencie weryfikacji.

**Analiza:** Weryfikacja zamówień pod kątem finansowania i zasilania to pierwsza publiczna deklaracja OEM, że ryzyko kredytowe odbiorcy (neocloudy, projekty DC) wchodzi do kwalifikacji backlogu. W praktyce mniejsi kupujący bez potwierdzonego przyłącza lub finansowania dostaną gorsze terminy albo zostaną wypchnięci z kolejki. Przesuwanie komponentów z PC do serwerów zaostrza niedobór pamięci w segmencie klienckim. Odnowienia floty end-user w 2027 r. trzeba planować z dłuższymi terminami i wyższą ceną.

Źródło: [The Register](https://www.theregister.com/systems/2026/11/25/dell-and-hpe-both-say-theyve-become-different-companies/5312003)

#### Amazon przejmuje technologię Decart Optimization Stack i zespół za ok. 7,5 mld USD – przygotowanie do sprzedaży Trainium na zewnątrz (23.11)
Amazon przejmuje technologię Decart Optimization Stack (DOS), założycieli i zespół badawczy. Kwota 7,5 mld USD pochodzi od Bloomberga i nie została potwierdzona przez strony (wcześniej WSJ podawał ok. 7 mld). DOS to warstwa optymalizacji wykonania modeli działająca na GPU Nvidii, TPU i Trainium. Według TechCrunch Amazon przygotowuje sprzedaż Trainium klientom zewnętrznym jako alternatywę dla Nvidii.

**Analiza:** Główną barierą dla Trainium nie jest krzem, tylko koszt portowania i strojenia kerneli (Neuron SDK, NKI) wobec dojrzałości CUDA. Kupno warstwy kompilacji i optymalizacji działającej na wielu backendach to próba obniżenia kosztu zmiany akceleratora, a więc osłabienia lock-in do CUDA. Jeśli Trainium trafi do on-prem lub kolokacji, pojawi się trzecia (po Nvidii i AMD) opcja zakupowa dla dużych klastrów inferencyjnych. Ryzyko: ekosystem narzędzi (profilery, NCCL-ekwiwalenty, obsługa sieci EFA poza AWS) jest dziś ściśle związany z AWS. Wycena 7,5 mld USD za zespół i stos (Anthropic odrzucił zakup przy ok. 6 mld) pokazuje, jak drogie są kompetencje kompilatorowe na rynku.

Źródło: [TechCrunch](https://techcrunch.com/2026/11/23/amazon-snaps-up-decarts-ai-optimization-tech-for-7-5b-as-it-prepares-to-sell-trainium-chips/), [The Register](https://www.theregister.com/ai-and-ml/2026/11/23/amazon-scoops-up-ai-accelerator-tech-to-keep-chips-busy-at-all-times/5311754)

#### BIS dopisuje Megaspeed i sześć powiązanych firm z Singapuru, Malezji i Tajlandii do Entity List (reguła złożona 23.11, publikacja w Federal Register 25.11)
Megaspeed to singapurski dostawca chmury badany w sprawie przekierowania GPU do Chin (rejestr 21.11: wezwanie od prokuratora generalnego Kalifornii). Na liście są także właściciel Swiftdata i malezyjska spółka zależna Speedmatrix. Dostawy towarów objętych EAR wymagają licencji z domniemaniem odmowy. W tle wymieniono też Apex Enterprise Solutions (singapurska spółka zależna Alibaby). Nvidia wpisała Megaspeed na wewnętrzną listę zakazu sprzedaży już w 06.2025.

**Analiza:** To pierwsze egzekucyjne zastosowanie nowego reżimu wobec „chmur pośredniczących” w ASEAN po regule z 15.11. Konsekwencje dla operatorów w regionie: obowiązek KYC klienta końcowego przy najmie GPU, ryzyko zamrożenia wsparcia (RMA, firmware, licencje NVAIE) dla sprzętu już zainstalowanego u podmiotów z listy, a także efekt mrożący dla bankowego finansowania DC w Malezji i Tajlandii (por. Aolani w rejestrze 17–18.11). Kto kupuje moc GPU w regionie, powinien sprawdzić łańcuch właścicielski dostawcy i wpisać do umowy klauzule o sankcjach z prawem do natychmiastowej migracji.

Źródło: [The Register](https://www.theregister.com/systems/2026/11/24/us-blacklists-megaspeed-hours-after-jensen-huang-returns-to-nvidia/5311901)

### Technologia

#### NCSA „Garnet”: 334 mln USD od Schmidt Sciences – HPE z 1 280 B200 (NVL4) i dwiema szafami NVL72 B300 do inferencji, 50 PB VAST (24.11)
- **Finansowanie:** ok. 240 mln USD na sprzęt i 94 mln na 5 lat eksploatacji. HPE wygrał przetarg.
- **Partycja główna:** 640 płyt NVL4 (2× B200 + Grace na płytę), łącznie ok. 107 PF FP64 peak.
- **Partycja inferencyjna:** dwie szafy NVL72 z 144 GPU B300 i 72 CPU Grace, optymalizowane pod FP8/FP4.
- **Sieć i storage:** ConnectX-8 800 Gb/s, Quantum-X InfiniBand w topologii rail-optimized, VAST 50 PB raw obok istniejącego Lustre 25 PB.
- **Termin i dostęp:** instalacja ok. połowy 2027 r. NCSA ma ok. 10% czasu, resztą dysponuje Schmidt Sciences, także dla badaczy spoza USA (z zastrzeżeniem kontroli eksportu).

**Analiza:** To wzorzec architektury naukowej na 2027 r.: osobna partycja FP64 (B200 zachowuje sensowne FP64) i osobna partycja inferencyjna na B300, gdzie FP64 jest szczątkowe. Rozdział wynika bezpośrednio z degradacji FP64 w kolejnych generacjach Nvidii. Ośrodki HPC muszą teraz utrzymywać dwie klasy węzłów zamiast jednej. Rail-optimized InfiniBand przy NVL4 (zamiast NVL72 w całości) pokazuje, że dla mieszanych obciążeń MPI/AI duża domena NVLink nie zawsze się opłaca. Dostęp dla badaczy spoza USA to rzadki kanał dostępu do mocy Blackwell dla europejskich zespołów naukowych.

Źródło: [The Next Platform](https://www.nextplatform.com/hpc/2026/11/24/ncsa-gets-supercomputing-capacity-with-334-million-schmidt-sciences-grant/5311971)

## euro neocloud

### Nscale

#### CEO do pracowników: „wystarczająco kapitału, by pozostać spółką prywatną w dającej się przewidzieć przyszłości” (FT, spotkanie 24.11)
**Fakty:**
- Na ogólnofirmowym spotkaniu 24.11 CEO Josh Payne powiedział pracownikom, że Nscale ma dość pozyskanego kapitału, by na razie nie wchodzić na giełdę. To parafraza FT na podstawie czterech źródeł, Nscale odmówiło komentarza.
- Cel to IPO „tak szybko, jak to możliwe”. Wcześniej zarząd zakładał debiut jeszcze w 2026 r. (cel ok. 35 mld USD).
- FT przypomina utratę projektu DC Microsoft pod Londynem o wartości 3,2 mld USD z powodu opóźnień przyłączenia do sieci.

Kontekst z rejestru: wstrzymanie rekrutacji i strata ok. 1,69 mld USD przy przychodzie 307,5 mln za 9 miesięcy (24.11), licencja BIS dla kontraktu z ByteDance (20.11).

**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:*
  - Deklaracja „mamy dość kapitału” sprzeczna z prospektem, który mówi o potrzebie dodatkowego finansowania na zobowiązania 2027 r. Najpewniej chodzi o horyzont operacyjny (opex), a nie o capex na Monarch i kolejne kampusy.
  - Brak kwot i źródeł finansowania w komunikacie.
  - Odsunięcie IPO przy rynku kredytowym, w którym LTV dla pożyczek pod GPU spadło do 50–60% (zob. sekcja ryzyk), oznacza droższy dług i większy wkład własny przy kolejnych transzach.
- *Ocena ryzyka:* **Wysokie** (bez zmian wobec 24.11). Uspokajający komunikat wewnętrzny nie zmienia struktury: wysoka dźwignia (dług ok. 5,5 mld USD), strata kilkukrotnie wyższa od przychodu, koncentracja klientów (Anthropic, Microsoft z historią wycofań) i zależność od licencji BIS. Klienci z kontraktami powyżej 12 miesięcy powinni żądać escrow przedpłat i praw step-in do sprzętu.
- *Linki:* [Stocktwits (za FT)](https://stocktwits.com/news-articles/markets/equity/nvidia-backed-nscale-reportedly-eyes-private-route-as-ai-market-sell-off-clouds-35-b-ipo-plans/cNbuDjYR9lc), [The Next Web](https://thenextweb.com/news/nscale-ipo-filing-debt-nvidia-anthropic)

## AI for networking

#### OpenAI i Cisco: Shirudo – open-source runtime eBPF do „tarcz” blokujących exploity bez restartu i bez łatki, tarcze generowane przez Codex (23.11)
Shirudo (BSD-2, programy BPF na licencji BSD/GPL) wywodzi się z Tetragon (Cilium/Isovalent).
- **Mechanizm:** agent skanuje infrastrukturę, Codex generuje program eBPF („tarczę”) podpinany do działającego procesu, a człowiek go przegląda.
- **Tryb obserwacji:** przed egzekwowaniem tarcza może działać w trybie podglądu wpływu na aplikację.
- **Integralność:** tarcze są podpisywane i weryfikowane przed załadowaniem, a TPM prowadzi weryfikowalny rejestr ładowania i usuwania.
- **Demonstracja:** w teście Java4Shell (CVE-2026-114938, rejestr 19.11) tarcza dodała walidację wejścia do dekodera Base64 w działającym Elasticsearch i odrzuciła exploit bez restartu.

Producenci nie podali narzutu wydajnościowego ani wspieranych wersji jądra. Repozytorium: [github.com/shirudo-dev/shirudo](https://github.com/shirudo-dev/shirudo).

**Analiza:**
- **Wartość:** przy medianie czasu od ujawnienia do eksploatacji ok. 33 dni (Datadog, zob. newslettery) i cyklu łatania floty liczonym w tygodniach eBPF daje warstwę kompensacyjną wdrażaną w minutach, z kontrolą zmian (podpis + TPM).
- **Failure modes:** uprobes na gorących ścieżkach (dekodery, parsery HTTP) dodają opóźnienie na każde wywołanie. Przy wysokim QPS trzeba to zmierzyć. Tarcza wygenerowana przez LLM może też przepuścić warianty exploita albo zablokować legalny ruch, więc tryb obserwacji jest obowiązkowy.
- **Ograniczenia:** w JVM z JIT podpinanie do kodu skompilowanego jest kruche (zmiany offsetów po deoptymalizacji). Wymaga też nowych jąder z BTF/CO-RE, czego nie spełnią starsze RHEL 7/8 i appliance'y.
- **Lock-in:** formalnie otwarte, ale przepływ pracy zakłada Codex jako generator tarcz. Dla zespołów na Cilium/Tetragon integracja jest naturalna, a dla platform bez eBPF (Windows, zamknięte appliance'y sieciowe) nie ma odpowiednika.

Źródło: [OpenAI](https://openai.com/index/introducing-shirudo/), [The Register](https://www.theregister.com/security/2026/11/24/cisco-and-openai-pitch-ebpf-kernel-plugins-as-the-way-to-tame-the-patchpocalypse/5311795)

#### Agent AI (OpenCode + Chrome + curl) przeszedł od rekonesansu do testowania kluczy API w instytucji finansowej w 5 min 44 s – wykryty po wzorcu zachowań, uwięziony w środowisku-pułapce (incydent 16.11, publikacja 24.11)
Według Jesta (startup deception z Tel Awiwu) napastnik z Bukaresztu użył open-source'owego agenta OpenCode, Chrome (desktop i podszywanie się pod Androida) i curl z jednego Maca. Sekwencja:
- 3 odrzucone próby wejścia,
- znalezienie wabika z dokumentacją API po 5 s i przejście po linku po 13 s,
- klasyfikacja ruchu jako agent AI po ok. 64 s i przekierowanie do fałszywego środowiska z fałszywym kluczem,
- testy kluczy (fałszywy, brak klucza, zmyślony) i enumeracja endpointów, po czym agent zrezygnował.

Adres IP nie miał złej reputacji. Nic nie wyciekło.

**Analiza:** To źródło sprzedającego rozwiązanie, więc szczegóły trzeba traktować ostrożnie. Sygnatury behawioralne są jednak przenośne do własnego WAF/NDR:
- zmiana User-Agent desktop → mobile → curl z jednego IP w ciągu sekund,
- odwołania do zasobów, które istnieją tylko w dokumentacji API,
- rotacja kluczy w krótkim oknie.

Reputacja IP i rate limiting per IP są przy tym tempie bezużyteczne. Potrzebna jest korelacja per sesja lub odcisk TLS (JA4) i tani deception (honeytokeny w dokumentacji OpenAPI). Skala: przy kosztach bliskich zeru takie skanowanie da się prowadzić równolegle na tysiące celów.

Źródło: [The Register](https://www.theregister.com/security/2026/11/24/an-ai-coding-agent-a-browser-and-a-mac-tool-found-a-financial-firms-access-key-in-under-6-minutes/5311922)

## Ryzyka rynku AI/GPU

#### Kredytodawcy GPU obniżają LTV z 80% do 50–60%; firmy średniej wielkości szukają pożyczek ratunkowych i pomostowych, ryzyko przejmowania sprzętu w 3–6 miesięcy (Odd Lots 19.11, analiza 23.11)
Stanley Lee (CPO, Liquid Compute) w podcaście Bloomberga Odd Lots opisał zmianę LTV dla pożyczek zabezpieczonych na GPU:
- **LTV:** latem 80%, po Labor Day ok. 70%, obecnie 50–60%.
- **Pożyczki ratunkowe:** wielu średnich operatorów zgłasza się po pożyczki ratunkowe i pomostowe z bardzo ciasnymi covenantami.
- **Ryzyko przejęcia sprzętu:** bez refinansowania (takeout) w ciągu 3–6 miesięcy kredytodawcy przejmą zastawiony sprzęt.

Dla porównania: CFO CoreWeave zakłada 7–7,5% kosztu kolejnej pożyczki zabezpieczonej kontraktem, IREN pozyskał 6,5 mld USD po 6–9,5%. Przy 50–60% LTV operator musi wnieść 40–50 USD własnego kapitału na każde 100 USD sprzętu zamiast 20 USD.

**Analiza:** To zmiana o największym wpływie systemowym w tym tygodniu, bo działa na całą warstwę neocloudów drugiego rzędu. Podwojenie wymaganego equity przy zamkniętym rynku IPO (Nscale, DayOne, Firmus) oznacza mniej nowych zamówień GPU od mniejszych graczy. To spójne z deklaracją Della o weryfikacji finansowania każdego zamówienia. Przejęcia sprzętu wyrzucą na rynek wtórny partie H100/H200/B200. Ceny najmu spot mogą wtedy spaść lokalnie mimo trendu wzrostowego z połowy listopada (rejestr 18.11). Wskaźniki do obserwacji: spready obligacji neocloudów, ogłoszenia sprzedaży sprzętu przez kredytodawców i ceny H100/h na rynkach spot.

Źródło: [Yahoo Finance / TIKR](https://finance.yahoo.com/markets/stocks/articles/bad-news-coreweave-iren-gpu-160609776.html)

#### Odwrót po rajdzie: Nvidia ok. −6% 24.11 po +31% dzień wcześniej; TSMC +9,9% (limit dzienny) i oświadczenie, że popyt na zaawansowane węzły „pozostaje niezwykle silny” (23–24.11)
- Nvidia oddała ok. 6%, a AMD, Broadcom, CoreWeave i Nebius też spadały w ramach ponownej presji na rynek AI. Dokładnych kursów zamknięcia źródło nie podało.
- TSMC w Tajpej zyskało 9,9% (limit dzienny, ok. +158 mld USD kapitalizacji).
- W oświadczeniu z 23.11 TSMC odrzuciło obawy o głębszą korektę AI: popyt na zaawansowane węzły jest „niezwykle silny”, a moce pozostaną napięte niezależnie od architektury. Jednocześnie przyznało słabość w segmentach konsumenckich i wrażliwych na cenę z powodu drożejących komponentów.

**Analiza:** Zmienność ±6–30% dziennie na Nvidii przy niezmienionych fundamentach potwierdza, że wycena jest dziś napędzana zdarzeniami politycznymi i zarządczymi (zeznania Huanga 1.12), a nie danymi o wykorzystaniu GPU. TSMC wskazuje strukturalną asymetrię: zaawansowane węzły ograniczone podażą, a segment konsumencki tłumiony kosztem pamięci. Dla kupujących oznacza to, że ceny akceleratorów i serwerów nie spadną mimo korekty kursów. Ryzyko przesuwa się z dostępności krzemu na koszt kapitału po stronie kupujących (zob. LTV).

Źródło: [Yahoo Finance](https://finance.yahoo.com/markets/article/nvidia-amd-broadcom-lead-chip-stocks-lower-as-ai-trade-comes-under-pressure-141635463.html), [Fortune](https://fortune.com/2026/11/24/tsmc-gains-158-billion-in-value-in-one-day-as-jensen-huang-returns-as-nvidia-ceo), [Taipei Times](https://taipeitimes.com/News/biz/archives/2026/11/24/2003868252)

#### Wzrost OEM z cen, nie z jednostek: HPE prognozuje +31% w FY27 głównie z ASP, marża spada; Dell filtruje backlog pod kątem finansowania i energii (23–24.11)
Szczegóły w sekcji AI-ML. Dla oceny ryzyka liczą się trzy elementy:
- HPE wprost przypisuje wzrost w 2027 r. cenom, a nie wolumenowi,
- marża HPE spadła o ok. 2 pp kw/kw,
- Dell (backlog 32 mld USD) kwalifikuje zamówienia według finansowania klienta i dostępu do gruntu i energii.

**Analiza:** Wzrost przychodów z ASP przy stałych lub rosnących wolniej jednostkach oznacza, że część „boomu” to inflacja kosztów pamięci, a nie przyrost mocy obliczeniowej. To istotne przy porównywaniu capex hyperscalerów z realnie dostarczonymi MW i GPU. Filtrowanie backlogu przez OEM to wczesny bufor przed anulacjami. Jednocześnie jest to sygnał, że OEM widzą ryzyko kontrahenta po stronie kupujących, spójny z obniżką LTV.

Źródło: [Blocks & Files](https://www.blocksandfiles.com/ai-ml/2026/11/24/hpe-customers-ravenous-for-ai-servers-in-its-latest-quarter/5311861), [The Register](https://www.theregister.com/systems/2026/11/25/dell-and-hpe-both-say-theyve-become-different-companies/5312003)

## Newsletters summary

#### Datadog Mitigeddon: wykorzystywane w praktyce jest tylko 0,6–0,8% CVE, a mediana od ujawnienia do eksploatacji spadła do 33 dni w 2026 r. ze 163 dni w 2024 r. (TLDR InfoSec)
Darmowy serwis łączy częstość występowania CVE w środowiskach klientów Datadog z osią czasu: ujawnienie, publiczny exploit, wpis do KEV (CISA/VulnCheck). Próba jest przesunięta w stronę środowisk cloud-native.

**Analiza:** Skrócenie okna o ok. 5× to twardy argument za priorytetyzacją opartą na danych o eksploatacji i występowaniu (KEV + prevalence + EPSS) zamiast CVSS. Uzasadnia też warstwę kompensacyjną typu Shirudo/WAF dla luk, których nie da się załatać w 30 dni.

Źródło: [Datadog Security Labs](https://securitylabs.datadoghq.com/articles/introducing-mitigeddon/)

#### Porównanie sandboxów dla agentów kodujących: nono, Docker Sandboxes, Microsandbox, NVIDIA OpenShell (Kudelski, TLDR InfoSec)
Wszystkie cztery dostały 4–4,5/5. Docker Sandboxes i Microsandbox to microVM, nono to kontrole na poziomie OS (najszybszy start), OpenShell obsługuje kilka backendów izolacji. Tylko nono i OpenShell wiążą poświadczenia jednocześnie z domeną i endpointem. Żaden nie pozwala zablokować pojedynczego pliku w dozwolonym katalogu ani dostosować listy syscalli.

**Analiza:** Przy agentach z dostępem do sekretów (zob. incydent Together AI w rejestrze 23.11) kluczowe jest wiązanie poświadczeń z endpointem, a nie sam typ izolacji. Brak deny na poziomie pliku oznacza, że sekrety nie mogą leżeć w drzewie repozytorium montowanym do sandboxa.

Źródło: [Kudelski Security](https://kudelskisecurity.com/research/sandboxing-coding-agents)

#### WordPress.org: karencja wydań zablokowała backdoor w sześciu wtyczkach WP Connect wypchniętych z przejętego konta autora (CVE-2026-135202, TLDR InfoSec)
Złośliwe wydania (m.in. Air WP Sync) przy każdym żądaniu łączyły się z C2 z wyłączoną weryfikacją TLS i wykonywały PHP przez `eval()`. Dawały webshelle i zrzut hashy haseł. Nowy okres karencji wydań („Protect The Shire”) zatrzymał aktualizacje, zanim trafiły do stron.

**Analiza:** To kolejny po GHES 3.23 (rejestr 24.11) dowód, że opóźnienie dystrybucji nowych wersji (cooldown) jest skuteczniejsze niż detekcja przy kompromitacji kont wydawców. Ten sam wzorzec warto egzekwować w wewnętrznych proxy pakietów (Artifactory/Nexus: minimalny wiek wersji 3–7 dni).

Źródło: [Wordfence](https://www.wordfence.com/blog/2026/11/supply-chain-compromise-backdoors-six-wp-connect-plugins-blocked-by-wordpress-orgs-release-hold/)

#### binary-parser < 2.3.1 (CVE-2026-134814, CVSS 9,2): wstrzyknięcie JavaScript przez definicje parsera (TLDR InfoSec)
Niewalidowane opcje tekstowe (`length`, `key`, `tag`, `offset` i inne) trafiają do kodu generowanego przy kompilacji parsera. Wykonanie następuje w procesie Node.js. Podatne są tylko aplikacje, w których definicja parsera pochodzi z niezaufanego źródła, a parsowanie niezaufanych danych zaufaną definicją jest bezpieczne.

**Analiza:** Ryzyko dotyczy głównie narzędzi telemetrii i IoT, które ładują definicje protokołów z konfiguracji dostarczanej przez użytkownika lub urządzenie. Należy podnieść wersję w zależnościach przechodnich (`npm ls binary-parser`).

Źródło: [The Hacker News](https://thehackernews.com/2026/11/critical-binary-parser-flaw-enables.html)

#### Project Zero (Forshaw): wyczyszczenie jednego bitu w nagłówku PE omija kontrolę integralności kodu – VirtualBox, UMCI w S Mode i PPL bez uprawnień admina (TLDR InfoSec)
Wyczyszczenie flagi `IMAGE_FILE_EXECUTABLE_IMAGE` w assembly .NET sprawia, że `NtCreateSection` odrzuca plik, ale .NET ładuje go ścieżką flat-image bez sprawdzania integralności. Efekty: CVE-2026-87268 (VirtualBox), CVE-2026-108398 (UMCI przez regasm.exe) oraz wykonanie kodu na poziomie PPL/TCB przez przejęcie obiektu COM. Microsoft poprawił WLDP w październiku. Obejście PPL pozostaje poza kryteriami serwisowymi MSRC.

**Analiza:** Środowiska z WDAC powinny włączyć opcję Dynamic Code Security, która blokuje atak. Bez niej ochrona PPL dla agentów EDR i LSASS ma znaną, niełataną lukę.

Źródło: [Project Zero](https://projectzero.google/2026/11/virtualbox-process-hardening.html)

#### Claude Ship: agenci Anthropic do przeglądu, poprawek i zatwierdzania PR według polityki zespołu (TLDR AI / TLDR)
Trzy komponenty:
- Code Review oznacza problemy,
- PR Steward wypycha poprawki,
- Claude Approvals sprawdza zmiany względem polityki zatwierdzeń zespołu i może zatwierdzić PR, jeśli polityka na to pozwala.

**Analiza:** Automatyczne zatwierdzanie przenosi kontrolę z ludzi na politykę. Kluczowe są ochrona gałęzi (wymóg człowieka dla ścieżek krytycznych: IaC, CI, uprawnienia), rozdzielenie tożsamości agenta piszącego i zatwierdzającego oraz audyt decyzji. Bez tego ten sam łańcuch prompt injection może zarówno wprowadzić zmianę, jak i ją zatwierdzić.

Źródło: [Claude Docs](https://code.claude.com/docs/en/claude-ship), [The Register](https://www.theregister.com/ai-and-ml/2026/11/24/claude-ship-puts-ai-at-the-helm-of-pull-request-approvals/5311883)

#### Microsoft AI „Rocket”: własny framework RL do post-treningu MAI Base 2 (3 bln parametrów) bez destylacji z modeli zewnętrznych (TLDR AI)
Microsoft AI opisał wewnętrzną infrastrukturę RL dla modelu o 3 bln parametrów.

**Analiza:** Po NVIDIA Dynamo dla RL i Prime Intellect Model Factory (rejestr 18–19.11) to kolejny sygnał, że RL post-training staje się osobną klasą obciążenia: generowanie rolloutów (inferencja), trening i dystrybucja wag w pętli. Klastry projektowane na 2027 r. muszą obsługiwać szybkie przełączanie ról węzłów i transfer delt wag między pulami.

Źródło: [Microsoft AI](https://microsoft.ai/news/rl-infrastructure-at-microsoft-ai/)

#### GLM-5.4 (Z.ai): pierwszy flagowy model na Loop Transformerze – współdzielony blok warstw iterowany dla głębszego rozumowania (TLDR AI)
Z.ai deklaruje poziom porównywalny z modelami 2–3 bln parametrów przy znacznie mniejszej liczbie parametrów. Model jest dostępny w GLM Coding Plan.

**Analiza:** Architektury pętlowe zamieniają pamięć (parametry) na obliczenia (iteracje). Wymagania HBM na same wagi spadają, ale rośnie latencja i koszt FLOP na token, a liczba iteracji może być dynamiczna. Komplikuje to planowanie batchy w vLLM/SGLang, bo czas dekodowania na żądanie przestaje być przewidywalny. Deklaracje wydajności pochodzą od producenta.

Źródło: [Z.ai](https://z.ai/blog/glm-5.4)

#### Google DeepCrossAttention: dynamiczne połączenia rezydualne, 30–40% mniej parametrów przy tej samej jakości (TLDR AI)
Każda warstwa dynamicznie wybiera cechy z wcześniejszych warstw (cross-attention po głębokości) zamiast prostej sumy rezydualnej. Szybsza zbieżność i niższa perplexity na C4 i LM1B.

**Analiza:** Dostęp do aktywacji wielu wcześniejszych warstw zwiększa pamięć aktywacji w treningu i komplikuje pipeline parallelism (zależności między etapami). Zysk parametrów może zostać częściowo skonsumowany przez narzut komunikacji.

Źródło: [Google Research](https://research.google/blog/supercharging-transformer-residual-connections/)

#### AWS: Systems Manager Automation dla Azure i Google Cloud, CloudWatch Omni (obserwowalność agentów na otwartych standardach), CodeDeploy z wstrzymywaniem procesów Auto Scaling (TLDR IT)
- SSM Automation wykonuje runbooki (remediacja, sprzątanie zasobów, wdrażanie agentów) w Azure i GCP z jednej płaszczyzny.
- CloudWatch Omni działa poza konsolą AWS, w IDE: auto-wykrywanie topologii, zapytania w języku naturalnym, ciągła ewaluacja śladów agentów.
- CodeDeploy może wstrzymać procesy ASG na czas wdrożenia, więc instancje nie są wymieniane w połowie aktualizacji.

**Analiza:** SSM jako orkiestrator multicloud to próba przejęcia płaszczyzny operacyjnej, która w innych chmurach wymaga szerokich uprawnień (service principal z prawami do zmian). Trzeba ograniczyć zakres i audytować w natywnych logach Azure/GCP. Wstrzymanie ASG zamyka znany race condition przy wdrożeniach in-place.

Źródło: [AWS What's New (SSM)](https://aws.amazon.com/about-aws/whats-new/2026/11/systems-manager-multicloud-automation/), [CloudWatch Omni](https://aws.amazon.com/cloudwatch/omni/), [AWS What's New (CodeDeploy)](https://aws.amazon.com/about-aws/whats-new/2026/11/aws-codedeploy-suspend-asg-processes/)

#### Tesla: AI4.5 z RAM obciętym z 32 do 24 GB, koszt układu AI4 podwojony przez ceny pamięci (TLDR)
Odświeżenie AI4.5 powstaje głównie dlatego, że Tesla potrzebuje typu pamięci, który da się kupić. Koszt AI4 wzrósł dwukrotnie z powodu cen pamięci i storage.

**Analiza:** Z prognozą HPE (niedobór DRAM/NAND do 2H 2028) to potwierdzenie, że niedobór pamięci wymusza zmiany projektowe poza DC. Konkurencja o wolumen LPDDR i HBM między edge, PC i serwerami utrzyma presję cenową na wszystkie segmenty.

Źródło: [Electrek](https://electrek.co/2026/11/23/tesla-fsd-v15-ai4-last-shot-compute-limit/)

#### NVIDIA cuDSS: bezpośrednie solwery rzadkie na GPU do symulacji inżynierskich (Own news / NVIDIA Technical Blog)
Biblioteka faktoryzacji i rozwiązywania rzadkich układów liniowych (mechanika strukturalna, CFD, obwody, sieci energetyczne) z równoległymi algorytmami na GPU, ukierunkowana na niezawodność produkcyjną. Uwaga: streszczenie z newslettera podaje niespójnie numer wersji (1.0 w tytule, 3.0 w treści).

**Analiza:** Bezpośrednie solwery to dotąd domena CPU (PARDISO, MUMPS), a przenosi je na GPU głównie faktoryzacja wielofrontalna. Zysk zależy od wypełnienia (fill-in) i pamięci GPU. Duże macierze wymagają trybu hybrydowego lub multi-GPU, czyli przepustowości NVLink lub PCIe.

Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/accelerating-direct-sparse-solvers-for-computational-engineering-with-nvidia-cudss-1-0/)

#### NVIDIA, IBM i Fermilab: dekodowanie QEC na GPU przez NVQLink i CUDA-Q (Own news / HPCwire)
Platforma współprojektowania sprzętu kwantowego z klasycznymi dekoderami korekcji błędów na GPU, połączonymi z QPU przez NVQLink. Czas iteracji projektowej spada z tygodni do minut.

**Analiza:** QEC w czasie rzeczywistym wymaga latencji dekodera w mikrosekundach. To łącze QPU–GPU, a nie FLOPS, jest ograniczeniem. Dla ośrodków HPC oznacza to nową klasę węzłów z deterministyczną siecią do QPU. Na razie to etap badawczy.

Źródło: [HPCwire](https://www.hpcwire.com/2026/11/23/inside-nvidias-plan-to-accelerate-quantum-error-correction/)

## Implikacje praktyczne

1. **Model finansowania capacity musi zakładać droższy kapitał u dostawców.** Spadek LTV do 50–60% i zamknięte okno IPO oznaczają, że neocloudy drugiego rzędu będą agresywnie szukać przedpłat albo zbankrutują z zastawionym sprzętem. W kontraktach na 2027–2029 trzeba mieć escrow przedpłat, prawo do przejęcia lub wykupu sprzętu (step-in), klauzule zmiany kontroli i gotowy plan migracji obciążeń w 60–90 dni. Ryzyko kontrahenta należy oceniać kwartalnie, a nie przy podpisaniu.
2. **Budżety sprzętowe liczone w jednostkach, nie w USD.** HPE i Dell rosną z ASP, a niedobór DRAM/NAND ma potrwać do 2H 2028. Plan na 3–5 lat powinien przyjmować spadek realnej siły nabywczej budżetu. Warto wynegocjować indeksację ceny pamięci w kontraktach ramowych i preferować architektury mniej zależne od pojemności HBM i DRAM (kwantyzacja FP4/FP8, offload KV cache do CXL/NVMe).
3. **Warstwa kompensacyjna jako standard procesu łatania.** Okno eksploatacji 33 dni, cztery luki pre-auth 9,8 w vCenter w kwartale i trzy paczki VMware w tydzień wymagają, by obok patchowania istniała warstwa eBPF/WAF/virtual patching z kontrolą zmian (podpis, tryb obserwacji, audyt). Interfejsy zarządzania (vCenter, BMC, MLOps) muszą być wyłącznie za ZTNA/PAM.
4. **Testy obciążeniowe na sprzęcie produkcyjnym.** Awaria M365 pokazała, że defekty skalujące się z liczbą rdzeni nie wychodzą na starszej flocie testowej. Przy wymianie floty na CPU 192–256 rdzeni i GPU nowej generacji staging musi obejmować docelową generację, a testy failover powinny odbywać się przy szczycie dobowym.
5. **Due diligence sankcyjne przy zakupie mocy GPU w Azji i nie tylko.** Wpis Megaspeed na Entity List pokazuje, że sprzęt u pośrednika może z dnia na dzień stracić wsparcie i legalność. Dla każdego dostawcy GPU poza USA i UE trzeba weryfikować łańcuch właścicielski i klientów końcowych oraz mieć klauzulę natychmiastowego wyjścia.

## Trend tygodnia

Warstwa finansowa rynku AI zacieśnia się szybciej, niż wskazują wyniki i komunikaty producentów. Kredytodawcy obniżyli LTV dla pożyczek pod GPU z 80% do 50–60% w ciągu jednego kwartału, a Dell oficjalnie filtruje backlog według finansowania i dostępu do energii u klienta. Jednocześnie strona podażowa raportuje rekordy: HPE +50%, Dell z prognozą 202 mld USD, TSMC zapewnia o „niezwykle silnym” popycie. Te rekordy coraz bardziej wynikają jednak z cen pamięci i ASP, a nie z przyrostu jednostek. Rozjazd między fizycznym niedoborem (pamięć do 2028 r., zaawansowane węzły) a pogarszającą się zdolnością kupujących do finansowania zakupów to dziś główne napięcie systemu. Rozładuje się albo przez konsolidację neocloudów i przejęcia sprzętu przez kredytodawców, albo przez dalszą koncentrację popytu u hyperscalerów i laboratoriów z dostępem do rynku kapitałowego. Równolegle polityka eksportowa przechodzi od reguł do egzekucji (Megaspeed), więc ryzyko regulacyjne staje się parametrem każdego kontraktu na moc GPU poza USA. W bezpieczeństwie tydzień pokazał, że tempo ataków wspieranych przez AI (minuty od rekonesansu do eksploatacji, okno 33 dni od ujawnienia) wymusza przejście od samego łatania do warstw kompensacyjnych i detekcji behawioralnej.

## To obserwować

- **Ceny H100/B200 na rynku spot i ogłoszenia sprzedaży sprzętu przez kredytodawców:** pierwszy sygnał przejmowania zastawionych GPU po obniżce LTV.
- **Zeznania Huanga przed Kongresem (1.12) i kolejne wpisy BIS na Entity List** dotyczące chmur w ASEAN (w tym ewentualny status Apex/Alibaba).
- **AWS re:Invent (30.11–4.12):** szczegóły sprzedaży Trainium poza AWS, integracja DOS z Neuron SDK i ceny Trainium 3.
- **Kolejne wyniki OEM i ODM (Supermicro, Lenovo, Quanta):** relacja wzrostu ASP do wzrostu jednostek oraz marża brutto jako miara przenoszenia kosztów pamięci.
- **Shirudo:** narzut wydajnościowy w produkcji, wspierane jądra i pierwsze wdrożenia poza Cisco i OpenAI. Do tego tempo wydań VMware dla vCenter (czy pojawi się piąta luka pre-auth).
