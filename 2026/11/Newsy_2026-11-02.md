# Newsy — 2026-11-02

**Okno przyrostowe:** od raportu 2026-11-01 do 2026-11-02. Pominięte sekcje: raport technologiczny, AI-ML, euro neocloud, AI for networking (brak nowych, zweryfikowanych pozycji; wyszukiwarka WWW zwraca głównie materiały starsze; RSS SemiAnalysis/NVIDIA bez nowych wpisów poza już opisanymi).

## Ryzyka rynku AI/GPU

#### Pipeline DC 2028: 62,2 GW netto w USA głównie przed rozpoczęciem budowy, ryzyko zamrożenia pozwoleń w Teksasie
(INFRASTRUKTURA) Wg modelu SemiAnalysis ok. 3/4 z 33,8 GW zaplanowanych na 2027 w top-20 stanów jest już w budowie, więc 2027 jest w dużej mierze zabezpieczony. Ryzyko dotyczy 2028: 62,2 GW netto, z czego 43,5 GW (70%) w 13 stanach, a 10 z nich wybiera gubernatora 3.11. Teksas (~30% GW 2028) ma zamrożone pozwolenia TCEQ od 21.09 oraz audyt kolejki przyłączeniowej ERCOT od 3.08 (raport ERCOT docelowo 10.12; samo złożenie raportu nie wznawia wydawania pozwoleń). Oregon rozszerzył 9.10 zamrożenie na pozwolenia powietrzne i zrzutowe do 1.07.2027. Scenariusz z wygraną kandydatów za moratorium: ok. 30 GW pipeline 2028 w ryzyku, przy kandydatach tylko „warunkowych" ok. 5,7 GW. Opóźnienie o 5–9 miesięcy względem restartu w grudniu, jeśli pozwolenia pozostaną wstrzymane do końca sesji legislacyjnej (12.01–31.05.2027). Dla neocloudów i kontraktów pod dostawy 2027/28: ryzyko opóźnień dostaw mocy (kary SLA, odroczone przychody przy już zaciągniętym długu). Źródło: newsletter (analiza własnego modelu, kandydaci „at face value").
Źródło: https://newsletter.semianalysis.com/p/the-datacenter-backlash-goes-bipartisan

## Newsletters summary

#### SemiAnalysis: backlash wobec datacenter staje się dwupartyjny — 2027 zablokowane, 2028 niepewne
Warunki, nie zakaz: 15 kandydatów postuluje pokrywanie przez DC kosztów sieci, 12 koniec ulg podatkowych, 11 raportowanie/limit zużycia wody. Najsilniejsze dźwignie: zwolnienia z podatku od sprzętu (potencjalnie miliardy USD na GW), komisje regulujące taryfy (depozyty, minimum bills, opłaty wyjścia), gubernatorzy mogą wstrzymać niewydane pozwolenia.
**Analiza:** Dla planowania mocy: hyperscalerzy udźwigną koszt (własna generacja, opłaty za przyłącze), ale nie czas — przeniesienie projektu do innego stanu nie omija kolejki mocy i pozwoleń. Projektując lokalizacje, uwzględnić projekty z niewydanymi pozwoleniami na kolejną fazę zasilania; chłodzenie o niskim zużyciu wody zmniejsza ekspozycję regulacyjną. Wyniki wyborów 3.11 zmienią warunki, ale nie wstrzymają budów już rozpoczętych.
Źródło: https://newsletter.semianalysis.com/p/the-datacenter-backlash-goes-bipartisan

#### SemiAnalysis GDPx: benchmark agentowy w 11 domenach — najlepszy model ok. 12%
GDPx obejmuje m.in. chip design, cyber, bio, finanse, prawo, quant trading, software engineering. Wg autorów Opus 5.5 wygrywa all-around (quant, hardware/chip design), Gemini 4 Carbon bio i SWE, GPT-6 Astra RSI i materiały; zadania finansowe i prawne dają ~0% dla wszystkich modeli. Klasyfikatory bezpieczeństwa obniżają wynik w cyber/bio/alignment, a Muse Wonder Full jest wg nich najlepszy w cyber.
**Analiza:** Dla doboru modeli w produkcji istotne są: front kosztu vs wydajności (Luna i DeepSeek na dole, Opus na górze, OpenAI w środku), oraz „spikiness" — model dobierać per domena, nie globalnie. Klasyfikatory bezpieczeństwa to ryzyko operacyjne (odmowy) przy zastosowaniach security. Wyniki autorskie, bez niezależnej replikacji; Gemini 4 Carbon jeszcze niedostępny GA.
Źródło: https://newsletter.semianalysis.com/p/gdpx-measuring-frontier-model-capabilities

## Implikacje praktyczne
1. Lokalizacje DC oceniać po statusie pozwoleń kolejnej fazy zasilania, nie tylko po umowie przyłączeniowej; klauzule opóźnień dostaw mocy w kontraktach z operatorami.
2. Zakładać wzrost kosztu własnego zasilania (opłaty za sieć, utrata ulg) w TCO 3–5 lat.
3. Chłodzenie bezwodne / niskie WUE jako zabezpieczenie przed regulacjami wodnymi.
4. Multi-region i multi-dostawca dla pojemności inferencji; nie opierać planu 2028 na jednym stanie (Teksas).
5. Dobór modeli per domena i monitorowanie odmów klasyfikatorów bezpieczeństwa w agentach security.

## Trend tygodnia
Ograniczeniem budowy infrastruktury AI w USA przesuwa się z podaży chipów na zgodę administracyjną. Rok 2027 jest w większości zabezpieczony fizycznie, bo ok. trzy czwarte mocy jest w budowie. O 2028 zdecydują pozwolenia, taryfy i ulgi podatkowe, a nie dostępność GPU. Sprzeciw wobec DC jest dwupartyjny i dotyczy kosztów sieci, wody i ulg, więc każda lokalizacja zapłaci więcej, a część będzie czekać. Hyperscalerzy udźwigną koszt, lecz nie czas, co obciąża zwłaszcza podmioty z długiem pod terminowe uruchomienie mocy. Równolegle rośnie rozrzut jakości modeli między domenami, co wspiera wielomodelowe architektury zamiast jednego dostawcy.

## To obserwować
- Wyniki wyborów 3.11 (gubernatorzy TX, OH, AZ, GA) i ich stanowisko ws. moratoriów.
- Termin raportu ERCOT (10.12) i decyzja TCEQ o wznowieniu pozwoleń.
- Oregon: zamrożenie do 1.07.2027; sesja legislacyjna Teksasu (12.01–31.05.2027).
- Publiczna dostępność Gemini 4 Carbon i niezależna weryfikacja GDPx.
