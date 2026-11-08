# Newsy — 2026-11-08

**Okno:** od raportu 2026-11-07. Pominięto sekcje: raport technologiczny, euro neocloud, AI for networking (brak nowych zweryfikowanych informacji w oknie; ostatnie zdarzenia neocloudów – Nebius/Citi 3,5 mld USD z 22.10 – już poza oknem). Źródła: newslettery z etykiety NEWSY, WWW.

## Raport AI-ML

### Technologia

#### SemiAnalysis: kampus Galaxy (Ulanqab, Wewn. Mongolia) – 800 MW IT, pilot 5 MW SST→800 V DC, chłodzenie bez central chillerów
Pierwsza faza 120 MW oddana w sierpniu 2026 w <12 mies. przy ok. 1/5 kosztów USA; Envision zintegrował własne wiatraki (420 MW na 240 MW obciążenia, >60% energii z wiatru, BESS 4 h), BESS, moduły LV, prefabrykowaną podstację (<3 mies. vs ~6) i 800 V DC z SST (10 kV AC → 800 V DC, bez transformatorów LV, UPS i prostowników). Jedna pętla wodna dla serwerów cieczowych i powietrznych. Chiny: capex <3 mln USD/MW, czynsz ok. 40 USD/kW/mies. (-50% w 5 lat), energia ok. 0,055 USD/kWh.
**Analiza:** Referencyjny punkt dla roadmapy 800 V DC (Kyber ~600 kW/rack) i dla debaty o chillerach (Vera Rubin na wodzie 45°C). Przewaga to czas energizacji i koszt, nie technologia GPU; ryzyko: 5 MW pilot ≠ skala GW, SST nadal niedojrzałe operacyjnie. Pełny raport za paywallem.
Źródło: https://newsletter.semianalysis.com/p/we-visited-chinas-stargate-in-inner

#### NVIDIA: Reshard Collective – odświeżanie wag w RL przy rozdzielonych GPU treningu i inferencji
W RL z rozdzielonym wdrożeniem (inferencja i trening na osobnych GPU) wagi polityki muszą być okresowo synchronizowane z GPU inferencyjnymi; artykuł opisuje przyspieszenie tego refitu (resharding między różnymi układami równoległości).
**Analiza:** Refit jest narzutem na każdą iterację RL; krótszy czas zmniejsza idle GPU inferencyjnych i poprawia on-policy-ness. Dotyczy planowania sieci (NVLink/IB) między pulami trening/inferencja. Opis ze streszczenia – szczegółów wydajnościowych nie zweryfikowano.
Źródło: https://developer.nvidia.com/blog/accelerating-reinforcement-learning-weight-refit-with-the-reshard-collective/

#### FLUX 3 Image Klein (Black Forest Labs) na NVIDIA Blackwell
Optymalizacja inferencji modeli dyfuzji (sekwencyjne przejścia, koszt rośnie z rozdzielczością) na Blackwell.
**Analiza:** Latencja dyfuzji jest ograniczona liczbą kroków; optymalizacje sprzętowe (niska precyzja) to główna dźwignia kosztu na obrazek. Wpis o niskiej wadze dla capacity planningu. 
Źródło: https://developer.nvidia.com/blog/speed-up-image-generation-with-flux-3-image-klein-on-nvidia-blackwell-gpus/

## Ryzyka rynku AI/GPU

#### Wyprzedaż spółek AI/chipów 5–6.11: skala niepotwierdzona jednoznacznie
The Business Engineer podaje: 12 spółek AI -1,45 bln USD w czwartek, Nvidia -9,3%, KOSPI wstrzymany po -8,6% w piątek, TSMC limit-down, rekordowy CDS SpaceX (286,56 pb), najgorszy dzień obligacji dolarowej SoftBank od 10 tyg.; 9 z 10 raportujących spółek AI pobiło szacunki, 4 spadły. Niezależne źródło (tracker) potwierdza wyprzedaż 5.11 (ok. -930 mld USD do 10:04 ET), lecz z innymi procentami (Nvidia ok. -5,4% intraday) i bez potwierdzenia halt KOSPI w tej dacie.
**Analiza:** Sygnał przejścia ryzyka z podaży do kredytu/wycen; kanał transmisji do neocloudów: koszt długu i wyceny zabezpieczeń (GPU-backed loans). Liczby traktować ostrożnie do czasu potwierdzenia z notowań.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-the-9e5 ; https://fourweekmba.com/ai-nvidia-leads-930b-drop-in-12-ai-stocks-by-10-04-a-m-et/

