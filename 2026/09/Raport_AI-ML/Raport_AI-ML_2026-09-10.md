# Raport AI-ML — 2026-09-10

**Data podsumowania:** 2026-09-10  
**Zakres przyrostowy:** od edycji 2026-09-09 do 2026-09-10 (Europe/Warsaw)

## Biznes

### Google: 13 mld EUR na infrastrukturę AI w Finlandii i 22-letni PPA jądrowy

Google zapowiedział co najmniej 13 mld EUR inwestycji w latach 2027–2028, obejmujących trzy nowe centra danych w północnej Finlandii, modernizację sieci, magazyny energii i projekty wytwórcze. Równolegle zawarł z Fortum 22-letnią umowę zakupu do 50% produkcji elektrowni Loviisa — pierwszy kontrakt jądrowy Google poza USA.

Dla capacity planning oznacza to sprzężenie harmonogramu mocy obliczeniowej z wieloletnią dostępnością energii, sieci przesyłowej i chłodzenia; chłodny klimat obniża koszt free-air cooling, ale nie usuwa ryzyka przyłączeniowego i koncentracji geograficznej. Długoterminowy PPA stabilizuje cenę i emisje energii, jednocześnie tworząc zależność od pojedynczego aktywa wytwórczego, którą należy objąć scenariuszami N-1, SLO zasilania i planem zastępczej mocy.

