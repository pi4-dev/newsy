# Raport AI-ML — 2026-08-21

**Data podsumowania:** 2026-08-21  
**Okres analizy:** 2026-08-20 – 2026-08-21 (Europe/Warsaw)

## Biznes

### Brazylia finansuje dwa odrębne stosy superkomputerowe AI

20 sierpnia rząd Brazylii ogłosił inwestycje o łącznej wartości około 2,3 mld BRL (444,2 mln USD). Około 1,3 mld BRL ma sfinansować w Rio de Janeiro infrastrukturę rozwijaną z Huawei i iFlytek do budowy modeli ogólnych i sektorowych, a około 1 mld BRL zostanie przeznaczone w przetargu na superkomputer w Rio Grande do Norte; urzędnicy oczekują oferty NVIDIA, ale dostawca nie został jeszcze formalnie wybrany. Finansowanie ma pochodzić z FNDCT, uruchomienie drugiego systemu zaplanowano do końca 2027 r., a współpracę z chińskimi partnerami od lipca 2027 r.

**Analiza architektoniczna:** rozdzielenie programu między dwa ekosystemy sprzętowe zmniejsza zależność geopolityczną, ale przenosi koszt na portability, operacje i obserwowalność. Organizator powinien z góry ujednolicić kryteria SLO i capacity planning — czas wykonania zadania, wykorzystanie akceleratorów, przepustowość fabric/storage, energia na job i koszt pełnego cyklu — oraz utrzymać formaty danych, checkpointów i interfejsy schedulerów niezależne od backendu. Największym ryzykiem jest powstanie dwóch niekompatybilnych wysp narzędziowych z osobnymi mechanizmami IAM, telemetrii i reagowania na awarie.

**Źródło:** [Reuters — ogłoszenie programu z 20 sierpnia 2026](https://www.reuters.com/world/americas/brazil-launches-ai-supercomputer-push-splits-projects-between-chinese-us-firms-2026-08-20/)

## Technologia

Brak odrębnych, istotnych nowych informacji w analizowanym okresie.

## Implikacje praktyczne

1. **Wymagać przenośnego kontraktu uruchomieniowego.** Kontenery, formaty checkpointów, pipeline danych i interfejs schedulerów powinny być testowane na co najmniej dwóch klasach akceleratorów, bez założenia zgodności CUDA.
2. **Normalizować telemetrykę między stosami.** Jedna warstwa obserwowalności powinna porównywać p50/p99 czasu joba, awaryjność node/fabric, przepustowość storage, energię/job oraz koszt użytecznej jednostki pracy.
3. **Oddzielić identity i policy plane od dostawcy sprzętu.** Federacja IAM i niezależny audyt ograniczają ryzyko, że wymiana dostawcy wymusi przebudowę kontroli dostępu.
4. **Planować capacity na poziomie całego systemu.** Kryteria przetargowe muszą obejmować sieć, pamięć, storage, chłodzenie, energię i dostępność części, nie tylko nominalną wydajność akceleratorów.

## Trend tygodnia

Suwerenność infrastruktury AI coraz częściej oznacza dywersyfikację całych stosów — dostawców układów, sieci, oprogramowania i jurysdykcji danych — zamiast prostego zakupu większej liczby GPU. Taka strategia ogranicza pojedynczy vendor lock-in, ale bez wspólnych SLO i warstwy przenośności może zwiększyć koszt operacyjny oraz obniżyć wykorzystanie zasobów. Capacity planning przesuwa się więc z porównania FLOPS do pomiaru całego execution path i ryzyka łańcucha dostaw.

## To obserwować

- wynik przetargu na superkomputer w Rio Grande do Norte i formalny wybór akceleratorów,
- topologię fabric oraz klasy storage dla obu brazylijskich systemów,
- wymagania dotyczące lokalizacji danych, IAM i audytu między ekosystemami,
- mierzalne kryteria efektywności energetycznej i harmonogram zasilania do końca 2027 r.,
- warstwę portability dla modeli, checkpointów i pipeline'ów między stosami.
