# Raport Zbiorczy IT – 2026-11-11

**Okno przyrostowe:** od raportu 2026-11-10 (zdarzenia z 5.11 ujęte tylko wtedy, gdy nie było ich w rejestrze). Pominięta sekcja: AI for networking (brak nowych, zweryfikowanych pozycji w oknie; ostatnie istotne materiały to SRv6 na OCP Summit, 15.10). Źródła: newslettery z etykiety NEWSY (TLDR, TLDR AI/IT/InfoSec, Own news/NVIDIA blog), RSS NVIDIA Developer Blog i HPCwire, WWW. SemiAnalysis RSS zwraca nieaktualny cache (ostatni wpis 09/2025), AIwire RSS – 403.

## Raport technologiczny

### Technologia

#### Qubole: przejęte poświadczenia dostawcy użyte do AssumeRole w kontach AWS klientów (14–26.10)
Atakujący użył kluczy IAM z konta AWS Qubole (805246085872), żeby przejąć role integracyjne, które klienci utworzyli dla tego dostawcy. Jedynym zabezpieczeniem był External ID, który CloudTrail zapisuje otwartym tekstem. Po przejęciu roli atakujący pobierał sekrety z Secrets Manager, parametry SSM i zmienne środowiskowe Lambdy, weryfikował je TruffleHogiem (AWS, Entra ID, Okta) i kopiował całe buckety S3. Większość przejętych ról nie była w tym miesiącu używana przez samego dostawcę. Według Wiz dodanie warunku `aws:SourceIp` w trust policy zablokowałoby każdą próbę. Do sprawdzenia w logach: sesje `quiam-*` i AssumeRole z adresów spoza AWS.
**Znaczenie:** To kolejny przypadek, w którym łańcuch zaufania do SaaS w modelu cross-account okazuje się najsłabszym ogniwem. External ID chroni przed confused deputy, ale nie jest sekretem. Do zrobienia: inwentaryzacja ról ufających kontom zewnętrznym, usunięcie nieużywanych, zawężenie trust policy do konkretnego principal z warunkiem SourceIp/SourceVpce. Ostatni krok to rotacja wszystkiego, co rola mogła odczytać. Uwaga: rotacja nie unieważnia już wydanych sesji STS. Ten sam wzorzec dotyczy integracji z platformami danych i AI (Databricks, Snowflake, bramki LLM).
Źródło: https://www.wiz.io/blog/qubole-cross-account-access-abused-in-the-wild

#### SonicWall CVE-2026-19868: obejście SAML/MFA w SSL-VPN, w CISA KEV od 6.11
Luka pozwala obejść kontrolę SAML, a potencjalnie także MFA wymuszane przez SAML. Warunek: atakujący ma ważne poświadczenia do alternatywnej metody logowania i SAML jest włączony dla SSL-VPN. Producent nie podaje obejścia. Termin dla agencji federalnych USA to 20.11. W tym samym biuletynie SonicWall załatał 6 innych błędów, m.in. pre-auth overflow (DoS) i słabe hashowanie haseł.
**Znaczenie:** Failure mode to współistnienie kilku metod uwierzytelniania na jednej bramie: MFA jest tak silne, jak najsłabsza aktywna ścieżka. Po patchu warto skorelować udane logowania VPN z logami IdP i szukać sesji bez odpowiadającego im zdarzenia SAML/MFA. Jeśli to możliwe, wyłączyć lokalne lub LDAP-owe metody na bramie.
Źródło: https://www.helpnetsecurity.com/?p=390676

#### Outlook.com odrzuca legalną pocztę po niezapowiedzianym zaostrzeniu SPF/DMARC (incydent EX1545193)
Od końca poprzedniego tygodnia konsumenckie usługi Microsoft z przerwami odrzucają wiadomości, gdy SPF nie przechodzi albo brakuje rekordu DMARC. Dotyczy to także nadawców poniżej progu 5000 wiadomości/dzień z reguł z 2025 r. Microsoft potwierdził, że błąd wywołała aktualizacja reguł antyspamowych. Osobno wycofał zmianę we front-endzie Exchange Online, która w EMEA zrywała sesje OWA i portali Purview/Defender.
**Znaczenie:** Domeny transakcyjne i systemowe (alerty, faktury, reset hasła) wymagają SPF pass, DKIM pass i DMARC co najmniej p=none z alignmentem. Wysyłki z narzędzi monitoringu i urządzeń, wysyłane z IP spoza SPF, są pierwsze na liście ryzyka. Warto dodać monitoring NDR do alertingu.
Źródło: https://www.theregister.com/applications/2026/11/09/outlookcom-bounces-legit-emails-as-microsofts-authentication-crackdown-goes-sideways/5308428

