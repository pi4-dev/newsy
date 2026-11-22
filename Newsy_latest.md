# Raport Zbiorczy IT – 2026-11-22

**Okno przyrostowe:** od raportu z 2026-11-21 (weekend). Uwzględniono zdarzenia z 15–21.11, których nie było w rejestrze z ostatnich 7 dni. Sekcja „AI for networking” została pominięta, bo nie pojawiły się w niej nowe, zweryfikowane informacje.

## Raport technologiczny

### Biznes

#### Retelit (Włochy): sprzedaż 38 DC – oferta Vaultica (Apollo), w procesie Ardian, Macquarie i Rai Way (19.11)
Według Bloomberga Vaultica, wspierana przez Apollo, złożyła ofertę na biznes DC Retelit. Wycena wynosi ok. 700–800 mln EUR. Ardian, Macquarie i Rai Way złożyły oferty niewiążące, a oferty wiążące mogą wpłynąć w grudniu. Właściciel, Asterion Industrial Partners, chce zmniejszyć zadłużenie i skupić Retelit na sieci światłowodowej. Portfel obejmuje 38 obiektów, głównie edge/regionalnych, dla operatorów telekomunikacyjnych, firm i chmur.

**Znaczenie:** Rozdzielenie światłowodu i DC to kolejny przykład modelu, w którym fiber zostaje u operatora z infrastrukturą, a colo trafia do funduszy z niższym kosztem kapitału. Dla klientów colo we Włoszech zmiana właściciela zwykle oznacza przegląd cennika przy odnowieniu, presję na zagęszczenie mocy i możliwe zmiany zasad cross-connectów. Warto teraz sprawdzić klauzule change-of-control i indeksacji cen w umowach.