#### Aktualizacja: Anthropic IPO – wycena 1,2–1,3 bln USD (Bloomberg, wg newslettera); S-1: Q3 ok. 17 mld USD, zobowiązania ok. 895 mld USD
Nowy element względem rejestru: widełki 1,2–1,3 bln (z 1,5–1,6 bln), OpenAI domknęło ok. 11 mld USD przy 1,25 bln (pierwsza transza rundy do 20 mld; SoftBank rozważa <1 mld). Press WSJ podaje 1,5–1,6 bln – źródła rozbieżne; cena nie ustalona.
**Analiza:** Zobowiązania 895 mld vs przychód ~68 mld/rok (run-rate z Q3) – koncentracja ryzyka po stronie dostawców compute. OpenAI+Anthropic mają kupić ok. 30% nowego compute w 2027 (Patel/SemiAnalysis) – ryzyko koncentracji kontrahentów.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-the-9e5 ; https://stocktwits.com/news-articles/markets/equity/anthropic-updates-ipo-filing-as-investors-push-for-lower-valuation-discloses-17-b-in-q3-revenue-slowing-growth-and-nearly-900-b-in-commitments/cNGQpAqR95R

#### Wybory 3.11: 40 z 41 lokalnych głosowań ograniczyło rozwój data center; nowi gubernatorzy w stanach z ok. 41% planowanej mocy DC
Ballotpedia (via newsletter): głosowania cross-party; Ratepayer Protection Act przeszła w Izbie 417:3 (koszty infrastruktury na DC). Capex Big Tech+SpaceX Q4 ok. 270 mld USD (> Q3). SemiAnalysis: ok. 3/4 mocy na 2027 już w budowie, większość dodatków 2028 jeszcze nie.
**Analiza:** Ryzyko przesuwa się na 2028 (brak startu budów + zgody). Dodatkowy koszt przyłączeń/taryf – kalkulować w PPA i modelach marży. Uzupełnia wpis o Teksasie z 7.11.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-the-9e5

#### Pamięć: przychód DRAM +369% do ok. 200 mld USD przy ok. 4% więcej bitów (Counterpoint); HBM +121% w 2027
**Analiza:** Wzrost to cena, nie wolumen – koszt serwera AI i capex rosną bez proporcjonalnej mocy; interakcja z deklaracją CXMT 4F² (7.11) jako potencjalnym czynnikiem korekty cen 2027. Dane przez newsletter.
Źródło: https://businessengineer.ai/p/this-week-in-the-ai-supercycle-the-9e5

## Newsletters summary

#### SemiAnalysis, The Business Engineer, "Own news" (n8n, NVIDIA blog)
Przetworzone wyżej (Galaxy/800 V DC; mapa ryzyka: pozwolenia, kredyt, pamięć; Reshard, FLUX 3). Pominięto: agentowe finanse (Claude + Plaid read-only; Visa: agenci zakupowi zawodzą w 42,8% testów) – niska waga infrastrukturalna.
Źródło: https://newsletter.semianalysis.com/p/we-visited-chinas-stargate-in-inner

## Implikacje praktyczne
1. Lokalizacja i energia: moratoria/taryfy w USA przesuwają ryzyko na kampusy 2028; dywersyfikować regiony, umowy energetyczne i opcje BYO-power.
2. Architektura zasilania: planować kompatybilność hali z 800 V DC/SST (miejsce, BESS zamiast UPS), ale kontraktować pilotażowo – dojrzałość SST niepotwierdzona na GW.
3. Kontrahenci compute: koncentracja popytu (2 labs ~30% nowej mocy 2027) i wyceny IPO – klauzule wyjścia, limity ekspozycji na jednego klienta.
4. Koszt pamięci: budżetować HBM/DRAM wg cen 2027 (+121% HBM), z wariantami przy korekcie CXMT.
5. Finansowanie: koszt długu GPU-backed rośnie przy wyprzedaży; unikać refinansowania zbieżnego z oknami zmienności.

## Trend tygodnia
Ograniczenie ekspansji AI przesunęło się z dostępności chipów na warunki, na jakich można budować i pożyczać. Wybory lokalne w USA (40 z 41 głosowań na „nie” dla DC) wprowadzają ryzyko zgody społecznej, które uderza głównie w moce planowane na 2028. Jednocześnie wyceny publiczne i obligacje zareagowały gwałtownie, mimo że wyniki raportujących spółek były w większości powyżej oczekiwań – rynek wycenia kredyt i koncentrację, nie popyt. Chiny pokazują alternatywny model: pionowa integracja energii, zasilania i DC, z czasem dostawy ~12 mies. i kosztem ~1/5 amerykańskiego. Ceny pamięci rosną bez wzrostu bitów, co podnosi koszt jednostki mocy. Dla architekta oznacza to, że przewaga przesuwa się do tych, którzy kontrolują energię, finansowanie i terminy energizacji.

## To obserwować
- Roadshow i wycena IPO Anthropic (ok. 10.11, nieoficjalnie) oraz cena zlecenia.
- Potwierdzenie skali wyprzedaży i spready CDS (SpaceX, Oracle, CoreWeave), SoftBank bond.
- Kolejny raport SemiAnalysis o Wewn. Mongolii (25 GW zapowiedzi) i wyniki fazy 2 Galaxy z SST.
- Ceny DRAM/HBM kontraktowe Q1 2027.
- Stan stanowych moratoriów DC (Teksas, sesja w styczniu) i liczba GW wstrzymanych.