#### Copeland XWEB PRO (kontrolery chłodnicze) przejmowane jako proxy i bramki spamu (CVE-2026-25085)
Wystawione do Internetu kontrolery chłodni supermarketowych są przejmowane, najpewniej przez obejście uwierzytelniania. Służą potem jako węzły wyjściowe proxy i do skoordynowanej wysyłki spamu.
**Znaczenie:** To drugi w ciągu tygodnia incydent z kontrolerami chłodniczymi/BMS (wcześniej Danfoss i EFS). Te same klasy urządzeń obsługują chłodzenie w DC i edge. Do zrobienia: segmentacja OT, brak ekspozycji do Internetu, egress deny-by-default dla sieci BMS. Ruch residential/OT proxy dalej osłabia reputacyjne filtrowanie IP.
Źródło: https://www.helpnetsecurity.com/2026/11/09/copeland-xweb-refrigeration-controllers-spam-botnet/

#### Plex2Shell: aktywnie wykorzystywane 4 luki w Plex na NAS QNAP
QNAP zaleca traktować NAS jako skompromitowany, jeśli podatny Plex był wystawiony do Internetu.
**Znaczenie:** NAS-y z aplikacjami zewnętrznymi często przechowują backupy, więc ich kompromitacja zagraża odtwarzalności po ransomware. Backup trzymać na osobnej tożsamości, z kopią immutable i bez usług multimedialnych na tym samym hoście.
Źródło: https://links.tldrnewsletter.com/dwAGkH (BleepingComputer via TLDR InfoSec)

#### Google DeepMind: agenci AI obniżają koszt ataku na RSA-1024 (forgery 1380 → 391 core-years)
Atak na podpisy wymaga ok. 4 mld zapytań podpisujących i nie odzyskuje klucza prywatnego. Zaproponowana, ale niezaimplementowana metoda batch factoring mogłaby obniżyć koszt każdego kolejnego klucza ok. 30x. RSA-2048+ jest bez zmian.
**Znaczenie:** Dług kryptograficzny: inwentaryzacja kluczy 1024-bit w legacy (HSM, firmware, DKIM, stare CA wewnętrzne). Kryptoanaliza wspierana przez AI skraca horyzont wycofania słabych parametrów niezależnie od harmonogramu PQC.
Źródło: https://links.tldrnewsletter.com/CdyNhn

#### Contagious Interview: loader w pluginie Tailwind uruchamiany przy buildzie CSS, nie przy `npm install`
JFrog opisał fikcyjną firmę rekrutacyjną z testem React. Złośliwy plugin uruchamia się podczas budowania CSS, więc skanery skryptów instalacyjnych go nie wykrywają. Plugin jest ponownie publikowany pod nowymi nazwami po każdym usunięciu z npm.
**Znaczenie:** Kontrola supply chain musi obejmować etap build (sandbox bez sekretów i egress), a nie tylko install hooks. Dotyczy też runnerów CI i agentów kodujących, które uruchamiają buildy obcych repozytoriów.
Źródło: https://research.jfrog.com/post/accurasofthire-fake-interview/

### Biznes

#### Google Cloud: self-managed VMware Cloud Foundation na bare metal (GA, 22 regiony)
Klient ma pełny root (ESXi, vCenter, NSX), sam patchuje i używa przenośnej subskrypcji VCF. To uzupełnienie zarządzanego VMware Engine; analogiczne oferty mają już AWS i Oracle.
**Znaczenie:** Ścieżka lift-and-shift bez refaktoryzacji, ale z pełną odpowiedzialnością operacyjną po stronie klienta. Model licencyjny Broadcom pozostaje głównym kosztem i źródłem lock-in, a hyperscaler sprzedaje jedynie metal i sieć.
Źródło: https://www.infoq.com/news/2026/11/google-cloud-self-managed-vmware/