Źródło: [Placera / Finwire (za Bloombergiem)](https://www.placera.se/telegram/apollo-stodda-vaultica-uppges-buda-pa-retelits-datacenterverksamhet-bn-20261119), [Private Equity Wire](https://www.privateequitywire.co.uk/retelit-weighs-potential-e700m-sale-of-data-centre-unit/)

### Technologia

#### Rackspace Spot (managed Kubernetes): wyciek poświadczeń dał dostęp do Secrets klientów (17.11, przywrócenie 21.11)
17.11 w godz. ok. 00:20–22:00 UTC atakujący, używając wycieku poświadczeń, działał w platformie Spot (współtworzonej z Platform9) jako uprawniony użytkownik części organizacji klientów. Mógł:
- tworzyć pody,
- czytać Kubernetes Secrets,
- listować węzły i wolumeny.

Rackspace dowiedział się o incydencie od klienta 18.11. Unieważnił tokeny i sesje, usunął obce pody i zrotował klucze. Konsola była niedostępna przez kilka dni, więc klienci nie mogli sami rotować poświadczeń. IoC:
- adres 23.224.79.106 i zakres 23.224.0.0/15,
- prefiksy obiektów `i1`, `glm-`, `probe`.

**Znaczenie:** To typowy failure mode zarządzanego control plane: kompromitacja tożsamości po stronie dostawcy omija RBAC klienta, bo napastnik ma uprawnienia legalnego użytkownika. Odcięcie konsoli na czas sanityzacji blokuje rotację u klienta, czyli wydłuża okno ekspozycji. Wnioski:
- Sekrety w etcd (nawet szyfrowane at-rest) są czytelne dla każdego z prawem `get secrets`. Lepszy jest zewnętrzny KMS/Vault z krótkim TTL i workload identity.
- Rotację poświadczeń dla downstreamu (bazy danych, rejestry, Git, chmury) trzeba móc przeprowadzić bez konsoli dostawcy.
- Należy przejrzeć logi downstreamu od 17.11.

Źródło: [BleepingComputer](https://www.bleepingcomputer.com/news/security/rackspace-spot-breach-let-hackers-access-customer-kubernetes-secrets/)

#### Microsoft Entra Connect Health: błąd w API wykorzystany do odczytu metadanych tenantów (luka od 23.07, poprawka 19.11, powiadomienia 20–21.11)
Błąd w API Entra Connect Health, wprowadzony 23.07 i wykryty przez Microsoft 9.11, pozwalał na nieautoryzowany odczyt metadanych tenantów. Ujawnione mogły zostać:
- e-maile kontaktów technicznych i administratorów,
- Tenant ID, domeny i region,
- poziom licencji Entra i status synchronizacji katalogu z czasem ostatniego synca,
- wygasły certyfikat rejestracji agentów.

Microsoft twierdzi, że hasła, hashe, tokeny i ważne klucze nie wyciekły, a zmian konfiguracji nie było, choć luka teoretycznie pozwalała modyfikować ustawienia monitoringu. Microsoft nie podał liczby tenantów ani sprawcy. MSP dostają powiadomienia dla wielu tenantów naraz. Poprawka jest po stronie serwera, a klient nie musi nic aktualizować.

**Znaczenie:** Same metadane wystarczają do precyzyjnego phishingu. Napastnik wie, kto jest adminem, które tenanty mają hybrydowy sync AD (a więc atakowalny serwer Entra Connect on-prem) i jaki mają poziom licencji, a więc czy działa Conditional Access lub PIM. Priorytety:
- MFA odporne na phishing (FIDO2) dla ról uprzywilejowanych,
- przegląd logów logowań kont wymienionych w powiadomieniu,
- traktowanie serwera Entra Connect jak Tier 0.

Problem trwał prawie 4 miesiące, zanim został wykryty. To argument za własnym monitoringiem anomalii w audycie Entra zamiast polegania na telemetrii dostawcy.

Źródło: [BleepingComputer](https://www.bleepingcomputer.com/news/security/microsoft-entra-connect-health-flaw-exploited-to-steal-tenant-data/)

#### AWS us-east-1: niemal 4 godziny awarii IAM Identity Center i 11 usług, status page z opóźnieniem 93 minut (15.11)
15.11 od 3:05 PT AWS notował błędy w us-east-1, przede wszystkim w IAM Identity Center (logowanie do kont): problemy z uwierzytelnianiem, łącznością i rejestracją. Pierwszy wpis na publicznej stronie statusu pojawił się ok. 93 minuty po początku błędów. Pełne przywrócenie nastąpiło o 6:53 PT, łącznie ok. 3 h 48 min. AWS nie podał root cause. Incydent opisał Corey Quinn 21.11.

**Znaczenie:** IAM Identity Center jest zarządzany z jednego home regionu, a jego awaria odcina ludzki dostęp do wszystkich kont organizacji niezależnie od regionu workloadów. Wymagania:
- kont break-glass z poświadczeniami IAM poza Identity Center (z MFA sprzętowym, przechowywanych offline),
- alertowania opartego na własnych syntetycznych testach logowania, a nie na AWS Health.

Przy ponad 90 minutach opóźnienia status page'a SLO wykrywania incydentu u dostawcy jest zawsze gorsze niż własny probe.

Źródło: [The Register (Corey Quinn)](https://www.theregister.com/columnists/2026/11/21/aws-got-hit-by-an-outage-too-heres-why-you-didnt-hear-about-it/5311511)

## Raport AI-ML

### Biznes

#### BIS zatwierdził 879 868 akceleratorów H200/MI325X dla Chin z 3,44 mln wnioskowanych; Alibaba i Tencent na liście firm wojskowych (ujawnienie 19.11)
Przewodniczący House Foreign Affairs Committee Brian Mast ujawnił dane licencyjne Commerce, korzystając z uprawnień ECRA. Do 11.05.2026 r. w 21 wnioskach zatwierdzono 879 868 układów Nvidia H200 i AMD MI325X z ok. 3,44 mln wnioskowanych. Przydziały:
- Tencent: 150 tys.,
- Alibaba: 75 tys.,
- Baidu: 43 tys.,
- BYD: 4 tys.

Odrzucono duże wnioski Nvidii dla Alibaby (610 tys.) i ByteDance (508 tys.) oraz 75 tys. MI325X dla UCloud. Wszystkie wnioski oceniały ryzyko dywersji jako „niskie”. Część zatwierdzonych zamówień nie została wysłana, bo Pekin blokował import. Nvidia deklaruje, że wstrzymała wysyłki H200 do Chin. Alibaba (Eddie Wu) zapowiada, że cel 20 GW mocy DC może zostać obniżony po nowej regule o zdalnym dostępie.

**Analiza:** Licencje zatwierdzone, ale niezrealizowane to martwy backlog. Nie należy go wliczać do popytu 2027. Obniżenie celu 20 GW przez Alibabę to bezpośredni sygnał spadku capex jednego z największych nabywców poza USA. Uderzy to w łańcuch dostaw z Azji (serwery, HBM, optyka). Ujawnienie listy z oceną ryzyka „niskie” wzmacnia presję na zaostrzenie procesu licencyjnego (KYC, audyty on-site). Dla neocloudów spoza USA z chińskimi klientami rośnie ryzyko licencji wydanej i potem cofniętej, co oznacza stranded capacity bez możliwości szybkiej realokacji kontraktu.

Źródło: [The Register](https://www.theregister.com/legal/2026/11/20/us-let-nvidia-and-amd-sell-bucketloads-of-chips-to-military-linked-chinese-companies/5311200), [The Standard HK (za Reuters)](https://www.thestandard.com.hk/world/article/349075/US-signed-off-on-sales-of-nearly-900000-AI-chips-to-China-in-six-months-committee-says)

### Technologia

#### E4 Computer Engineering i Openchip: MoU w sprawie europejskich systemów inferencyjnych na akceleratorach RISC-V BERNIA (SC26, 20.11)
Włoski integrator HPC E4 i barceloński fabless Openchip (ok. 300 osób) podpisały niewiążące MoU. Chcą zbadać wspólną rodzinę energooszczędnych systemów do inferencji na dużą skalę i HPC na akceleratorach BERNIA (RISC-V). Docelowo systemy mają trafić do programów AI Factories i AI Gigafactories UE. Kolejny krok to badanie rynku, potem ewentualnie definitywna umowa, projekt i walidacja. Nie podano dat, wydajności, poboru mocy ani wolumenów. Openchip nie ma jeszcze układu komercyjnego (dostępność zapowiadana na 2027 r.).

**Analiza:** To sygnał polityczny pod kryteria „suwerenności” w przetargach EuroHPC, a nie realna alternatywa capacity w horyzoncie 2027–2028. Brak dojrzałego stosu (kompilatory, kernele attention i MoE, integracja z vLLM/SGLang) oznacza, że nawet przy dobrym perf/W koszt portowania i niska utylizacja zjedzą przewagę TCO. W planowaniu warto to traktować jako opcję na niszowe obciążenia inferencyjne w sektorze publicznym. Platformę bazową nadal wyznaczają Nvidia i AMD.

Źródło: [HPCwire](https://www.hpcwire.com/off-the-wire/e4-computer-engineering-and-openchip-sign-mou-to-explore-eu-sovereign-ai-compute-platforms/)

## euro neocloud

### Nebius

#### Trzecia umowa z Mega DC (Mega Or): kolejne 134 MW IT w Izraelu (20.11)
**Fakty:**
- Mega DC, spółka zależna Mega Or, podpisała z podmiotem zależnym Nebius umowę na 134 MW mocy IT w dwóch lokalizacjach. Pierwsza to druga faza kampusu Beit Shemesh (maks. 135 MW IT, w budowie), druga to planowana lokalizacja w Masmiya (rada regionalna Yoav).
- Nie ujawniono okresu najmu ani wartości umowy.
- Wcześniejsza umowa ze stycznia 2026 r. obejmowała 80 MW na 5 lat z opcją przedłużenia, a koszt budowy wyceniono na ok. 880 mln USD.
- Według niemieckiego serwisu giełdowego w Masmiya brakuje jeszcze zgody na przyłączenie do sieci.

**Ocena analityczna:**
- *Wykryte sygnały ostrzegawcze:*
  - Nebius dokłada leasing zewnętrznych obiektów, a w III kw. obniżył prognozę przychodów 2026 r. właśnie z powodu opóźnień w lokalizacjach, których nie kontroluje.
  - Masmiya bez potwierdzonego przyłącza powtarza ten sam wzorzec ryzyka.
  - Koncentracja ok. 330 MW w jednym kraju i u jednego dewelopera zwiększa ekspozycję na ryzyko geopolityczne i ryzyko kontrahenta.
  - Przy rentownościach długu neocloudów na poziomie 14–16% stałe zobowiązania leasingowe pogarszają dźwignię operacyjną, jeśli najem GPU się nie zapełni.
- *Ocena ryzyka:* **Średnie.** Kontrakty z Microsoft i Meta oraz rosnące ceny najmu GPU wspierają popyt, ale wykonanie zależy od harmonogramu przyłączeń u stron trzecich.
- *Linki:* [Globes (umowa Mega Or)](https://en.globes.co.il/en/article-1001531296), [DCD](https://www.datacenterdynamics.com/en/news/nebius-expands-israel-data-center-lease-with-mega-or-by-134mw/), [Börse Global (przyłącze Masmiya)](https://www.boerse-global.de/nebius-aktie-netzanschluss-auf-eis/844194)

## Ryzyka rynku AI/GPU

#### Senat USA uchwalił BAAJA 62–33: DC ≥20 MW mają pokrywać koszty sieci, ok. 55 mld USD w 10 lat (19.11)
Senat przyjął Bipartisan American Affordability and Jobs Act. Ustawa:
- wymaga, by DC i obiekty high-density o mocy ≥20 MW pokrywały dodatkowe koszty wytwarzania, magazynowania i sieci, które powodują;
- zleca FERC ustanowienie ochrony odbiorców.

Według Grid Strategies (dane komisji energii Senatu) DC zapłacą ok. 55 mld USD za przesył w 10 lat, ok. 35 mld USD więcej niż obecnie. Ustawa przyspiesza też procedury pozwoleń (NEPA, CWA, ESA) dla elektrowni i linii przesyłowych. Teraz trafia do Izby Reprezentantów, która w 2025 r. przyjęła węższą wersję. Trump zapowiada podpis. Od stycznia Kongres będą kontrolować Demokraci, część z nich chce bardziej restrykcyjnej wersji.

**Analiza:**
- *Infrastruktura i energia:* koszt przyłączenia staje się jawną pozycją capex projektu, a nie ukrytą subsydią taryfową. Przy ok. 3,5 mld USD rocznie, rozłożonych na ok. 30–50 GW nowych przyłączeń, daje to rzędu 70–120 USD/kW rocznie, czyli kilka procent TCO mocy IT. Największy wpływ dotknie projektów w regionach z dużymi kosztami rozbudowy przesyłu (PJM, ERCOT).
- *Finanse:* dla deweloperów finansowanych długiem pod przedsprzedaż mocy (neocloudy, BTS dla hyperscalerów) wyższy upfront capex przy zamkniętym rynku obligacji przesuwa projekty w czasie albo wymusza renegocjację stawek najmu.
- *Strona pozytywna:* szybsze pozwolenia skracają czas od wniosku do przyłączenia, co może zmniejszyć ryzyko stranded GPU.

Netto: wyższy koszt, ale mniej niepewności co do terminu. Do monitorowania, czy FERC dopuści rozliczenie kosztów w taryfie przez umowy typu „large load tariff”, czy wymusi płatności z góry.

Źródło: [Yahoo News / HuffPost](https://www.yahoo.com/news/politics/articles/senate-votes-ai-data-centers-160341239.html), [The Hill](https://thehill.com/homenews/senate/6229829-senate-passes-permitting-reform/), [DCD](https://www.datacenterdynamics.com/en/news/us-senate-passes-bill-requiring-data-centers-to-pay-more-for-grid-infrastructure/)

## Newsletters summary

#### The Business Engineer: rekordowy rozjazd między „zegarem fizycznym” (+4) a „zegarem finansowym” (−5); HBM sprzedane na lata naprzód (21.11)
Autor prowadzi cztery wskaźniki cyklu AI: infrastruktura fizyczna +4, finanse −5, efektywność +4, adopcja +2. Różnica 9 punktów między stroną fizyczną a finansową jest najwyższa w historii pomiarów. Nowe elementy względem rejestru:
- trzej producenci HBM poinformowali klientów, że znaczna część mocy jest sprzedana z wyprzedzeniem (według autora „do 2029”), a mimo to ich kursy spadły;
- według wyliczeń autora z IPO Anthropic i emisji dla dostawców wpłynęło łącznie ok. 119,5 mld USD;
- banki zaostrzają warunki kredytów na DC.

Teza autora: ryzyko przechodzi na akcjonariuszy, a nie na dostawców komponentów.

**Analiza:** Wyprzedanie HBM na lata oznacza, że ceny i wolumeny pamięci są zablokowane po stronie podaży. Spadek popytu nie przełoży się szybko na tańsze HBM, ale na kary za take-or-pay lub renegocjacje. Dla capacity planningu wynika z tego, że HBM, a nie wafle logiki, pozostaje twardym limitem dostaw akceleratorów w 2027–2028. Nabywcy, którzy nie mają slotów w alokacjach Nvidii lub AMD, nie kupią mocy nawet przy spadku cen GPU. Rozjazd fizyka–finanse to klasyczny sygnał późnej fazy cyklu, w której moce są zakontraktowane przy kosztach kapitału, których rynek już nie akceptuje.

Źródło: [The Business Engineer](https://businessengineer.ai/p/this-week-in-the-ai-supercycle-a)

## Implikacje praktyczne

1. **Tożsamość dostawcy jako wektor ataku na Kubernetes i chmurę.** Rackspace Spot i Entra Connect Health pokazują, że kompromitacja po stronie dostawcy omija kontrole klienta. Decyzja na 3–5 lat: sekrety poza etcd (zewnętrzny KMS/Vault, workload identity, krótki TTL) oraz procedura rotacji niezależna od konsoli dostawcy. Ryzyko: dłuższe okno ekspozycji, gdy dostawca odcina panel na czas sanityzacji.
2. **Break-glass i własna detekcja awarii hyperscalera.** Awaria IAM Identity Center w us-east-1 i ponad 90 minut opóźnienia status page'a wymagają kont awaryjnych poza federacją SSO oraz syntetycznych testów logowania i API. Bez tego RTO zależy od komunikacji dostawcy.
3. **Koszt przyłączenia energii jako jawna pozycja TCO DC w USA.** BAAJA (jeśli przejdzie w Izbie) przenosi ok. 55 mld USD kosztów przesyłu na DC ≥20 MW. W modelach colo, BTS i leasingu GPU trzeba zakładać wzrost stawek lub opłaty przyłączeniowe pass-through. W wyborze lokalizacji większą wagę zyskują regiony z istniejącą rezerwą przesyłu.
4. **Ryzyko regulacyjne kontraktów z klientami z Chin.** Ujawnienie licencji BIS i reguła o zdalnym dostępie zwiększają prawdopodobieństwo cofnięcia lub zaostrzenia licencji. Dla dostawców mocy oznacza to klauzule wyjścia i realokacji w kontraktach. Dla odbiorców: due diligence struktury klientów u neocloudu, bo licencja cofnięta sąsiadowi może zmienić finanse operatora.
5. **Suwerenne akceleratory UE jako opcja, nie plan bazowy.** MoU w rodzaju E4–Openchip warto śledzić pod przetargi publiczne, ale do 2028 r. platformę bazową wyznaczają Nvidia i AMD z dojrzałym stosem inferencyjnym. Przenośność warstwy serwującej (vLLM/SGLang, OpenAI-compatible API) ogranicza koszt ewentualnej migracji.

## Trend tygodnia

Weekend potwierdza, że ograniczeniem infrastruktury AI coraz częściej są polityka i kapitał, a nie krzem. Senat USA przerzuca koszty sieci na duże DC, a Kongres ujawnia, jak licencjonowano eksport akceleratorów do Chin. Obie decyzje zmieniają ekonomikę projektów, nie zmieniając ich technologii. Alibaba sygnalizuje możliwe obniżenie celu 20 GW, co jest pierwszym wyraźnym sygnałem cięcia capex dużego nabywcy spoza USA. Jednocześnie HBM pozostaje wyprzedane na lata, więc podaż jest sztywna, a ryzyko spadku popytu przenosi się na bilanse kupujących i operatorów. Neocloudy, jak Nebius w Izraelu, dalej dokładają moc w leasingu u stron trzecich, choć opóźnienia przyłączeń już raz kosztowały je obniżkę prognozy. W warstwie operacyjnej rośnie liczba incydentów, w których źródłem jest tożsamość lub control plane dostawcy (Rackspace Spot, Entra Connect Health, IAM Identity Center). To przesuwa ciężar odporności z „wielu regionów” na „niezależność od płaszczyzny zarządzania dostawcy”. Kierunek na najbliższe kwartały to droższa energia i kapitał, bardziej uregulowany eksport oraz architektury projektowane pod awarię dostawcy, a nie tylko sprzętu.

## To obserwować

1. **Ścieżka BAAJA w Izbie Reprezentantów i ewentualne rulemaking FERC** (próg 20 MW, płatności z góry czy rozliczenie w taryfie). Kluczowe jest, czy ustawa przejdzie przed zmianą większości w styczniu.
2. **Alibaba i Tencent: zmiany celów capex i mocy DC w kolejnych komunikatach** (cel 20 GW) oraz ewentualne cofnięcia licencji BIS po ujawnieniu danych przez Masta.
3. **Nebius: status przyłącza w Masmiya i harmonogram Beit Shemesh II**, a także czy w IV kw. pojawią się kolejne korekty prognozy z powodu lokalizacji stron trzecich.
4. **Rackspace / Platform9: raport poincydentalny Spot** (wektor wycieku poświadczeń, liczba klientów) oraz Microsoft: liczba tenantów dotkniętych luką Entra Connect Health.
5. **Kontraktowe ceny HBM i DRAM na 2027 r. oraz alokacje HBM4/HBM4E** jako wskaźnik, czy wyprzedana podaż przechodzi w renegocjacje take-or-pay.
