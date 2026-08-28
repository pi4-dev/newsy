# NVIDIA Network Equipment

> Static catalog generated from [`data/nvidia-interconnects.json`](data/nvidia-interconnects.json). Snapshot date: **2026-08-28** (Europe/Warsaw). Only active/current portfolio records are shown. Transceiver detail reflects normalized schema **v4**.

## Ethernet switches

| Model | Family | Speed | Connectors | Port options | Throughput | Height | Cooling | Software | Source |
|---|---|---|---|---|---|---|---|---|---|
| SN6810-LD | Spectrum-6 SN6000 | 800GbE | 128x MMC-12 CPO | 128x800G / 256x400G / 512x200G / 514x100G | 102.4 Tb/s | 2U | liquid-cooled/CPO family | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN6800-LD | Spectrum-6 SN6000 | 200GbE | 512x MMC-12 CPO | 2048x200G / 2056x100G | 409.6 Tb/s (4x102.4) | 5U | liquid-cooled/CPO family | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN6600-LD | Spectrum-6 SN6000 | 800GbE | 64x OSFP 2x800G | 128x800G / 256x400G / 512x200G / 514x100G | 102.4 Tb/s | 2U | liquid-cooled | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN6600 | Spectrum-6 SN6000 | 800GbE | 64x OSFP 2x800G | 128x800G / 256x400G / 512x200G / 514x100G | 102.4 Tb/s | 3U | air-cooled | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN6200-LD | Spectrum-6 SN6000 | 200GbE | 32x OSFP 2x800G front; 256x200G backplane | 256x200G / 258x100G | 102.4 Tb/s | 1U | liquid-cooled | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN5610 | Spectrum-4 SN5000 | 800GbE | 64x OSFP 800G + 2x SFP28 25G | 64x800G / 128x400G / 256x200G / 256x100G / 258x25G | 51.2 Tb/s; 33.3 Bpps | 2U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN5600 | Spectrum-4 SN5000 | 800GbE | 64x OSFP 800G + 1x SFP28 25G | 64x800G / 128x400G / 256x200G / 256x100G / 257x25G | 51.2 Tb/s; 33.3 Bpps | 2U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN5600D | Spectrum-4 SN5000 | 800GbE | 64x OSFP 800G + 1x SFP28 25G | 64x800G / 128x400G / 256x200G / 256x100G / 257x25G | 51.2 Tb/s; 33.3 Bpps | 2U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN5400 | Spectrum-4 SN5000 | 400GbE | 64x QSFP-DD 400G + 2x SFP28 25G | 64x400G / 128x200G / 256x100G / 258x25G | 25.6 Tb/s; 33.3 Bpps | 2U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN4600C | Spectrum-3 SN4000 | 100GbE | 64x QSFP28 100G | 64x100G / 128x50G / 64x40G / 128x25G | 6.4 Tb/s; 8.4 Bpps | 2U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN4700 | Spectrum-3 SN4000 | 400GbE | 32x QSFP-DD 400G | 32x400G / 64x200G / 128x100G / 128x50G / 64x40G / 128x25G | 12.8 Tb/s; 8.4 Bpps | 1U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN3420 | Spectrum-2 SN3000 | 100GbE | 12x QSFP28 100G + 48x SFP28 25G | 12x100G / 12x40G / 96x25G | 2.4 Tb/s; 3.57 Bpps | 1U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |
| SN2201 | Spectrum SN2000 | 100GbE | 48x RJ45 + 4x QSFP28 100G | 4x100G / 8x50G / 4x40G / 16x25G / 48x1G | 448 Gb/s; 667 Mpps | 1U | — | Cumulus Linux, Pure SONiC, NetQ, DSX Air | [NVIDIA](https://www.nvidia.com/en-us/networking/ethernet-switching/) |

## Silicon Photonics

| Product | Family | Speed / Architecture | Throughput | Availability / Cooling | Compatibility | Source |
|---|---|---|---|---|---|---|
| Q3450-LD | Quantum-X InfiniBand Photonics | 800Gb/s XDR; CPO | 115.2 Tb/s | liquid-cooled | Quantum-X800, 200G SerDes, co-packaged optics | [NVIDIA](https://www.nvidia.com/en-us/networking/products/silicon-photonics/) |
| Spectrum-X Ethernet Photonics Switches | Spectrum-X Ethernet Photonics | 200G SerDes CPO architecture | up to 409.6 Tb/s | 2H 2026 | Spectrum-X Ethernet, co-packaged optics | [NVIDIA](https://www.nvidia.com/en-us/networking/products/silicon-photonics/) |

## InfiniBand switches and appliances

| Model | Family | Speed | Connectors / Ports | Throughput / Reach | Height | Cooling | Software / Compatibility | Source |
|---|---|---|---|---|---|---|---|---|
| Q3200-RA | Quantum-X800 | 800Gb/s XDR | 2x18 OSFP; 72x800G | 2x28.8 Tb/s | 2U | air-cooled | NVOS, UFM; Quantum-X800, Quantum-2 storage migration | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| Q3400-RA | Quantum-X800 | 800Gb/s XDR | 72x OSFP; 144x800G | 115.2 Tb/s | 4U | air-cooled | NVOS, UFM | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| Q3401-RD | Quantum-X800 | 800Gb/s XDR | 72x OSFP; 144x800G | 115.2 Tb/s | 4U | air-cooled; DC power | NVOS, UFM | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| Q3450-LD | Quantum-X Photonics / Quantum-X800 | 800Gb/s XDR | 144x MPO12 CPO; 144x800G | 115.2 Tb/s | 4U | 85% liquid / 15% air | NVOS, UFM | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| QM9700 family | Quantum-2 | 400Gb/s NDR | OSFP | 51.2 Tb/s class | — | — | NVOS, UFM; ConnectX-7, NDR/HDR | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| Skyway | InfiniBand-to-Ethernet Gateway | 100/200 Gb/s per port | 8 ports per InfiniBand and Ethernet side | 1.6 Tb/s | — | — | Gateway software; InfiniBand, Ethernet | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |
| MetroX-3 XC | Long-haul InfiniBand | InfiniBand extension | — | up to 40 km reach | — | — | NVOS/UFM ecosystem; DWDM, encrypted long-haul | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband-switching/) |

## DPU

| Product | Family | Speed | Software | Primary capabilities | Source |
|---|---|---|---|---|---|
| BlueField-4 DPU | BlueField | 800Gb/s | NVIDIA DOCA | networking, storage, cybersecurity, secure multi-tenancy | [NVIDIA](https://www.nvidia.com/en-us/networking/products/data-processing-unit/) |
| BlueField-4 STX Storage Processor | BlueField STX | 800Gb/s network-enforcement class | NVIDIA DOCA | AI-native storage, Vera CPU, in-silicon security | [NVIDIA](https://www.nvidia.com/en-us/networking/products/data-processing-unit/) |
| BlueField-3 DPU | BlueField | 400Gb/s | NVIDIA DOCA | SDN, storage, cybersecurity, HPC, 5G | [NVIDIA](https://www.nvidia.com/en-us/networking/products/data-processing-unit/) |

## Ethernet SuperNICs

| Product | Family | Speed | Compatibility / Use | Source |
|---|---|---|---|---|
| ConnectX-9 SuperNIC | ConnectX SuperNIC | up to 1.6 Tb/s per GPU | Spectrum-X Ethernet, AI fabrics | [NVIDIA](https://www.nvidia.com/en-us/networking/products/ethernet/supernic/) |
| ConnectX-8 SuperNIC | ConnectX SuperNIC | up to 800 Gb/s total network bandwidth | PCIe Gen6, Spectrum-X Ethernet, AI compute fabrics | [NVIDIA](https://www.nvidia.com/en-us/networking/products/ethernet/supernic/) |
| BlueField-3 SuperNIC | BlueField SuperNIC | up to 400 Gb/s | Spectrum-X Ethernet, secure cloud multi-tenancy, deterministic isolated performance | [NVIDIA](https://www.nvidia.com/en-us/networking/products/ethernet/supernic/) |

## UFM management products

| Product | Family | Deployment | Compatibility | Source |
|---|---|---|---|---|
| UFM Telemetry | UFM platform | container / dedicated appliance | switch/adapters/cables telemetry; on-prem/cloud database | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband/ufm/) |
| UFM Enterprise | UFM platform | container / dedicated appliance | Slurm, IBM Spectrum LSF, REST API | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband/ufm/) |
| UFM Cyber-AI | UFM platform | dedicated on-prem appliance | UFM Telemetry, UFM Enterprise | [NVIDIA](https://www.nvidia.com/en-us/networking/infiniband/ufm/) |

## Active NVIDIA LinkX transceivers — normalized detail

> Fields below are rendered directly from the normalized transceiver records in `data/nvidia-interconnects.json`: protocol, medium, reach, cable-side interface count/speed/type, wavelength, part numbers and source notes.

| Model | Speed | Protocol | Medium | Reach | Cable interfaces | λ | Part number(s) | Notes | Source |
|---|---|---|---|---|---|---|---|---|---|
| MMS4B10-XM | 1600G | Ethernet | SMF | 500 m | 2 × 800Gb/s — MPO-12/APC | 1310 nm | — | Ordering-information URL was not retrievable during this refresh. | [Docs](https://networking-docs.nvidia.com/mms4b10xmtro1600g) |
| MMS4C1X | 1600G | InfiniBand XDR (MMS4C10); Ethernet (MMS4C11) | SMF | 500 m | 2 × 800Gb/s — MPO-12/APC | 1310 nm | 980-9IAU0-00XM00; 980-9IAU0-00XM01; 980-9IAU1-00XM00; 980-9IAU1-00XM01 | — | [Docs](https://networking-docs.nvidia.com/9iau000xmosfptcvr1600) |
| MMS4A00 | 1600G | InfiniBand XDR | SMF | 500 m | 2 × 800Gb/s — MPO-12/APC | 1310 nm | 980-9IAH1-00XM00; 980-9IAH0-00XM00 | — | [Docs](https://networking-docs.nvidia.com/9iahx00xmosfptcvr1600) |
| MMA1Z00-NS400 | 400G | InfiniBand; Ethernet | MMF | OM3 30 m; OM4 50 m | 1 × 400Gb/s — MPO-12/APC | 850 nm | 980-9I693-00NS00 | — | [Docs](https://networking-docs.nvidia.com/mms1z00ns400sr4) |
| MMS1V00-WM | 400G | Ethernet | SMF | 500 m | 1 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I16Y-00W000 | — | [Docs](https://networking-docs.nvidia.com/mms1v00wm10) |
| MMS4X00-NS400 | 400G | InfiniBand; Ethernet | SMF | 100 m | 1 × 400Gb/s — MPO-12/APC | — | 980-9I31N-00NM00 | — | [Docs](https://networking-docs.nvidia.com/mms4x00ns400) |
| MMA4Z00-NS400 | 400G | InfiniBand; Ethernet | MMF | OM3 30 m; OM4 50 m | 1 × 400Gb/s — MPO-12/APC | 850 nm | 980-9I51S-00NS00 | — | [Docs](https://networking-docs.nvidia.com/mma4z00ns400) |
| MMA4Z00-NS400-T | 400G | Ethernet | MMF | 50 m | 1 × 400Gb/s — MPO-12/APC | 850 nm | 980-9I51S-F4NS00 | — | [Docs](https://networking-docs.nvidia.com/mma4z00ns400t) |
| MMA1Z00-NS400-T | 400G | Ethernet | MMF | 50 m | 1 × 400Gb/s — MPO-12/APC | 850 nm | 980-9I693-F4NS00 | — | [Docs](https://networking-docs.nvidia.com/mma1z00ns400t) |
| MMS1X00-NS400 | 400G | InfiniBand; Ethernet | SMF | 500 m normalized; intro: 500 m; ordering description: 100 m | 1 × 400Gb/s — MPO-12/APC | — | 980-9I068-00NM00 | NVIDIA introduction states 500 m while ordering description states up to 100 m; both values retained in JSON. | [Docs](https://networking-docs.nvidia.com/mms1x00ns400) |
| MMS1V70-CM | 100G | Ethernet | SMF | 500 m | 1 × 100Gb/s — LC/LC duplex-compatible DR1 path | 1310 nm | 980-9I042-00C000 | — | [Docs](https://networking-docs.nvidia.com/mms1v70cm10) |
| MMA1B00-C100D | 100G | Ethernet | MMF | OM3 70 m; OM4 100 m | 1 × 100Gb/s — MPO-12/UPC | 850 nm | 980-9I149-00CS00 | — | [Docs](https://networking-docs.nvidia.com/mma1b00c100dspec) |
| MMS4C10-XM800 | 800G | InfiniBand XDR; Ethernet 800GbE | SMF | 500 m | 1 × 800Gb/s — MPO-12/APC | 1310 nm | 980-9IAY0-00XM00 | Product-level docs identify single-port DR4/1×MPO; catalog title is inconsistent and says twin-port 2×DR4/2×MPO. | [Docs](https://networking-docs.nvidia.com/mms4c1x800) |
| MMS4X00-NM-T | 800G | Ethernet | SMF | 500 m | 2 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I30G-F4NM00 | — | [Docs](https://networking-docs.nvidia.com/mms4x00nmt800g) |
| MMS4X00-NM | 800G | InfiniBand; Ethernet | SMF | 500 m | 2 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I30G-00NM00; 980-9I301-00NM00 | — | [Docs](https://networking-docs.nvidia.com/mms4x00nm800g500m) |
| MMS4X50-NM | 800G | InfiniBand; Ethernet | SMF | 2 km | 2 × 400Gb/s — Duplex LC | 1310 nm | 980-9I30L-00N000 | — | [Docs](https://networking-docs.nvidia.com/mms4x50nm800g2kmpub) |
| MMS4X00-NS | 800G | InfiniBand; Ethernet | SMF | 100 m | 2 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I30H-00NM00; 980-9I30I-00NM00 | — | [Docs](https://networking-docs.nvidia.com/800gmms4x00ns) |
| MMA4Z00-NS-T | 800G | Ethernet | MMF | OM3 30 m; OM4 50 m | 2 × 400Gb/s — MPO-12/APC | 850 nm | 980-9I510-F4NS00 | — | [Docs](https://networking-docs.nvidia.com/800gmma4z00nst) |
| MMS4X00-NS-T | 800G | Ethernet | SMF | 100 m | 2 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I30H-F4NM00 | — | [Docs](https://networking-docs.nvidia.com/800gmms4x00nst) |
| MMS4A20 | 800G | InfiniBand XDR | SMF | 500 m | 1 × 800Gb/s — MPO-12/APC | 1310 nm | 980-9IAT0-00XM00 | — | [Docs](https://networking-docs.nvidia.com/9iat0mosfp800sprhs) |
| MMS4X00-NM16 | 800G | Ethernet | SMF | 500 m | 1 × 800Gb/s — MPO-16/APC | 1310 nm | 980-9I30J-F4NM00 | — | [Docs](https://networking-docs.nvidia.com/mms4x00nm16) |
| MMS4X00-NM-HGX | 800G | InfiniBand; Ethernet | SMF | 500 m | 2 × 400Gb/s — MPO-12/APC | 1310 nm | 980-9I302-00NM00 | — | [Docs](https://networking-docs.nvidia.com/mms4x00nmhgx500) |
| MMS4X90-NR | 800G | InfiniBand; Ethernet | SMF | 10 km | 2 × 400Gb/s — Duplex LC | 1310 nm | — | Current NVIDIA ordering page displays MMS4X50-NM / 980-9I30L-00N000 (2 km FR4), inconsistent with the 10 km LR4 product page; unverified OPN omitted. | [Docs](https://networking-docs.nvidia.com/mms4x90nr800g) |
| MMS1W50-HM | 200G | InfiniBand HDR | SMF | 2 km | 1 × 200Gb/s — Duplex LC/UPC | 1310 nm | MMS1W50-HM | — | [Docs](https://networking-docs.nvidia.com/mms1w50hmspec) |
| MMA2P00-AS | 25G | Ethernet | MMF | OM3 70 m; OM4 100 m | 1 × 25Gb/s — Duplex LC/UPC | 850 nm | MMA2P00-AS | — | [Docs](https://networking-docs.nvidia.com/mma2p00asspec) |

## Active NVIDIA LinkX interconnects

| Model | Bandwidth | Category | Description | Source |
|---|---|---|---|---|
| MCA4K00 | 1600G | Copper | 1600Gbps to 1600Gbps OSFP Active Copper Cable | [Docs](https://networking-docs.nvidia.com/mca4k00hw) |
| MCA4K50 | 1600G | Copper | 1600Gbps to 1600Gbps OSFP Active Copper Cable | [Docs](https://networking-docs.nvidia.com/mca4k50osfp1600) |
| MCA7K10 | 1600G | Copper | 1600Gb/s to 2x800Gb/s OSFP to 2xOSFP Active Copper Splitter Cable | [Docs](https://networking-docs.nvidia.com/9809iao500xxxxrhs2x800) |
| MMS4B10-XM | 1600G | Transceiver | TRO 1600Gbps OSFP Twin-Port 2xDR4 2xMPO 1310nm SMF up to 500m | [Docs](https://networking-docs.nvidia.com/mms4b10xmtro1600g) |
| MMS4C1X | 1600G | Transceiver | FRO 1600Gbps 2xDR4 Twin-port OSFP 1310nm SMF up to 500m | [Docs](https://networking-docs.nvidia.com/9iau000xmosfptcvr1600) |
| MMS4A00 | 1600G | Transceiver | 1600Gbps 2xDR4 Twin-port OSFP 1310nm SMF up to 500m | [Docs](https://networking-docs.nvidia.com/9iahx00xmosfptcvr1600) |
| MCA4J80-Nxxx | 800G | Copper | 800Gb/s Twin-port OSFP to 2x400Gb/s OSFP InfiniBand ACC | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MCP4Y10-Nxxx | 800G | Copper | Twin-port 2x400Gb/s OSFP to 2x400Gb/s OSFP Passive DAC | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MCP7Y00-Nxxx | 800G | Copper | 800Gb/s Twin-port 2x400G OSFP to 2x400G OSFP Passive DAC Splitter | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4C10-XM800 | 800G | Transceiver | Gen2 800Gb/s DR4 single-port OSFP, 1xMPO-12/APC, 1310nm SMF up to 500m; XDR + Ethernet | [Docs](https://networking-docs.nvidia.com/mms4c1x800) |
| MMS4X00-NM-T | 800G | Transceiver | 800Gbps Twin-port OSFP 2x400Gb/s Single Mode 500m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X00-NM | 800G | Transceiver | 800Gbps Twin-port OSFP 2x400Gb/s InfiniBand and Ethernet 2xDR4 500m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X50-NM | 800G | Transceiver | 800Gbps Twin-port OSFP 2xFR4 Single Mode 2km | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X00-NS | 800G | Transceiver | 800Gbps Twin-port OSFP 2x400Gb/s Single Mode 2xDR4 100m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMA4Z00-NS-T | 800G | Transceiver | 800Gb/s Twin-port OSFP 2x400Gb/s Multimode 50m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X00-NS-T | 800G | Transceiver | 800Gbps Twin-port OSFP 2x400Gb/s Single Mode 100m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4A20 | 800G | Transceiver | 800Gbps DR4 Single-port OSFP MPO 1310nm SMF RHS 500m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X00-NM16 | 800G | Transceiver | Ethernet 800Gbps OSFP Twin-port Finned 1xMPO16 SMF up to 500m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X00-NM-HGX | 800G | Transceiver | 800Gbps Twin-port OSFP InfiniBand and Ethernet 2xDR4 500m | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS4X90-NR | 800G | Transceiver | 800Gbps 2xLR4 OSFP 1310nm SMF up to 10km | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MFA7U10-H00x | 400G | AOC | 400Gb/s OSFP to 2x200Gb/s QSFP56 HDR Active Optical Splitter Cable | [Docs](https://networking-docs.nvidia.com/mfa7u10h00x10) |
| MMA1Z00-NS400 | 400G | Transceiver | 400Gb/s QSFP112 Multimode SR4 | [Docs](https://networking-docs.nvidia.com/mms1z00ns400sr4) |
| MMS1V00-WM | 400G | Transceiver | 400GbE QSFP-DD DR4 | [Docs](https://networking-docs.nvidia.com/mms1v00wm10) |
| MMS4X00-NS400 | 400G | Transceiver | 400Gb/s OSFP Single Mode DR4 | [Docs](https://networking-docs.nvidia.com/mms4x00ns400) |
| MMA4Z00-NS400 | 400G | Transceiver | 400Gb/s OSFP Multimode SR4 50m | [Docs](https://networking-docs.nvidia.com/mma4z00ns400) |
| MMA4Z00-NS400-T | 400G | Transceiver | 400Gb/s OSFP Multimode 50m | [Docs](https://networking-docs.nvidia.com/mma4z00ns400t) |
| MMA1Z00-NS400-T | 400G | Transceiver | 400Gb/s QSFP112 Multimode | [Docs](https://networking-docs.nvidia.com/mma1z00ns400t) |
| MMS1X00-NS400 | 400G | Transceiver | 400Gb/s QSFP112 Single Mode | [Docs](https://networking-docs.nvidia.com/mms1x00ns400) |
| MFS1S00-HxxxV | 200G | AOC | 200Gb/s QSFP56 MMF AOC | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MMS1W50-HM | 200G | Transceiver | 200Gb/s QSFP56 FR4 | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MCP1600-E0xxEyy | 100G | Copper | 100Gb/s QSFP28 DAC Cable | [Docs](https://networking-docs.nvidia.com/mcp1600e0xxxeyyspec) |
| MMS1V70-CM | 100G | Transceiver | 100GbE QSFP28 DR1 | [Docs](https://networking-docs.nvidia.com/mms1v70cm10) |
| MMA1B00-C100D | 100G | Transceiver | 100GbE QSFP28 MMF SR4 | [Docs](https://networking-docs.nvidia.com/mma1b00c100dspec) |
| MMA2P00-AS | 25G | Transceiver | 25GbE SFP28 MMF Transceiver | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MAM1Q00A-QSA | Accessories | Adapter | DynamiX QSA QSFP+ to SFP+ Adapter | [Docs](https://networking-docs.nvidia.com/mam1q00aqsadnmx) |
| MAM1Q00A-QSA28 | Accessories | Adapter | DynamiX QSA28 QSFP28 to SFP28 Adapter | [Docs](https://networking-docs.nvidia.com/mam1q00aqsa28dnmx) |
| MFP7E10-Nxxx | Accessories | Fiber | Optical Multimode Fiber Cable | [Docs](https://networking-docs.nvidia.com/mfp7e10nxxx) |
| MFP7E20-Nxxx | Accessories | Fiber | Optical Multimode Splitter Fiber Cable | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MFP7E30-Nxxx | Accessories | Fiber | MPO-to-MPO Single-mode Fiber Cable | [Docs](https://networking-docs.nvidia.com/interconnect) |
| MFP7E40-Nxxx | Accessories | Fiber | Single mode 1:2 Fiber Splitter Cable | [Docs](https://networking-docs.nvidia.com/interconnect) |

---

Source of truth: [`data/nvidia-interconnects.json`](data/nvidia-interconnects.json). The catalog intentionally excludes records marked `no_longer_for_sale`.