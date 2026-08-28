# Raport AI-ML — 2026-08-28

Data podsumowania: **2026-08-28**  
Zakres przyrostowy: od edycji z 2026-08-27 do 2026-08-28, Europe/Warsaw.

## Biznes

### NVIDIA wstrzymuje część umów revenue-sharing z małymi chmurami AI

27 sierpnia Reuters, za „Wall Street Journal”, podał, że NVIDIA w poprzednim tygodniu wycofała się z części programu kredytowego dla małych dostawców chmury AI. Model miał gwarantować odkup niewykorzystanej mocy, ułatwiać finansowanie GPU i przekazywać NVIDIA 50% przychodu ponad ustalony próg; producent zastrzega, że szerszy mechanizm udostępniania mocy nadal ewoluuje.

**Analiza techniczno-biznesowa:** zawieszenie osłabia bankowalność projektów opartych na gwarantowanym wykorzystaniu i zwiększa koszt kapitału oraz ryzyko niedoszacowania popytu. Dla klientów oznacza to większe ryzyko zmian cen, opóźnień capacity i niewypłacalności mniejszych operatorów; w umowach należy oddzielić SLO usługi od finansowania sprzętu oraz wymagać przenośności obrazów, checkpointów i danych.

**Źródło:** [Reuters, 2026-08-27](https://www.reuters.com/business/nvidia-pauses-revenue-sharing-deals-with-ai-cloud-companies-wsj-reports-2026-08-27/)

### Anthropic bada własny układ treningowy i współpracę z MatX

Reuters ujawnił 27 sierpnia, że Anthropic rozważał zakup MatX za około 7 mld USD, a rozmowy przesunęły się w stronę partnerstwa; MatX ma pozyskiwać kapitał przy wycenie około 4 mld USD. Anthropic nie wybrał jeszcze architektury, ale rozbudowuje zespół silicon i utrzymuje strategię wielodostawcy obejmującą NVIDIA, Google i Amazon.

**Analiza techniczno-biznesowa:** własny ASIC może obniżyć koszt i energię na token tylko przy stabilnym profilu modeli, wysokim wykorzystaniu i kilkuletniej skali produkcji; kosztem jest długi cykl projektu, ryzyko yield/packaging i konieczność utrzymywania osobnego toolchainu. Dla platform enterprise najbezpieczniejszy pozostaje kontrakt oparty na SLO i cenie za jednostkę pracy, z testami zgodności modeli na wielu backendach.

**Źródło:** [Reuters, 2026-08-27](https://www.reuters.com/business/finance/anthropic-planned-then-abandoned-7-billion-purchase-matx-sources-say-2026-08-27/)

## Technologia

W obowiązkowym przeglądzie SemiAnalysis, NVIDIA Developer Blog, AIwire i HPCwire nie pojawiła się po poprzedniej edycji nowa publikacja infrastrukturalna przekraczająca próg istotności. Najnowszy materiał NVIDIA o NVLink Fusion/NVHBM pochodzi z 26 sierpnia i został odrzucony jako wcześniejszy niż poprzedni przebieg.

## Implikacje praktyczne

1. W kontraktach z neocloudami wymagać ujawnienia struktury finansowania, wskaźników wykorzystania GPU, praw do migracji danych i checkpointów oraz planu wyjścia przy zmianie gwarancji producenta.
2. Utrzymywać warstwę serving/training niezależną od konkretnego ASIC: kontenery, otwarte formaty modeli, testy numerycznej zgodności i automatyczne benchmarki price/performance.
3. Capacity planning prowadzić dla scenariuszy bez gwarantowanego buy-backu: rezerwa mocy, alternatywny region/operator i limity koncentracji dostawcy.
4. Dla custom silicon mierzyć koszt całej ścieżki — kompilacja, kernel coverage, obserwowalność, debugowanie i odzysk po awarii — nie tylko szczytowy throughput.
5. SLO inferencji wiązać z p95/p99 TTFT, inter-token latency, error budget i kosztem energii na token, aby porównanie GPU/ASIC było operacyjne.

## Trend tygodnia

Kapitał, układy i kontrakty compute coraz częściej są jednym systemem ekonomicznym. NVIDIA ogranicza najbardziej bezpośredni model revenue-sharing, podczas gdy laboratoria próbują budować własne układy, by obniżyć koszt krańcowy i zmniejszyć zależność od GPU. W horyzoncie 3–5 lat przewagę da nie pojedynczy akcelerator, lecz możliwość przenoszenia workloadu między kilkoma backendami bez utraty SLO i kontroli kosztów.

## To obserwować

- czy NVIDIA wznowi program gwarantowanego odkupu mocy w zmienionej formule;
- finansowanie i wycena MatX oraz wybór architektury przez Anthropic;
- realny koszt kapitału i utilization małych dostawców GPU po zmianie programu;
- dostępność NVIDIA Rubin i alternatywnych układów treningowych do 2027 r.;
- dojrzałość kompilatorów, profilerów i observability dla custom ASIC.
