2026-09-15

# Newsletters summary

Nowa wiadomość SemiAnalysis z 14 września. Aktualizacja tematu edge/datacenter: poniżej wyłącznie nowe ilościowe porównanie, bez powtarzania wcześniejszego opisu podziału warstw sterowania.

## Ilościowy koszt obsługi floty robotów

**Technologia / Zdarzenie:** [SemiAnalysis — on-device vs datacenter, 14 września 2026](https://newsletter.semianalysis.com/p/a-brain-too-big-to-carry-on-device).

**Mechanizm działania:** Autorzy odtworzyli profil obliczeniowy RoboTTT na checkpointach GR00T N1.7, z blokami test-time training i 151 MB stanu fast weights na robota. To rekonstrukcja do pomiaru czasu i ruchu pamięci, nie walidacja skuteczności oryginalnej polityki; B300 obsługiwał 12 robotów na GPU, RTX PRO 6000 cztery, przy budżecie chunku 500 ms.

**Wpływ na architekturę:** Model dla 96 robotów daje agregat TCO 18,63 USD/h dla B300, 15,61 dla RTX PRO 6000 i 14,97 dla Jetson Thor. Przewaga offloadu po uwzględnieniu wykorzystania zależy od przyjętych 90% dla serwera i 40% dla urządzeń; to ekonomika owner/operator, nie cena chmury. Ocena analityczna: wymagać pomiaru równoczesności floty i tail latency przed centralizacją.

**Failure modes i edge cases:** Nie utożsamiać dotrzymania deadline z poprawnością działań. Ocena analityczna: utrata łącza, jitter i przeciążenie kolejki wymagają lokalnego bezpiecznego zatrzymania; obniżenie wykorzystania serwera może zniwelować korzyść poolingową.
