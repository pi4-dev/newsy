# Newsletters summary — 2026-09-02

**Data podsumowania:** 2026-09-02  
**Źródło:** nieprzeczytane wiadomości w Inboxie z etykietą NEWSY, otrzymane 2026-09-01.

## Microsoft UFO MCP — zdalne sterowanie urządzeniem bez uwierzytelnienia

- **Technologia / Zdarzenie:** [CVE-2026-73296 w Microsoft UFO MCP](https://gbhackers.com/critical-microsoft-ufo-mcp-flaw/)
- **Mechanizm działania:** W wersjach do 3.0.7 usługi na TCP 8020/8021 mogą przy konfiguracji z bindem do `0.0.0.0` przyjmować bez uwierzytelnienia polecenia ADB: zrzuty ekranu, odczyt drzewa UI, tap/swipe i wprowadzanie tekstu.
- **Wpływ na architekturę:** Agentowy control plane urządzeń mobilnych staje się zdalnym interfejsem administracyjnym i wymaga osobnej strefy sieciowej, mTLS lub uwierzytelnionego reverse proxy, ograniczenia źródeł oraz pełnego audytu działań.
- **Failure modes i edge cases:** Samo ukrycie portów za NAT nie jest kontrolą dostępu; błędna publikacja w Kubernetes/VM może ponownie wystawić usługę. Do czasu poprawki potrzebny jest deny-by-default, blokada egress ADB i ręczny fallback.

## Prompt injection w źródłach OSINT

- **Technologia / Zdarzenie:** [Ukryte instrukcje w dokumentach używanych przez agentów OSINT](https://www.dutchosintguy.com/post/when-the-source-attacks-back-prompt-injection-is-coming-for-osint)
- **Mechanizm działania:** Instrukcje są ukrywane poza viewportem CSS, białym tekstem w PDF lub w niewidocznych komórkach arkusza. Parser przekazuje je modelowi, choć analityk nie widzi ich w renderowanym materiale, co może wymusić fałszywą narrację.
- **Wpływ na architekturę:** Pipeline ingestu powinien przechowywać hash oryginału, render i niezależnie wyekstrahowany tekst oraz wskazywać rozbieżności. Model analizujący niezaufane źródła powinien działać read-only bez credentiali i narzędzi mutujących.
- **Failure modes i edge cases:** OCR i parsery mogą różnie normalizować warstwy, a prosty filtr słów nie wykryje instrukcji rozproszonych między obiektami. Deterministyczny fallback to porównanie Tika/OCR z renderingiem, provenance per fragment i ręczna akceptacja przed działaniem.

## Meraki AI Assistant tworzy zgłoszenia dla Client VPN i switchingu

- **Technologia / Zdarzenie:** [AI-powered support cases w Meraki Dashboard](https://community.cisco.com/t5/cloud-networking-feature-announcements/coming-aug-31st-ai-powered-support-cases-in-client-vpn-and/ba-p/5571255)
- **Mechanizm działania:** Asystent analizuje stan środowiska, proponuje kroki diagnostyczne i generuje podsumowanie zgłoszenia, które administrator zatwierdza przed wysłaniem do wsparcia.
- **Wpływ na architekturę:** Skraca to triage, lecz wymaga jawnej kontroli zakresu telemetrii przekazywanej dostawcy, redakcji sekretów oraz korelacji sugestii z surowymi logami i zmianami konfiguracji.
- **Failure modes i edge cases:** Halucynacja przyczyn może skierować diagnostykę na złą warstwę, a niepełna telemetria ukryć zależność od RADIUS, DNS lub routingu. Zatwierdzenie przez człowieka, niezmienny bundle dowodowy i klasyczny kanał TAC muszą pozostać fallbackiem.
