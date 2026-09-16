# Newsletters summary — 2026-09-16

**Data podsumowania:** 2026-09-16  
**Zakres:** nieprzeczytane wiadomości z etykietą `NEWSY`, zweryfikowane do 07:32 CEST

## Databricks: wymuszone jawne entitlementy dla nowych principalów

- **Technologia / Zdarzenie:** [Databricks wymusza jawne entitlementy workspace od 14.09.2026](https://docs.databricks.com/aws/en/security/auth/entitlements)
- **Mechanizm działania:** To zmiana control plane IAM, nie algorytmu AI. Grupy systemowe `users` i `admins` nie przenoszą już edytowalnych entitlementów: `users` ma ich zero, `admins` komplet, a przy dodawaniu użytkownika lub service principal trzeba jawnie nadać m.in. `workspace-access`, `databricks-sql-access`, `allow-cluster-create` i `allow-instance-pool-create`.
- **Wpływ na architekturę:** Automatyzacje SCIM/Terraform i onboarding workload identities muszą przypisywać uprawnienia deterministycznie; inaczej nowy principal może istnieć, lecz nie uruchomić jobu ani uzyskać dostępu do SQL. Zmiana poprawia least privilege i czytelność audytu, ale wymaga testów driftu oraz alarmów dla principalów bez oczekiwanych entitlementów.
- **Failure modes i edge cases:** Najbardziej prawdopodobna jest cicha utrata funkcjonalności po utworzeniu principalu, niespójność między starszymi workspace’ami z grupą-klonem a nowym modelem oraz nadanie zbyt szerokich praw jako obejście. Fallback powinien pozostać deklaratywny: wersjonowany mapping ról, test po provisioningu i break-glass admin poza ścieżką agenta.
