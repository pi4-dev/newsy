# Newsy — 2026-11-04

**Okno przyrostowe:** od raportu 2026-11-03. Pominięte sekcje: euro neocloud (brak nowych zweryfikowanych pozycji, ostatnie znalezione: Nebius 26.08, Nscale 26.09), AI for networking. Źródła: newslettery z etykiety NEWSY, wyszukiwanie WWW.

## Raport technologiczny

### Biznes

#### Marvell przejmuje Lumentum: umowa ostateczna (2.11), ok. 100–115 mld USD
Aktualizacja wpisu z 1.11 (wcześniej tylko rozmowy). Transakcja all-stock, 4,5 akcji Marvell za akcję Lumentum; wg SEC implikowany EV ok. 115 mld USD (premia ok. 24%), Marvell 67% / Lumentum 33% akcjonariatu. Część doniesień podawała 80 mld USD (to wartość rynkowa przed ogłoszeniem).
**Znaczenie:** Konsolidacja dostaw lasera/optyki (CPO, transceivery) z DSP/ASIC w jednym dostawcy; ryzyko koncentracji i vendor lock-in w optyce skali AI oraz długi czas zamknięcia (regulatorzy).
Źródło: https://www.sec.gov/Archives/edgar/data/0001633978/000119312526442760/d163169d425.htm

#### Microsoft i Lumen: wspólna sprzedaż sieci „AI-ready”
Co-sell łączności hybrydowej/multicloud, rozważane przeniesienie dodatkowych usług Lumen do Azure.
**Znaczenie:** Sygnał, że przepustowość WAN do workloadów agentowych jest sprzedawana pakietowo z chmurą; sprawdzić koszty egress i zależność od jednego carriera.
Źródło: https://ir.lumen.com/news/news-details/2026/Lumen-and-Microsoft-Expand-Collaboration-to-Co-Sell-Enterprise-AI-Ready-Networks/default.aspx

### Technologia

#### Langflow: dwie krytyczne luki bez uwierzytelnienia (CVE-2026-93680, CVE-2026-110186)
Wersje 1.0.0–1.12.8. Pierwsza omija uwierzytelnienie, druga daje wykonanie kodu przez enumerację narzędzi MCP; publiczny moduł Metasploit już istnieje. Jedyna poprawka: 1.13.0 (wyłącza automatyczne logowanie), brak obejścia.
**Znaczenie:** Instancje Langflow wystawione do sieci traktować jako skompromitowane do czasu aktualizacji; rotować klucze API i sekrety w flow, odciąć od Internetu.
Źródło: https://thehackernews.com/2026/11/critical-langflow-flaws-could-let.html

#### Atak na chłodnictwo sklepów w 7 krajach UE
Podejrzewany cyberatak przełączył parowniki w tryb odszraniania (Belgia, Luksemburg, Szwecja, Irlandia, Holandia, Estonia, Finlandia); temperatury do 50°C, straty (Delhaize ok. 100 tys. EUR). Dostawca wskazuje, że jednostki wymagają zmian fizycznych; zdalna konserwacja pozostaje w modelu operacyjnym.
**Znaczenie:** Zdalny dostęp serwisowy OT/IoT jako wektor; segmentacja, brokered access z MFA i fizyczne limity bezpieczeństwa (niezależne od sterownika).
Źródło: https://www.bbc.com/news/articles/cq1kd1nee80eo

#### Cloudflare DDoS Q3 2026: więcej krótkich roju, mniej gigantów
18,2 mln ataków L3/4; ataki <10 min i <1 Gbps +208% r/r, >1 Tbps −69% względem Q2; amplifikacja DNS +215%, UDP flood +122%, CLDAP −99,5%.
**Znaczenie:** Mitygacja oparta o progi i ręczną reakcję zawodzi przy krótkich seriach; potrzebna automatyczna detekcja w sekundach i ochrona resolverów przed amplifikacją.
Źródło: https://blog.cloudflare.com/ddos-threat-report-2026-q3/

#### ChatGPT: przejęcie kont przez HTML Previews (łańcuch MathLive → iframe → YouTube oEmbed)
Obejście CSP OpenAI przez iframe ładujący skrypty z domeny dozwolonej, następnie dostęp do endpointów API i kradzież danych odwiedzających link udostępnienia.
**Znaczenie:** Allowlisty CSP na domenach z endpointami oEmbed/JSONP są słabe; dla własnych aplikacji z renderowaniem treści generowanych przez LLM stosować sandbox z osobnym originem.
Źródło: https://simonwillison.net/2026/Oct/31/how-we-took-over-chatgpt-accounts/

#### WorkOS: incydent z użyciem AI-owego frameworku ofensywnego (30.10)
Dostęp do wewnętrznej aplikacji sprzedażowej: dane kontaktowe, historie kont klientów, fragmenty korespondencji. Zewnętrzny dostęp wyłączony, analiza trwa.
**Znaczenie:** Klienci WorkOS powinni zakładać phishing ukierunkowany z użyciem wyciekłej korespondencji; sprawdzić logi SSO/Directory Sync pod kątem anomalii.
Źródło: https://workos.com/security/bulletins/october-30th-2026-data-security-incident