#### Broadcom przekazuje VM API do CNCF Sandbox; badanie LF/CNCF: VM to średnio 53% aplikacji produkcyjnych
VM API to neutralne zarządzanie VM przez API Kubernetes. Badanie objęło 492 praktyków (sponsor: CNCF i Broadcom). Większość respondentów nie spodziewa się spadku footprintu VM. Równolegle Red Hat wydał OpenShift 5 (GA) z AI-assisted troubleshooting, detekcją anomalii i nadzorowaną remediacją oraz wspólnym zarządzaniem kontenerami, VM i AI.
**Znaczenie:** Rynek zmierza do jednej płaszczyzny sterowania (K8s API) nad mieszanym VM i kontenerami. Decyzja architektoniczna: KubeVirt/OpenShift Virtualization albo VM API nad vSphere. Lock-in przesuwa się z hypervisora do warstwy operatorów. Badanie jest sponsorowane przez Broadcom, więc liczby trzeba traktować ostrożnie.
Źródła: https://investors.broadcom.com/news-releases/news-release-details/broadcom-contributes-vm-api-open-source-project-cloud-native ; https://www.cncf.io/blog/2026/11/09/new-research-virtual-machines-remain-central-to-cloud-native-infrastructure-as-ai-workloads-evolve/ ; https://www.itpro.com/cloud/hybrid-cloud/red-hat-eyes-automated-cloud-management-gains-with-openshift-5-release

#### Kryzys pamięci wydłuża cykle wymiany sprzętu: rośnie rynek refurbished z gwarancją do 3 lat
**Znaczenie:** W planach odświeżenia na lata 2027–28 warto uwzględnić wariant wydłużenia życia serwerów i rozbudowy RAM przy obecnych cenach DRAM. Ryzyko: mieszane generacje i wsparcie firmware.
Źródło: https://www.theregister.com/on-prem/2026/11/09/memory-crunch-makes-refurbished-kit-look-a-lot-less-secondhand/5307758

## Raport AI-ML

### Technologia

#### Google TPU 8i Max i TPU 8tx (9.11)
TPU 8i Max to dwa układy 8i w jednym pakiecie: 2x compute, SRAM i przepustowości HBM względem 8i, 2x krzemu w tym samym footprincie racka. Według kart z kwietnia 8i to 288 GB HBM, 8,6 TB/s i 384 MB SRAM, więc Max daje ok. 576 GB HBM na pakiet; to wyliczenie, nie oficjalna specyfikacja. TPU 8tx bazuje na krzemie 8t z hostami AMD EPYC 5. generacji i ma sprzętową izolację danych i wag w trakcie przetwarzania (confidential inference). Sieć Boardfly ma do 2x niższą latencję niż 8t. Google nie podał FLOPS, rozmiaru podów, cen ani dat dostępności.
**Analiza:** 8i Max celuje w long-context i KV-cache-bound serving: więcej HBM i SRAM na pakiet zmniejsza liczbę układów na replikę i ruch międzyukładowy. Podwójna gęstość w racku oznacza jednak wyższe kW/rack, więc trzeba zweryfikować budżet chłodzenia w regionach. 8tx otwiera regulowane workloady (sektor publiczny, finanse) dla TPU, ale atestacja i model zaufania nie są opisane. Lock-in pozostaje w JAX/XLA. Na decyzje capacity planning trzeba poczekać do publikacji specyfikacji i cen.
Źródło: https://blog.google/innovation-and-ai/infrastructure-and-cloud/global-network/tpu-8i-max-and-8tx/

#### AMD 6th Gen EPYC (Zen 6, „Venice”) – GA 10.11, systemy u OEM
Flagowy SKU to EPYC 9996, a wersje host-node mają do 5,3 GHz. Deklaracje AMD: do 1,8x przepustowości LLM output względem poprzedniej generacji na hoście GPU, ok. 3,3x agentów/W CPU względem NVIDIA Vera, 2,2x wydajności względem Xeon 6980P. Meta jest klientem wiodącym. Azure ma HDv2 w preview, a OCI zapowiada E7.
**Analiza:** CPU wraca jako zasób krytyczny: sandboxy agentów, tool calling, RL rollouts i preprocessing skalują się rdzeniami, nie GPU. Przy planowaniu klastrów trzeba policzyć proporcję CPU:GPU i gęstość agentów na kW. Porównania z Vera i Xeon pochodzą od producenta i wymagają niezależnych benchmarków (SPECrate, MLPerf host). Uwaga na migrację platformy (nowe sockety SP7/SP8) i dojrzałość BIOS i firmware w pierwszych kwartałach.
Źródło: https://www.hpcwire.com/off-the-wire/amd-announces-general-availability-of-6th-gen-epyc-cpus/

