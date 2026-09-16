# Raport technologiczny — 2026-09-16

**Data podsumowania:** 2026-09-16  
**Okno przyrostowe:** od edycji z 2026-09-14 do 2026-09-16 07:32 CEST

## Biznes

Brak odrębnych, materialnych zdarzeń biznesowych spełniających kryteria raportu.

## Technologia

### Pierwsze zgłoszenie do AEPD naruszenia danych przypisanego agentowi AI

Hiszpański organ ochrony danych AEPD poinformował 14 września o pierwszym otrzymanym zgłoszeniu naruszenia danych osobowych, w którym atak miał zostać wykonany przez agenta AI korzystającego ze znanego LLM. Według opisu agent wyszukał podatności, poprawnie zalogował się do systemu, zmienił dane osobowe i uzyskał dostęp do faktur przy ograniczonym udziale człowieka; AEPD zastrzega, że zgłoszenie jest nadal analizowane i nie dowodzi kompromitacji modelu ani jego dostawcy.

**Znaczenie:** To praktyczny sygnał, że agentowe narzędzia ofensywne trzeba traktować jako autonomiczne wykonawcze obciążenia, a nie tylko interfejs konwersacyjny. Wymaga to krótkotrwałych poświadczeń, izolacji narzędzi, pełnego audytu wywołań, limitów skutków operacji i detekcji sekwencji „rekonesans → logowanie → modyfikacja/eksfiltracja”, z deterministycznym odcięciem agenta po przekroczeniu polityki.

**Źródło:** [AEPD, 14.09.2026](https://www.aepd.es/prensa-y-comunicacion/blog/primera-notiviacion-brecha-datos-personales-causada-por-ataque-ejecutado-mediante-agente-ia)
