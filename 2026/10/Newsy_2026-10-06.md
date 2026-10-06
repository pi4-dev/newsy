# Newsy — 2026-10-06

**Data podsumowania:** 2026-10-06  
**Okno przyrostowe:** od raportu 2026-10-05 do 2026-10-06 08:29 CEST.

## Raport technologiczny

### Technologia

#### KVM/Firecracker: zgłoszony guest-to-host escape, ale bez publicznego root-cause

Badacz Paulos Yibelo poinformował o uzyskaniu guest→host root w ramach Vercel Sandbox bounty; Vercel Sandbox wykorzystuje Firecracker/KVM. Na moment raportu brak publicznego CVE, reproduktora i technicznego advisory pozwalającego stwierdzić, czy błąd leży w KVM, Firecracker, konfiguracji Vercel czy w interakcji tych warstw.

**Znaczenie:** dla platform uruchamiających nieufne workloady/agent sandboxes jest to sygnał wysokiego priorytetu do ograniczenia blast radius: separacja tenantów na hostach, ograniczenie nested virtualization i device exposure, szybki kernel/VMM patching oraz telemetryka host-side. Nie ma podstaw do awaryjnego traktowania wszystkich instalacji KVM jako podatnych przed publikacją szczegółów.

**Źródła:** https://www.theregister.com/offbeat/2026/10/06/security-researcher-claims-to-they-found-kvm-guest-host-escape-flaw/ ; https://cybernews.com/security/critical-kvm-zero-day-vulnerability-allows-vm-escape/

## Raport AI-ML

### Biznes

#### Power availability staje się twardym ograniczeniem harmonogramu AI factory

Morgan Stanley wskazuje, że niedobór dostępnej mocy zaczyna przesuwać terminy wdrożeń AI i może zmieniać profil popytu w łańcuchu dostaw: najbardziej narażone są komponenty, których dostawy są zsynchronizowane z uruchomieniem całych kampusów. NVIDIA i Broadcom są oceniane jako relatywnie lepiej zabezpieczone kontraktowo, ale opóźnienia energii przenoszą ryzyko na pamięci, optykę i pozostałe elementy BOM.

**Analiza:** capacity planning powinien traktować secured energization date jako zasób równie krytyczny jak przydział GPU. Zakup akceleratorów przed potwierdzeniem harmonogramu grid/substation może generować stranded inventory, koszty finansowania i skrócenie ekonomicznego okresu użycia generacji GPU.

**Źródło:** https://www.reuters.com/business/nvidia-broadcom-shielded-ai-power-crunch-hits-chip-supply-chain-says-morgan-2026-10-05/

### Technologia

#### OpenAI Jalapeño: dojrzałość host platform wygrywa z teoretycznie nowszym CPU

OpenAI wdraża Jalapeño ASIC z hostami AMD EPYC Turin i 1.5 TB RAM zamiast NVIDIA Vera. Według OpenAI kluczowe są dojrzałość platformy, doświadczenie operacyjne partnerów i serwisowalność socketed CPU; różnica benchmarkowa Vera nie kompensuje obecnie ryzyka platformowego.

**Analiza:** w rack-scale AI host CPU nadal wpływa na failure domains, provisioning i MTTR, mimo że nie jest głównym compute engine. Heterogeniczny ASIC+EPYC zmniejsza lock-in względem jednego dostawcy, ale zwiększa macierz walidacji firmware, NUMA/IOMMU, PCIe/CXL i telemetryki.

**Źródło:** https://www.tomshardware.com/pc-components/cpus/openais-jalapeno-asics-are-deployed-alongside-amd-epyc-turin-cpus-as-hosts-hardware-vp-says-nvidias-vera-standalone-is-a-little-bit-behind-on-that-maturity-level

### Implikacje praktyczne

1. Rezerwować i kontraktować moc, substation oraz energization milestones przed finalnym GPU purchase schedule.
2. W TCO uwzględniać stranded-GPU risk wynikający z opóźnienia energii, chłodzenia i sieci.
3. Host CPU/platform wybierać również według firmware maturity, field replaceability i MTTR, nie tylko benchmarków.
4. Dla heterogenicznych ASIC/GPU utrzymywać osobną macierz kompatybilności host CPU, IOMMU/PCIe/CXL, NIC/DPU i firmware.

### Trend tygodnia

Wąskim gardłem AI factory coraz częściej nie jest dostępność samego akceleratora, lecz zdolność do uruchomienia kompletnego megawatowego systemu w terminie. Jednocześnie operatorzy zaczynają optymalizować komponenty pomocnicze pod ryzyko operacyjne i dojrzałość, zamiast maksymalizować każdy benchmark. To przesuwa przewagę z samego procurement GPU na koordynację power, cooling, network, host platform i finansowania.

### To obserwować

- secured MW vs announced MW w nowych kampusach AI;
- opóźnienie grid connection / energization względem GPU delivery;
- stranded inventory i wykorzystanie GPU w pierwszych 6–12 miesiącach;
- adopcję AMD EPYC Turin jako host CPU dla custom ASIC;
- relację czasu życia platformy hosta do cyklu wymiany akceleratorów.

## euro neocloud

### Nscale

#### Loughton: ryzyko wieloletniego opóźnienia zasilania dla kampusu do 90 MW

Nscale nadal opisuje Loughton jako lokalizację zdolną skalować przydział mocy do 90 MW. Nowsze doniesienia wskazują jednak, że wystarczająca moc sieciowa może nie być dostępna do wczesnych lub środkowych lat 2030., podczas gdy pierwotne plany zakładały uruchomienie znacznie wcześniej. Nscale analizuje przyspieszenie przyłączenia i generację on-site.

**Ocena analityczna:** wykryty sygnał ostrzegawczy dotyczy przede wszystkim execution/capacity risk, a nie płynności. Przy planowanym GPU cloud wieloletnia różnica między compute delivery a grid energization może wymusić przenoszenie capacity do innych lokalizacji, własną generację lub zmianę harmonogramu klientów.

**Ocena ryzyka: Wysokie** dla terminowości capacity w Loughton; **nie przenoszę tej oceny automatycznie na całą kondycję finansową Nscale**.

**Źródła:** https://www.nscale.com/ai-infrastructure ; https://www.techradar.com/pro/nvidia-backed-neocloud-nscale-faces-years-of-delay-because-uks-largest-ai-supercomputer-cant-get-enough-power-from-the-grid-to-feed-its-data-centers

## Newsletters summary

W bieżącym przebiegu nie było nowych nieprzeczytanych wiadomości z etykietą `NEWSY`. Wiadomości sklasyfikowane podczas wcześniejszej próby 2026-10-06 zachowują nadane etykiety; nie wykonano ponownej modyfikacji, aby utrzymać idempotencję.
