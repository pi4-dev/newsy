# Newsy — 2026-11-09

**Okno:** od raportu 2026-11-08. Pominięto sekcje bez nowych zweryfikowanych informacji: raport technologiczny, AI-ML, euro neocloud, AI for networking, Newsletters (jedyny mail NEWSY – powiadomienie Substack Live – oznaczony IGNORED).

## Ryzyka rynku AI/GPU

#### CoreWeave: rentowność obligacji 2032 (kupon 9,625%) >16%, CDS 5Y do 1 289 pb, potem ok. 1 155 pb
CNBC (6.11, za Deutsche Bank): CDS CoreWeave skoczył ponad 1 289 pb w czwartek i spadł ok. 8% w piątek do ok. 1 155 pb; Nvidia ok. 139 pb (+76% m/m), Oracle ok. 298 pb, Broadcom +49%, Amazon +44% m/m. Goldman Sachs szacuje, że moratoria na DC (Ohio, Teksas, Georgia) mogą dotknąć ok. 45% planowanych mocy USA; hyperscalerzy mieli w Q3 ujemny FCF (capex > operacyjny CF). Rozbieżności źródeł co do poziomu CDS (1 100–1 289 pb).
**Analiza:** Nowy element względem rejestru: wycena długu neocloudów na poziomie ~16% efektywnie zamyka rynek refinansowania bez zabezpieczenia; koszt kapitału przy takim yield przekracza typową marżę brutto GPU-as-a-service, więc nowe kontrakty trzeba finansować z zabezpieczeniem kontraktowym (take-or-pay hyperscalera). Wysokie wzrosty % CDS dużych emitentów startują z niskiej bazy – sygnał, nie dowód stresu kredytowego. Kanał ryzyka: moratoria → opóźnienia energizacji → opóźnione przychody przy stałych kuponach.
Źródło: https://www.cnbc.com/2026/11/06/coreweave-bond-yield-tops-16percent-as-ai-credit-stress-rises.html

#### Aktualizacja: Anthropic IPO – brak widełek cenowych w S-1/A z 2.11, start roadshow niepotwierdzony
Znowelizowany S-1 z 2.11 dodał wyniki Q3 i brokerów detalicznych, ale cena, liczba akcji i opcja gwarantów pozostają puste; brak późniejszej nowelizacji wg trackerów. Termin roadshow (tydzień 9.11 vs 16.11) wg doniesień, nie dokumentów; widełki wyceny rozbieżne (Bloomberg 1,2–1,3 bln, FT 1,25–1,75 bln USD). Dane wtórne – zweryfikować w EDGAR (S-1/A po 2.11).
**Analiza:** Ryzyko opóźnienia okna IPO w warunkach zmienności; brak ceny to brak sygnału wyceny dla całego łańcucha dostawców compute (zobowiązania ok. 895 mld USD).
Źródło: https://financefeeds.com/anthropic-ipo-roadshow-looms-with-no-price-range/ ; https://fourweekmba.com/ai-anthropic-said-to-start-ipo-roadshow-next-week-no-price-yet/

## Implikacje praktyczne
1. Finansowanie neocloudów: przy yield >16% projektować pojemność wyłącznie pod kontrakty z kontrahentem inwestycyjnym; unikać GPU bez zabezpieczonego popytu.
2. Klauzule wobec dostawców GPU-cloud: step-in/escrow i prawa do danych na wypadek restrukturyzacji; wielodostawcowość.
3. Lokalizacja: dywersyfikować poza stany z ryzykiem moratoriów (Teksas, Ohio, Georgia).
4. Kontrahenci: monitorować CDS jako wskaźnik wyprzedzający i wpisać progi do przeglądów ryzyka dostawców.
5. Harmonogramy: zakładać poślizgi energizacji 2027–28 w planach capacity.

## Trend tygodnia
Rynek wycenia teraz ryzyko kredytowe AI osobno od ryzyka akcyjnego. Po wyprzedaży 5–6.11 najmocniej widać to w długu neocloudów, gdzie rentowności przekroczyły 16%. Głównym kanałem jest polityka: możliwe moratoria na data center uderzają w moce planowane na 2028 i w przychody, które mają obsługiwać dług. Jednocześnie duzi emitenci (Nvidia, Oracle) mają CDS wciąż niskie w wartościach bezwzględnych, co wskazuje na stres skoncentrowany w segmencie lewarowanym. Wynik IPO Anthropic będzie pierwszym twardym testem apetytu na ekspozycję compute o skali bilionowej. Do czasu ustalenia widełek cenowych niepewność wyceny pozostaje główną zmienną.

## To obserwować
- S-1/A Anthropic z widełkami (EDGAR) i start roadshow.
- CDS 5Y CoreWeave (poziom 1 155 pb) i yield obligacji 2032.
- CDS Oracle, Broadcom, Amazon.
- Stanowe moratoria DC (Ohio, Teksas, Georgia) i szacunki Goldmana (45% mocy).
- Kolejny kwartalny FCF hyperscalerów vs capex.
