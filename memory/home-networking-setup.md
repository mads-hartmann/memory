# Home Network Setup

This document explains different options for setting up my home network.

## Overview

- Provider: Fibernet with cables owned by TDC Net
- Access type: Fiber
- Internet plan: 1000 Mbps

## Unknowns

- **ONT Ethernet port speed (Options A & B):** If the ONT only has a 1 GbE Ethernet port, it becomes the bottleneck regardless of router or AP. Check the model number on the ONT label and look up its spec sheet. Not relevant for Option C since it bypasses the ONT.
- **In-wall Ethernet cable category:** The cable category in the wall sockets (Cat5e, Cat6, etc.) determines the max speed of the runs to the basement. Cat5e supports 1 GbE, Cat6 supports up to 10 GbE (short runs) or reliable 2.5 GbE. Check the cable jacket print or the wall socket markings.
- **FTP fiber connector type (Option C):** The FTP likely uses an SC/APC connector (green, angled), but this needs to be verified. The GPON SFP module will have an LC/UPC connector, so you'll need a patch cable with the right connector on each end (e.g. SC/APC to LC/UPC).
- **TDC Net GPON credentials (Option C):** To replace the ONT with a GPON SFP module, you need the GPON serial number and possibly a PLOAM password or OMCI configuration that TDC Net expects. Check Danish forums (e.g. Pair.dk) for confirmed working setups, or contact TDC Net / Fibernet directly.
- **TDC Net VLAN configuration (Option C):** TDC Net likely uses VLAN tagging for internet traffic (commonly VLAN 101 on their network). The UCG Fiber's WAN interface would need to be configured with the correct VLAN ID.

## Terminology

- Router — a device that forwards traffic between networks (e.g. your LAN and the internet). It assigns local IP addresses (DHCP), performs NAT, and typically includes a firewall.
- Switch — a device that connects multiple Ethernet devices on the same local network. Unlike a router, it doesn't route between networks — it just forwards frames between its ports.
- UniFi Gateway — a router/firewall appliance from Ubiquiti's UniFi line (e.g. Dream Machine, UCG Fiber). It handles routing, NAT, DHCP, and acts as the central controller for other UniFi devices.
- Access point (AP) — a device that creates a Wi-Fi network and connects wireless clients to the wired LAN. Unlike a router, it doesn't do routing or NAT—it just bridges wireless traffic onto the Ethernet network.
- WAN (Wide Area Network) — the "internet side" of your router. The WAN port connects to your ISP's network (via the ONT), as opposed to LAN ports which connect to your local devices.
- PoE (Power over Ethernet) — delivers electrical power alongside data over an Ethernet cable, eliminating the need for a separate power adapter. Common standards: PoE (802.3af, 15W), PoE+ (802.3at, 30W), PoE++ (802.3bt, up to 60–100W).
- FTP (Fiber termination point) - A passive box where the street fiber terminates inside the house.
- ONT (Optical Network Terminal) - the active box that converts the optical signal to Ethernet.
- SFP / SFP+ (Small Form-factor Pluggable) — a hot-swappable transceiver module that plugs into a network device. SFP supports up to 1 Gbps, SFP+ supports up to 10 Gbps. Modules come in different media types: fiber optic (with LC connectors) or copper RJ45.
- GPON (Gigabit Passive Optical Network) — the fiber-to-the-home technology used by TDC Net. A GPON SFP module lets a router authenticate directly on the ISP's optical network, replacing a standalone ONT.
- VLAN (Virtual LAN) — a logical network segment within a physical network. ISPs often use VLAN tagging to separate services (internet, TV, VoIP) on the same fiber connection.

## Fiber & Ethernet sockets

1. Fiber enters through the wall into a TDC Net FTP
2. Short fiber patch cable from the FTP to a separate TDC Net ONT
3. Next to the TDC boxes are two wall ethernet sockets that run to the basement.

## Requirements

1. I want to wifi access points in both the 1st floor and basement.
2. In the basement I have a Macbook Pro and a Linux server that both need a wired ethernet connection.

## Relevant equipment

