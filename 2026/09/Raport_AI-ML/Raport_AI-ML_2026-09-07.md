# Raport AI-ML — 2026-09-07

Zakres: od ostatniej edycji z 2026-09-04 do 2026-09-07 (Europe/Warsaw).

## Biznes

### HyperVault: kampus AI do 1 GW i 70 000 crore INR

5 września TCS poinformował, że HyperVault zabezpieczył 264 akry w Hyderabadzie pod kampus data center o docelowej mocy do 1 GW; HyperVault i partnerzy mają zainwestować do 70 000 crore INR (około 7,4 mld USD). Budowa ma przebiegać etapami według popytu, z infrastrukturą high-density GPU, chłodzeniem cieczą, energią odnawialną i projektem neutralnym wodnie. Skala przesuwa główne ryzyko z dostępności serwerów na przyłącze, harmonogram faz, łańcuch dostaw chłodzenia i zdolność utrzymania SLO przy mieszanych profilach trening/inference.

**Znaczenie architektoniczne:** capacity planning powinien traktować 1 GW jako pułap programu, nie dostępną od razu moc. Dla klientów kluczowe będą ujawnione MW oddane do eksploatacji, gęstość na rack, PUE/WUE, redundancja przyłącza oraz parametry fabric i storage na etapie każdej fazy.

**Źródła:** [TCS — komunikat z 5 września 2026](https://www.tcs.com/who-we-are/newsroom/press-release/tcs-hypervault-establish-large-scale-ai-data-center-campus-telangana), [Reuters — wartość inwestycji](https://www.reuters.com/world/india/indias-tcs-unit-invest-up-74-billion-ai-data-center-campus-2026-09-05/)

### Chińscy producenci akceleratorów zwiększają presję na ekosystem CUDA

Reuters Breakingviews wskazał 7 września przyspieszenie komercjalizacji Enflame, Moore Threads, MetaX i Biren: Enflame wykazał wzrost sprzedaży o 1475% r/r i pozyskał około 908 mln USD w IPO, a udział NVIDIA w chińskim rynku układów AI oszacowano na 55%, wobec wcześniejszej niemal monopolistycznej pozycji. To nie dowodzi jeszcze równoważności pełnego stosu, ale zwiększa prawdopodobieństwo regionalnej fragmentacji toolchainów, kompilatorów, bibliotek komunikacyjnych i formatów wdrożeniowych, szczególnie dla energooszczędnego inference.

**Znaczenie architektoniczne:** portability nie może kończyć się na ONNX lub zgodności API. Należy mierzyć pokrycie operatorów, stabilność kompilatora, collectives, profilery, debugowanie i observability dla każdego backendu oraz utrzymywać obrazy i testy wydajnościowe niezależne od dostawcy.

**Źródło:** [Reuters Breakingviews — 7 września 2026](https://www.reuters.com/commentary/breakingviews/chinas-ai-dragons-breathe-fire-nvidias-moat-2026-09-07/)

## Technologia

### Reasoning na Jetson: NVFP4 i speculative decoding w vLLM

NVIDIA opublikowała 4 września receptury dla Nemotron 3.5 Lightning 30B-A3B i Qwen3.8-27B na Jetson AGX Thor/Orin z JetPack 7.2 i vLLM 0.28.0. NVFP4 wraz ze speculative decoding dały w testach do 6,28× wzrostu decode throughput względem BF16, ale optymalny mechanizm zależał od modelu: DSpark dla Nemotron i DFlash2 dla Qwen; wyniki aplikacyjne wynosiły odpowiednio 123,01–138,02 oraz 27,69–34,44 tokena/s zależnie od kategorii promptu.

**Znaczenie architektoniczne:** wzrost throughput nie jest przenośnym parametrem SKU. Trzeba monitorować acceptance rate draft tokenów, TTFT, ITL, zużycie KV cache, jakość po kwantyzacji i profil energetyczny na reprezentatywnych promptach; przy niskiej akceptacji koszt draftowania może zniwelować zysk. Lokalne inference ogranicza zależność sieciową i ekspozycję danych, ale wzmacnia lock-in do NVFP4, JetPack i zoptymalizowanych checkpointów.

**Źródło:** [NVIDIA Technical Blog — 4 września 2026](https://developer.nvidia.com/blog/frontier-reasoning-reaches-the-edge-how-to-deploy-and-optimize-models-on-nvidia-jetson/)

## Implikacje praktyczne

1. Kontraktować moce data center etapami, wiążąc płatności z dostarczonymi MW, PUE/WUE, gęstością racków i zweryfikowanym SLO fabric/storage.
2. Utrzymywać sprzętowo niezależny zestaw testów: operator coverage, collectives, kompilacja, profilowanie, checkpoint conversion i jakość po kwantyzacji.
3. Dla edge inference porównywać konfiguracje na poziomie aplikacji, nie tylko tokenów/s; acceptance rate speculative decoding powinien być metryką produkcyjną.
4. Projektować fallback z lokalnego agenta do regionalnego endpointu oraz tryb deterministyczny przy utracie modelu, draft checkpointu lub łączności.
5. W horyzoncie 3–5 lat zakładać jednocześnie presję na energię/przyłącza i fragmentację stosów akceleratorów; największym kosztem migracji może być oprogramowanie operacyjne, nie sam model.

## Trend tygodnia

Ograniczeniem skali staje się zdolność oddania kompletnej infrastruktury energetycznej i chłodniczej, dlatego nowe kampusy są komunikowane jako programy wielofazowe, a nie jednorazowe klastry. Jednocześnie inference rozchodzi się w przeciwnych kierunkach: do kampusów gigawatowych i na urządzenia edge z agresywną kwantyzacją. Rosnąca liczba regionalnych akceleratorów wymusza traktowanie przenośności jako testowanego SLO całego toolchainu.

## To obserwować

- Faktycznie oddane MW, PUE/WUE i pierwszych klientów HyperVault w Hyderabadzie.
- Terminy przyłącza, dostępność wody i gęstość mocy w kolejnych fazach kampusu.
- Pokrycie operatorów i collectives w chińskich stosach akceleratorów względem CUDA.
- Acceptance rate i jakość NVFP4 dla DSpark/DFlash2 na workloadach produkcyjnych.
- Stabilność vLLM 0.28.0 i kompatybilność checkpointów na JetPack 7.2.

## Kontrola źródeł

Sprawdzono wskazane kanały SemiAnalysis, NVIDIA Developer Blog, AIwire i HPCwire. Do raportu włączono tylko informacje nowe względem rejestru z poprzednich edycji i mające bezpośredni wpływ na architekturę infrastruktury AI/ML.