#### NVIDIA Snapshot (open source, K8s CRD): checkpoint/restore zainicjowanych workloadów inferencyjnych
CRIU przywraca stan CPU, a osobny etap przywraca pamięć GPU. CRD: `PodSnapshot`, `SnapshotJob` i adnotacja `nvidia.com/restore-from`. Na B200 z VAST NFS (vLLM, SGLang, TRT-LLM) restore trwał 3,6–42,7 s wobec cold startu od ok. 88 s do ok. 19 min (8–36x szybciej). Artefakt dla 8-GPU MoE ma do ok. 1 TB. Ograniczenia: wymagany ten sam model GPU i ta sama wersja sterownika, KV cache jest zwalniany przed checkpointem, w Dynamo funkcja jest eksperymentalna.
**Analiza:** Skraca czas reakcji autoscalingu i MTTR replik, więc pozwala utrzymywać mniejszy zapas ciepłych GPU, co bezpośrednio obniża koszt. Wąskim gardłem staje się storage: TB na replikę i przepustowość odczytu przy jednoczesnym skalowaniu wielu replik grozi efektem thundering herd. Pinning wersji sterownika utrudnia rolling upgrade floty, a snapshoty stają się wrażliwym artefaktem z wagami w pamięci (szyfrowanie, kontrola dostępu).
Źródło: https://developer.nvidia.com/blog/start-and-restore-gpu-workloads-in-seconds-with-nvidia-snapshot/

#### Agent Substrate (CNCF Sandbox): multipleksowanie sesji agentów na puli ciepłych podów
Bezczynny agent jest snapshotowany (pamięć i FS, sandbox gVisor lub microVM) i przywracany na dowolny wolny worker. Restore omija ścieżkę API server, scheduler i kubelet. Router parkuje żądania, gdy pula jest pełna. Integracja z Dynamo (przewidywany TTFT, saturacja puli GPU) decyduje o suspend/resume w trakcie oczekiwania na model. Deklaracja to 10x gęstości względem kontenerów; to cel projektowy, nie pomiar.
**Analiza:** Odpowiada na realny problem: sesja agenta przez większość czasu czeka, a pod-per-sesję marnuje RAM. Ryzyka: opóźnienie restore przy dużym stanie, spójność stanu FS przy migracji między węzłami, izolacja tenantów w multipleksowanym workerze. W tym samym dniu NVIDIA ogłosiła NVKE i NVKR (zarządzanie K8s na infrastrukturze GPU, public preview). Stos K8s dla AI coraz silniej łączy się z narzędziami NVIDIA (Dynamo, DSX), co zwiększa lock-in na poziomie operacji.
Źródła: https://developer.nvidia.com/blog/agent-substrate-advancing-kubernetes-for-agents-at-scale/ ; https://developer.nvidia.com/blog/manage-kubernetes-clusters-for-gpu-infrastructure-with-nvidia-kubernetes-engine/

#### Cisco IT: wewnętrzna „AI factory” – utylizacja GPU 15% → ok. 50%
Zamiast przydziału GPU per projekt Cisco zbudowało pooled fabric (Silicon One) i multi-tenant scheduling. Używa mieszanki NVIDIA i AMD (tańszy krzem do inferencji), rear-door heat exchangers (2 węzły AI na szafę bez nowej powierzchni) i obserwowalności przez Splunk. Liczba GPU wzrosła z ok. 250 do ponad 1568 w tym samym DC, a szczytowa utylizacja sięga ok. 80%. Ok. 80% stałej inferencji on-prem uznano za optimum kosztowe.
**Analiza:** To typowy profil enterprise: główną dźwignią TCO jest pooling i scheduling, nie sam zakup GPU. Kolejne kroki (batch w okresach bezczynności, automatyczne odzyskiwanie GPU) wymagają preemption i limitów, inaczej zespoły przewymiarowują rezerwacje. Liczby pochodzą od producenta i dotyczą własnego wdrożenia.
Źródło: https://blogs.cisco.com/cisco-on-cisco/how-cisco-built-an-enterprise-ai-factory

#### NVIDIA AutoCompress: pruning świadomy sprzętu (Nemotron 3.5 Super 120B → 74B)
Optymalizacja pruningu odbywa się pod docelowe ograniczenia latency, throughput i pamięci, a nie pod liczbę parametrów.
**Analiza:** Ok. 38% mniej parametrów przy deklarowanej jakości zmniejsza liczbę GPU na replikę i zapotrzebowanie na HBM, co ma znaczenie przy obecnych cenach pamięci. Model jest jednak optymalizowany pod konkretny sprzęt (przenośność między akceleratorami), a jakość trzeba zweryfikować na własnych ewaluacjach.
Źródło: https://arxiv.org/abs/2611.08290

