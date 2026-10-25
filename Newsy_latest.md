# Newsy — 2026-10-25

**Okno przyrostowe:** od raportu 2026-10-24 do 2026-10-25 07:28 CEST (weekend, mała liczba nowych publikacji). Pominięto sekcje: Raport technologiczny, Raport AI-ML, euro neocloud, AI for networking – brak nowych, wiarygodnych i datowanych pozycji (feed NVIDIA Developer Blog bez wpisów po 23.10; SemiAnalysis bez nowych wpisów dostępnych w feedzie; HPCwire zwrócił 403). Źródła: newsletter z etykiety NEWSY (The Business Engineer), WWW.

## Raport Ryzyka rynku AI/GPU

#### SpaceX: 40 mld USD długu na chipy NVIDIA – aktualizacja: kredytowe CDS i wyceny obligacji pogorszone, zamknięcie dopiero w 2027
**Nowy element względem 24.10:** wg FT (via media wtórne) struktura to ok. 10 mld USD kredytów bankowych i 30 mld USD obligacji, a zamknięcie oczekiwane jest w 2027. 5-letnie CDS SpaceX wzrosły o do 14,5 pb do ok. 195 pb, a obligacje 6,65% z terminem 2056 handlują wyraźnie szerzej niż spread emisyjny 175 pb z czerwca. Newsletter The Business Engineer przytacza Bloomberga, wg którego transakcja została „roiled” przez turbulencje rynkowe – tej informacji nie udało się potwierdzić w źródle pierwotnym.
**Ryzyko:** Okno finansowania sięga 2027, więc zamówienie GPU jest zabezpieczone dopiero warunkowo; rozszerzanie spreadów na istniejącym długu podnosi koszt refinansowania całej klasy emitentów finansujących sprzęt AI długiem. Wniosek analityczny: kolateral w GPU nie neutralizuje ryzyka rynku kredytowego.
Źródło: https://investinglive.com/stocks/spacex-seeks-40-billion-in-debt-led-by-apollo-to-fund-nvidia-chip-order-ft-says/ , https://www.advisorperspectives.com/articles/2026/10/07/spacex-credit-risk-gauge-record-high-fundraising-reports

#### AWS: pojemność 2027 w większości zarezerwowana, backlog 496 mld USD, capex 2026 ok. 220 mld USD przy ujemnym FCF
**Fakty:** Wg relacji z rozmowy wynikowej Amazona (koniec lipca 2026) Jassy oczekuje braku mocy względem popytu w 2026 i utrzymania tej sytuacji w 2027; backlog AWS wzrósł do 496 mld USD z 364 mld USD kwartał wcześniej, a prognoza capex 2026 poszła w górę z ok. 200 do ok. 220 mld USD, głównie przez wyższe ceny pamięci. Free cash flow Amazona stał się ujemny. Cytat „2027 is largely sold out” przypisany w newsletterze Reutersowi nie został potwierdzony dosłownie w źródle pierwotnym; sformułowania w mediach różnią się („most”/„vast majority”). Dane pochodzą z lipca, ale newsletter prezentuje je jako aktualne w tym tygodniu.
**Ryzyko:** Backlog to zobowiązania klientów, nie przychód – jego jakość zależy od kontrahentów (patrz koncentracja popytu poniżej). Wzrost capex napędzany ceną pamięci (a nie wolumenem) oznacza spadek marży na jednostkę mocy; dla klientów – brak pojemności on-demand w 2027 i presja na kontrakty rezerwacyjne.
Źródło: https://www.stheadline.com/realtime-finance/3599608/亞馬遜AWS營收增37%創四年最快 資本支出上調至2200億 惟自由現金流轉負 , https://www.theregister.com/2026/04/10/amazon_may_sell_homegrown_chips/

## Newsletters summary

Przetworzono 1 z 1 maili NEWSY (The Business Engineer – „This Week In The AI Supercycle”); 0 zignorowanych. Wpisy OpenAI/DeepSeek/SpaceX opisane już wcześniej pominięto.

