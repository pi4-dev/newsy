# Raport AI-ML — 2026-09-09

**Zakres:** 2026-09-08–2026-09-09 (Europe/Warsaw)

## Biznes

### Amazon i Qualcomm: wieloletnia umowa na niestandardowe układy inference

Amazon może kupić do 60 mld USD układów i produktów Qualcomm dla centrów danych, a warranty o wartości około 4 mld USD będą nabywane wraz z realizacją zakupów. Współpraca obejmuje niestandardowe akceleratory inference i łączność optyczną do 1,6 Tb/s; Qualcomm deklaruje cel 15 mld USD przychodów data-center do 2029 r.

**Analiza architektoniczna:** Umowa wzmacnia alternatywę dla GPU w stabilnych, wysokowolumenowych ścieżkach inference, ale przenosi część lock-in z CUDA na własny toolchain, kompilator i warstwę runtime. Przy ocenie platformy trzeba mierzyć nie tylko tokens/s i koszt energii, lecz także czas kompilacji modeli, dostępność operatorów, observability per accelerator i możliwość awaryjnego przeniesienia workloadu.

**Źródło:** [Reuters, 8 września 2026](https://www.reuters.com/technology/qualcomm-amazon-develop-custom-chips-ai-data-centers-2026-09-08/)

## Technologia

### Karmada uzyskuje status CNCF Graduated

Karmada, warstwa orkiestracji wielu klastrów Kubernetes, uzyskała 7 września status Graduated. Wersja 1.19 rozwija wielokomponentowe planowanie dla rozproszonego treningu AI oraz wprowadza planowanie priorytetowe jako funkcję Beta domyślnie włączoną.

**Analiza architektoniczna:** Dojrzałość governance zmniejsza ryzyko projektu, ale nie eliminuje kosztu spójności polityk, danych i telemetrii między klastrami. Dla treningu rozproszonego scheduler powinien uwzględniać topologię sieci, locality danych, dostępność akceleratorów i wspólne SLO; inaczej globalne rozmieszczenie może pogorszyć tail latency i koszt transferu.

**Źródło:** [CNCF, 7 września 2026](https://www.cncf.io/announcements/2026/09/07/cloud-native-computing-foundation-announces-karmada-graduation/)

## Implikacje praktyczne

1. Wprowadzić neutralny interfejs model-serving i zestaw testów zgodności operatorów przed przyjęciem niestandardowych ASIC.
2. Porównywać koszt inference na poziomie kompletnej usługi: akcelerator, pamięć, sieć optyczna, kompilacja, retry i rezerwa SLO.
3. Dla multi-cluster AI wymagać topologicznego schedulingu oraz wspólnego modelu capacity, quota i priorytetów.
4. Utrzymywać deterministyczną ścieżkę fallback na co najmniej drugiej klasie akceleratora lub dostawcy.
5. Standaryzować metryki: queue time, TTFT, inter-token latency, tokens/J, failure rate i czas relokacji między klastrami.

## Trend tygodnia

Infrastruktura inference coraz częściej łączy własne układy, dedykowaną optykę i kontrakty zakupowe o skali hyperscalera. Jednocześnie warstwa orkiestracji dojrzewa w kierunku świadomego topologii zarządzania wieloma klastrami. Przewaga kosztowa będzie zależeć od jakości całego stosu, nie wyłącznie od parametrów układu.

## To obserwować

- realny harmonogram i dostępność akceleratorów Qualcomm dla Amazon;
- kompletność toolchainu i przenośność modeli poza platformę docelową;
- stabilność wielokomponentowego schedulingu Karmada przy awariach części klastra;
- koszt i opóźnienia sieci między klastrami przy treningu rozproszonym;
- metryki produkcyjne łączności optycznej 1,6 Tb/s.

## Kontrola źródeł

Sprawdzono również bieżące publikacje SemiAnalysis, NVIDIA Developer Blog, AIwire i HPCwire; nie znaleziono w nich dodatkowego, nieopisanego wcześniej zdarzenia spełniającego próg istotności.