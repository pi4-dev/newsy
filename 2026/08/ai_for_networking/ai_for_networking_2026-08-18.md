# AI for networking — 2026-08-18

**Data podsumowania:** 2026-08-18  
**Okres analizy:** 2026-08-11 – 2026-08-18 (Europe/Warsaw)

## 1. Cisco Nexus Dashboard 4.3: MCP jako kontrolowany interfejs dla agentów NetOps

**Technologia / Zdarzenie:** [AI Ops reimagined: LLMs, Agentic AI, and the ND 4.3 MCP server](https://community.cisco.com/t5/technology-and-support-events-and-webinars/ai-ops-reimagined-llms-agentic-ai-and-the-nd-4-3-mcp-server/ev-p/5559093)

**Mechanizm działania:** Cisco prezentuje planowany MCP Server w Nexus Dashboard 4.3 jako warstwę udostępniającą narzędzia i kontekst operacyjny agentom LLM. Wzorzec łączy LLM/RAG do interpretacji intencji i wiedzy operacyjnej z agentami wywołującymi kontrolowane funkcje platformy oraz z klasyczną automatyzacją closed-loop. Kluczowy element nie polega na zastąpieniu kontrolera modelem generatywnym, lecz na wystawieniu ograniczonego zestawu funkcji przez ustandaryzowany interfejs narzędziowy.

**Wpływ na architekturę:** MCP może stać się warstwą pośrednią pomiędzy agentem AI a kontrolerem fabric, co upraszcza integrację z ACI/NDFC/NDO i umożliwia rozdzielenie reasoning od deterministycznego wykonania. Dla produkcyjnego wdrożenia należy traktować telemetrykę, topologię, stan konfiguracji i wyniki assurance jako ground truth; model nie powinien samodzielnie rekonstruować stanu sieci. Architektura wymaga szczegółowych RBAC, audytu wywołań narzędzi, limitów zakresu działania, pre-change validation i deterministycznej ścieżki rollback.

**Failure modes i edge cases:** Największe ryzyka to halucynowane parametry wywołań, błędna interpretacja intencji, prompt injection z danych operacyjnych, nieaktualny kontekst RAG oraz wyścigi pomiędzy agentem a klasycznym systemem automatyzacji. Closed-loop bez limitów częstotliwości i histerezy może powodować oscylacje konfiguracji. Krytyczne operacje powinny wymagać policy gate, symulacji/pre-check, transakcyjnego commit oraz niezależnej walidacji po zmianie.

**Źródła:**
- https://community.cisco.com/t5/technology-and-support-events-and-webinars/ai-ops-reimagined-llms-agentic-ai-and-the-nd-4-3-mcp-server/ev-p/5559093
- https://www.cisco.com/c/m/en_us/customer-experience/customer-success/resources.html

---

## 2. A2A przechodzi do Agentic AI Foundation — znaczenie dla wieloagentowego zarządzania siecią

**Technologia / Zdarzenie:** [Agent2Agent Protocol przechodzi do Agentic AI Foundation](https://www.axios.com/2026/08/17/a2a-agentic-ai-foundation-open-ai-standards)

**Mechanizm działania:** A2A standaryzuje komunikację pomiędzy niezależnymi agentami, podczas gdy MCP koncentruje się na połączeniu agenta z narzędziami, API i źródłami danych. W kontekście zarządzania siecią istnieją już prace IETF opisujące model Controller Agent ↔ Device Agent, w którym A2A służy do delegowania zadań i koordynacji agentów, a NETCONF/RESTCONF/gNMI pozostają deterministycznymi interfejsami do odczytu stanu i egzekwowania konfiguracji. Przeniesienie A2A do wyspecjalizowanej fundacji zwiększa szansę na stabilniejszy, wielodostawcowy ekosystem agentów.

**Wpływ na architekturę:** Dla NetOps oznacza to możliwy podział płaszczyzny automatyzacji na trzy poziomy: A2A do koordynacji agentów, MCP do bezpiecznego udostępniania narzędzi oraz YANG/gNMI/NETCONF/API kontrolerów do deterministycznego odczytu i wykonania. Taki model ogranicza vendor lock-in na warstwie agentowej, ale wymaga wspólnej semantyki intencji, identyfikacji zasobów, autoryzacji między agentami i spójnego modelu stanu.

**Failure modes i edge cases:** Wieloagentowość zwiększa ryzyko sprzecznych decyzji, pętli delegacji, niejawnej eskalacji uprawnień i utraty spójności stanu. Agent nie powinien uznawać odpowiedzi innego agenta za ground truth bez ponownej walidacji w kontrolerze lub telemetryce. Niezbędne są correlation IDs, budżety kroków, TTL dla delegacji, idempotentne operacje, mechanizmy leader/arbiter dla konfliktów oraz globalny kill switch dla warstwy agentowej.

**Źródła:**
- https://www.axios.com/2026/08/17/a2a-agentic-ai-foundation-open-ai-standards
- https://datatracker.ietf.org/doc/draft-yan-a2a-device-agent-applicability/
- https://datatracker.ietf.org/doc/html/draft-zeng-nmrg-mcp-usecases-requirements-00

## Wniosek architektoniczny

Najbardziej dojrzały wzorzec AIOps dla infrastruktury sieciowej nie polega obecnie na oddaniu agentowi bezpośredniego CLI, lecz na rozdzieleniu **reasoning → policy/validation → deterministic execution → post-change verification**. MCP i A2A zaczynają układać się w warstwę integracyjną powyżej istniejących kontrolerów, natomiast bezpieczeństwo i niezawodność nadal muszą opierać się na klasycznych mechanizmach transakcyjnych, telemetryce i pre-change assurance.