**Źródło:** [Reuters — Google to invest $15 billion in AI infrastructure and buy nuclear power in Finland, 9 września 2026](https://www.reuters.com/business/media-telecom/google-invest-15-billion-ai-infrastructure-finland-2026-09-09/)

### NVIDIA i partnerzy: cel do 2 GW infrastruktury AI w Australii do 2027 r.

NVIDIA współpracuje z Firmus, CDC, NEXTDC i AirTrunk nad rozbudową do 2 GW mocy związanej z AI do 2027 r.; według branżowych danych cytowanych przez Reuters obecna australijska baza obliczeniowa to około 1,6 GW. Partnerzy mają budować centra danych z wykorzystaniem platformy określonej w komunikacie jako „DSX”, a projekt ma być powiązany z dodatkowym wytwarzaniem energii.

Deklarowane 2 GW jest celem budowy, a nie potwierdzoną mocą oddaną do eksploatacji, dlatego kluczowe są kamienie milowe energizacji, dostępność wody, dostawy systemów i realny współczynnik wykorzystania GPU. Dla odbiorców zwiększa to potencjalną regionalną podaż i możliwość rezydencji danych, ale pogłębia zależność od stosu NVIDIA; kontrakty powinny wiązać opłaty z dostarczoną pojemnością, SLO i telemetrią energetyczną, a nie wyłącznie z zapowiedzianą mocą.

**Źródło:** [Reuters — Nvidia plans major expansion of data centre capacity in Australia, 10 września 2026](https://www.reuters.com/world/asia-pacific/nvidia-teams-up-with-australian-partners-build-ai-factory-capacity-2026-09-10/)

### Niedobór HBM podnosi ceny chińskich akceleratorów o 20–50%

Według źródeł Reuters Huawei podniosło wskazywaną cenę karty Ascend 950DT powyżej 250 tys. CNY, o 20–50% wobec wycen sprzed dwóch miesięcy; Cambricon 690 zdrożał o 20–30%, a podobne ruchy wykonali MetaX i Iluvatar CoreX. Ograniczenia eksportowe zwiększyły zależność od szarego rynku HBM, gdzie pamięć kosztuje wielokrotnie więcej niż poza Chinami; drożeją też wcześniejsze 950PR i 910C.

To materialnie zmienia TCO alternatyw dla NVIDIA: koszt i dostępność pamięci, a nie sam układ obliczeniowy, stają się głównym ograniczeniem throughputu i harmonogramu klastrów. Organizacje oceniające Ascend lub inne krajowe akceleratory powinny oddzielnie modelować koszt HBM, lead time, zapas serwisowy i ryzyko alokacji dla największych klientów oraz testować przenośność modeli i obserwowalności poza CUDA.

**Źródło:** [Reuters — China's AI chipmakers raise prices as HBM shortage bites, 10 września 2026](https://www.reuters.com/world/asia-pacific/chinas-ai-chipmakers-raise-prices-high-bandwidth-memory-shortage-bites-2026-09-10/)

## Technologia

### Gdzie „myśli” robot: lokalna pętla sterowania, planowanie w centrum danych

Analiza SemiAnalysis wskazuje na architekturę hierarchiczną: lokalne pętle działania i bezpieczeństwa pracują z częstotliwością setek herców, natomiast planowanie wysokiego poziomu działa asynchronicznie z częstotliwością kilku herców i może być oddelegowane do centrum danych. Obecne modele robotyczne mają zwykle około 5–14 mld parametrów; przykładowo DreamZero 14B wymaga według autorów dwóch GB200 przy około 7 Hz, podczas gdy Jetson Thor ma ułamek wydajności B200/B300.

Ruch physical AI jest stałym strumieniem zamkniętej pętli z obrazem i stanem robota, a nie krótkim burstem tokenów, więc SLO musi obejmować opóźnienie „sensor-to-action”, jitter, utratę pakietów i czas przełączenia na fallback lokalny. Sensowny wzorzec to deterministyczny safety/control plane na urządzeniu oraz adaptacyjne kodowanie i planowanie w DC, ale tylko z degradacją funkcji, buforowaniem i autonomicznym zatrzymaniem przy utracie łączności.

**Źródło:** [SemiAnalysis — Where Does a Robot Think: On-Device vs Datacenter Inference, 9 września 2026](https://newsletter.semianalysis.com/p/where-does-a-robot-think-on-device)

## Implikacje praktyczne

1. W nowych regionach AI wiąż roadmapę GPU z potwierdzonymi kamieniami milowymi przyłączenia, PPA, chłodzenia i testów obciążeniowych; raportuj osobno moc zakontraktowaną, dostępną i faktycznie używaną.
2. Buduj modele TCO per warstwa BOM — HBM, akcelerator, host CPU, sieć i energia — oraz utrzymuj scenariusze wzrostu cen pamięci o 20–50% i wydłużenia lead time.
3. Dla robotyki rozdziel certyfikowaną pętlę safety/control od niedeterministycznego planowania AI; zdefiniuj lokalny fallback, maksymalny jitter i zachowanie po utracie WAN przed dopuszczeniem offloadu.
4. Ogranicz vendor lock-in przez kontraktowe SLO telemetrii, eksport metryk na poziomie GPU/HBM i okresowe testy przenośności workloadów pomiędzy CUDA, TPU i alternatywnymi stosami.
5. Dla wieloletnich PPA i pojedynczych aktywów energetycznych wykonuj testy N-1/N-2 oraz licz koszt zapasowej mocy jako część kosztu usługi, nie jako zewnętrzne ryzyko obiektu.

## Trend tygodnia

Capacity planning AI coraz wyraźniej przesuwa się z samej liczby akceleratorów na łańcuch energii, HBM, sieci i terminy energizacji. Projekty liczone w gigawatach wymagają finansowych i operacyjnych zabezpieczeń podobnych do infrastruktury energetycznej, a nie typowego zamówienia serwerowego. Jednocześnie physical AI wymusza rozdzielenie niskolatencyjnej, deterministycznej kontroli od kosztownego planowania w centrum danych. Wspólnym mianownikiem jest potrzeba pomiaru dostarczonej zdolności i SLO zamiast polegania na deklaracjach mocy lub teoretycznym FLOPS.

## To obserwować

- Faktyczne terminy energizacji trzech fińskich centrów Google i postęp modernizacji Loviisa.
- Ile z australijskiego celu 2 GW przejdzie do mocy zakontraktowanej i operacyjnej do końca 2027 r.
- Dostępność Huawei Ascend 950DT w IV kwartale 2026 i dalsze ceny HBM/kompletnych kart.
- Pomiary end-to-end latency, jitteru i failoveru dla robotycznego planowania oddelegowanego do DC.
- Narzędzia przenoszące telemetrię i SLO między CUDA, TPU i alternatywnymi akceleratorami.

## Kontrola źródeł

Sprawdzono wymagane kanały SemiAnalysis, NVIDIA Developer Blog, AIwire i HPCwire; poza pozycją SemiAnalysis nie znaleziono dodatkowej, nowej informacji przekraczającej próg istotności i nieopisanej we wcześniejszych raportach.