### Biznes

#### ChatGPT Enterprise w AWS Marketplace (zaliczane do commitów AWS)
**Analiza:** Zakup przez marketplace omija budżet SaaS i zużywa istniejące commity EDP, co obniża barierę wejścia, ale rozmywa kontrolę kosztów AI w FinOps. Konkurencja dostawców modeli przenosi się do kanałów zakupowych hyperscalerów.
Źródło: https://links.tldrnewsletter.com/SmAOO4

## euro neocloud

### Nscale
#### Microsoft rezygnuje z części projektów Nscale (Bloomberg/FT, 5.11)
**Fakty:** Microsoft anulował kontrakt o wartości ok. 3 mld USD na DC w Wielkiej Brytanii (Loughton, Essex), bo przyłączenie do sieci przesunęło się na lata 2030+. Nie wykorzysta też opcji ponad dwukrotnego zwiększenia kontraktu w Teksasie. Według zestawienia opartego na S-1 Nscale na każde ok. 18 podpisanych GPU przypada ok. 1 uruchomione (źródło wtórne, StackPath). Nscale nie skomentował.
**Ocena analityczna:**
- Wykryte sygnały ostrzegawcze: utrata anchor tenanta na projekcie w Europie; ryzyko przyłączeniowe zmaterializowało się jako anulacja; bardzo duża luka między kontraktami a mocą oddaną.
- Ocena ryzyka: **Wysokie**. Model zależy od kilku dużych najemców i terminowego oddania mocy, a opóźnienia sieciowe w UK są wieloletnie (por. Teesside 2029 w rejestrze). Kolejne anulacje uderzą w finansowanie projektowe.
- Linki: https://www.investing.com/news/stock-market-news/microsoft-tightens-oversight-of-neocloud-data-center-projects-bloomberg-news-reports-5005281 ; https://stackpath.ie/europe/neoclouds

### Nebius
#### Nadzór Microsoft nad projektami, kara 1 mln USD za generatory bez pozwoleń (NJ); wyniki Q3 12.11
**Fakty:** Microsoft wysłał personel do projektu Nebius w New Jersey, ukaranego grzywną ok. 1 mln USD za pracę generatorów bez pozwoleń. Akcje Nebius spadły 5.11 o ok. 14–15% i odbiły 6.11. Wyniki Q3 2026 zostaną opublikowane 12.11 przed otwarciem rynku.
**Ocena analityczna:**
- Wykryte sygnały ostrzegawcze: problemy z pozwoleniami przy generacji on-site (wąskie gardło regulacyjne, nie tylko sprzętowe); większa kontrola głównego klienta nad harmonogramem.
- Ocena ryzyka: **Średnie**. Nebius ma dywersyfikację klientów (m.in. Meta) i dostęp do rynku kapitałowego, ale wrażliwość wyceny na sentyment wobec neocloudów jest wysoka. Kluczowe w raporcie 12.11: capex, oddana moc w MW, warunki finansowania i ewentualne korekty harmonogramu Microsoft.
- Linki: https://www.investing.com/news/stock-market-news/microsoft-tightens-oversight-of-neocloud-data-center-projects-bloomberg-news-reports-5005281 ; https://nebius.com/newsroom/nebius-group-announces-date-of-third-quarter-2026-results-and-conference-call

## Ryzyka rynku AI/GPU