## Raport AI-ML

### Biznes

#### SpaceX/xAI: cel 13 GW mocy DC w 2027 (SemiAnalysis), 3,6 GW w budowie
Czasy budowy: Colossus 1 retrofit 122 dni/150 MW, Colossus 2 91 dni/210 MW, nowy obiekt „Project SPAM” od 5.10 z celem uruchomienia w grudniu; Starbase może przekroczyć 1 GW; BESS ok. 5,9 GWh. Moc pod zarządzaniem >8 GW vs >9 GW DC online. Colossus 2: limit Southaven 2,7 GW nameplate obniża się do 1,24 GW od lipca 2027 (umowa z 30.07.2026), przyłącze TVA ok. 1,2 GW, możliwy niedobór ok. 300 MW. Prawdopodobne dzierżawy dla podmiotów trzecich (TeraFab).
**Analiza:** Modularne MEP i turbiny mobilne skracają build do ok. 4–5 mies., ale zasilanie opiera się na turbinach tymczasowych i BESS cyklowanym przez gaz; ryzyka: ograniczenia środowiskowe, TCEQ, derating. Nowy dostawca mocy GPU skali wielu GW zmienia ceny dzierżaw dla neocloudów. Analiza częściowo za paywallem, dane niezależnie niezweryfikowane.
Źródło: https://newsletter.semianalysis.com/p/elon-is-doing-it-again-spacex-will

#### Gartner: koszty oparte o tokeny mogą przekroczyć plany o >50% do 2027
Agenci AI w triage incydentów, walidacji zmian i remediacji; zalecenie: mierzalne use case'y, observability, hybrydowa infrastruktura.
**Analiza:** Budżetować na podstawie p95 zużycia tokenów na zadanie i wprowadzić routing modeli (Snowflake raportuje do 3x lepszą efektywność tokenów przy routingu).
Źródło: https://www.itpro.com/infrastructure/ai-is-crucial-to-infrastructure-and-it-operations-but-only-if-its-done-right-gartner-report-says

### Technologia

#### Anthropic Message Threads (public beta): przesyłanie tylko nowych wiadomości
Redukcja mediany TTFT o ok. 35% dla żądań 500K–1M tokenów, bez zmiany zużycia tokenów.
**Analiza:** Eliminuje retransmisję i ponowne przetwarzanie długiego kontekstu po stronie klienta; dla agentów długiego kontekstu mniejszy ruch i latencja, kosztem stanu po stronie dostawcy (lock-in, retencja danych).
Źródło: https://claude.com/resources/articles/message-threads

#### GPU-accelerated Weight Teleportation: synchronizacja wag trainer→sampler w RL
Pauza <0,23 s dzięki kompresji/dekompresji na GPU (LZ4/Snappy + warstwa Zstd).
**Analiza:** Skraca bubble w pętlach on-policy RL; przepustowość sieci między pulami staje się mniej krytyczna. Wynik z jednego opisu producenta.
Źródło: https://humansand.ai/blog/gpu-accelerated-weight-teleportation

#### Trening na „wyspach obliczeniowych” (mieszane generacje i dostawcy)
Pojedynczy run może łączyć mniejsze klastry z różnymi akceleratorami, każdy z własnym harmonogramem; zwiększa pulę użytecznej mocy, w tym starszych GPU.
**Analiza:** Ryzyko integralności: jeden wadliwy lub skompromitowany węzeł może po cichu zdegradować model; potrzebna weryfikacja gradientów i atestacja.
Źródło: https://www.coreauto.com/hitchhikers-guide-to-training-on-compute-islands

#### OpenAI Pando: nowy backend Git dla monorepo
Mediana latencji push spadła ok. 5x, ~1 PB danych Git dziennie, wdrożenie bez zmiany workflow.
**Analiza:** Ruch generowany przez agentów kodujących (równoległe pushe do wielu gałęzi) wymaga redesignu warstwy VCS; GitHub raportował wcześniej 5x r/r commitów.
Źródło: https://links.tldrnewsletter.com/sN4IQP

## Ryzyka rynku AI/GPU

#### Anthropic: zmieniony S-1 z wynikami Q3
Przychód 33,24 mld USD za 9 mies. 2026 (+1200%), strata netto 104 mld USD, zobowiązania compute/infrastruktura ok. 895,4 mld USD; spółka zastrzega, że może wkrótce osiągnąć wewnętrzną definicję RSI. Uzupełnia wcześniejszy wpis o S-1 (pierwszy kwartał rentowny) o twarde liczby.
**Analiza:** Zobowiązania ok. 27x przychodu 9-miesięcznego; strata netto tej skali przy rentownym kwartale sugeruje duże pozycje niegotówkowe (nie zweryfikowano) — sprawdzić w S-1. Wszystkie wieloletnie kontrakty compute u dostawców są kotwiczone w jednym kredycie.
Źródło: https://www.cnbc.com/2026/11/02/anthropic-ant-s-1-quarter-financials.html

