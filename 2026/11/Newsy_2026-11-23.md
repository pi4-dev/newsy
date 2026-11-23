# Raport Zbiorczy IT – 2026-11-23

**Okno przyrostowe:** od raportu z 2026-11-22 (weekend 21–23.11). Uwzględniono zdarzenia z 20–23.11, których nie ma w rejestrze z ostatnich 7 dni. Sekcje „euro neocloud” i „AI for networking” pominięto, bo nie pojawiły się w nich nowe, zweryfikowane informacje.

## Raport technologiczny

### Technologia

#### Kradzież promptów u dostawców inferencji: badanie canary Dragos, OpenRouter usuwa 3 dostawców i 125 endpointów z ZDR; incydenty Together AI i Phala (21–22.11)
Dragos (z udziałem HD Moore'a) między lipcem a listopadem wysłał ok. 2 mln promptów z unikalnymi linkami i poświadczeniami-pułapkami (canary) przez routery i dostawców inferencji, a potem obserwował, kto ich używa. Wyniki:
- Trzech nienazwanych dostawców inferencji było skompromitowanych przez miesiące i o tym nie wiedziało. Najwcześniejszy wyciek trwał od połowy lipca.
- W największym przypadku (koniec października) skradziono każde żądanie z danego okna, także odrzucone przed uruchomieniem modelu. Ruch był więc kopiowany już w punkcie wejścia (gateway/LB), a nie w warstwie modelu.
- Część skradzionych żądań miała flagę Zero Data Retention.
- Atakujący testował klucze AWS w ciągu kilku godzin: listował S3, przeszukiwał Secrets Manager i Parameter Store, próbował tworzyć klucze API Bedrock i użytkowników IAM (dziesiątki tysięcy wywołań API). Do sprawdzania kluczy użył frameworka ARTEX (user-agenty `artex-agent`, `artex-judge-r121`).
- Sprzedawcy na marketplace inferencji wystawiali tanie modele z włączonym logowaniem żądań. Jeden router przekazywał ruch do marketplace bez wiedzy klientów. Analiza ok. 2 000 ścieżek wykazała kilkaset niezidentyfikowanych systemów pośrednich i sprzedawcę doklejającego ok. 1 000 ukrytych tokenów do każdego promptu.
- Modele jednego chińskiego laboratorium pobierały sekretne linki z ponad połowy promptów, w tym z ok. 2/3 oznaczonych jako ZDR lub „no data collection”. Ten sam scraper uruchamiał się także u amerykańskich hostów odsprzedających te modele.

OpenRouter usunął trzech dostawców (bez nazw), wycofał 125 endpointów z kwalifikacji ZDR i powiadomił klientów. Ujawnione incydenty:
- **Phala:** przejęty serwer inferencji, żądania do DeepSeek V4 Flash 0731 od 21.10 00:00 do 8.11 05:35 UTC należy traktować jako ujawnione.
- **Together AI (komunikat CEO 21.11):** agent AI do kodowania wcommitował do publicznego repozytorium GitHub poświadczenie do prywatnego repo Hugging Face z próbkami promptów testowych z czerwca i lipca (MiniMax M3, GLM-5.2, Kimi K2.6). Skaner sekretów cofnął commit, ale atakujący zdążył skopiować dane, a potem przetestował znalezione w nich poświadczenia, w tym 5 canary Dragos. Together wprowadził krótkotrwałe i zawężone poświadczenia, skan sekretów w każdym PR, ZDR domyślnie dla nowych kont i nowe zasady dla agentów kodujących.
- **Novita:** błąd konfiguracji routingu przekazywał część żądań MiniMax do oficjalnego API MiniMax.

**Znaczenie:** ZDR to deklaracja kontraktowa, a nie kontrola techniczna. Jeśli ruch jest kopiowany na wejściu, flaga nie ma znaczenia. Przy łańcuchu router → reseller → host → laboratorium klient nie zna faktycznej ścieżki danych, więc ryzyko rośnie z każdym hopem. Wnioski:
- Prompty agentów (kontekst repo, logi, zmienne środowiskowe) zawierają sekrety. Potrzebne są filtrowanie formatów kluczy na egress do LLM, poświadczenia krótkotrwałe (STS, workload identity) zamiast statycznych kluczy oraz własne canary w kontekstach agentów, których każde użycie oznacza incydent.
- Dla danych wrażliwych: self-hosting (vLLM/SGLang) albo bezpośrednia umowa z dostawcą pierwszego poziomu z prawem audytu i jawnie zadeklarowaną ścieżką ruchu, bez marketplace i resellerów.
- Przypadek Together to kolejny failure mode agentów kodujących: commit sekretu do publicznego repo. Pre-commit i push protection muszą obejmować też tokeny HF i klucze inferencji, a agent nie powinien mieć dostępu do długożyjących poświadczeń.
- W SIEM warto dodać user-agenty ARTEX i alerty na `CreateUser` oraz tworzenie kluczy Bedrock z nietypowych ASN.

Źródło: [The Hacker News](https://thehackernews.com/2026/11/openrouter-drops-3-providers-after.html), [Unite.ai](https://www.unite.ai/openrouter-cuts-three-providers-after-ai-prompt-exposure-findings/), [Fox News – relacja z komunikatu Together AI](https://www.foxnews.com/live-news/ai-news-markets-correction-11-22-26)

## Raport AI-ML

### Biznes

#### Dell, HPE, Lenovo i Supermicro żądają podwyżek do 45% na serwery AI po podpisaniu umów (FT, 22.11)
Według Financial Times (relacje Digitimes 23.11 i innych) czterej główni dostawcy serwerów żądają od klientów dopłat sięgających 45% miesiące po podpisaniu umów, gdy sprzęt zbliża się do wysyłki. 45% to górna granica, nie średnia. Według części relacji serwery GPU podrożały po podpisaniu umów o ok. 20–30%, a niektóre systemy CPU ponad trzykrotnie, bo pamięć stanowi w nich większą część BOM. JPMorgan szacuje wzrost cen pamięci o ponad 400% od początku 2024 r. HPE zmienił warunki handlowe tak, by móc zmieniać cenę między złożeniem zamówienia a wysyłką (Neri nazywa to „agile pricing posture”). Precedens sporu: XTX (Finlandia) pozwał Dell w irlandzkim High Court o ok. 62–70 mln USD dopłaty do kontraktu na 1 680 serwerów (ok. 549 mln USD). XTX twierdzi, że cena była stała, a sprawę odroczono do grudnia. Dostawcy nie skomentowali doniesień.

**Analiza:**
- **Ryzyko kontraktowe:** cena serwera przestaje być stała między PO a dostawą. Kontrakty powinny mieć twardy price lock z limitem indeksacji powiązanym z publicznym indeksem DRAM i HBM, prawo odstąpienia bez kary przy przekroczeniu progu oraz escrow spornej kwoty, jak w modelu XTX. Bez tego dostawca może opóźniać wysyłkę do czasu akceptacji podwyżki.
- **Capacity planning:** przy budżecie stałym podwyżka o 20–30% oznacza o 17–23% mniej węzłów. W TCO na 2027 r. trzeba uwzględnić scenariusz cenowy pamięci, a nie tylko GPU. Systemy CPU-heavy (pamięć dla agentów, KV-cache offload, bazy wektorowe) są bardziej wrażliwe niż węzły GPU.
- **Kto jest najbardziej narażony:** hyperscalerzy przeniosą to w capex, ale neocloudy z kontraktami offtake o stałej cenie za GPU-h i finansowaniem pod te kontrakty (DDTL) tracą marżę albo muszą renegocjować z klientem. To kolejny element nożyc: rosnący koszt wejścia przy spadającej wycenie rynkowej.
- **Vendor lock-in:** odejście od jednego OEM (ODM direct, np. Foxconn/Quanta/Wiwynn, lub zakup przez chmurę) zmniejsza ekspozycję na jednostronne repricing, ale przenosi na kupującego integrację i wsparcie.

Źródło: [Digitimes](https://www.digitimes.com/news/a20261123VL206/ai-server-dell-price-hpe-lenovo.html), [Startup Fortune (za FT)](https://startupfortune.com/dell-hpe-lenovo-and-supermicro-raise-ai-server-prices-up-to-45-after-signing/), [Irish Times – XTX vs Dell](https://www.irishtimes.com/crime-law/courts/2026/06/15/finnish-company-sues-dell-over-70m-price-increase-for-data-centre-servers/)

## Ryzyka rynku AI/GPU

#### Apollo i Goldman wyceniły ok. 44 mld USD obligacji pod leasing GPU Vera Rubin dla SpaceX – największe finansowanie GPU w historii (20.11)
20.11, w dniu kapitulacji rynku, Apollo i Goldman Sachs (sole structuring agent) wyceniły ok. 44 mld USD obligacji senioralnych i podporządkowanych na zakup GPU Nvidia Vera Rubin i powiązanego sprzętu dla naziemnej infrastruktury compute SpaceX. Emisja była nadsubskrybowana i zwiększona. Inwestorzy kotwiczący: BlackRock, Goldman Sachs Asset Management i rachunki doradzane przez KKR. Struktura (8-K SpaceX z 26.10):
- podmiot zarządzany przez Apollo kupuje sprzęt i leasinguje go SpaceX w modelu master finance lease,
- łączne płatności kapitałowe ok. 40 mld USD,
- harmonogramy po ok. 5–5,25 roku, czynsz kwartalny z góry, ok. 1/20 kosztu transzy plus stała stopa,
- start po dostawie sprzętu w I połowie 2027 r.

Zawarcie leasingu podlega warunkom zamknięcia i żaden dokument nie potwierdza jeszcze podpisania. Podział transz z jednego źródła (CNA, niepotwierdzony): ok. 25,2 mld USD senior secured A3 do 2033 z kuponem ok. 8,26%, ok. 4,4 mld USD A2 do 2031 (7,66%), ok. 2,6 mld USD klasy B (8,36%).

**Analiza:**
- **Rynek otwarty, ale drogi.** Kredyt pod GPU jest dostępny dla kredytobiorcy z silnym bilansem. Przy kuponie ok. 8,3% dla transzy senior secured koszt długu znacznie przekracza spready IG hyperscalerów. Rynek wycenia ryzyko wartości rezydualnej GPU, a nie tylko ryzyko kredytowe najemcy.
- **Niedopasowanie terminów.** Amortyzacja w ok. 5 lat przy cyklu architektury Nvidii ok. 12–18 miesięcy oznacza, że na koniec leasingu zabezpieczenie będzie 3–4 generacje za czołówką. Pokrycie długu opiera się na czynszu, nie na odsprzedaży sprzętu.
- **Koncentracja popytu.** Ok. połowa kwartalnych wydatków Anthropic na compute trafia do SpaceX (rejestr 20.11). Ten dług pośrednio finansuje więc capacity dla jednego głównego najemcy. Spadek kursu ANT i zobowiązania wypowiadalne z S-1 (ok. 186 mld USD) to ryzyko łańcuchowe.
- **Koszt pieniądza rośnie.** Rynek wycenia ok. 94% prawdopodobieństwa podwyżki Fed o 25 pb na posiedzeniu 8–9.12 (tracker OIS na 23.11), a rentowność 10-latki była ok. 4,95% (10.11). Każde refinansowanie DDTL neocloudów w 2027 r. odbędzie się drożej. Kontrast: emisja SpaceX przeszła, a w tym samym tygodniu QTS ograniczył emisję, a banki zbywały ekspozycje DC (rejestr 19–20.11). Rynek się bifurkuje: kapitał trafia do kredytobiorców z bilansem, a nie do neocloudów z jednym klientem.

Źródło: [Reuters via Investing.com](https://www.investing.com/news/stock-market-news/apollo-goldman-lead-44-billion-spacex-gpu-debt-financing-5042346), [Ultrathink (analiza 8-K i transz)](https://ultrathink.ai/news/apollo-goldman-price-44-billion-spacex-gpu-notes), [Rate Probability (OIS)](https://rateprobability.com/fed), [CNBC](https://www.cnbc.com/2026/11/10/market-spooked-because-now-the-fed-may-need-a-supersized-hike-in-december.html)

## Newsletters summary

#### „The Persistent Agent”: agenci działający w tle stają się kategorią obciążenia; SemiAnalysis – 45% własnych wydatków na AI (The Business Engineer, 22.11)
Przegląd zmiany od czatu do agentów trwałych, którzy budzą się sami, mają pamięć i działają w oparciu o cel. Produkty z ostatnich 10 tygodni: Meta Muse, OpenAI Dots, Perplexity „Teammates”, a LangChain pozyskał 300 mln USD na infrastrukturę dla takich agentów. Według Dylana Patela (SemiAnalysis, 11.11) agenci trwali to 45% wydatków firmy na AI, wobec zera na początku roku. Harrison Chase (webinar LangChain 20.11) wskazuje cztery właściwości: cel, harmonogram wybudzeń, pamięć i uprawnienia. Test Arrington Capital to pięć pytań: co agent obserwuje, co go wybudza, jaki stan trzyma, co może zrobić bez pytania i jak ocenia ukończenie zadania. Wątek prymitywów runtime'u (event inbox, trwałe timery) jest w rejestrze z 17.11, więc tutaj nowe są dane popytowe i mapa rynku.

**Analiza:** Dla infrastruktury agent trwały ma inny profil obciążenia niż czat:
- **Ruch:** to nie interaktywny ruch z pikami w godzinach pracy, lecz ciągłe, zdarzeniowe wywołania o długim horyzoncie. Pozwala to na lepszą utylizację (batching, off-peak), ale wymaga trwałego stanu: pamięci epizodycznej, checkpointów i kolejek zdarzeń z gwarancją dostarczenia.
- **Koszt:** dominuje prefill długiego kontekstu przy każdym wybudzeniu, więc rośnie znaczenie współdzielonego i rozproszonego KV cache (np. LMCache/Valkey, CXL-pool) oraz prompt cachingu.
- **Failure modes:** pętle wybudzeń bez warunku stopu (niekontrolowany koszt), dryf pamięci i eskalacja uprawnień w czasie. Konieczne są budżety tokenów per agent, circuit breakery i przegląd uprawnień, jak dla kont serwisowych.
- Incydent Together AI (sekcja 1) pokazuje, że agent z dostępem do repo i poświadczeń to nowa klasa ryzyka wycieku.

Udział 45% to deklaracja jednej firmy, a nie dane rynkowe.

Źródło: [businessengineer.ai](https://businessengineer.ai/p/the-persistent-agent)

## Implikacje praktyczne

1. **Kontrakty sprzętowe z twardym price lockiem.** Wobec repricingu serwerów do 45% po podpisaniu każde PO na 2027 r. powinno mieć limit indeksacji powiązany z indeksem DRAM/HBM, prawo odstąpienia i escrow spornej kwoty. W TCO na 3–5 lat trzeba modelować pamięć jako główną zmienną kosztową, także dla serwerów CPU (agenci, bazy wektorowe, KV offload).
2. **Inferencja przez strony trzecie jako kanał wycieku.** ZDR deklarowane przez routery i resellerów nie jest kontrolą techniczną. Dla danych wrażliwych: self-hosting albo jeden dostawca pierwszego poziomu z audytem ścieżki ruchu, DLP na egress do LLM, krótkotrwałe poświadczenia w kontekstach agentów i canary jako stały mechanizm detekcji.
3. **Koszt kapitału jako ograniczenie capacity.** Kupony ok. 8% dla zabezpieczonego długu pod GPU i oczekiwana podwyżka Fed w grudniu oznaczają, że finansowanie leasingowe GPU w horyzoncie 3–5 lat będzie dostępne głównie dla kredytobiorców z silnym bilansem. Przy wyborze neocloudu jako dostawcy trzeba ocenić strukturę jego długu i terminy refinansowania, a nie tylko cenę GPU-h.
4. **Ryzyko wartości rezydualnej.** Leasingi 5-letnie na sprzęt z 12–18-miesięcznym cyklem generacji wymagają planu drugiego życia dla GPU (inferencja, kaskadowanie do tańszych tierów). Własne klastry warto amortyzować w 3–4 lata, a nie 6.
5. **Agenci trwali jako nowa klasa obciążenia.** Planując capacity na 2027–2029 r., trzeba uwzględnić ciągłe obciążenie tła (prefill długiego kontekstu, trwały stan, kolejki zdarzeń) i zarządzać nim jak kontami serwisowymi: budżety, circuit breakery, okresowy przegląd uprawnień.

## Trend tygodnia

Po kapitulacji rynku 20.11 rynek kapitałowy wyraźnie rozdziela kredytobiorców: SpaceX zamyka rekordowe 44 mld USD długu pod GPU, a w tym samym tygodniu banki pozbywają się ekspozycji DC, a neocloudy z jednym klientem tracą dostęp do finansowania. Jednocześnie koszt wejścia rośnie zamiast spadać: OEM-y przenoszą wzrost cen pamięci na podpisane kontrakty, a Fed wycenia kolejną podwyżkę. To ściska marżę każdego operatora, który sprzedał GPU-h po stałej cenie i finansuje się długiem. Fizyczny niedobór (pamięć, PCB) i finansowa ostrożność działają więc w tym samym kierunku: mniej, ale droższej mocy, skoncentrowanej u graczy z bilansem. Równolegle dojrzewa warstwa aplikacyjna. Agenci trwali stają się realną pozycją w budżetach, a to generuje stałe obciążenie inferencyjne niezależne od nastrojów giełdowych. Ta sama zmiana poszerza powierzchnię ataku: łańcuch dostaw inferencji (routery, resellerzy, hosty) okazał się nieszczelny, a agenci kodujący sami wyciekają poświadczenia. W najbliższych kwartałach przewagę zyskają organizacje, które kontrolują jednocześnie koszt kapitału, kontrakty sprzętowe i ścieżkę danych do modeli.

## To obserwować

- **Decyzja FOMC 8–9.12** (25 pb czy 50 pb) i jej wpływ na spready obligacji DC/ABS oraz kursy obligacji CoreWeave.
- **Repricing serwerów:** czy Dell/HPE potwierdzą podwyżki w komentarzach kwartalnych (Dell raportuje pod koniec listopada) oraz rozstrzygnięcie XTX vs Dell w irlandzkim Commercial Court (grudzień).
- **Lista dostawców inferencji usuniętych przez OpenRouter** i ewentualne ujawnienie nazw przez Dragos; zmiany w definicjach ZDR u routerów i resellerów.
- **Zamknięcie leasingu SpaceX/Apollo** (8-K) oraz harmonogram dostaw Vera Rubin w I połowie 2027 r. jako wskaźnik realnych wolumenów Rubin.
- **AWS re:Invent (29.11–3.12):** ceny instancji GPU i Trainium, ewentualne podwyżki za pamięć i storage.