#### CoreWeave Q3 2026 (10.11): backlog 132,5 mld USD, capex 2026 37–39 mld, rezygnacja z niezabezpieczonego długu na poziomie spółki matki
Przychód wyniósł 3,661 mld USD (+168% r/r), strata netto 709 mln USD (rok wcześniej 110 mln), adjusted operating income 379 mln. Capex w Q3 to ok. 14,5 mld (powyżej widełek, część dostaw przyspieszona z Q4). Dług całkowity ok. 47 mld. Spółka nie planuje nowego niezabezpieczonego długu na poziomie spółki matki przy obecnych rentownościach (obligacje >16%). Do 2027 r. zakłada poniżej 5 mld USD finansowania na tym poziomie; resztę mają pokryć przedpłaty klientów, finansowanie od partnerów sprzętowych i DDTL przypisane do kontraktów. Aktywna moc przekracza 1,7 GW, cel to 1,85 GW na koniec roku. Spółka celowo wstrzymuje część mocy do sprzedaży w późniejszym terminie.
**Analiza:** FINANSE: rynek długu niezabezpieczonego jest dla CoreWeave praktycznie zamknięty (zgodnie z sygnałem CDS z rejestru). Model przesuwa się na finansowanie aktywami i kontraktami, co zwiększa zależność od kontrahentów (Anthropic, OpenAI, Microsoft) i od Nvidia jako finansującego. Ryzyko przenosi się na łańcuch: przedpłaty klientów oznaczają, że ich płynność (np. IPO Anthropic) staje się płynnością CoreWeave. Strata netto rośnie szybciej niż przychód, co odzwierciedla koszt odsetek i deprecjacji. POPYT: backlog +27% kw/kw, a wstrzymywanie mocy sugeruje oczekiwanie wyższych cen GPU-h.
Źródła: https://www.cnbc.com/2026/11/10/coreweave-crwv-q3-earnings-report-2026.html ; https://convergedigest.com/coreweave-q3-2026-earnings-financing-nvidia-backlog/

#### Microsoft zaostrza nadzór nad neocloudami (Nebius, IREN, Nscale) – reakcja rynku 5.11
IREN przesuwa terminy dostaw dla Microsoft po zmianach projektu sieci (według źródeł Bloomberg problemem było okablowanie w DC w Teksasie). Spadki 5.11: Nebius ok. 14%, IREN ok. 17%, CoreWeave ok. 10%. Microsoft deklaruje strategię „flexibility and optionality”.
**Analiza:** POPYT i DOSTAWY: największy kontrahent selektywnie wycofuje się ze słabiej realizowanych projektów. Ryzyko koncentracji klienta jest realne, a opóźnienie wykonawcze (sieć, kable, pozwolenia) zamienia się w utratę kontraktu. Brakuje danych o łącznej skali wycofanej mocy, a to kluczowy wskaźnik do śledzenia.
Źródło: https://www.investing.com/news/stock-market-news/microsoft-tightens-oversight-of-neocloud-data-center-projects-bloomberg-news-reports-5005281

#### Anthropic kupuje turbiny gazowe Caterpillar: do 1,7 GW za 3,4 mld USD (przejęte zamówienie OnePWR)
OnePWR złożył zamówienie, ale nie sfinansował zakupu. Anthropic zawarł umowę bezpośrednio z Caterpillar. Lokalizacje nie zostały ujawnione. Kolejki na turbiny gazowe sięgają wczesnych lat 2030.
**Analiza:** INFRASTRUKTURA: laboratorium, które dotąd wynajmowało moc, przechodzi do posiadania aktywów energetycznych. Ok. 2 mld USD/GW za samą generację to capex poza GPU i budynkami. Upadek finansowania OnePWR sygnalizuje stres w łańcuchu power-for-AI (zamówienia slotów turbinowych bez kapitału). Failure modes: pozwolenia emisyjne (por. kara Nebius w NJ), ceny gazu, redundancja N+1 przy pracy wyspowej.
Źródło: https://techcrunch.com/2026/11/09/anthropic-joins-ais-natural-gas-frenzy-buys-3-4b-worth-of-turbines/

#### Aktualizacja: Anthropic IPO – roadshow od tygodnia 9.11, notowania planowane do końca przyszłego tygodnia (WSJ); szacowany próg przychodu ok. 200 mld USD/rok dla pokrycia zobowiązań compute
Nowe elementy względem rejestru: WSJ podaje plan rozpoczęcia notowań do końca przyszłego tygodnia mimo zmienności rynku. Część dużych inwestorów naciska na wycenę niższą niż ok. 1,5 bln USD. Analiza zewnętrzna szacuje, że utrzymanie średnich zobowiązań compute do 2030 przy obecnym stosunku wydatków wymaga ok. 200 mld USD przychodu rocznie.
**Analiza:** EKONOMIKA: luka między run-rate (ok. 68 mld z rejestru) a ok. 200 mld oznacza, że zobowiązania wobec dostawców (CoreWeave, xAI, Nscale, hyperscalerzy) zależą od ok. 3x wzrostu przychodu przy jednoczesnej deflacji ceny tokena (Fable 5.2 wprowadza niższe ceny). Opóźnienie IPO byłoby sygnałem kredytowym dla całego łańcucha neocloudów.
Źródła: https://www.wsj.com/finance/stocks/anthropic-prepares-to-push-ahead-with-ipo-despite-market-jitters-843dd16c ; https://links.tldrnewsletter.com/Zm8I7b