#### Grok: ograniczenie limitów użycia z powodu kosztów
Musk potwierdził obniżenie limitów, bo koszty obsługi były nie do utrzymania względem przychodu.
**Analiza:** Sygnał ujemnej marży inferencji po stronie konsumenckiej, w kontraście do planów 13 GW.
Źródło: https://links.tldrnewsletter.com/jqaKVm

#### Marvell–Lumentum all-stock i niedobór zasilania Colossus 2
Patrz wpisy wyżej: transakcja wyceniona w papierach (ryzyko wyceny przy korekcie półprzewodników) oraz obniżenie limitu turbin Southaven do 1,24 GW od 7/2027.
Źródło: https://www.sec.gov/Archives/edgar/data/0001633978/000119312526442760/d163169d425.htm

## Newsletters summary

#### GPT-6.2 Astra w Codex (A/B), pominięty 6.1 ze względu na obawy o oszustwo
Doniesienia testingcatalog; premiera możliwa w ciągu dni, niepotwierdzona przez OpenAI.
**Analiza:** Planować ewaluację regresji promptów dopiero po ogłoszeniu.
Źródło: https://www.testingcatalog.com/gpt-6-2-astra-appears-codex-usage/

#### Weight-space programming: edycja wag przez LLM
MIT: GPT-6 Astra poprawił matematykę modelu Qwen 0,8B edytując wagi bezpośrednio; Goedecke zauważa, że normalny trening pozostaje lepszy.
**Analiza:** Kierunek istotny dla bezpieczeństwa łańcucha dostaw modeli (modyfikacja wag po walidacji); praktycznie wciąż badawczy.
Źródło: https://www.seangoedecke.com/llms-are-not-superhuman-at-reading-and-writing-neural-network-weights/

#### „Agent z jednym narzędziem exec”
Katalog narzędzi zwijany do jednego exec w sandboxie; skrypty łączą akcje w jednym przebiegu; Framer Harness 2: −33% kosztu, −43% kroków narzędziowych.
**Analiza:** Mniej round-tripów modelu, ale cała powierzchnia ataku przenosi się na sandbox; wymaga izolacji OS i limitów sieci.
Źródło: https://www.philschmid.de/code-mode

## Implikacje praktyczne
1. Wieloletnie umowy compute z dostawcami opartymi na turbinach tymczasowych/BESS (SpaceX, neocloudy) wymagają klauzul SLA zasilania i planu B w innym regionie.
2. Konsolidacja optyki (Marvell–Lumentum) przy CPO: wymagać interoperacyjności i drugiego źródła transceiverów w przetargach 2027+.
3. Narzędzia low-code do agentów (Langflow) i dostęp serwisowy OT traktować jako powierzchnię zewnętrzną: brak ekspozycji do Internetu, brokered access, MFA.
4. Budżetowanie AI na podstawie zużycia tokenów z routingiem modeli i twardymi limitami (spend caps) zamiast stałych założeń.
5. Zakładać krótsze, częstsze ataki DDoS: automatyczna mitygacja w sekundach, ochrona resolverów DNS.

## Trend tygodnia
Skala budowy infrastruktury AI przyspiesza szybciej niż jej zasilanie: SpaceX celuje w 13 GW w 2027, budując obiekty w 4–5 miesięcy, lecz opiera się na turbinach mobilnych, BESS i przyłączu, które nie pokrywa całego zapotrzebowania. Jednocześnie finansowanie pozostaje wrażliwe: zmieniony S-1 Anthropic pokazuje zobowiązania około 895 mld USD wobec 33 mld USD przychodu w 9 miesięcy, a Grok ogranicza użycie z powodu kosztów. Konsolidacja łańcucha dostaw (Marvell–Lumentum za ok. 100–115 mld USD w akcjach) koncentruje optykę w jednym dostawcy. Po stronie bezpieczeństwa dominują zdalnie dostępne narzędzia i interfejsy serwisowe (Langflow, chłodnictwo w 7 krajach, ChatGPT HTML Previews), a ataki DDoS fragmentują się na krótkie roje. Równolegle agenci kodujący zmieniają profil obciążenia (Pando, message threads), przesuwając wąskie gardła z obliczeń na stan, VCS i sandbox.

## To obserwować
- Zamknięcie transakcji Marvell–Lumentum i reakcja regulatorów.
- Faktyczne uruchomienie Project SPAM (grudzień) i decyzje TVA/Southaven.
- Poprawki Langflow 1.13.0 i skala eksploatacji (Metasploit).
- Premiera GPT-6.2 Astra oraz ceny kontekstu w Message Threads.
- Pozycje niegotówkowe i harmonogram zobowiązań w S-1 Anthropic.