- **Rozjazd łańcucha dostaw AI: sprzedawcy zakontraktowani, płatnicy mają droższy kapitał** – Micron i Sandisk mają łącznie ok. 225 mld USD sprzedaży zakontraktowanej (wyliczenie autora newslettera z raportów, nie liczba spółek). Meta: wg newslettera odsetki +330% r/r i ujemny kwartalny FCF; weryfikacja własna: FCF Q2 2026 to ok. 0,8 mld USD przy capex ok. 30 mld USD (ok. 95% operacyjnego cash flow), a ujemny FCF za Q3 (−2,1 mld USD) pochodzi z jednego niepotwierdzonego źródła; liczby odsetek nie zweryfikowano. Wniosek: koszt kapitału, nie popyt na komponenty, jest ograniczeniem budowy; pojemność pamięci HBM/DRAM jest zarezerwowana na lata, więc ceny pamięci przenoszą się na capex (zob. AWS wyżej). https://businessengineer.ai/p/this-week-in-the-ai-supercycle-execution
- **Koncentracja popytu na API modeli frontier** – wg danych Ramp (firmowe karty) 1% firm odpowiada za 80% wydatków biznesowych na OpenAI i Anthropic, w dużej części spółki finansowane z VC. Ryzyko retencji: przychód dostawców modeli (i pośrednio kontrakty compute) opiera się na wąskiej grupie klientów zależnych od następnych rund finansowania; dane z jednego dostawcy kart, próba nie jest reprezentatywna. https://businessengineer.ai/p/this-week-in-the-ai-supercycle-execution
- **Presja cenowa na tokeny** – Moonshot wycenił model klasy frontier na połowę ceny najwyższego modelu Anthropic; wg jednego panelu danych ceny tokenów spadły ok. 50% r/r. Dla architektów: koszt inferencji na token spada szybciej niż amortyzacja GPU (3–5 lat), co obniża marżę dostawców inferencji z drogim zadłużeniem; w planach kosztowych zakładać dalszą deflację i unikać długich zobowiązań wolumenowych po obecnych cenach. Dane z newslettera, bez weryfikacji niezależnej. https://businessengineer.ai/p/this-week-in-the-ai-supercycle-execution

## Implikacje praktyczne
1. Pojemność on-demand w chmurach hyperscale w 2027 będzie ograniczona (AWS: backlog 496 mld USD); wymagane są kontrakty rezerwacyjne lub zapewnienie capacity przez alternatywnych dostawców z wyprzedzeniem 12–18 miesięcy.
2. Finansowanie GPU długiem (SpaceX, neocloudy) jest wrażliwe na spready kredytowe; przy wyborze dostawcy moc obliczeniowej oceniać strukturę finansowania i ekspozycję na refinansowanie w 2027–28.
3. Ceny tokenów spadają ok. 50% r/r: unikać wieloletnich zobowiązań wolumenowych po cenach stałych i projektować warstwę routingu modeli umożliwiającą zmianę dostawcy.
4. Koncentracja popytu (1% firm = 80% wydatków u dwóch dostawców) to ryzyko kontrahenta dla całego łańcucha – klauzule step-in i przenaszalność danych/obciążeń w umowach compute.
5. Ceny pamięci przenoszą się na capex: planować budżet sprzętowy z buforem na wzrost kosztu HBM/DRAM.

## Trend tygodnia
Obraz z ostatnich dni jest asymetryczny: strona podaży (komponenty, energia, pojemność chmur) jest zakontraktowana na lata, a strona płatników – hyperscalerzy z ujemnym lub bliskim zera FCF oraz emitenci długu pod GPU – mierzy się z droższym kapitałem. SpaceX pokazuje to najwyraźniej: CDS szerzej, obligacje poniżej emisji, a zamknięcie finansowania odsunięte do 2027. Jednocześnie przychód końcowy opiera się na wąskiej grupie klientów, a cena tokena spada o połowę rocznie, więc marża jednostkowa inferencji jest ściskana z obu stron. Capex rośnie dziś głównie przez ceny pamięci, nie przez wolumen mocy, co oznacza gorszą produktywność każdego wydanego dolara. Tydzień jest słaby informacyjnie (weekend), więc wnioski opieram na pojedynczych źródłach wtórnych i oznaczam je jako niezweryfikowane. Dla architekta praktyczny wniosek: zabezpieczyć pojemność kontraktowo, ale unikać stałych cen i monokultury dostawcy.

## To obserwować
1. Zamknięcie i ostateczna cena 40 mld USD długu SpaceX oraz poziom 5-letnich CDS (ok. 195 pb).
2. Wyniki Q3 Meta, Microsoft, Alphabet i Amazon (tydzień wyników): capex vs FCF, koszt odsetek.
3. Ceny DRAM/HBM kontraktowe i wpływ na capex hyperscalerów.
4. Rozstrzygnięcie rundy OpenAI (30 mld USD, ok. 1,25 bln USD wyceny) – jakość kontrahenta dla backlogu compute.
5. Cena tokena modeli frontier (Moonshot vs Anthropic/OpenAI) jako wskaźnik marży inferencji.