#### Tunguz: USA mogą stać się importerem netto tokenów w latach 2028–2030
Krajowa moc obsłuży około 2,5x rocznego wzrostu tokenów. Szybszy popyt lub anulowane projekty DC (por. wyniki lokalnych głosowań w rejestrze) przesuną obliczenia za granicę.
**Analiza:** INFRASTRUKTURA: jeśli moratoria i opóźnienia przyłączeń się utrzymają, wzrośnie wartość mocy poza USA (UE, Bliski Wschód, Azja), a wraz z nią znaczenie suwerenności danych i latencji transkontynentalnej. To szansa dla euro neocloudów, ale tylko przy terminowym oddawaniu mocy, czego dziś brakuje (Nscale).
Źródło: https://tomtunguz.com/when-does-the-us-become-a-net-importer-of-tokens

## Newsletters summary

#### Claude Fable 5.2 i Mythos 5.2: do 30% szybsze generowanie, niższe ceny, wspólna architektura
**Analiza:** Szybsze dekodowanie i niższe ceny przesuwają koszt na token w dół, co sprzyja agentom o długim horyzoncie, ale nasila presję na przychód/GPUh dostawców. Przy migracji modeli trzeba ponownie zbadać limity rate i budżety tokenów. Testy społeczności sugerują też zmiany w zachowaniu rozumowania istniejących modeli bez zmiany wersji, co wymaga regresji ewaluacyjnych w pipeline.
Źródło: https://www.anthropic.com/claude-fable-and-mythos-5-2

#### Meta Muse Ember: mały model rozumujący do tool calling i parsowania dokumentów
Meta deklaruje jakość zbliżoną do Claude Haiku 5.5 (high reasoning) przy ok. 1/3 kosztu.
**Analiza:** Dla wolumenowych kroków agentowych (routing, ekstrakcja) mniejsze modele obniżają koszt i latencję. Deklaracje producenta trzeba zweryfikować na własnym zestawie zadań.
Źródło: https://research.meta.ai/blog/muse-ember

#### OpenAI Control Room: centralna autoryzacja żądań inferencji badawczej, zatrzymanie outputu w ok. 35 s
**Analiza:** To wzorzec kill switch na warstwie inferencji z mierzalnym SLO zatrzymania. Warto zdefiniować analogiczne RTO dla własnych agentów: czas od decyzji do odcięcia, zarówno dla gateway, jak i dla poświadczeń.
Źródło: https://links.tldrnewsletter.com/qKKt6Y

#### Fireworks: incydent z 6.10 przypisany grupie odsprzedającej nielegalny dostęp do LLM (illicit routing gateways)
**Analiza:** To nowy element względem rejestru (atrybucja). Skradzione poświadczenia do inferencji są monetyzowane przez pośredników, więc potrzebne są limity wydatków i anomalia zużycia per klucz (zgodnie z wcześniejszym raportem Mandiant).
Źródło: https://fireworks.ai/security/bulletins/october-2026-security-incident-update

#### Torvalds: fala zgłoszeń błędów generowanych przez AI mogła osiągnąć szczyt w eBPF
**Analiza:** W eBPF poprawki napływają szybciej niż nowe zgłoszenia, ale większość podsystemów jądra jeszcze nie osiągnęła szczytu. Na najbliższe kwartały trzeba zakładać podwyższoną kadencję patchowania jądra.
Źródło: https://www.theregister.com/security/2026/11/09/linus-torvalds-sees-signs-ai-prompted-patch-flood-has-peaked/5308345

#### Detekcja ataków wspomaganych AI i agentowych
Charakterystyczne sygnały: wzorzec map-reduce (fan-out rekonesansu, potem jednorazowa synteza), błędy w syntetycznych nagłówkach, przełączanie frameworków.
**Analiza:** Przydatne do reguł SIEM/NDR: korelacja wielu krótkich, równoległych sesji z jednego źródła.
Źródło: https://www.sudorem.dev/blog/detecting-and-responding-to-ai/

