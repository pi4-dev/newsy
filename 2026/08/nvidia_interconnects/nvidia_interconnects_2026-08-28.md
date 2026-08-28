# nvidia interconnects — 2026-08-28

Data podsumowania: **2026-08-28**

Porównano znormalizowany snapshot `data/nvidia-interconnects.json` z katalogiem NVIDIA LinkX oraz stronami Ethernet, InfiniBand, Silicon Photonics, DPU i SuperNIC.

## Dodane

### LinkX / 800G Transceivers — MMS4C10-XM800

- **Produkt / numer części:** MMS4C10-XM800; NVIDIA OPN `980-9IAY0-00XM00`
- **Poprzednia wartość:** brak produktu w aktywnym snapshotcie
- **Nowa wartość:** aktywny transceiver Gen2 800 Gb/s DR4, single-port OSFP, 1 × MPO-12/APC, SMF 1310 nm, zasięg do 500 m
- **Kompatybilność:** Ethernet 800GbE oraz InfiniBand XDR; serwery z ConnectX-9
- **Parametry dodatkowe:** 4 × 200G-PAM4 elektrycznie i optycznie, maks. 15 W, CMIS 4.0+, secure firmware boot/update
- **Status dokumentacji:** rewizja 1.0, sierpień 2026; ostatnia aktualizacja 2026-08-27
- **Źródła:** [karta produktu](https://networking-docs.nvidia.com/mms4c1x800), [specyfikacja](https://networking-docs.nvidia.com/mms4c1x800/specifications), [ordering information](https://networking-docs.nvidia.com/mms4c1x800/ordering-information)

Uwaga: nagłówek strony katalogowej błędnie opisuje produkt jako twin-port 2×DR4/2×MPO; treść karty, specyfikacja oraz OPN konsekwentnie podają single-port DR4/1×MPO. Snapshot zachowuje zweryfikowaną wartość single-port i odnotowuje niespójność źródła.

## Usunięte

Brak.

## Zmienione

Brak innych potwierdzonych zmian modeli, parametrów, kompatybilności lub statusów. Zmiany kolejności i formatowania pominięto.
