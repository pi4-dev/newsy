# Newsy — 2026-11-01

**Okno przyrostowe:** od raportu 2026-10-31 do 2026-11-01. Pominięte sekcje: raport technologiczny, AI-ML, euro neocloud, AI for networking (brak nowych wiarygodnych pozycji; wyszukiwarka WWW zwraca głównie materiały nieaktualne, RSS HPCwire niedostępny – 403; Marvell/Lumentum bez oficjalnego potwierdzenia). Dzień wolny (niedziela).

## Ryzyka rynku AI/GPU

#### Capex hyperscalerów vs operating cash flow (Q3): przekroczenie przepływów operacyjnych
(FINANSE) Wg newslettera The Business Engineer (liczone z raportów kwartalnych, „by our arithmetic"): Alphabet 51,4 mld USD capex vs 42,3 mld USD OCF (121%), Amazon 77,3 mld vs 42,4 mld (ok. 1,8x, 48,1 mld USD nowego długu w kwartale), Microsoft 39,4 mld vs 58,0 mld (68%). S&P zmienił perspektywę Meta na negatywną. AWS +39% r/r do 46,0 mld USD, capex 2026 ok. 220 mld USD. Finansowanie budowy przesuwa się z rachunku wyników na bilans (dług, leasingi niestartowane, gwarancje) – wrażliwość na koszt długu rośnie. Źródło wtórne (newsletter), liczby do weryfikacji w filingach.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-anthropics

#### Selektywność kapitału: spread MSFT ~64 pb, raportowany rating poniżej investment grade dla Anthropic, OpenAI tnie rundę
(FINANSE/POPYT) Microsoft sprzedał 25 mld USD obligacji ok. 64 pb nad Treasuries; Blackstone miał zapowiedzieć inwestorom rating poniżej investment grade dla Anthropic; OpenAI (wg FT/WSJ) pozyskał „mniej niż połowę" celu 30 mld USD, ok. 20 mld przy wycenie 1,25 bln (cel 1,4 bln). Koncentracja popytu: top 5 klientów Anthropic ≈25% przychodów (pierwszy publiczny filing), jeden klient AI = 34,5% przychodów SpaceX w Q3. Dla neocloudów kontraktujących pod laboratoria oznacza to wyższy koszt refinansowania i ryzyko kontrahenta. Wszystko raportowane, niepotwierdzone przez strony.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-anthropics

#### Pamięć jako ograniczenie: marże SK hynix 80% (operating), Sandisk 85% gross
(DOSTAWY/EKONOMIKA) SK hynix operating margin 80%, Sandisk przychód 11,0 mld USD (+378% r/r, DC 4,8 mld USD) przy 85% marży brutto. Sprzedawcy komponentów (HBM/NAND) przejmują marżę, co podnosi BOM serwerów AI i koszt capacity planningu; kontrakty wieloletnie i alokacje stają się ważniejsze niż cena spot.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-anthropics

## Newsletters summary

#### NVIDIA Dynamo na Kubernetes: zwalidowany runtime
Dynamo (rozproszona inferencja na wielu GPU) wymaga zintegrowanego stosu: sterowniki GPU, alokacja zasobów, scheduling, sieć i monitoring. Wpis opisuje zwalidowaną ścieżkę od konfiguracji modelu do endpointu.
**Analiza:** Kluczowe dla throughput/latency jest topologia sieci (RDMA/NVLink) i gang scheduling disaggregated prefill/decode; ryzyko lock-in w stos NVIDIA. Przed produkcją sprawdzić zgodność wersji operatora GPU, DRA i obserwowalność na poziomie GPU. Treść znana tylko z podsumowania n8n, nie z pełnego wpisu.
Źródło: https://developer.nvidia.com/blog/build-a-validated-kubernetes-runtime-for-nvidia-dynamo/

#### Francja: podatek €0,055 za GB wysyłany przez dostawców treści; Google w rozmowach o Golden Dome
Dolna izba francuskiego parlamentu zagłosowała podatek od ruchu; CrowdStrike: grupa powiązana z Chinami uruchomiła agentów AI przeciw ponad 200 organizacjom (wg newslettera).
**Analiza:** Podatek od wolumenu danych zmienia ekonomikę CDN/peeringu i dystrybucji dużych artefaktów (obrazy, wagi modeli) do UE – uwzględnić w kosztach egressu i lokalizacji cache. Ustawa nie jest jeszcze w mocy; ryzyko precedensu dla innych państw UE. Źródło wtórne.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-anthropics

(Pominięte jako niedopasowane: NVIDIA edge AI quick-start.)

## Implikacje praktyczne
1. Koszt kapitału zastępuje podaż chipów jako ograniczenie: w umowach z neocloudami zabezpieczać się na ryzyko kontrahenta (escrow, step-in, ekspozycja na GPU kupione z długu).
2. Kontraktować pamięć/HBM z wyprzedzeniem; zakładać utrzymanie marż dostawców w BOM.
3. Wielodostawcza strategia modeli: koncentracja klientów wokół kilku laboratoriów to ryzyko dla ich dostawców mocy.
4. Egress i dystrybucja artefaktów do UE: uwzględnić ryzyko podatków od ruchu w TCO.
5. Inferencja rozproszona (Dynamo/vLLM) – standaryzować runtime K8s i obserwowalność, ograniczając lock-in.

## Trend tygodnia
Narracja budowy infrastruktury AI przesuwa się z historii wzrostu do historii kredytowej. Trzech z czterech największych budujących wydaje więcej niż generuje z operacji, więc różnicę pokrywają dług i leasingi. Rynek obligacji nadal przyjmuje najlepszych emitentów, ale podmioty o wąskiej bazie klientów (laboratoria) płacą wyraźnie więcej lub pozyskują mniej. Jednocześnie dostawcy pamięci zbierają marże bliskie 80%, co przenosi koszt na całą stawkę. Popyt rośnie (AWS +39%), więc ryzykiem nie jest brak popytu, lecz trwałość jego ekonomiki względem już podjętych zobowiązań. Regulacje (podatki od ruchu, kontrola danych) dodają kolejny wymiar niepewności. Wszystko wg źródła wtórnego – wymaga weryfikacji w filingach.

## To obserwować
- Oficjalne potwierdzenie transakcji Marvell/Lumentum i sposób finansowania.
- Rating i warunki długu Anthropic oraz finalna kwota rundy OpenAI.
- Stosunek capex/OCF hyperscalerów i perspektywy S&P (Meta).
- Marże i alokacje HBM/NAND (SK hynix, Sandisk).
- Losy francuskiego podatku od ruchu w Senacie.
