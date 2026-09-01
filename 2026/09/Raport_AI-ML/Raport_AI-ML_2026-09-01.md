# Raport AI-ML — 2026-09-01

## Biznes

### Anthropic–Lambda: nowy kontrakt chmurowy o wartości 35 mld USD i mocy około 350 MW

Według Reutersa Anthropic podpisał z neocloudem Lambda umowę na infrastrukturę NVIDIA w centrum danych Hut 8 w hrabstwie Nueces w Teksasie. Projekt obejmuje około 350 MW; informacja nie została jeszcze publicznie skomentowana przez strony. Jest to materialna aktualizacja wcześniejszego tematu Anthropic–Nscale: nowa umowa tworzy drugi bardzo duży strumień capacity i dodatkowy łańcuch zależności Anthropic–Lambda–NVIDIA–Hut 8.

**Analiza architektoniczna:** 350 MW oznacza skalę wymagającą planowania nie tylko liczby GPU, lecz także przyłączy energetycznych, chłodzenia, zapasu części, sieci międzyhalowej i fazowania odbiorów. Wielostronna struktura kontraktowa zwiększa ryzyko skorelowanych opóźnień i utrudnia przypisanie odpowiedzialności za SLO, capacity shortfall oraz odzyskanie danych. Portability powinna obejmować obrazy, scheduler, warstwę serving i obserwowalność, ponieważ sama dywersyfikacja nazw dostawców nie usuwa koncentracji na sprzęcie NVIDIA.

**Źródło:** [Reuters — Anthropic signs $35 billion cloud deal with Nvidia-backed Lambda](https://www.reuters.com/technology/anthropic-signs-35-billion-cloud-deal-with-nvidia-backed-lambda-source-says-2026-08-31/)

### NVIDIA inwestuje 3,5 mld USD w obligacje zamienne MediaTek; MediaTek wdraża NVLink Fusion

NVIDIA i MediaTek ogłosiły 31 sierpnia rozszerzenie współpracy od custom XPU po systemy rack-scale; NVIDIA zainwestowała 3,5 mld USD w obligacje zamienne MediaTek. MediaTek ma oferować klientom projektowanie akceleratorów korzystających z NVLink Fusion, NVLink-C2C i NVHBM oraz z kwalifikowanego przez NVIDIA otoczenia packagingu, pamięci i systemów rack-scale.

**Analiza architektoniczna:** skraca to drogę hyperscalera od własnego bloku compute do produkcyjnego systemu, lecz przenosi lock-in z pojedynczego GPU na interconnect, chiplet, pamięć, packaging i kwalifikację racka. Przy ocenie custom XPU trzeba osobno mierzyć przepustowość i energię scale-up, koszt HBM, yield zaawansowanego packagingu oraz możliwość pracy poza domeną NVLink; deklarowana zgodność nie zastępuje testów failure isolation i degradacji fabric.

**Źródło:** [MediaTek — NVIDIA and MediaTek Deepen Long-Standing Partnership](https://www.mediatek.com/press-room/nvidia-and-mediatek-deepen-long-standing-partnership-to-build-ai-edge-to-cloud-computing-platforms)

## Technologia

### LUMI-AI: europejski system za 387,8 mln EUR na AMD, IBM i Nokia

EuroHPC wybrał państwową spółkę Bull do budowy LUMI-AI w Finlandii; uruchomienie zaplanowano na drugą połowę 2027 r. System ma łączyć akceleratory i procesory AMD, storage IBM, sieć Nokia oraz platformę Bull, a popyt na obecne zasoby EuroHPC już przekracza dostępną podaż. Jest to szóste zamówienie w programie AI Factories.

**Analiza architektoniczna:** heterogeniczny stos zmniejsza zależność od NVIDIA, ale zwiększa ciężar integracji ROCm, kolektywów, systemu plików, telemetryki i schedulera. Dla użytkowników kluczowe będą kolejki, mierzalne SLO, dojrzałość kontenerów i przenośność workloadów między LUMI-AI a chmurami CUDA; termin 2027 wymaga także planu na zmianę generacji sprzętu przed końcem amortyzacji.

**Źródła:** [Reuters — EU orders AI supercomputer from Bull](https://www.reuters.com/world/europe/europe-expands-ai-computing-network-with-390-million-order-frances-bull-2026-08-31/), [Bull — LUMI-AI](https://www.globenewswire.com/news-release/2026/08/31/3353040/0/en/bull-selected-to-deliver-europe-s-387-8-million-lumi-ai-supercomputer-in-finland.html)

## Implikacje praktyczne

1. W kontraktach na duże pule GPU rozdzielić SLO dla energii, hali, sieci, sprzętu, schedulera i warstwy serving; określić odpowiedzialność każdego podmiotu w wielostronnym łańcuchu dostaw.
2. Utrzymywać przenośny control plane i telemetrykę oraz co najmniej jeden regularnie testowany profil workloadu poza CUDA/NVLink, zamiast opierać strategię wyjścia wyłącznie na zapisach umownych.
3. Dla custom XPU wymagać benchmarków całego racka: TTFT/ITL, tokens/J, wykorzystanie HBM, wydajność kolektywów, zachowanie przy awarii linku i czas rekonfiguracji.
4. W capacity planning uwzględniać datę realnego przyłączenia mocy i rampę odbiorów, nie tylko zakontraktowane MW; monitorować opóźnienia energetyczne, yield packagingu i dostępność HBM.
5. Dla środowisk publicznych EuroHPC projektować kolejki i checkpointing pod ograniczoną podaż, a artefakty MLOps utrzymywać w formacie możliwym do uruchomienia na AMD i NVIDIA.

## Trend tygodnia

Kapitał i integracja przesuwają się z zakupu pojedynczych akceleratorów do wieloletnich, wielopodmiotowych umów obejmujących energię, centrum danych, fabric i cały rack. Jednocześnie custom silicon nie oznacza automatycznie otwartości: NVLink Fusion pozwala różnicować blok compute, ale utrwala wspólną domenę interconnectu, pamięci i kwalifikacji systemowej. Europejski LUMI-AI pokazuje równoległy kierunek — budowę publicznej capacity na stosie AMD/IBM/Nokia — lecz jej wartość zależy od dostępności operacyjnej, a nie samej mocy szczytowej.

## To obserwować

- harmonogram i faktyczny ramp-up 350 MW dla projektu Lambda/Hut 8 w Teksasie;
- sposób podziału zobowiązań i gwarancji pomiędzy Anthropic, Lambda, NVIDIA i Hut 8;
- pierwsze produkcyjne XPU MediaTek z NVLink Fusion oraz ich parametry packagingu, HBM i fabric;
- gotowość ROCm, kolektywów i narzędzi observability dla LUMI-AI przed uruchomieniem w drugiej połowie 2027 r.;
- kolejki i odsetek odrzucanych wniosków w EuroHPC jako miernik rzeczywistego niedoboru capacity.