- Own today:
    - [Dream Machine EU Version from 2022 SKU UDM-EU](https://eu.store.ui.com/eu/en/products/udm)
    - [U7 Pro XG](https://eu.store.ui.com/eu/en/category/wifi-flagship/products/u7-pro-xg?variant=u7-pro-xg)
- Would have to buy, depending on the setup I choose
    - [Cloud Gateway Fiber](https://eu.store.ui.com/eu/en/category/cloud-gateways-compact/collections/cloud-gateway-fiber/products/ucg-fiber)
    - [10G PoE++ Adapter (60W)](https://eu.store.ui.com/eu/en/category/accessories-poe-power/collections/pro-store-poe-and-power-adapters/products/uacc-poe-plus-plus-10g)
    - [USW Pro XG 8 PoE](https://eu.store.ui.com/eu/en/category/switching-utility/products/usw-pro-xg-8-poe) — 8× 10 GbE RJ45 (all PoE++, up to 60W/port, 155W total), 2× 10G SFP+, Layer 3

## Option A

Dream Machine in the basement, U7 Pro XG on the 1st floor. Uses both in-wall Ethernet runs: one carries WAN down to the UDM, the other carries LAN back up to the AP.

### Topology

```mermaid
graph TD
  subgraph floor1 ["1st Floor"]
    ONT["TDC Net box / ONT<br>Fiber in, Ethernet out"]
    J1_UP["Wall jack #1"]
    J2_UP["Wall jack #2"]
    POE["UniFi 10G PoE++ Adapter (60W)<br>UACC-PoE++-10G"]
    AP["U7 Pro XG<br>Wi-Fi 7 AP"]
  end

  subgraph basement ["Basement"]
    J1_BASE["Wall jack #1"]
    J2_BASE["Wall jack #2"]
    UDM["Dream Machine (UDM-EU)<br>Router + Wi-Fi 5 AP"]
    MAC["MacBook Pro"]
    SRV["Linux server"]
  end

  ONT -->|"Ethernet (WAN)"| J1_UP -->|"In-wall run #1"| J1_BASE -->|"Ethernet (WAN)"| UDM
  UDM -->|"Ethernet (LAN)"| MAC
  UDM -->|"Ethernet (LAN)"| SRV
  UDM -->|"Ethernet (LAN)"| J2_BASE -->|"In-wall run #2"| J2_UP -->|"Ethernet"| POE -->|"PoE++"| AP

  style floor1 stroke-dasharray: 5 5
  style basement stroke-dasharray: 5 5
```

### How it works

- **In-wall run #1 (WAN):** ONT Ethernet out → 1st floor jack → basement jack → UDM WAN port.
- **In-wall run #2 (LAN):** UDM LAN port → basement jack → 1st floor jack → PoE++ adapter → U7 Pro XG.
- **Wired devices:** MacBook Pro and Linux server plug directly into the UDM's remaining LAN ports (it has 4× GbE LAN).
- **Wi-Fi:** UDM provides Wi-Fi 5 coverage in the basement. U7 Pro XG provides Wi-Fi 7 on the 1st floor.

### Limitations

- All UDM LAN ports are 1 GbE, so the U7 Pro XG's 10 GbE uplink is underutilized (capped at ~940 Mbps). The 1000 Mbps internet plan still fits within this ceiling, so WAN throughput is unaffected.
- The UDM's built-in AP is Wi-Fi 5 — adequate for the basement but noticeably slower than the Wi-Fi 7 AP upstairs.
- The PoE++ adapter's data input also appears to be 1 GbE based on its spec sheet, reinforcing the 1G ceiling.
- The in-wall Ethernet runs must be Cat5e or better for reliable Gigabit over the full distance. Cat6 is ideal.

### Equipment to buy

- UniFi 10G PoE++ Adapter (60W) (UACC-PoE++-10G-EU)
    - Sits on the 1st floor near wall jack #2
    - Injects PoE++ power to the U7 Pro XG over a short patch cable
- Patch cables
    - 1st floor: wall jack #2 → PoE++ adapter, PoE++ adapter → U7 Pro XG
    - Basement: wall jack #1 → UDM (WAN), UDM (LAN) → wall jack #2, UDM (LAN) → MacBook, UDM (LAN) → Linux server

## Option B

UCG Fiber in the basement (replaces the Dream Machine entirely), with two U7 Pro XG access points — one per floor. Uses both in-wall Ethernet runs: one carries WAN down to the UCG Fiber, the other carries LAN back up to the 1st floor AP.

### Topology

```mermaid
graph TD
  subgraph floor1 ["1st Floor"]
    ONT["TDC Net box / ONT<br>Fiber in, Ethernet out"]
    J1_UP["Wall jack #1"]
    J2_UP["Wall jack #2"]
    POE_UP["UniFi 10G PoE++ Adapter (60W)<br>UACC-PoE++-10G"]
    AP_UP["U7 Pro XG<br>Wi-Fi 7 AP"]
  end

  subgraph basement ["Basement"]
    J1_BASE["Wall jack #1"]
    J2_BASE["Wall jack #2"]
    UCG["Cloud Gateway Fiber (UCG-Fiber)<br>Router, 4x 2.5 GbE LAN, 30W PoE budget"]
    AP_BASE["U7 Pro XG<br>Wi-Fi 7 AP"]
    MAC["MacBook Pro"]
    SRV["Linux server"]
  end

  ONT -->|"Ethernet (WAN)"| J1_UP -->|"In-wall run #1"| J1_BASE -->|"Ethernet (WAN)"| UCG
  UCG -->|"2.5 GbE + PoE (LAN)"| AP_BASE
  UCG -->|"2.5 GbE (LAN)"| MAC
  UCG -->|"2.5 GbE (LAN)"| SRV
  UCG -->|"2.5 GbE (LAN)"| J2_BASE -->|"In-wall run #2"| J2_UP -->|"Ethernet"| POE_UP -->|"PoE++"| AP_UP

  style floor1 stroke-dasharray: 5 5
  style basement stroke-dasharray: 5 5
```

### How it works

- **In-wall run #1 (WAN):** ONT Ethernet out → 1st floor jack → basement jack → UCG Fiber WAN port (1 GbE RJ45 or via SFP+ with an RJ45 transceiver module).
- **In-wall run #2 (LAN):** UCG Fiber LAN port → basement jack → 1st floor jack → PoE++ adapter → U7 Pro XG upstairs.
- **Basement AP power:** The UCG Fiber has a 30W PoE budget across its LAN ports. The U7 Pro XG draws up to 22W and accepts PoE+ (802.3at), so the UCG Fiber can power the basement AP directly — no separate injector needed.
- **1st floor AP power:** The PoE budget is consumed by the basement AP (~22W of 30W), so the 1st floor AP needs an external PoE++ adapter.
- **Wired devices:** MacBook Pro and Linux server plug into the UCG Fiber's remaining 2.5 GbE LAN ports.
- **Wi-Fi:** Both floors get Wi-Fi 7 via dedicated U7 Pro XG access points. No built-in AP on the UCG Fiber — it's a gateway only.

### Advantages over Option A

- Wi-Fi 7 on both floors (Option A has Wi-Fi 5 in the basement via the UDM).
- 2.5 GbE LAN ports on the UCG Fiber vs 1 GbE on the UDM — faster local wired and AP uplink speeds.
- Cleaner separation of concerns: dedicated gateway + dedicated APs.
- UCG Fiber powers the basement AP directly via built-in PoE — one fewer device in the basement.

### Limitations

- The 30W PoE budget only covers one U7 Pro XG (~22W). The 1st floor AP still needs an external PoE++ adapter.
- The UCG Fiber's RJ45 WAN port is 1 GbE. With a 1000 Mbps internet plan this is fine, but it doesn't leave headroom. The SFP+ port supports 10G but would need an RJ45 transceiver if the in-wall run is Ethernet.
- Uses all 4 of the UCG Fiber's LAN ports (basement AP, MacBook, Linux server, upstairs run). No spare ports without adding a switch.
- The in-wall Ethernet runs must be Cat5e or better for reliable Gigabit. Cat6 for 2.5 GbE.

### Equipment to buy

- Cloud Gateway Fiber (UCG-Fiber) — replaces the Dream Machine
- Second U7 Pro XG — for the basement (you already own one for the 1st floor)
- 1× UniFi 10G PoE++ Adapter (60W) (UACC-PoE++-10G-EU) — for the 1st floor AP only (basement AP is powered by the UCG Fiber's built-in PoE)
    - 1st floor: between wall jack #2 and 1st floor U7 Pro XG
- Patch cables
    - Basement: wall jack #1 → UCG Fiber (WAN), UCG Fiber (LAN) → basement AP, UCG Fiber (LAN) → MacBook, UCG Fiber (LAN) → Linux server, UCG Fiber (LAN) → wall jack #2
    - 1st floor: wall jack #2 → PoE++ adapter, PoE++ adapter → U7 Pro XG



## Option C

UCG Fiber on the 1st floor with a GPON SFP module plugged directly into the FTP — no ONT needed. A USW Pro XG 8 PoE switch in the basement provides 10 GbE connectivity and PoE++ power to the basement AP. Uses one in-wall Ethernet run for LAN; the second run is free.

### Topology

```mermaid
graph TD
  subgraph floor1 ["1st Floor"]
    FTP["TDC Net FTP<br>Fiber termination point"]
    UCG["Cloud Gateway Fiber (UCG-Fiber)<br>Router, SFP+ WAN, 4x 2.5 GbE LAN, 30W PoE"]
    AP_UP["U7 Pro XG<br>Wi-Fi 7 AP"]
    J1_UP["Wall jack #1"]
    J2_UP["Wall jack #2 (spare)"]
  end

  subgraph basement ["Basement"]
    J1_BASE["Wall jack #1"]
    J2_BASE["Wall jack #2 (spare)"]
    SW["USW Pro XG 8 PoE<br>8x 10 GbE PoE++, 2x 10G SFP+"]
    AP_BASE["U7 Pro XG<br>Wi-Fi 7 AP"]
    MAC["MacBook Pro"]
    SRV["Linux server"]
  end

  FTP -->|"Fiber patch cable<br>(SC/APC to LC/UPC)"| UCG
  UCG -->|"2.5 GbE + PoE (LAN)"| AP_UP
  UCG -->|"2.5 GbE (LAN)"| J1_UP -->|"In-wall run #1"| J1_BASE -->|"Ethernet"| SW
  SW -->|"10 GbE + PoE++"| AP_BASE
  SW -->|"10 GbE"| MAC
  SW -->|"10 GbE"| SRV

  style floor1 stroke-dasharray: 5 5
  style basement stroke-dasharray: 5 5
```

### How it works

- **Fiber WAN:** A short fiber patch cable goes directly from the FTP to the UCG Fiber's SFP+ port (with a GPON SFP module). The UCG Fiber authenticates on TDC Net's GPON network — no ONT needed.
- **1st floor AP:** The UCG Fiber powers the 1st floor U7 Pro XG via its built-in PoE (~22W of 30W budget). Short Ethernet run, no separate PoE adapter needed.
- **In-wall run #1 (LAN):** UCG Fiber LAN port → 1st floor wall jack → basement wall jack → USW Pro XG 8 PoE.
- **Basement switch:** The USW Pro XG 8 PoE fans out to all basement devices over 10 GbE. It powers the basement AP via PoE++ (155W total budget, U7 Pro XG draws ~22W).
- **In-wall run #2:** Free — available as a spare or for a future device.
- **Wired devices:** MacBook Pro and Linux server plug into the switch's 10 GbE ports.
- **Wi-Fi:** Both floors get Wi-Fi 7 via dedicated U7 Pro XG access points.

### Advantages over Options A and B

- **Eliminates the ONT** — one fewer device, one fewer power adapter, one fewer point of failure.
- **10G WAN capability** — the SFP+ port supports up to 10 Gbps. If TDC Net upgrades your plan beyond 1 Gbps, you're ready without hardware changes. Options A and B are capped at 1 GbE on the WAN side.
- **Frees up an in-wall Ethernet run** — Options A and B consume both runs (one WAN, one LAN). Option C only needs one (LAN to basement), leaving the second for future use.
- **Shorter, simpler WAN path** — fiber goes directly from FTP to UCG Fiber with a single patch cable. No Ethernet conversion, no in-wall run for WAN.
- **10 GbE in the basement** — the USW Pro XG 8 PoE provides 10 GbE to all basement devices. Options A and B are limited to 1 GbE (A) or 2.5 GbE (B) from the gateway's LAN ports.
- **Plenty of spare ports** — the switch has 8x 10 GbE + 2x SFP+ ports. Only 3 are used (AP, MacBook, server), leaving 7 for future devices.
- **Wi-Fi 7 on both floors** — same as Option B.
- **No PoE++ adapter needed** — the UCG Fiber powers the 1st floor AP, the switch powers the basement AP. No standalone PoE injectors anywhere.

### Limitations

- **GPON SFP compatibility** — you need an SFP module that works with both TDC Net's GPON network and the UCG Fiber. Common options are the Ubiquiti UF-GP-C+ or third-party modules like the ODI DFP-34X-2C2. This requires research and possibly trial-and-error.
- **ISP credentials** — the ONT normally handles GPON authentication transparently. With your own SFP module, you need to configure the GPON serial number, PLOAM password, or OMCI settings. TDC Net may or may not be cooperative.
- **VLAN configuration** — TDC Net likely uses VLAN tagging (commonly VLAN 101). The UCG Fiber's WAN interface needs the correct VLAN ID.
- **Higher cost** — the USW Pro XG 8 PoE is a significant addition compared to Options A and B. However, it future-proofs the basement with 10 GbE and ample PoE.
- **UCG Fiber LAN ports are 2.5 GbE** — the uplink from UCG Fiber to the basement switch is capped at 2.5 GbE over the in-wall Ethernet run (assuming Cat6). This is the ceiling for aggregate basement traffic to/from the internet, though local traffic between switch ports is 10 GbE.

### Equipment to buy

- **Cloud Gateway Fiber (UCG-Fiber)** — replaces the Dream Machine, sits on the 1st floor next to the FTP
- **GPON SFP module** — e.g. Ubiquiti UF-GP-C+ or compatible third-party module for TDC Net's network
- **Second U7 Pro XG** — for the basement (you already own one for the 1st floor)
- **USW Pro XG 8 PoE** — sits in the basement, provides 10 GbE + PoE++ to all basement devices
- **Fiber patch cable** — SC/APC to LC/UPC (verify FTP connector type), short run from FTP to UCG Fiber SFP+ port
- Patch cables
    - 1st floor: UCG Fiber (LAN) → 1st floor AP, UCG Fiber (LAN) → wall jack #1
    - Basement: wall jack #1 → switch, switch → basement AP, switch → MacBook, switch → Linux server