#### NVIDIA: Alpamayo 2 Edge (modele rozumujące AV w limitach onboard) i Holoscan 5 (kompensacja grawitacji ramienia robota)
**Analiza:** Edge AI wymaga destylacji i walidacji latencji na docelowym SoC. Dla infrastruktury to głównie popyt na trening i symulację w DC, nie zmiana w serwerowni.
Źródła: https://developer.nvidia.com/blog/running-av-reasoning-models-within-onboard-compute-and-latency-limits/ ; https://developer.nvidia.com/blog/make-a-robot-arm-feel-weightless-with-nvidia-holoscan-5/

## Implikacje praktyczne
1. **Zaufanie do dostawców SaaS w modelu cross-account (AWS):** przegląd wszystkich ról z zewnętrznym principal, warunki SourceIp/SourceVpce, okresowe usuwanie nieużywanych integracji. W horyzoncie 3–5 lat przejście na krótkotrwałe, federacyjne poświadczenia (workload identity) zamiast External ID i statycznych kluczy.
2. **Ryzyko kontrahenta w umowach GPU:** neoclouds finansują się przedpłatami klientów i długiem pod kontrakt. W umowach wymagać kamieni milowych mocy (MW oddane), kar za opóźnienia, praw step-in i escrow. Dywersyfikować na 2–3 dostawców plus hyperscalera, bo anulacje Microsoft pokazują, że opóźnienie wykonawcze kończy się utratą kontraktu.
3. **Proporcja CPU:GPU i gęstość agentów:** przy Venice, Agent Substrate i sandboxach planować osobne pule CPU pod agentów i RL, z metryką agentów/kW. Skalowanie wyłącznie GPU przy rosnącym ruchu agentowym przesunie wąskie gardło na CPU i pamięć.
4. **Warstwa szybkiego startu inferencji (snapshot/restore):** pozwala zmniejszyć zapas ciepłych GPU, ale wymaga storage o wysokiej przepustowości, polityki wersji sterowników (pinning vs. rolling upgrade) oraz szyfrowania artefaktów zawierających wagi.
5. **Energia on-site jako standard projektowy:** turbiny gazowe i ogniwa paliwowe wymagają pozwoleń emisyjnych, kontraktów gazowych i architektury wyspowej N+1. Ryzyko regulacyjne (kary, moratoria) trzeba wpisać do harmonogramów i TCO.

## Trend tygodnia
Ryzyko w łańcuchu AI przesuwa się z dostępności GPU na zdolność wykonawczą i finansowanie. CoreWeave wykazuje rekordowy backlog, ale zamyka ścieżkę niezabezpieczonego długu i opiera się na przedpłatach klientów, co splata jego płynność z płynnością laboratoriów i wynikiem IPO Anthropic. Microsoft jako największy odbiorca zaczyna dyscyplinować dostawców: opóźnienia przyłączeń, okablowania czy pozwoleń kończą się anulacją kontraktów, co najmocniej uderza w Nscale i europejskie projekty zależne od sieci UK. Odpowiedzią jest pionowa integracja energii: laboratoria kupują turbiny bezpośrednio, a upadające pośredniki (OnePWR) oddają sloty. Po stronie sprzętu konkurencja przenosi się na specjalizację: TPU 8i Max pod długi kontekst, 8tx pod confidential inference, EPYC Venice pod gęstość agentów na wat. Wszystkie te rozwiązania są pozycjonowane względem ekonomiki agentów, a nie czystego treningu. Warstwa operacyjna (Snapshot, Agent Substrate, NVKE) standaryzuje Kubernetes dla AI i jednocześnie wiąże go z ekosystemem NVIDIA. W bezpieczeństwie dominuje nadużywanie zaufania (role cross-account, alternatywne metody uwierzytelniania obchodzące MFA, kontrolery OT jako proxy), a nie klasyczne exploity.

## To obserwować
- **Nebius Q3 2026 (12.11):** oddana moc w MW vs. kontraktowana, capex, koszt finansowania, komentarz o Microsoft i harmonogramie NJ.
- **Anthropic IPO:** zakres cenowy w S-1/A, data notowań, wycena (próg ok. 1–1,5 bln USD) jako sygnał dla wycen i długu neocloudów.
- **CoreWeave:** nowe DDTL pod kontrakty (koszt, zabezpieczenia), rentowność obligacji 2032 i CDS po wynikach.
- **Google TPU 8i Max/8tx:** specyfikacje, ceny, regiony i model atestacji confidential inference.
- **Skala wycofania Microsoft z neocloudów:** łączna liczba MW lub kontraktów i ewentualne komunikaty IREN/Nscale.
