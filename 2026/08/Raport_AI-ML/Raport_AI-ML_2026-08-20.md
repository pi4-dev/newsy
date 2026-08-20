# Raport AI-ML — 2026-08-20

**Data podsumowania:** 2026-08-20  
**Okres analizy:** 2026-08-19 – 2026-08-20 (Europe/Warsaw)  
**Aktualizacja:** wydanie uzupełnione o informacje niewystępujące we wcześniejszej edycji z tego dnia.

## Biznes

### Google dywersyfikuje custom silicon przez rozszerzenie współpracy z Marvell

Google i Marvell rozszerzyły współpracę przy niestandardowych układach dla infrastruktury AI. Umowa komercyjna została zawarta 29 lipca, natomiast 18 sierpnia Marvell wyemitował warrant dający Google prawo zakupu do ok. 58,97 mln akcji po 206,58 USD; większość vestingu jest powiązana z kwalifikowanymi zakupami custom silicon, które przy pełnym vestingu odpowiadałyby ok. 120 mld USD przychodów do roku fiskalnego 2033. Zakres obejmuje nie tylko akceleratory inference, ale również kontrolery sieciowe, storage, memory-interface oraz near-memory compute.

**Analiza architektoniczna:** jest to sygnał dalszej dywersyfikacji hyperscalerowego łańcucha dostaw poza jeden ASIC partner i poza GPU jako jedyną klasę akceleratora. Dla dużych organizacji zwiększa to znaczenie warstwy abstrakcji nad różnymi backendami obliczeniowymi, ponieważ TPU/custom ASIC będą coraz częściej miały inne charakterystyki throughput, pamięci, sieci i narzędzi observability niż klastry GPU. Ryzykiem pozostaje lock-in do toolchainu konkretnego hyperscalera oraz nierówna przenośność workloadów między TPU, GPU i innymi ASIC.

**Źródła:** [Reuters](https://www.reuters.com/technology/marvell-grants-google-122-billion-stock-warrant-custom-chip-deal-2026-08-19/), [Marvell Investor Relations](https://investor.marvell.com/sec-filings/current-reports)

## Technologia

### MLflow CVE-2026-64849: krytyczny unauthenticated SSRF trafił do CISA KEV

MLflow ma krytyczną podatność CVE-2026-64849 (CVSS 9.3) w mechanizmie webhooków. Domyślny Tracking Server może udostępniać bez uwierzytelnienia endpoint testowy webhooka; walidacja sprawdza pierwotny URL, ale starsza implementacja pozwalała na obejście przez przekierowanie HTTP lub DNS rebinding, co umożliwiało odczyt usług wewnętrznych oraz cloud instance metadata. Poprawka w MLflow 3.15.0 wprowadza ochronę przy zestawianiu połączenia i walidację adresu peer również dla kolejnych połączeń po redirectach; CISA dodała CVE do katalogu Known Exploited Vulnerabilities 19 sierpnia.

**Analiza architektoniczna:** Tracking Server nie powinien być traktowany jako zwykła aplikacja pomocnicza — często posiada dostęp sieciowy i IAM do registry, artifact stores, baz danych oraz usług chmurowych. SSRF może więc przełamać granicę między MLOps control plane i wewnętrzną siecią. Oprócz aktualizacji do 3.15.0 należy ograniczyć ingress do MLflow, wymusić auth, zastosować egress filtering blokujący link-local/RFC1918 tam, gdzie nie są wymagane, a w chmurze ograniczyć rolę instancji i używać mechanizmów odpornych na kradzież metadata credentials.

**Źródła:** [MLflow GitHub Security Advisory GHSA-7gwp-5pfp-969j](https://github.com/mlflow/mlflow/security/advisories/GHSA-7gwp-5pfp-969j), [MLflow 3.15.0](https://github.com/mlflow/mlflow/releases/tag/v3.15.0), [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

## Implikacje praktyczne

1. **Traktować MLOps jako część security-sensitive control plane.** MLflow, registries, model gateways i artifact stores powinny mieć własne segmenty, RBAC, egress policy i telemetrykę bezpieczeństwa, a nie domyślne szerokie trasy do sieci zarządzającej.
2. **Wprowadzić politykę multi-accelerator portability.** Dla nowych platform warto utrzymywać warstwę API/runtime pozwalającą benchmarkować GPU, TPU i custom ASIC według tego samego SLO: p50/p99 latency, tokens/s lub samples/s, koszt/job i energia/job.
3. **Nie opierać capacity planning wyłącznie na FLOPS.** Rosnąca liczba custom ASIC zwiększa znaczenie przepustowości HBM, interconnectu, host-network path i pamięci zewnętrznej; modele kosztowe powinny uwzględniać cały execution path.
4. **Skrócić SLA dla podatności w MLOps.** Publicznie osiągalne komponenty tracking/serving powinny mieć priorytet patchowania zbliżony do systemów IAM i API gateway, szczególnie gdy CVE trafia do KEV.

## Trend tygodnia

Warstwa infrastruktury AI coraz wyraźniej rozdziela się na dwa równoległe zjawiska: hyperscalerzy pogłębiają inwestycje w custom silicon i dywersyfikują dostawców, a jednocześnie software control plane wokół MLOps staje się pełnoprawnym celem ataków. W efekcie przewaga architektoniczna nie będzie wynikała tylko z wyboru najszybszego akceleratora, ale z przenośności workloadów, kontroli kosztu oraz bezpieczeństwa całego łańcucha od schedulerów i registry po storage i sieć.

## To obserwować

- tempo wdrażania MLflow 3.15.0+ i dalsze CVE w webhook/model-registry paths,
- faktyczny udział Marvell w kolejnych generacjach Google TPU i kontrolerów infrastrukturalnych,
- wpływ dywersyfikacji custom silicon na Broadcom i na dostępność alternatyw dla GPU,
- narzędzia observability porównujące SLO i koszt między GPU/TPU/custom ASIC,
- standardy egress policy i workload identity dla komponentów MLOps.