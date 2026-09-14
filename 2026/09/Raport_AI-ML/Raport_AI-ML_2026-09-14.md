# Raport AI-ML — 2026-09-14

Okres: od edycji 2026-09-11 do 2026-09-14, 07:30 CEST.

## Biznes

### UAE-US AI Campus: rozważane rozproszenie zamiast pojedynczego kampusu 5 GW

11 września Reuters, na podstawie sześciu źródeł, podał, że po atakach na infrastrukturę regionu rozważana jest zmiana planu 5 GW kampusu w Abu Zabi na sieć lokalizacji w ZEA; G42 stwierdził, że prace przebiegają zgodnie z planem, a szczegóły pozostają w przeglądzie. Nie ustalono jeszcze ostatecznego projektu, kosztu ani wpływu na terminy; pierwsze 200 MW fazy Stargate UAE było planowane na 2026 r. Rozproszenie może poprawić odporność na zdarzenia fizyczne, ale zwiększa zależność od DCI, synchronizacji danych, zapasu mocy/chłodzenia w wielu miejscach i automatyzacji przełączania obciążenia.

**Źródło:** [Reuters, 11.09.2026](https://www.reuters.com/world/middle-east/uae-revises-ai-data-center-plan-after-iranian-attacks-sources-say-2026-09-11/).

## Technologia

### 4-hi HBM4E: przepustowość bez nadmiarowej pojemności w decode

W analizie z 13 września SemiAnalysis wskazuje, że cztery warstwy DRAM wystarczają do obsługi 2048 linii I/O stosu HBM4E, więc redukcja wysokości z 8-hi do 4-hi nie musi obniżać przepustowości pojedynczego stosu, ale zmniejsza pojemność. Dla inferencji ograniczonej przepustowością może to poprawić koszt na jednostkę pasma i wykorzystanie deficytowych wafli HBM; autorzy modelują również niższą pamięć Rubin Ultra (192 GB wobec 288 GB), co należy traktować jako ustalenie analityków, nie potwierdzoną konfigurację produktu. Korzyść zanika przy długich kontekstach, dużym KV cache, wysokiej współbieżności lub treningu, gdzie pojemność i transfer między akceleratorami znów dominują; wymagane są pomiary throughput, p95/p99 latency i kosztu/token dla własnego workloadu.

**Źródło:** [SemiAnalysis, 13.09.2026](https://newsletter.semianalysis.com/p/long-live-the-short-king-why-4-hi).

## Implikacje praktyczne (3–5 lat)

1. Rozdzielić profile capacity-planning dla prefill, decode i treningu; modelować HBM GB, GB/s, KV cache i przepustowość sieci zamiast jednego wskaźnika „GPU na model”.
2. W przetargach wymagać pomiarów p95/p99 token latency, tokenów/s/W oraz kosztu na token dla rzeczywistych długości kontekstu i współbieżności; nie zakładać, że większy HBM zawsze poprawia TCO.
3. Dla geograficznie rozproszonego klastra weryfikować przepustowość DCI, RPO/RTO, odporność control plane i koszt replikacji checkpointów; nie przenosić automatycznie kolektywów treningowych przez WAN.
4. Utrzymywać opcję przenośności modeli i orkiestracji między GPU a ASIC; plan dostaw HBM i energii oraz awarie regionów uwzględnić w SLO i umowach capacity reservation.

## Trend tygodnia

Ograniczeniem skali AI jest coraz częściej nie samo FLOPS, lecz dostępna pamięć o wysokim paśmie, zasilanie i odporność lokalizacji. SemiAnalysis proponuje ograniczanie pojemności HBM tam, gdzie profil decode wykorzystuje pasmo, lecz nie dodatkowe gigabajty; to hipoteza wymagająca testów obciążeń. Dyskusja o rozproszeniu kampusu w ZEA pokazuje, że architektura fizyczna i ryzyko geopolityczne wpływają na projekt sieci i koszt niezawodności. Oba przypadki przemawiają za planowaniem na poziomie całego systemu, nie katalogowej karty GPU.

## To obserwować

- Oficjalne parametry pamięci Rubin Ultra, liczba stosów i dostępność HBM4E 4-hi/8-hi.
- Zależność p99 decode latency i kosztu/token od długości kontekstu oraz zajętości KV cache.
- Ostateczny układ lokalizacji UAE-US AI Campus i status pierwszych 200 MW.
- Koszt i SLO replikacji/odtwarzania obciążeń między regionami oraz dostępność mocy i chłodzenia.
