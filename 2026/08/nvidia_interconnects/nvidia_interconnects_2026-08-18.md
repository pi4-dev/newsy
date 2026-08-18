# NVIDIA interconnects — 2026-08-18

Data podsumowania: **2026-08-18**  
Strefa czasowa: **Europe/Warsaw**

Porównano bieżące strony NVIDIA z `data/nvidia-interconnects.json`. W istniejących zakresach wykryto jedną zmianę parametru. Ponadto monitoring rozszerzono o Silicon Photonics, DPU i SuperNIC; pozycje z tych trzech nowych zakresów są **inicjalizacją baseline'u**, a nie dowodem, że produkty zostały wprowadzone 2026-08-18.

## Dodane

### Nowy zakres monitoringu: Silicon Photonics
- **Zakres:** `silicon_photonics`
- **Produkty/rodziny w baseline:** NVIDIA Quantum-X InfiniBand Photonics / Q3450-LD; NVIDIA Spectrum-X Ethernet Photonics Switches.
- **Stan:** aktywne / oferowane na stronie NVIDIA; Spectrum-X Ethernet Photonics opisano jako dostępne w 2H 2026.
- **Źródło:** https://www.nvidia.com/en-us/networking/products/silicon-photonics/

### Nowy zakres monitoringu: DPU
- **Zakres:** `data_processing_units`
- **Produkty w baseline:** NVIDIA BlueField-4 DPU, NVIDIA BlueField-4 STX Storage Processor, NVIDIA BlueField-3 DPU.
- **Stan:** aktywne w portfolio NVIDIA.
- **Źródło:** https://www.nvidia.com/en-us/networking/products/data-processing-unit/

### Nowy zakres monitoringu: SuperNIC
- **Zakres:** `ethernet_supernics`
- **Produkty w baseline:** NVIDIA ConnectX-9 SuperNIC, NVIDIA ConnectX-8 SuperNIC, NVIDIA BlueField-3 SuperNIC.
- **Stan:** aktywne w portfolio NVIDIA.
- **Źródło:** https://www.nvidia.com/en-us/networking/products/ethernet/supernic/

## Usunięte

Brak wykrytych usunięć w dotychczas monitorowanych zakresach.

## Zmienione

### Ethernet switching — SN3420
- **Produkt:** SN3420
- **Pole:** `throughput`
- **Poprzednio:** `2.4 Tb/s; 3.58 Bpps`
- **Obecnie:** `2.4 Tb/s; 3.57 Bpps`
- **Źródło:** https://www.nvidia.com/en-us/networking/ethernet-switching/

Zmiana dotyczy wartości packet rate publikowanej obecnie przez NVIDIA w tabeli produktowej. Nie wpływa na deklarowany switching throughput 2.4 Tb/s.

## Pozostałe zakresy

Nie wykryto merytorycznych zmian w indeksie aktywnych i wycofanych produktów LinkX ani w głównym portfolio InfiniBand względem bieżącego snapshotu.

Źródła:
- https://networking-docs.nvidia.com/interconnect
- https://www.nvidia.com/en-us/networking/ethernet-switching/
- https://www.nvidia.com/en-us/networking/infiniband-switching/
- https://www.nvidia.com/en-us/networking/products/silicon-photonics/
- https://www.nvidia.com/en-us/networking/products/data-processing-unit/
- https://www.nvidia.com/en-us/networking/products/ethernet/supernic/
