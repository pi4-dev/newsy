# Raport AI-ML — 2026-08-31

## Biznes

### Audyt bezpieczeństwa staje się kryterium wyboru neocloudu, nie dodatkiem do benchmarku

SemiAnalysis opublikował 30 sierpnia wyniki czteromiesięcznych testów ClusterMAX 3.0 obejmujących 25 dostawców i 32 klastry. Badacze nie ujawnili nazw środowisk z lukami, ale opisali ekspozycję danych klientów oraz scenariusze cross-tenant RCE; jednocześnie podali, że wszystkie zgłoszone problemy zostały usunięte lub potwierdzono wdrożenie poprawek przed upływem 90 dni. Dla nabywcy GPU cloud oznacza to konieczność traktowania izolacji tenantów, procesu aktualizacji firmware'u i oprogramowania oraz kontroli płaszczyzn zarządzania jako warunków kontraktowych i odbiorowych, a nie wyłącznie deklaracji dostawcy.

**Znaczenie:** ryzyko kontrahenta obejmuje dziś nie tylko płynność i dostępność GPU, lecz także możliwość utraty poufności promptów, kluczy i danych treningowych przez błędy wspólnej warstwy sieciowej, storage lub observability.

**Źródło:** [SemiAnalysis — Most Neoclouds Suck At Security](https://newsletter.semianalysis.com/p/most-neoclouds-suck-at-security)

## Technologia

### ClusterMAX 3.0: powtarzalne błędy izolacji w stosie GPU cloud

W testach wykryto m.in. publicznie osiągalne kubelety, brak domyślnej polityki `default deny` w Kubernetes, błędne RBAC storage, dostęp tenantów do BMC/IPMI oraz wspólne dashboardy Grafana z uprzywilejowanym kluczem Prometheus. W InfiniBand błędna konfiguracja P_Key, SA_Key i M_Key ujawniała hosty innych tenantów albo pozwalała wpływać na konfigurację fabric; w jednym przypadku aktywna pozostała domyślna partycja obejmująca setki endpointów. Testy potwierdziły też działające ścieżki ucieczki z kontenerów przy przestarzałym NVIDIA Container Toolkit i innych komponentach runtime.

**Analiza architektoniczna:** kontrola wersji CUDA, sterownika, GPU Operatora, runc/Docker i firmware'u ConnectX powinna być ciągła oraz powiązana z admission control i automatycznym wycofaniem węzła z puli. Izolacja wyłącznie namespace'em lub kontenerem nie stanowi wystarczającej granicy dla nieufnych workloadów; osobne VM-y, jawne polityki sieciowe, klucze fabric i niezależna płaszczyzna zarządzania ograniczają blast radius. Observability musi zachowywać separację tenantów również na poziomie źródłowych API i credentiali, nie tylko dashboardu.

**Źródła:** [SemiAnalysis](https://newsletter.semianalysis.com/p/most-neoclouds-suck-at-security), [kryteria ClusterMAX](https://www.clustermax.ai/criteria)

## Implikacje praktyczne

1. Wprowadzić przedprodukcyjny test odbiorowy klastra obejmujący cross-tenant discovery, dostęp do kubeleta/BMC, `default deny`, storage RBAC, Grafana/Prometheus oraz P_Key/M_Key/SA_Key.
2. Wymagać od dostawcy mierzalnego SLO na poprawki krytyczne oraz dowodów automatycznej zgodności wersji sterownika, CUDA, runtime'u kontenerowego, GPU Operatora i firmware'u NIC/DPU.
3. Uruchamiać nieufne obrazy i agentów w dedykowanych VM-ach lub microVM-ach; kontener pozostawić warstwą pakowania, nie granicą bezpieczeństwa.
4. Oddzielić credential plane od workload plane, ograniczyć egress i rotować klucze infrastrukturalne per tenant/klaster; wspólny klucz root do całej floty traktować jako ryzyko krytyczne.
5. W planowaniu 3–5 lat uwzględnić koszt ciągłej walidacji izolacji po każdej zmianie firmware'u, sieci, storage i orkiestratora; statyczny audyt raz w roku nie pokrywa tempa zmian stosu AI.

## Trend tygodnia

Bezpieczeństwo AI infrastructure przesuwa się z kontroli pojedynczego hosta na walidację całej współdzielonej platformy. Najgroźniejsze awarie nie wynikają z jednego egzotycznego exploita, lecz z kompozycji starych wersji, źle ustawionej fabric, nadmiarowych credentiali i pozornej separacji w warstwie prezentacji. Dla architekta kluczowym artefaktem staje się powtarzalny test izolacji tenantów wykonywany przy każdym rolloutcie, a nie sama lista certyfikatów dostawcy.

## To obserwować

- pełny raport ClusterMAX 3.0 i sposób identyfikowania dostawców bezpiecznych operacyjnie;
- czas od publikacji poprawki do aktualizacji sterownika NVIDIA, GPU Operatora, runc/Docker i firmware'u ConnectX u poszczególnych neocloudów;
- wymuszanie `default deny` i blokad privileged/hostPath przez admission policy;
- separację API Prometheus/Grafana oraz danych telemetrycznych między tenantami;
- praktyczne wdrożenia P_Key, M_Key, SA_Key i VS_Key w wielotenantowych fabric InfiniBand.
