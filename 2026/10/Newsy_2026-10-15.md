# Newsy — 2026-10-15

**Okno przyrostowe:** od raportu 2026-10-14 do 2026-10-15 07:28 CEST.
**Uwaga:** RSS niedostępne z sandboxa; wyszukiwanie WWW nie zwróciło nowych, datowanych pozycji (Raport technologiczny, AI-ML, euro neocloud, AI for networking pominięte). Treść pochodzi z newsletterów TLDR (14.10). Maia 300 pominięta jako duplikat z 2026-10-14.

## Newsletters summary

#### GitLab CVE-2026-97229 (CVSS 10.0): nieuwierzytelniony RCE przez walidację regex w konfiguracji CI/CD
**Zdarzenie:** integer overflow w walidacji regex konfiguracji CI/CD pozwala w określonych warunkach nieuwierzytelnionemu użytkownikowi wykonać kod na serwerze. Poprawki: CE/EE 19.2.8, 19.3.5, 19.4.2 (+ XSS, authz, response-cache poisoning). **Wpływ:** instancje self-managed dostępne z internetu to priorytet patchowania; runnery i sekrety CI na tym samym hoście zwiększają blast radius. **Failure modes:** warunki eksploatacji niepublikowane w newsletterze; do czasu patcha ograniczyć ekspozycję (VPN/allowlist), rotacja tokenów po podejrzeniu kompromitacji.
Źródło: https://thehackernews.com/2026/10/gitlab-patches-cvss-100-flaw-that-could.html

#### Wycieki kluczy AWS testowane pod Bedrock (Datadog)
**Mechanizm:** platformy harvestujące klucze sprawdzają dostęp wywołaniami ListFoundationModels, InvokeModel i Converse oraz dostępne kredyty promocyjne. **Wpływ:** skradzione klucze monetyzowane przez inferencję na koncie ofiary (koszt, LLMjacking); alerty na Bedrock API z nowych ASN/kluczy, SCP ograniczające modele/regiony, budżety i anomaly detection. **Failure modes:** brak CloudTrail data events dla Bedrock maskuje nadużycie; klucze długoterminowe IAM user nadal główny wektor.
Źródło: https://securitylabs.datadoghq.com/articles/beyond-valid-credentials-how-exposed-aws-keys-are-tested-for-amazon-bedrock-access/

#### Ghostscript RCE przez przetwarzanie PDF (Wordfence)
**Mechanizm:** spreparowany PDF przetwarzany przez podatny Ghostscript przy generowaniu podglądów/miniatur daje RCE; wymagany upload PDF. **Wpływ:** WordPress i każdy pipeline thumbnailowania/konwersji (DMS, mail gateway, ImageMagick→gs). Izolować konwertery (kontener bez sieci, seccomp), zaktualizować gs. **Failure modes:** brak numeru CVE i wersji naprawionych w newsletterze – zweryfikować w advisory dystrybucji.
Źródło: https://links.tldrnewsletter.com/QwoarX

#### Nvidia Run:ai na open source (KAI Scheduler, GPU Fractioning Operator)
**Zdarzenie:** pełna platforma Run:ai opublikowana jako OSS w ramach DSX AI Factory: KAI Scheduler, GPU Fractioning Operator, KAI Resource Management, Workload Map. **Wpływ:** darmowy scheduler GPU z fractioningiem i kolejkami dla K8s – mniejsza zależność od licencji, ale pogłębia ekosystem Nvidia (DSX). Dla neocloudów/on-prem: wyższa utylizacja GPU przy współdzieleniu. **Failure modes:** izolacja fractioningu (brak twardej izolacji pamięci/QoS), dojrzałość OSS i model wsparcia nieznane.
Źródło: https://developer.nvidia.com/blog/nvidia-runai-is-now-open-source-as-part-of-nvidia-dsx-ai-factory-platform

#### AWS „priming": hybrydowe modele SSM z wag Transformera, koszt pretrainingu −99%
**Mechanizm:** reuse wag pretrenowanego Transformera, część warstw attention zastąpiona warstwami state-space (Gated KalmaNet); deklarowane dopasowanie benchmarków rozumowania i dekodowanie do 2× szybsze. Wydano modele, kod treningowy i plugin vLLM (Apache 2.0). **Wpływ:** mniejszy KV-cache → większy batch/dłuższy kontekst na GPU, niższy koszt/token. **Failure modes:** wyniki z publikacji własnej, jakość na long-context retrieval i dojrzałość pluginu vLLM do zweryfikowania.
Źródło: https://www.amazon.science/blog/can-we-change-an-ai-models-architecture-without-retraining-it-from-scratch

#### Broadcom VMware HyperArmor dla VCF
**Mechanizm:** virtual patching, application allowlisting i MFA na poziomie hypervisora bez agentów w VM i bez przestoju. **Wpływ:** odpowiedź na ataki na ESXi (kradzione poświadczenia, niezałatane luki); kompensuje okno patchowania. **Failure modes:** virtual patch ≠ fix, ryzyko fałszywych blokad allowlisty; dodatkowy lock-in w VCF. Szczegóły licencjonowania nieznane.
Źródło: https://investors.broadcom.com/news-releases/news-release-details/broadcom-introduces-vmware-hyperarmor-expanding-hypervisor

#### Gemini 4 Argon GA (Gemini API, Google Cloud), promocyjna cena 2/10 USD za 1M tokenów
**Zdarzenie:** model flagowy dostępny w płatnym API, AI Studio, Antigravity i na Google Cloud; cena „limited-time" (2 USD wejście / 10 USD wyjście wg tytułu VentureBeat). **Wpływ:** presja cenowa na inferencję; budżetować po cenie docelowej, nie promocyjnej. **Failure modes:** brak ceny docelowej i limitów rate w źródle; ryzyko lock-in przez Antigravity/Gemini Enterprise.
Źródła: https://venturebeat.com/technology/google-makes-gemini-4-argon-broadly-available-with-limited-time-low-api-pricing-of-2-10-per-1m-token, https://cloud.google.com/blog/products/ai-machine-learning/gemini-4-argon-available-on-google-cloud

#### Anthropic: własny system kontroli wersji Scion zamiast git
**Zdarzenie:** migracja monorepo w 5 miesięcy; merge'e z ok. 2500/dzień (maj) do ponad 30 000/dzień; własne code review, search i IAM kodu. **Wpływ:** przy ruchu generowanym przez agentów git/PR-review staje się wąskim gardłem – wzorzec dla platform engineering (merge queue, IAM per repo, automatyczna weryfikacja). **Failure modes:** koszt utrzymania własnego VCS; wnioski nieprzenoszalne na typowe skale. Źródło pierwotne Anthropic (opis własny).
Źródło: https://www.anthropic.com/engineering/redesigning-source-control

#### Drobne (krótko)
Racket: koparka na build serwerze Northwestern ujawniła klucze podpisu Apple/Windows – rotacja, reinstalacja macOS 8.12–9.3 (https://blog.racket-lang.org/2026/10/infrastructure-compromise.html). Intel Connectivity Performance Suite powoduje awarie IPv6 w OneDrive/OneNote – usunąć/zaktualizować (https://www.theregister.com/networks/2026/10/13/intel-optimizes-ipv6-networking-into-nothingness-say-users/5302622). Trezor One/T z implantami z nieautoryzowanych kanałów sprzedaży (https://www.zetter-zeroday.com/trezor-wallets-also-compromised-with-implants/). Pominięto: Character.ai Chapter 11, aresztowanie pracownika OpenAI, produkty konsumenckie.
