# Raport Zbiorczy IT – 2026-11-17

Okres: od edycji 2026-11-16 do 2026-11-17 (Europe/Warsaw). Uwzględniono też pozycje z 11–15.11, których nie było w poprzednich edycjach. Pominięte, bo opisane już wcześniej: awaria M365, forensyka checkpointów (Blackwell), SGLang Consortium, GPUStack/BroadWorks w KEV, TSMC, wycofanie Microsoftu z neocloudów, wyniki Nebius Q3. Ten raport powstał 17.11 rano, przed publikacją wyników Nvidii (17.11, 22:20 czasu warszawskiego).

## Raport technologiczny

### Technologia

#### Google Project Zero porzuca „90+30” na rzecz „Full X”: 90 → 60 → 30 dni od zgłoszenia, bez okresu soak (13.11)
Od 13.11 Project Zero publikuje szczegóły podatności X dni po zgłoszeniu do producenta, niezależnie od tego, czy jest poprawka. X wynosi 90 dni, od 2.02.2027 – 60 dni, od 3.01.2028 – 30 dni. Zniknęły 30-dniowy okres soak na wdrożenie łatki, grace period i reguła wariantów. Wyjątki: błędy wykorzystywane in-the-wild (osobna polityka Google), błędy już publiczne i prośba producenta. Uzasadnienie: z pomocą LLM łatkę można przerobić na exploit niemal natychmiast, więc soak time nie chroni. Do tego P0 znajduje teraz „kilka podatności tygodniowo” w celach, w których wcześniej znajdował „kilka rocznie”.
**Znaczenie:** Spada górna granica czasu od łatki do publicznego exploita. Proces patchowania oparty na comiesięcznym oknie zmian (Patch Tuesday + 30 dni testów) przestaje wystarczać dla komponentów badanych przez P0: przeglądarek, jąder, hypervisorów, firmware. Inne zespoły badawcze zwykle idą za P0, więc należy się liczyć z tym, że rynek przejdzie na 30 dni do 2028. Do zrobienia: kanał wdrażania łatek ≤7 dni dla warstwy brzegowej i hypervisorów, testy kanarkowe zamiast ręcznego UAT, inwentarz SBOM pozwalający w ciągu godzin ustalić, czego dotyczy publikacja.
Źródło: [Project Zero](https://projectzero.google/2026/11/disclosure-policy.html)

<hr>

#### SignDrop: publiczny exploit LPE do SYSTEM w Windows przez CLFS i Defendera, bez poprawki (10–13.11)
Nightmare Eclipse (Abdelhamid Naceri) opublikował działający PoC. Defender otwiera kontener CLFS utworzony przez użytkownika, a CLFS tworzy brakujący plik uwierzytelniający z uprawnieniami SYSTEM procesu antymalware. Exploit omija mechanizm uwierzytelniania CLFS wprowadzony w październiku 2025. Według autora działa na Windows 11 26H2 i Windows Server 2025 przy trybie uwierzytelniania CLFS 0 lub 1 i może posłużyć do wyłączenia AV/EDR (obejście tamper protection). Microsoft nie wydał poprawki. Exploitacji in-the-wild dotąd nie potwierdzono. To kolejny zero-day tego autora, według Byteiota szesnasty w siedem miesięcy.
**Znaczenie:** Publiczny PoC LPE, który przy okazji potrafi wyłączyć EDR, to gotowy element łańcucha ransomware po initial access. Jako warstwa obrony zostaje: ASR/WDAC blokujące uruchamianie nieznanych binariów z profilu użytkownika, telemetria tworzenia plików `.blf` i kontenerów CLFS w nietypowych ścieżkach oraz alert na zatrzymanie sensora EDR. Na serwerach terminalowych i VDI, gdzie wielu użytkowników ma interaktywny dostęp, ryzyko jest najwyższe.
Źródło: [BleepingComputer](https://bleepingcomputer.com/news/security/new-windows-signdrop-zero-day-gives-hackers-system-privileges), [The Hacker News](https://thehackernews.com/2026/11/new-signdrop-zero-day-exploit-turns.html)

<hr>

#### FAA: wymiana miedzi na światłowód w zdalnej lokalizacji sparaliżowała sieć łączności, ponad 11 tys. odwołanych lotów (13–14.11)
13.11 po pracach L3Harris (wymiana kabli miedzianych na światłowód, dzień wcześniej ok. 15:00) router zaczął przeciążać krytyczny system łączności FAA. Według p.o. CTO FAA dwa źródła generowały ok. 1000 połączeń co 4–5 s. Ground stop trwał ponad 4 godziny. Cirium podaje ponad 8 200 odwołanych lotów w piątek i co najmniej 3 000 w sobotę. Stabilność wróciła po odcięciu obu źródeł. Root cause (błąd instalatora czy wadliwy router) jest jeszcze ustalany. FAA wyklucza udział AI.
**Znaczenie:** To klasyczna awaria płaszczyzny sterowania przez burzę połączeń z jednego węzła brzegowego. Sieć krytyczna bez rate limitingu i CoPP na wejściu od zdalnych lokalizacji pozwala jednemu wadliwemu urządzeniu położyć całość. Wnioski dla każdej sieci OT/krytycznej: limity sesji i pps per port lub per lokalizacja, segmentacja domen awarii, okna zmian dla lokalizacji brzegowych poza godzinami szczytu (FAA zapowiada ich przegląd) oraz automatyczna izolacja źródła anomalii zamiast ręcznej diagnostyki.
Źródło: [TechCrunch](https://techcrunch.com/2026/11/14/transportation-secretary-blames-routine-equipment-upgrade-for-grounding-flights-nationwide)

<hr>

#### AntiFox (autonomiczny pentest open-source) w realnych włamaniach: od IIS do `GetSecretValue` w AWS (Arctic Wolf, 11.11)
Pierwsza ścieżka: przejęta aplikacja IIS, web shell ASPX przez `w3wp.exe`, odczyt `web.config` i connection stringów, próba logowania do produkcyjnej bazy, staging plików w katalogu uploadu. Druga ścieżka: ważne poświadczenia AWS, a potem enumeracja kilkudziesięciu usług (IAM, EC2, S3, Lambda, Secrets Manager, KMS, RDS, EKS, Bedrock). Większość wywołań kończyła się AccessDenied, ale co najmniej jedno `GetSecretValue` na produkcyjnym sekrecie się powiodło. Związek z agentem AI jest poszlakowy: szeroki, systematyczny wzorzec wywołań. AWS: usługi nie zostały przełamane, poświadczenia pochodziły z aplikacji klientów.
**Znaczenie:** Agentowe narzędzia ofensywne skracają czas od wycieku klucza do eksfiltracji sekretów. Wzorzec „dziesiątki usług, głównie AccessDenied” to sygnatura dla detekcji: alert na liczbę unikalnych `eventSource` z jednego principal w oknie 5–15 min. Kontrole: brak długoterminowych kluczy w konfiguracji aplikacji, IMDSv2 wymuszone przez SCP, polityki zasobowe Secrets Manager ograniczone do VPC endpointów.
Źródło: [The Hacker News](https://thehackernews.com/2026/11/antifox-ai-pentesting-platform-linked.html)

<hr>

#### Post-quantum w praktyce (warsztat IAB, Praga): algorytmy działają, ale rozmiar podpisów psuje protokoły
Wdrażający ML-DSA i SLH-DSA nie zgłosili problemów kryptograficznych. Problemem są rozmiary: podpis ML-DSA-65 ma 3 309 B wobec 64 B dla Ed25519. Przez to handshake'i przekraczają początkowe okno przeciążenia, podpisane żądania przekraczają limity URL, a obrazy firmware'u nie mieszczą się w pamięci urządzeń. Współprzewodniczący podkreślają, że migracja kończy się dopiero, gdy weryfikatorzy przestają akceptować klasyczne root CA. Cisco spodziewa się, że część certyfikatów routerów będzie klasyczna do ok. 2035. Nowe źródło rozbieżności to agenci AI, którzy piszą własne formaty kryptograficzne zamiast wołać biblioteki.
**Znaczenie:** Budżet bajtów to realny problem projektowy: łańcuch certyfikatów PQ w TLS 1.3 dokłada kilkanaście KB, czyli dodatkowe RTT przy initcwnd=10 i fragmentację w QUIC. Uwaga na middleboxy z limitami rozmiaru ClientHello/Certificate, na MTU w tunelach IPsec/IKEv2 z PQ i na urządzenia OT z małym flashem. Inwentaryzację urządzeń bez możliwości aktualizacji do PQ warto zrobić teraz, bo to one wyznaczą datę końca migracji.
Źródło: [IETF Blog](https://www.ietf.org/blog/iab-pq-workshop-impressions/)

<hr>

#### Cloudflare Application Services for Private Origins – GA
Publiczny ruch trafia do aplikacji w sieciach prywatnych bez wystawiania IP origin i bez otwierania portów przychodzących. Prywatne IP w proxowanych rekordach DNS są wykrywane automatycznie. Obsługiwane są aplikacje HTTP, prywatne originy TCP/UDP (przez Spectrum) i Workers. Istniejące polityki WAF, bot management i cache stosują się automatycznie. Wymagania: Cloudflare One, Unified Routing (chyba że używany jest Tunnel) i trasy powrotne dla 100.64.0.0/12 oraz 2606:4700:cf1:5000::/64. Ruch private-to-private ma wejść w closed beta na początku 2027.
**Znaczenie:** To odpowiednik CloudFront VPC Origins i Azure Front Door Premium z Private Link. Eliminuje klasę ataków omijających WAF przez bezpośredni adres origin. Koszt: głębsze uzależnienie od Cloudflare One jako warstwy łączności (WAN/Mesh), a przy awarii edge Cloudflare nie ma ścieżki awaryjnej przez publiczny origin. Należy też sprawdzić kolizje 100.64.0.0/12 z CGNAT lub adresacją Tailscale w sieci własnej.
Źródło: [InfoQ](https://www.infoq.com/news/2026/11/cloudflare-private-origins/)

<hr>

#### AWS Secrets Manager: replikacja sekretów między kontami
Replikacja multi-Region rozszerzona na granice kont. Sekret ma jednego właściciela i jedną rotację, a kopie w innych kontach są synchronizowane bez współdzielonych kluczy KMS i polityk cross-account.
**Znaczenie:** Upraszcza DR w modelu „konto-na-workload” i wyposażenie kont odtworzeniowych (isolated recovery account). Jednocześnie replikacja przenosi kompromitację: przejęty sekret w koncie źródłowym propaguje się do kont DR. Konta odtworzeniowe dla scenariusza ransomware powinny mieć osobny, ręcznie zatwierdzany proces importu sekretów.
Źródło: [AWS What's New](https://aws.amazon.com/about-aws/whats-new/2026/11/secrets-manager-cross-account/)

<hr>

### Biznes

#### Microsoft zmienia ceny chmury od 1.01.2027: Indie +15,7%, Nowa Zelandia +10%, obniżki w Chinach, Australii i Norwegii
Microsoft wyrównuje lokalne ceny chmury do kursów walut wobec USD. Indie: +15,7% na usługi chmurowe i do +20% na część produktów on-prem. Nowa Zelandia: +10%. Chiny (21Vianet): do −8,5%. Australia: −4,5%. Norwegia: −5,9%.
**Znaczenie:** Ceny chmury hyperscalerów w walutach lokalnych są de facto indeksowane do USD i korygowane skokowo. W kontraktach EA/MCA w PLN/EUR warto negocjować zamrożenie kursu na okres umowy albo rozliczać w USD z własnym hedgingiem. Przy porównaniu TCO z on-prem ryzyko walutowe trzeba ujmować jawnie.
Źródło: [The Register](https://www.theregister.com/off-prem/2026/11/15/microsoft-lifts-indian-cloud-prices-by-15-percent-cuts-costs-in-china/5309993)

<hr>

#### Pozew o zmowę cenową DRAM przeciw Samsungowi, SK hynix i Micronowi idzie dalej
Sąd federalny w USA w całości oddalił wniosek o odrzucenie pozwu. Uznał za prawdopodobny zarzut, że producenci ograniczali podaż konwencjonalnego DRAM i podnieśli ceny o 697% w 18 miesięcy („hawks flying in formation”).
**Znaczenie:** Postępowanie nie zmieni cen w horyzoncie 2027, ale discovery może ujawnić dane o alokacjach i podnieść presję regulacyjną (USA, a potem pewnie UE i Chiny) na dyscyplinę podażową. Dla planowania pojemności ceny DRAM/HBM to nadal największy czynnik kosztu serwera. Według zapowiedzi analityków koszt pamięci obniża też marżę Nvidii.
Źródło: [Tom's Hardware](https://www.tomshardware.com/tech-industry/big-tech/judge-denies-motion-to-dismiss-dram-price-fixing-lawsuit-against-micron-samsung-and-sk-hynix-says-chipmakers-acted-like-hawks-flying-in-formation-finds-evidence-of-dram-collusion-plausible-after-500-percent-price-surge)

<hr>

#### Applied Materials: przychód +53% r/r do 10,41 mld USD, systemy półprzewodnikowe +64% do 7,99 mld (12.11)
Applied Materials podał też plan centrum EPIC za 5 mld USD do wspólnych prac R&D.
**Znaczenie:** Rekordowe zamówienia na sprzęt fab to wskaźnik wyprzedzający podaży logiki i pamięci o 12–24 miesiące. Ulga dla cen DRAM/HBM może przyjść najwcześniej w 2028, gdy nowe moce wejdą do produkcji.
Źródło: [SiliconANGLE](https://siliconangle.com/2026/11/12/applied-materials-posts-an-impressive-53-jump-in-revenue-as-it-beats-expectations-again)

## Raport AI-ML

### Technologia

#### Nvidia Rubin HPC: osobny GPU pod FP64, ponad 50 tys. rdzeni CUDA, 3× więcej jednostek wektorowych niż Blackwell, dostępny w 2028 (SC26, 16.11)
Nvidia zapowiedziała GPU przeznaczony do HPC. Nie podała wydajności FP64. HPCwire szacuje przy liniowym skalowaniu ok. 100 TFLOPS FP64. Dla porównania: Rubin (AI) ma 33 TFLOPS natywnego FP64 (H100/H200: 34/67 TFLOPS z Tensor Core), a AMD MI430X (2027) ma 288 TFLOPS sprzętowego FP64. Nvidia przyznaje, że emulacja Ozaki nie zastępuje FP64 we wszystkich obciążeniach. AMD ostrzega, że w niektórych przypadkach daje błędne wyniki.
**Analiza:** Rozwidlenie linii produktowych (AI z FP4/FP8 i HPC z FP64) oznacza, że jeden zakup GPU nie obsłuży już obu typów centrów. Dla ośrodków HPC kupujących w latach 2026–27 jedyną opcją z dużym natywnym FP64 jest AMD MI430X (Alice Recoque, Discovery). Rubin HPC zmniejsza to lock-in dopiero od 2028. Do planowania: kodów CFD i klimatycznych nie migrować na emulację Ozaki bez walidacji numerycznej na referencyjnych przypadkach. Hopper H100/H200 zachowują wartość rezydualną jako akceleratory FP64.
Źródło: [HPCwire](https://www.hpcwire.com/2026/11/16/nvidia-recommits-to-fp64-with-a-new-gpu-the-rubin-hpc/)

<hr>

#### MLPerf Training v6.1: debiut Rubin i TPUv7 Ironwood, nowy benchmark post-treningu GRPO, 37% systemów w chmurze (16.11)
Wyniki zgłosiło 24 organizacje (w tym Nebius, CoreWeave, Lambda, Crusoe – debiut, Firmus, Oracle, Azure, AMD, Google, tiny corp): 153 systemy, 250 wyników. Nowe benchmarki: DLRMv4-HSTU (rekomendacje z attention, dataset Yambda) oraz Qwen3.5-397B z GRPO (RL post-training z tool-callingiem w zadaniach SWE). Systemy chmurowe stanowiły 37% (w v6.0: 29%). 12 systemów miało co najmniej 1 000 akceleratorów, największy – 10 240. Nvidia deklaruje do 2× przewagi Vera Rubin NVL72 nad Blackwell Ultra.
**Analiza:** Benchmark GRPO po raz pierwszy mierzy obciążenie, które dziś dominuje w budżetach compute laboratoriów: pętlę RL z generowaniem rolloutów i refitem wag. Kluczowe są w niej throughput inferencji i synchronizacja wag, a nie sam GEMM. Neocloudy coraz częściej publikują wyniki MLPerf, bo kontrahenci (Microsoft, Meta) i kredytodawcy traktują je jako dowód zdolności operacyjnej. Przy wyborze dostawcy warto żądać wyników w podziale na skalę (np. 512 vs 4 096 GPU), bo w klastrach >1 000 GPU o efektywności decyduje fabric.
Źródło: [MLCommons via HPCwire](https://www.hpcwire.com/off-the-wire/mlcommons-releases-mlperf-training-v6-1-results-spotlighting-recent-advances-in-ai-training/), [NVIDIA Blog](https://developer.nvidia.com/blog/nvidia-vera-rubin-nvl72-achieves-up-to-2x-blackwell-ultra-performance-in-mlperf-training-v6-1/)

<hr>

#### MLPerf Endpoints v1.0: inferencja agentowa mierzona jako throughput na kW przy minimalnej interaktywności; Vera Rubin ~10× GB300 na Kimi K3 (wyniki wstępne)
Nowy benchmark mierzy pojemność serwisu przy minimalnej interaktywności E2E na użytkownika (tokeny/s/użytkownika), TTFT P90, system TPS i moc roboczą (kW), przy równej współbieżności. Poprawność wywołań narzędzi i intencji jest weryfikowana podczas przebiegu. Kimi K3: Vera Rubin (32 GPU, 100,9 kW) – 41 414 TPS, 50,4 tok/s/użytkownika, TTFT P90 4,67 s; GB300 (48 GPU, 60 kW) – 2 210 TPS, 16,8 tok/s/użytkownika. Daje to ok. 10× throughput/kW. DeepSeek-V4.1-Flash: ok. 9×. Qwen3.6-35B-A3B: ok. 5×. Stos: Dynamo (routing świadomy prefiksów), disaggregated prefill/decode, MegaMoE, MTP. Wyniki Rubin czekają na weryfikację MLCommons. Nvidia nie porównuje się z innymi dostawcami.
**Analiza:** Metryka „tokeny na kW przy SLO interaktywności” to właściwa jednostka planowania pojemności, gdy limitem jest moc DC, a nie liczba GPU. Uwaga: konfiguracje porównania są niesymetryczne (inna liczba GPU i inne kW), a TTFT P90 rzędu 4,7 s przy 1 024 równoległych sesjach pokazuje, że prefill przy długim kontekście agentów pozostaje wąskim gardłem. Większość przewagi wynika z oprogramowania (prefix caching, disaggregation), więc można ją częściowo odtworzyć na starszym sprzęcie przez vLLM/SGLang z LMCache/NIXL. Przy dużym udziale cache hit to obniża realny wzrost dla starszych flot.
Źródło: [NVIDIA Technical Blog](https://developer.nvidia.com/blog/nvidia-vera-rubin-nvl72-leads-agentic-inference-performance-on-mlperf-endpoints-v1-0/)

<hr>

#### Thinking Machines: pretrening i RL w NVFP4 end-to-end na GB300 – ok. 2% straty efektywności obliczeniowej, +8% (pretrening) i +20% (RL) czasu zegarowego wobec MXFP8 (13.11)
Model Inkling 2 Small trenowano 68 h na GB300 NVL72. Forward warstw MoE: GEMM w NVFP4 (bloki 1×16 E4M3 + skala FP32, wybór skali „4/6” dla aktywacji, skale per ekspert/per token). Backward: MXFP8 ze stochastycznym zaokrąglaniem, gradienty w MXFP8. Ostatnia 1/6 warstw zostaje w MXFP8. Komunikacja: dispatch w NVFP4, combine w 8,25-bitowym S2F6 z losową transformacją Hadamarda 32-wymiarową (fuzja w epilogu GEMM). Pełne NVFP4 w backwardzie powodowało duże straty. W RL nie widać różnicy w nagrodzie.
**Analiza:** To pierwszy publiczny, kompletny przepis na produkcyjne 4-bit trening MoE z wykorzystaniem 15 PFLOPS NVFP4 (6× BF16) na GB300. Kluczowe jest skompresowanie komunikacji all-to-all, a nie tylko GEMM: przy MoE na NVL72 to ona ogranicza skalowanie. Przewaga Blackwell/Rubin nad Hopper i MI3xx rośnie, bo dostęp do NVFP4 z odpowiednią granulacją skal jest dziś sprzętowo specyficzny dla Nvidii (lock-in na poziomie przepisu numerycznego). Przy RL, gdzie dominuje generowanie rolloutów, zysk (+20%) jest większy niż w pretreningu.
Źródło: [Thinking Machines Lab](https://thinkingmachines.ai/blog/end-to-end-4-bit-training/)

<hr>

#### TOP500 (68. edycja, 16.11): bez zmian w top 10, tylko 27 nowych systemów – najmniej w historii listy
Kolejność: LineShine (2,198 EF), El Capitan (1,809), Frontier (1,353), Aurora (1,012), JUPITER Booster (1,000). Nowe systemy to 5,4% listy. Łączna wydajność wzrosła o 4,4% do 19,6 EF, a próg wejścia (miejsce 500) to 2,6 PF (+7%). Największy debiut to Hangang (KISTI, HPE Cray EX z GH200): 342,6 PF, miejsce 12. Chiny nie zgłosiły Sugon 8000 (Zhengzhou). Według Epoch AI kwartalne przyrosty globalnego compute AI rosną ok. 2,5× rocznie (łącznie ponad 15 mln ekwiwalentów H100), a pięciu hyperscalerów z USA miało w Q4 2025 ok. 71% tej mocy.
**Analiza:** TOP500 przestaje opisywać rynek. Nowy sprzęt trafia do AI factories, które nie zgłaszają wyników HPL, a Chiny ukrywają część exascale. Dla ośrodków HPC oznacza to konkurencję o GPU, HBM i moc przyłączeniową z hyperscalerami o znacznie większym budżecie. Benchmarkiem decyzji zakupowych stają się MLPerf, HPL-MxP i wydajność aplikacji, a nie miejsce na liście.
Źródło: [HPCwire](https://www.hpcwire.com/2026/11/16/no-change-at-the-top-of-top500-lineshine-still-no-1/), [TOP500](https://www.hpcwire.com/off-the-wire/top500-leadership-holds-steady-as-new-system-turnover-falls-to-a-historic-low/)

<hr>

### Biznes

#### Schmidt Sciences: 334 mln USD dla NCSA na system Garnet; Berkeley/NERSC dołączają do Schmidt Science Computing Network (16.11)
SSCN to rozproszona sieć systemów należących do instytucji, finansowana filantropijnie. NCSA zbuduje Garnet w NPCF (dawna hala Blue Waters). Berkeley Science AI Platform rozbuduje compute i storage Doudny (NERSC, Dell ORv3 DLC + Vera Rubin). To pierwsza współpraca akademicko-filantropijna typu public-private w ekosystemie DOE.
**Analiza:** Finansowanie filantropijne w skali kilkuset mln USD zaczyna uzupełniać budżety DOE/NSF w compute dla nauki. Wydłuża to kolejki zamówień Vera Rubin u Della i HPE w 2027. Ryzyko: brak stałego budżetu operacyjnego (energia, personel) po wygaśnięciu grantu, co jest typowym problemem systemów finansowanych jednorazowo.
Źródło: [HPCwire / NCSA](https://www.hpcwire.com/off-the-wire/ncsa-receives-334m-grant-for-new-garnet-supercomputer/), [HPCwire / Berkeley Lab](https://www.hpcwire.com/off-the-wire/uc-berkeley-and-berkeley-lab-join-schmidt-science-computing-network/)

## euro neocloud

### Nscale / Verda (konsorcjum AI Gigafactory Northern)

#### Nokia, Telia, Nscale, Verda i Skeleton tworzą AIGF Northern pod przetarg EuroHPC na AI Gigafactory (16.11)
**Fakty:** Konsorcjum złoży ofertę na budowę i operowanie AI Gigafactory w ramach przetargu EuroHPC JU. Telia (z Telia Cygate) odpowiada za łączność, operacje i suwerenną chmurę. Nokia dostarcza sieć między lokalizacjami. Nscale i Verda dają platformę chmurową i infrastrukturę AI do treningu i inferencji. Skeleton dostarcza systemy wysokiej mocy (stabilizacja poboru, mniejsze obciążenie sieci). Partnerami badawczymi są VTT i KTH. Lokalizacje: Oulu (FI), region Tallinna/Iru (EE), Valmiera (LV) i niewybrana lokalizacja w Szwecji. Nie podano mocy (MW), liczby GPU ani wartości inwestycji. Według Converge Digest termin dla Lot 2 to 3.12.2026.

**Ocena analityczna**
- *Wykryte sygnały ostrzegawcze:* Nscale wchodzi w kolejne zobowiązanie infrastrukturalne, mając lukę finansową ok. 30 mld USD pod kontrakt Anthropic i brak wiążącego finansowania na 2027 (zob. 14.11). Federacja czterech lokalizacji w czterech krajach komplikuje SLA, synchronizację danych i treningi rozproszone, bo duże joby wymagają jednej lokalizacji z niskim opóźnieniem. Nie podano parametrów technicznych, co jest typowe dla ofert przetargowych na wczesnym etapie.
- *Ocena ryzyka:* **Średnie** (dla projektu), **Wysokie** (dla Nscale – bez zmian). Udział Telii i Nokii jako operatorów infrastruktury oraz współfinansowanie publiczne (InvestAI) obniżają ryzyko projektu wobec samodzielnego neocloudu. Dla Nscale to dywersyfikacja przychodu w stronę UE, ale też dodatkowe zapotrzebowanie na capex przy ograniczonym dostępie do długu. Verda (ex-DataCrunch) zyskuje skalę, której sama nie osiągnęłaby w przetargu.
- *Linki:* [EuropaWire (komunikat Telii)](https://news.europawire.eu/telia-and-european-technology-partners-form-ai-gigafactory-northern-to-expand-sovereign-ai-capacity/eu-press-release/2026/11/16/13/42/27/188028/), [AIwire](https://www.hpcwire.com/aiwire/2026/11/16/european-tech-leaders-form-ai-gigafactory-northern-consortium/), [Converge Digest](https://convergedigest.com/nokia-nscale-ai-gigafactory-northern-eurohpc-bid/)

## AI for networking

#### Astera Labs + Hyve (TD SYNNEX): scale-up przez PCIe/Scorpio w jednym hopie zamiast dwóch przełączników – do +89% tokenów/s na 8 GPU, pod do 160 akceleratorów, chłodzenie powietrzem (16.11)
Platforma Hyve Apex z linią Orion: Compute Board (8 akceleratorów PCIe pod jednym przełącznikiem fabric), Fabric Tray (wspólny scale-up i pooling pamięci) i Link Board (z tacki do skali racka). Według testów Astera pojedynczy przełącznik Scorpio X 320-lane dał do 89% wyższy TPS niż dwa przełączniki w klasycznej konfiguracji. Architektura jest agnostyczna wobec akceleratorów i CPU, a in-network compute i Hypercast obsługują kolektywy.
**Analiza:** To alternatywa dla NVLink dla neocloudów i enterprise, które chcą inferencji na kartach PCIe (różni dostawcy akceleratorów) bez DLC. Zysk wynika z eliminacji hopu i współdzielenia uplinków przy all-reduce/all-to-all w TP=8. Wartość +89% pochodzi z testów producenta i dotyczy topologii referencyjnej z dwoma przełącznikami. Ograniczenia: przepustowość PCIe Gen6 per GPU jest wyraźnie niższa niż NVLink 5/6, więc dla dużych MoE z EP na wiele węzłów ten fabric nie zastąpi NVL72. Lock-in przenosi się z dostawcy GPU na dostawcę przełącznika fabric (Astera).
Źródło: [HPCwire / Astera Labs](https://www.hpcwire.com/off-the-wire/astera-labs-and-hyve-solutions-partner-to-bring-ai-inference-capacity-to-customers-faster/)

<hr>

#### Shopify Cyber Chaos Monkey: agent ofensywny na produkcji z allowlistą egress, proxy podmieniającym poświadczenia i agentem-sędzią z innego modelu
Agent atakuje żywe systemy Shopify, korzystając z poświadczeń testowych, kodu źródłowego i historii zgłoszeń. Wysyła sondy, czyta logi produkcyjne i porównuje wyniki z kodem, żeby dopracować exploit. Według Shopify dziesiątki tysięcy przebiegów dały dziesiątki poważnych znalezisk i ani jednej awarii. Ograniczenia: efemeryczne poświadczenia tylko do odczytu, proxy podmieniające poświadczenia (agent nie widzi sekretów), allowlista egress oraz osobny agent-sędzia na innym modelu. Sędzia przerywa przebieg, gdy agent łamie rules of engagement albo przestaje odpowiadać.
**Analiza:** To wzorzec referencyjny dla izolacji sieciowej agentów (nie tylko ofensywnych): egress przez proxy z allowlistą FQDN i wstrzykiwaniem poświadczeń na poziomie proxy, a nie w kontekście modelu. Uwaga na wcześniej opisany failure mode (13.11: obejście egress przez spoofing DNS i domain fronting na współdzielonych IP CDN). Allowlista musi działać na SNI, Host i certyfikacie, a resolver DNS musi być kontrolowany. Agent-sędzia na innym modelu zmniejsza ryzyko wspólnych podatności na prompt injection, ale podwaja koszt inferencji przebiegu.
Źródło: [Shopify Engineering](https://shopify.engineering/cyber-chaos-monkey)

## Ryzyka rynku AI/GPU

#### Nvidia raportuje Q3 FY2027 dziś (17.11, 22:20 CET) – pierwszy raport bez Huanga; wytyczne 108 mld USD ±2%, marża 74%
Guidance (Kress, 26.08): przychód 108,0 mld USD ±2% bez sprzedaży DC do Chin, marża brutto 74,0% ±50 pb (75,0% w Q2), Rubin ok. 20% przychodu DC. Konsensus (LSEG) ok. 109,2 mld. Kress łączy funkcje CEO i CFO, więc podpisuje oba certyfikaty SOX. 18.11 przewodniczący trzech komisji Izby Reprezentantów chcą zamkniętego briefingu z Huangiem. Kress wcześniej mówiła o ok. 70% wzrostu w FY2028, ograniczonym podażą. Rynek przed raportem był słabszy: CoreWeave i Nebius prowadziły spadki neocloudów 16.11. Petycja o przywrócenie Huanga ma ponad 160 tys. podpisów.
**Analiza:** Ważniejsze od przychodu będą trzy elementy: (1) wytyczne na Q4 i marża (koszt HBM/DRAM; część analityków zakłada ok. 71,5% wobec konsensusu 72,3%), (2) pozycje pozabilansowe: gwarancje, backstopy i zobowiązania zakupowe po zaostrzeniu kryteriów (zob. 16.11) oraz ich ujęcie w 10-Q, (3) komentarz o alokacji Rubin. Spadek marży przy rosnącym przychodzie oznaczałby, że koszt pamięci przenosi się na łańcuch. Ograniczenie gwarancji uderzy najpierw w neocloudy drugiej linii.
Źródło: [Tech Times](https://www.techtimes.com/articles/330779/20261116/nvidia-earnings-tuesday-kress-guided-numbers-cfo-now-reports-them-interim-ceo.htm), [Seeking Alpha](https://seekingalpha.com/news/4673004-coreweave-nebius-lead-neocloud-decline-as-ai-stocks-prepare-for-nvidia-earnings), [CNBC](https://www.cnbc.com/2026/11/16/nvidia-jensen-huang-return-tech.html)

<hr>

#### Syndykacja kredytu 488 mln USD na DC w Malezji (offtake ByteDance/BytePlus) wstrzymana – rzadka pauza na azjatyckim rynku kredytów DC (Bloomberg, 16.11)
Według Bloomberga syndykacja kredytu 488 mln USD dla projektu DC w Malezji zarządzanego przez Actis została opóźniona w trakcie oferowania innym bankom. Z dokumentów AIIB wynika, że chodzi o Project Mercury (Epoch Digital). Pierwsza faza 60 MW jest w pełni zakontraktowana przez BytePlus. Łączne zapotrzebowanie wynosi ok. 870 mln USD, a AIIB ma wziąć do 125 mln w syndykacie. Bloomberg wiąże opóźnienie ze złożonością finansowania rozbudowy AI.
**Analiza:** Nawet projekt z pełnym offtake od dużego najemcy nie domyka syndykacji. Ryzyko kredytowe przenosi się więc z wykonawcy na najemcę. Prawdopodobny dodatkowy czynnik (to wniosek, a nie fakt z artykułu) to zbliżająca się reguła BIS o zdalnym dostępie chińskich firm do GPU w siedmiu krajach, w tym w Malezji (zob. 15.11). Banki mogą wyceniać ryzyko, że najemca nie będzie mógł legalnie użyć mocy. To sygnał, że kontrakty z chińskimi hyperscalerami poza Chinami przestają być aktywem bankowalnym.
Źródło: [Bloomberg](https://bloomberg.com/news/articles/2026-11-16/asian-data-center-loan-syndication-is-delayed-in-rare-deal-pause), [AIIB – Project Mercury](https://www.aiib.org/en/projects/details/2026/approved/malaysia-project-mercury-malaysia-green-data-center-financing.html)

## Newsletters summary

#### QPU jako zasób w Slurm: IQM (wtyczka HRES, Slurm 26.11) i IBM/Fermilab/NVIDIA (QRMI); kontrolery FPGA z NVQLink
IQM wspiera inicjatywę Nvidii, w której QPU są opisane jako zasoby planowane w Slurmie, a MQSC Orchestrator łączy HRES z QDMI. IBM, Fermilab i NVIDIA planują platformę co-design (DGX B200 + IBM Nighthawk) z otwartym interfejsem QRMI do Slurma. Altera (Agilex 9 Direct RF) z OQC i NVIDIA pokazuje kontroler kwantowy połączony z GPU przez NVQLink (styczeń 2027). Quantum Machines uruchomił dekodery CUDA-Q QEC na OPX1000 z mikrosekundowymi callbackami (na razie przez CPU, GPU dispatch będzie później).
**Analiza:** Integracja z harmonogramem zadań oznacza, że QPU w ośrodkach HPC staje się kolejnym zasobem w kolejce, jak GPU. To wymaga polityk fair-share dla zasobu o czasie kalibracji i niskiej przepustowości. Fakt, że NVQLink staje się standardem łącza kontroler-GPU, wiąże stos kwantowo-klasyczny z Nvidią na etapie, w którym wybór technologii QPU jest jeszcze otwarty.
Źródło: [HPCwire / IQM](https://www.hpcwire.com/off-the-wire/iqm-supports-nvidias-work-to-make-quantum-computers-schedulable-in-slurm/), [HPCwire / IBM-Fermilab](https://www.hpcwire.com/off-the-wire/ibm-fermilab-and-nvidia-plan-quantum-computing-co-design-platform/), [HPCwire / Quantum Machines](https://www.hpcwire.com/off-the-wire/quantum-machines-delivers-1st-results-from-cuda-q-qec-real-time-decoders/)

<hr>

#### Zasilanie DC AI: koszt energii skorygowany o szybkość budowy – mikrosieci DC (PV + magazyn) wygrywają w 85% lokalizacji
Praca porównuje sieć, PV z magazynem sprzężone AC i DC oraz wyspowy gaz. Dodaje metrykę „speed-adjusted cost of energy”, która wycenia compute utracone przez wolną budowę. Sprzężenie DC obniża koszt energii w cyklu życia o 18%, bo eliminuje falowniki. Mikrosieci DC rzadko mają najniższy koszt za kWh, ale powstają najszybciej, więc wygrywają w 85% lokalizacji (w 55%, jeśli skrócą się terminy dostaw turbin gazowych).
**Analiza:** Przy kolejkach przyłączeniowych 3–7 lat i turbinach z dostawą w 2029+ czas uruchomienia waży w TCO więcej niż cena kWh. Sprzężenie DC dobrze pasuje do zasilania 800 VDC w rackach (Kyber/Rubin Ultra). Ryzyko: dostępność mocy zależy od pogody i pojemności magazynu, więc SLA wymaga albo nadwymiarowania BESS, albo gazowego backupu. To przesuwa projekt w stronę hybrydy.
Źródło: [arXiv 2611.17098](https://arxiv.org/abs/2611.17098)

<hr>

#### Anthropic: „Building always-on agents” – pięć prymitywów: event inbox, trwałe timery, pamięć, system uprawnień, outbox
Agenci działający stale w tle opierają się na pętli z kolejką zdarzeń, trwałymi timerami, pamięcią między sesjami, warstwą uprawnień i wyjściem przez outbox.
**Analiza:** To architektura bliższa systemom kolejkowym (durable execution, w stylu Temporal) niż aplikacjom czatowym. W infrastrukturze oznacza długotrwałe sesje, stan poza kontekstem modelu i outbox jako punkt kontroli akcji zewnętrznych, czyli miejsce na polityki i audyt. Failure modes: duplikacja akcji przy ponowieniu (wymagana idempotencja outboxa) i dryf pamięci.
Źródło: [claude.com](https://claude.com/resources/articles/building-always-on-agents)

<hr>

#### OpenAI Codex Security (CLI/SDK) i Daybreak Defense Factory: od skanu do łatki w pipeline
Codex Security skanuje repozytoria, ścieżki i diffy, weryfikuje znaleziska, generuje łatki i eksportuje wyniki do SARIF/JSON/CSV oraz do Jira/Linear/GitHub. Działa jako GitHub Action z limitem kosztu na skan i obsługuje też Bedrock, OpenRouter i Fireworks. Daybreak Defense Factory przyjmuje znaleziska z narzędzi bezpieczeństwa i generuje poprawki.
**Analiza:** Razem ze skróceniem okien disclosure przez Project Zero automatyczne łatanie staje się warunkiem nadążania za wolumenem CVE. Ryzyko: łatki generowane przez LLM bez przeglądu wprowadzają regresje i (zob. warsztat IAB) własne implementacje kryptografii. Konieczne są reguły CI blokujące zmiany w modułach krypto i auth bez przeglądu człowieka.
Źródło: [GitHub – openai/codex-security](https://github.com/openai/codex-security/)

<hr>

#### Quark: pełny stos treningowy (pretrening, rozszerzenie kontekstu, RL, ewaluacja) dla modelu 165M za ok. 4 200 USD (ok. 1 300 H100-h)
Model 165M ma najlepszą średnią benchmarków wśród otwartych modeli tej wielkości przy podobnym budżecie tokenów. Projekt służył przetestowaniu całego pipeline'u przed dużym treningiem.
**Analiza:** Praktyka „przetestuj cały stos na małej skali” ogranicza kosztowne awarie na tysiącach GPU (błędy w danych, w RL, w ewaluacji). Dla planowania pojemności warto rezerwować ok. 1–2% budżetu GPU na takie przebiegi walidacyjne.
Źródło: [empero.org](https://empero.org/writing/quark)

## Implikacje praktyczne

1. **Patchowanie w cyklu ≤7 dni dla warstw brzegowych i hypervisorów (2026–2028).** Project Zero skraca okno do 30 dni do 2028 i znosi soak time, a exploit z łatki powstaje w godzinach (AntiFox, SignDrop). Architektura musi obsługiwać łatanie kroczące bez przestoju: niemutowalne obrazy, blue/green na brzegu, kanarki. Ryzyko: systemy legacy bez możliwości szybkiej aktualizacji stają się kandydatami do izolacji lub wymiany.
2. **Rozdzielenie floty HPC (FP64) i AI (FP4/FP8).** Rubin HPC dopiero w 2028, a MI430X w 2027. Zakupy HPC na lata 2026–27 powinny uwzględniać AMD albo utrzymanie Hopperów, a nie liczyć na emulację Ozaki bez walidacji. Ryzyko: wspólny klaster „AI+HPC” na Rubin daje słabe FP64 i konkurencję o kolejkę.
3. **Planowanie pojemności inferencji w tokenach/kW przy SLO interaktywności.** MLPerf Endpoints daje jednostkę porównawczą. Kontrakty z neocloudami i budżety mocy DC warto formułować w tej metryce, z wymaganiem disaggregated serving i prefix cache, które dają większość zysku niezależnie od generacji GPU.
4. **Ryzyko kontrahenta przed ceną GPU-h.** Pauza syndykacji przy pełnym offtake (Malezja), zaostrzenie backstopów Nvidii i wyniki Nvidii w jednym tygodniu pokazują, że dostępność mocy w latach 2027–28 zależy od bankowalności najemcy i operatora. W umowach z neocloudami: escrow lub step-in, prawo do przeniesienia obciążeń i klauzule na wypadek nieuruchomienia DC.
5. **PQC jako problem rozmiaru pakietów i urządzeń.** Plan migracji do ML-DSA musi uwzględniać MTU, initcwnd, limity middleboxów i pamięć firmware. Inwentaryzacja urządzeń bez ścieżki PQ (routery z certyfikatami klasycznymi do ok. 2035) wyznacza termin wygaszenia klasycznych root CA.

## Trend tygodnia

Infrastruktura AI rozdziela się na wyspecjalizowane ścieżki. Nvidia po raz pierwszy wydziela GPU dla FP64 (Rubin HPC), a trening przechodzi na 4-bitową numerykę end-to-end (NVFP4 w pretreningu i RL), więc architektura „jeden akcelerator do wszystkiego” przestaje mieć sens ekonomiczny. Benchmarki idą za obciążeniami: MLPerf mierzy już post-trening GRPO i inferencję agentową w tokenach na kW, a TOP500 z rekordowo niską rotacją przestaje opisywać miejsce, w którym faktycznie instalowany jest nowy sprzęt (71% compute AI u pięciu hyperscalerów). Po stronie bezpieczeństwa AI skraca cykl od łatki do exploita do godzin, a Project Zero formalnie dostosowuje do tego politykę disclosure. Atakujący używają agentowych narzędzi pentestowych w realnych włamaniach do chmury, a obrońcy (Shopify) budują agentów ofensywnych z izolacją egress i agentem-sędzią. Po stronie finansowej rynek wchodzi w tydzień weryfikacji: pierwszy raport Nvidii bez Huanga, pauza syndykacji nawet przy pełnym offtake i europejskie neocloudy szukające publicznego współfinansowania (AIGF Northern). Wspólny mianownik: dostęp do mocy obliczeniowej zależy coraz bardziej od jakości kontrahenta, energii i oprogramowania serwującego, a coraz mniej od samej generacji GPU.

## To obserwować

- **Wyniki Nvidii Q3 FY2027 (17.11, 22:20 CET):** marża brutto wobec 74% i wytyczne na Q4, ujawnienia w 10-Q dotyczące gwarancji/backstopów i zobowiązań zakupowych, alokacja Rubin.
- **Status SignDrop:** poprawka poza cyklem albo w Patch Tuesday grudnia; pierwsze potwierdzenia exploitacji in-the-wild i ewentualny wpis do CISA KEV.
- **Przetarg EuroHPC AI Gigafactories (Lot 2, 3.12.2026):** lista konsorcjów (AIGF Northern i inne) z deklarowanymi MW i GPU oraz udział finansowania publicznego.
- **Weryfikacja MLCommons wyników Vera Rubin w Endpoints v1.0** i pierwsze zgłoszenia konkurencji (AMD MI455X, TPUv7) w tym benchmarku.
- **Syndykacja Project Mercury i podobnych projektów z chińskimi najemcami w ASEAN** w kontekście finalnej reguły BIS o zdalnym dostępie do GPU.
