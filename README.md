# PUDGALA 300
### High-Power-Density 250W Synchronous Buck Converter (28V–60V In → 24.88V @ 10A Out)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![KiCad Version](https://img.shields.io/badge/KiCad-v10.0-blue.svg)](https://kicad.org/)
[![Hardware](https://img.shields.io/badge/Hardware-Open%20Source-green.svg)](https://en.wikipedia.org/wiki/Open-source_hardware)
[![PCB Layers](https://img.shields.io/badge/Layers-6--Layer%20FR4-orange.svg)]()
[![Form Factor](https://img.shields.io/badge/Dimensions-50x50%20mm-lightgrey.svg)]()
[![Tested Output](https://img.shields.io/badge/Bench%20Tested-12.5A%20Continuous-brightgreen.svg)]()
[![Efficiency](https://img.shields.io/badge/Peak%20Efficiency->96%25-brightgreen.svg)]()

> *"Ex umbra, in solem."* — [HEX Aerospace](https://hexaerospace.in)

---

![PUDGALA 300 Bare 6-Layer PCB](images/irl/pcb_bare_board.jpg)

## Overview & Motivation

The **PUDGALA 300** is an open-source, high-power-density synchronous buck converter developed with [HEX Aerospace](https://hexaerospace.in). It steps down wide-range DC voltages (nominal 36V–54V battery buses, 7S–14S LiPo / Li-ion packs) down to a tightly regulated **24.88V (25V / 24V bus compatible)** rail delivering **10A continuous (250W nominal)**, and has been bench-tested up to **12.5A continuous (~312W)** with peak efficiency exceeding **96%**.

### Field FPV Charging
The primary motivation for this project was powering high-power FPV drone battery chargers directly from repurposed **13S electric scooter (E-scooter) battery packs** in the field.

* **The Problem:** 13S Li-ion battery packs swing from **39.0V** (depleted) to **54.6V** (fully charged). However, most popular field chargers (such as ISDT, ToolkitRC, Hota, and SkyRC) have a maximum input voltage limit of **30.0V DC**. Connecting a 13S battery directly would instantly destroy the charger.
* **The Solution:** The PUDGALA 300 steps down the 39V–54.6V input into a stable **24.88V** rail at up to **10A–12.5A (>300W)**. This allows pilots to rapidly parallel-charge multiple 4S and 6S drone packs in the field using salvaged e-mobility batteries.

---

> [!CAUTION]
> ### Critical Operational Notice: Anti-Spark Connectors & Inrush Caution
> During early testing of the prototype, the board was connected to a high-voltage 13S battery (~54.6V) with an active load already plugged into the output. The raw XT60 connector sparked violently upon contact, and the severe inrush arc and inductive voltage spike ($L_{lead} \cdot \frac{di}{dt}$ resonance between cable inductance and ceramic input caps) exceeded the MOSFET breakdown rating, killing one of the switching FETs.
>
> **Mandatory Safe Operating Procedures:**
> 1. **Always Use Anti-Spark Connectors:** When connecting $>30\text{V}$ sources (especially 13S / 48V–54.6V packs), use an anti-spark connector with a built-in pre-charge resistor (such as **XT90-S** or **XT60-S**) or an external pre-charge switch.
> 2. **Power-On Sequence:** Always connect the battery input **first** with no output load connected. Confirm the output power indicator LED is on (verifying stable 24.88V regulation) **before** plugging in your charger.
> 3. **Lead Length:** Keep input battery leads as short as possible to minimize parasitic wiring inductance.

---

## Key Specifications

| Parameter | Specification | Notes / Conditions |
| :--- | :--- | :--- |
| **Input Voltage ($V_{IN}$)** | **28.0V to 60.0V DC** | Designed for 13S E-scooter packs (39V–54.6V), 7S–14S LiPo, 36V/48V buses |
| **Output Voltage ($V_{OUT}$)** | **24.88V nominal ($\approx 25\text{V}$)** | Calibrated via precision feedback divider ($301\text{ k}\Omega / 10\text{ k}\Omega$) |
| **Continuous Output Current ($I_{OUT}$)** | **10.0 A (Nominal) / 12.5 A (Tested)** | 250W nominal; bench-tested to 312W continuous (requires active cooling) |
| **Peak / Overcurrent Limit ($I_{OCP}$)** | **12.0 A peak** | Cycle-by-cycle limit ($V_{CS-TH} = 60\text{ mV} / 5\text{ m}\Omega$) with hiccup restart |
| **Continuous Output Power ($P_{OUT}$)** | **250 W nominal (up to 312 W)** | High continuous power density in a pocket-sized 50×50 mm board |
| **Switching Frequency ($f_{SW}$)** | **~1.0 MHz (966 kHz nominal)** | Programmed via $R_{RT} = 9.09\text{ k}\Omega$ |
| **Peak Efficiency** | **> 96%** | Typically 94.5%–96.2% across 36V–52V input range |
| **PCB Dimensions** | **50.0 mm × 50.0 mm × 12.0 mm** | Low profile; horizontal bent-lead electrolytic capacitors |
| **PCB Layer Stackup** | **6 Layers (1 oz / 35 µm Cu each)** | Solid internal GND reference planes on L2 & L5 |
| **Connectors** | **AMASS XT30PW-M (Right-Angle)** | High-current, vibration-resistant power in/out connectors |
| **Auxiliary Fan Header** | **2-pin test points (+Fan / -Fan)** | 24V regulated output for an active cooling fan |
| **Enclosure** | **CAD STEP File Coming Soon** | 3D-printable / CNC enclosure in progress |

---

## System Architecture & Block Diagram

```mermaid
flowchart LR
    subgraph Input_Stage ["Input Stage (13S Battery: 39V - 54.6V)"]
        BATT["13S E-Scooter Pack\n(Anti-Spark XT90-S)"] --> J1["Power In\nXT30PW-M"]
        J1 --> D2["TVS Surge Clamp\nSMBJ60A (60V)"]
        D2 --> C_IN_BULK["Bulk Damping\n47µF Elec. (C9)"]
        C_IN_BULK --> C_IN_HF["High-Freq MLCC\n2x 22µF 2220 + 100nF"]
    end

    subgraph Power_Stage ["Synchronous Buck Stage"]
        C_IN_HF --> Q1["High-Side NexFET\nCSD19534Q5A (100V)"]
        Q1 --> SW["Switch Node (SW)"]
        Q2["Low-Side Sync NexFET\nCSD19502Q5B (80V, 2.8mΩ)"] --- SW
        SW --> L1["Molded Power Choke\n2.2µH 20A (BMRF)"]
        L1 --> RSENSE["Current Shunt\n5mΩ 2512 (R3)"]
    end

    subgraph Controller_Loop ["Control & Protection (LM5149-Q1)"]
        U1["TI LM5149-Q1\nSync Controller"] -.->|HO Drive| Q1
        U1 -.->|LO Drive| Q2
        RSENSE -.->|Kelvin ISNS±| U1
        U1 -.->|DRSS Active| R2["CNFG: 40.2kΩ"]
        U1 -.->|1 MHz Osc| R1["RT: 9.09kΩ"]
        U1 -.->|PFM / Diode Emul.| VDDA["VDDA"]
    end

    subgraph Output_Stage ["Output Filtering & Protection (24.88V / 10A-12.5A)"]
        RSENSE --> C_OUT_HF["MLCC Bank\n4x 10µF 1210"]
        C_OUT_HF --> C_OUT_BULK["Output Bulk\n220µF Elec. (C10)"]
        C_OUT_BULK --> D3["OVP TVS Clamp\nSMBJ28A (28V)"]
        D3 --> FB_DIV["Feedback Divider\n301kΩ / 10kΩ"] -.->|FB| U1
        D3 --> LED_IND["Power Good LED\n(D1 + R8 11.5k)"]
        D3 --> J2["Power Out\nXT30PW-M"] --> CHARGER["FPV Charger\n(< 30V Input Limit)"]
        D3 --> FAN_HDR["Fan Header\n(+Fan / -Fan)"]
    end

    classDef stage fill:#1e1e24,stroke:#4a90e2,stroke-width:2px,color:#ffffff;
    classDef controller fill:#2c3e50,stroke:#e74c3c,stroke-width:2px,color:#ffffff;
    class Input_Stage,Power_Stage,Output_Stage stage;
    class Controller_Loop controller;
```

---

## Component Analysis & IC Explanations

### 1. Primary Controller: Texas Instruments LM5149-Q1 (`U1`)
The **LM5149-Q1** (`PLM5149QRGYRQ1`) is an AEC-Q100 qualified, 80V/100V-capable synchronous buck DC-DC controller designed for high-efficiency, low-EMI industrial and automotive applications.
* **Dual-Random Spread Spectrum (DRSS):** Programmed via a **$40.2\text{ k}\Omega$** resistor ($R_2$) between `CNFG` and `AGND`. DRSS combines triangular frequency modulation with pseudorandom dithering to disperse EMI energy across a wide frequency band, slashing fundamental and harmonic emission peaks.
* **Switching Frequency ($f_{SW} \approx 1\text{ MHz}$):** Programmed via a **$9.09\text{ k}\Omega$** 1% resistor ($R_1$) between `RT` and `AGND`:
  $$f_{SW} \approx \frac{6 \times 10^7}{R_{RT}(\text{k}\Omega) + 53} = \frac{60000}{9.09 + 53} \approx 966\text{ kHz}$$
  High-frequency operation enables using a compact $2.2\mu\text{H}$ inductor and tiny MLCCs while delivering fast transient response.
* **PFM / Diode Emulation Mode:** The `PFM/SYNC` pin is tied directly to `VDDA`. Under light or no-load conditions, the controller automatically enters Pulse Frequency Modulation, preventing reverse inductor current and reducing standby battery drain.
* **Differential Current Sense & Cycle-by-Cycle Limiting:** Connected to pins `ISNS+` (20) and `VOUT/ISNS-` (21). The internal $60\text{ mV}$ threshold triggers immediate high-side turn-off and initiates hiccup-mode timeout under short circuits.
* **Internal Gate Drivers & VCC Regulator:** Integrated 5V VCC regulator bypassed with $2.2\mu\text{F}$ ceramic capacitor ($C_8$). Delivers robust peak drive currents with adaptive deadtime control to prevent shoot-through.

---

### 2. High-Side Power MOSFET: TI CSD19534Q5A (`Q1`)
* **Package:** SON 5 mm × 6 mm (TDSON-8)
* **Ratings:** $V_{DS} = 100\text{ V}$, $I_D = 44\text{ A}$ (continuous), $137\text{ A}$ (pulsed)
* **$R_{DS(on)}$:** $10.3\text{ m}\Omega$ typical / $12.6\text{ m}\Omega$ max @ $V_{GS} = 10\text{ V}$
* **Gate Charge ($Q_g$):** Ultra-low $13.8\text{ nC}$
* **Selection Rationale:** At 1.0 MHz, high-side FET switching losses ($P_{SW} \propto f_{SW} \times V_{IN} \times I_{OUT} \times Q_g$) dominate over conduction losses. The CSD19534Q5A's low gate-drain capacitance ($C_{GD}$) and low total gate charge minimize switching losses.

---

### 3. Low-Side Synchronous Rectifier MOSFET: TI CSD19502Q5B (`Q2`)
* **Package:** SON 5 mm × 6 mm (TDSON-8)
* **Ratings:** $V_{DS} = 80\text{ V}$, $I_D = 138\text{ A}$ (silicon limited), $200\text{ A}$ (pulsed)
* **$R_{DS(on)}$:** Ultra-low **$2.8\text{ m}\Omega$ typical / $3.4\text{ m}\Omega$ max** @ $V_{GS} = 10\text{ V}$
* **Selection Rationale:** The low-side MOSFET conducts during the $(1 - D)$ freewheeling interval (~50% duty cycle). Conduction losses ($I_{RMS}^2 \times R_{DS(on)}$) dominate. With $2.8\text{ m}\Omega$ on-resistance, conduction loss at 10A is only $\approx 0.15\text{ W}$, keeping the low-side switch cool.

---

### 4. Power Inductor: Chilisin BMRF00101040 (`L1`)
* **Inductance:** $2.2\mu\text{H} \pm 20\%$
* **Saturation Current ($I_{sat}$):** **20.0 A** (rated for $>1.6\times$ the maximum converter peak current)
* **Shielding:** Shielded composite iron-powder construction eliminating radiated magnetic flux and EMI.

---

### 5. Current Sense Shunt: 5 mΩ Metal Element 2512 (`R3`)
* **Value:** $0.005\ \Omega\ (5\text{ m}\Omega)$, 1% tolerance, 2W/3W rating
* **Current Limit Threshold ($I_{PEAK}$):**
  $$I_{PEAK(CL)} = \frac{V_{CS-TH}}{R_{SENSE}} = \frac{60\text{ mV}}{5\text{ m}\Omega} = 12.0\text{ A}$$
  With average $I_{OUT} = 10\text{ A}$ and inductor ripple $\Delta I_L \approx 3.5\text{A} - 4.5\text{A}$, the peak inductor current is $10 + \Delta I_L / 2 \approx 11.75\text{ A}$. The 12.0A threshold provides clean margin while clamping overloads.
* **Kelvin 4-Wire Routing:** Instrumented with net-ties `NT1` and `NT2` providing true Kelvin differential sense traces directly into the LM5149-Q1.

---

### 6. Protection Clamps: SMBJ60A & SMBJ28A TVS Diodes
* **Input TVS (`D2` — SMBJ60A):** 600W peak pulse power dissipation, $60\text{ V}$ reverse standoff voltage ($V_{RWM}$). Clamps inductive spikes during battery connections.
* **Output TVS (`D3` — SMBJ28A):** $28\text{ V}$ reverse standoff voltage, $31.1\text{ V}$ breakdown. Clamps transient overvoltages to prevent exceeding the 30V maximum input limit of downstream FPV chargers.
* **Fuse Recommendation:** To guarantee complete protection against a catastrophic high-side FET failure, **always install an inline 15A fuse** (or use a battery with a short-circuit-protected BMS) on the battery input lead. If an overvoltage fault occurs, the TVS clamps the rail to ground and blows the fuse instantly, ensuring the charger never sees excessive voltage.

---

### 7. Hybrid Capacitive Filtering
* **Input Stage:** Two $22\mu\text{F}$ 2220 MLCCs (`C1`, `C2`) handle high-frequency ripple. A $100\text{ nF}$ 0603 MLCC (`C14`) is placed directly across the high-side FET drain to ground. A $47\mu\text{F}$ electrolytic capacitor (`C9`, horizontal mount) damps supply line ringing.
* **Output Stage:** Four $10\mu\text{F}$ 1210 MLCCs (`C4`, `C11`, `C12`, `C13`) yield $40\mu\text{F}$ of ultra-low ESR capacitance. A $220\mu\text{F}$ horizontal electrolytic (`C10`) provides bulk energy storage for step loads.

---

## Operating Constraints & Thermal Realities

### 1. Continuous 12.5A Bench Testing
The PUDGALA 300 was successfully tested on the bench at **12.5 A continuous** at ~24.9V, delivering **~312 W output power**:
* **$I^2 R$ Thermal Rise:** Conduction loss scales quadratically ($P = I^2 R$). Increasing current from 10A to 12.5A increases resistive power dissipation by **$+56\%$**.
* **Thermal Rise:** Under 12.5A continuous on an open test bench with still air, the board heats up substantially, with component temperatures quickly reaching $90^\circ\text{C}-100^\circ\text{C}$.
* **Cooling Requirement:** While the board handles 12.5A electrically, **active forced airflow or heatsink coupling is mandatory** for sustained operation above 10A.

### 2. Cooling Modes & Recommendations
1. **Natural Convection (Passive):** Suitable for loads up to **100W–120W continuous** (approx. 4A–5A).
2. **Forced Air Cooling:** Rated for **250W–300W+ continuous** when a 24V cooling fan is connected to the onboard `+Fan / -Fan` test points.
3. **Chassis / Heatsink Conduction:** The bottom layer has **no components** ("*Yes, there's nothing on this side*"). A 1.0 mm thermal pad placed between the bottom PCB ground plane and an aluminum enclosure allows sinking heat directly into external metalwork.
4. **Protective Enclosure:** A custom 3D-printable / CNC enclosure STEP model with integrated fan mounting and heatsink interface will be uploaded to this repository soon.

---

## 6-Layer PCB Stackup & Layout Engineering

The PCB utilizes a tailored 6-layer FR4 stackup (1.6 mm thickness, 1 oz / 35 µm copper on all layers) engineered for low parasitic inductance, optimal thermal spreading, and electromagnetic compliance.

```
┌────────────────────────────────────────────────────────┐ Layer 1 (F.Cu) - 35µm
│  Components, High-di/dt Switching Loop, Gate Drives, AGND │
├────────────────────────────────────────────────────────┤ Dielectric 1 (Prepreg) - 0.100mm
│  SOLID GROUND PLANE (In1.Cu) - 35µm                    │ (Extremely tight RF return path)
├────────────────────────────────────────────────────────┤ Dielectric 2 (Core) - 0.535mm
│  GND / Aux Shielding (In2.Cu) - 35µm                   │
├────────────────────────────────────────────────────────┤ Dielectric 3 (Prepreg) - 0.100mm
│  POWER PLANES: +BATT & +24V Distribution (In3.Cu) - 35µm│
├────────────────────────────────────────────────────────┤ Dielectric 4 (Core) - 0.535mm
│  SOLID GROUND PLANE (In4.Cu) - 35µm                    │ (Shielding & Thermal Conduction)
├────────────────────────────────────────────────────────┤ Dielectric 5 (Prepreg) - 0.100mm
│  FLAT BOTTOM GROUND PLANE (B.Cu) - 35µm                │
└────────────────────────────────────────────────────────┘ ("Yes, there's nothing on this side")
```

---

## Hardware Gallery

### Assembled & Bare PCB
| Populated SMD Board | Bare 6-Layer ENIG PCB |
| :---: | :---: |
| ![Populated SMD Board](images/irl/pcb_smd_assembled.jpg) | ![Bare PCB](images/irl/pcb_bare_board.jpg) |
| *Reflowed SMD components on hotplate* | *Bare 50x50 mm board with ENIG finish* |

### PCB Artwork & Stencil Inspection
| Copper Signature Detail | Silkscreen Motto & Fan Header | Solder Paste Deposition (Microscope) |
| :---: | :---: | :---: |
| ![Signature Detail](images/irl/pcb_signature_detail.jpg) | ![Motto Detail](images/irl/pcb_motto_detail.jpg) | ![Microscope View](images/irl/microscope_solder_paste_qfn.jpg) |
| *"Sid J" etched in top copper* | *"Ex umbra, in solem." & fan pads* | *Solder paste stencil on VQFN-24* |

### 3D CAD Renders & Schematic
| Isometric 3D View | Top Component Placement | Bottom Heatsink Plane |
| :---: | :---: | :---: |
| ![Isometric 3D](images/pcb_3d_isometric.png) | ![Top View](images/pcb_3d_top.png) | ![Bottom Plane](images/pcb_3d_bottom.png) |
| *3D isometric view* | *Component layout* | *Flat ground interface* |

> 📄 **Schematic:** Full vector schematic available at [`images/schematic.svg`](images/schematic.svg).

---

## Bill of Materials (BOM)

All components are standard, readily sourcable parts with matching **LCSC Part Numbers** for seamless turn-key assembly via JLCPCB, PCBWay, or other PCBA houses.

| Designator | Value / Part | Description | Package / Footprint | Qty | LCSC Part # |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **U1** | **PLM5149QRGYRQ1** | TI 80V/100V Synchronous Buck DC-DC Controller with DRSS | VQFN-24 (4.5×3.5mm) | 1 | **C7528454** |
| **Q1** | **CSD19534Q5A** | TI 100V 44A 10.3mΩ N-Ch NexFET (High-Side Switch) | TDSON-8 / SON 5×6mm | 1 | **C114200** |
| **Q2** | **CSD19502Q5B** | TI 80V 138A 2.8mΩ N-Ch NexFET (Low-Side Sync Rectifier) | TDSON-8 / SON 5×6mm | 1 | **C2864118** |
| **L1** | **2.2u, 20A** | Chilisin BMRF00101040 Shielded Power Inductor | SMD 10×10×4.0mm | 1 | **C3002681** |
| **R3** | **5m (0.005Ω)** | Current Sense Shunt Resistor, 1%, 2W/3W | SMD 2512 | 1 | **C48982662** |
| **D2** | **SMBJ60A** | 600W 60V Unidirectional TVS Surge Diode | DO-214AA (SMB) | 1 | *Standard* |
| **D3** | **SMBJ28A** | 600W 28V Unidirectional TVS Overvoltage Diode | DO-214AA (SMB) | 1 | *Standard* |
| **J1** | **Power In** | AMASS XT30PW-M Right-Angle 2-Pin Male Connector | XT30PW-M Horizontal | 1 | *Standard* |
| **J2** | **Power Out** | AMASS XT30PW-M Right-Angle 2-Pin Male Connector | XT30PW-M Horizontal | 1 | *Standard* |
| **C1, C2** | **22u** | Ceramic Capacitor, 50V/100V, X7R | SMD 2220 | 2 | **C7499406** |
| **C4, C11, C12, C13** | **10u** | Ceramic Capacitor, 35V/50V, X7R | SMD 1210 | 4 | **C77102** |
| **C9** | **47u** | Low-ESR Radial Aluminum Electrolytic, 63V/100V (Horizontal) | Rad 8×15mm Horizontal | 1 | **C5110663** |
| **C10** | **220u** | Low-ESR Radial Aluminum Electrolytic, 35V/50V (Horizontal) | Rad 10×16mm Horizontal | 1 | **C95082** |
| **C3** | **100n** | Bootstrap Capacitor (CBOOT), 50V, X7R | SMD 0603 | 1 | **C15725** |
| **C14** | **100n** | High-Frequency Input Bypass, 100V, X7R | SMD 0603 | 1 | **C7499406** |
| **C5** | **1u** | VDDA Analog Bypass Capacitor, 16V, X7R | SMD 0402 | 1 | **C29266** |
| **C6** | **680p** | Loop Compensation Zero Capacitor, 50V, C0G/X7R | SMD 0402 | 1 | *Standard* |
| **C7** | **24p** | Loop Compensation HF Pole Capacitor, 50V, C0G | SMD 0402 | 1 | **C2171081** |
| **C8** | **2.2u** | VCC Gate Drive Bias Bypass, 16V, X7R | SMD 0805 | 1 | **C170101** |
| **R1** | **9k09 (9.09kΩ)** | Switching Frequency Resistor (1% tolerance, sets ~1MHz) | SMD 0402 | 1 | **C861611** |
| **R2** | **40k2 (40.2kΩ)** | Mode & Dual-Random Spread Spectrum Resistor (1%) | SMD 0402 | 1 | **C491131** |
| **R4** | **301k (301kΩ)** | Feedback Voltage Divider Upper Resistor (0.5% or 1%) | SMD 0402 | 1 | **C2981491** |
| **R5** | **10k (10.0kΩ)** | Feedback Voltage Divider Lower Resistor (0.5% or 1%) | SMD 0402 | 1 | **C95204** |
| **R6** | **5k9 (5.9kΩ)** | External Error Amplifier Compensation Resistor (1%) | SMD 0402 | 1 | **C491142** |
| **R7** | **100k (100kΩ)** | Power Good (PG) Pull-Up Resistor (1%) | SMD 0402 | 1 | **C122538** |
| **R8** | **11k5 (11.5kΩ)** | LED Current Limiting Ballast Resistor (~2mA @ 24V) | SMD 0805 | 1 | *Standard* |
| **D1** | **LED** | Green / Blue Status Indicator LED | SMD 0603 | 1 | **C46610608** |
| **+FAN, -FAN** | **+Fan / -Fan** | Keystone 5015 Micro-Mini Solder Test Points | TP Keystone 5015 | 2 | *Standard* |
| **H1, H2, H3, H4** | **M3 Holes** | Mounting Holes with Grounded Annular Pad & Via Array | 3.2mm M3 Pad | 4 | *Mechanical* |
| **NT1, NT2, NT3** | **NetTie_2** | Kelvin-Sense & AGND Star Ground Isolation Net Ties | Net Tie SMD | 3 | *PCB Feature* |

> An interactive browser-based BOM with cross-probing component highlighting is located at [`250W Bat/bom/ibom.html`](250W%20Bat/bom/ibom.html).

---

## Manufacturing & Fabrication Guide

### PCB Fabrication Parameters
* **Layers:** 6
* **Board Dimensions:** 50.0 mm × 50.0 mm
* **Corner Radius:** 4.0 mm
* **PCB Thickness:** 1.6 mm
* **Copper Weight:** 1 oz outer / 1 oz inner (35 µm Cu across all 6 layers)
* **Surface Finish:** ENIG (Electroless Nickel Immersion Gold) recommended for high-current pads and flat thermal contact; Lead-Free HASL is acceptable.
* **Solder Mask:** Green (or Matte Black)
* **Silkscreen:** White
* **Minimum Trace / Space:** 0.15 mm / 0.15 mm (6 mil / 6 mil)
* **Minimum Drill:** 0.3 mm (via hole) / 0.5 mm (via pad)

### Production Files
All fabrication and assembly deliverables are located in the [`250W Bat/production/`](250W%20Bat/production/) directory:
* **Gerbers & Drill Archive:** [`250W_Bat.zip`](250W%20Bat/production/250W_Bat.zip)
* **BOM File:** [`bom.csv`](250W%20Bat/production/bom.csv)
* **Pick-and-Place (CPL) Centroid:** [`positions.csv`](250W%20Bat/production/positions.csv)
* **Netlist:** [`netlist.ipc`](250W%20Bat/production/netlist.ipc)
* **3D CAD STEP Model:** [`Converter.step`](250W%20Bat/Converter.step) & [`board.step`](250W%20Bat/board.step)

---

## Repository Structure

```
├── LICENSE                         # MIT Open Source License
├── README.md                       # Main project documentation (this file)
├── images/                         # Project graphics, renders, and diagrams
│   ├── pcb_3d_isometric.png        # Raytraced 3D perspective view
│   ├── pcb_3d_top.png              # Raytraced top component view
│   ├── pcb_3d_bottom.png           # Raytraced bottom ground plane view
│   ├── schematic.svg               # Vector schematic export
│   └── irl/                        # Real-world hardware photography
│       ├── README.md               # Photography index and descriptions
│       ├── pcb_bare_board.jpg      # Studio shot of bare ENIG PCB
│       ├── pcb_smd_assembled.jpg   # Populated SMD board post-reflow
│       ├── pcb_signature_detail.jpg# Macro of "Sid J" signature
│       ├── pcb_motto_detail.jpg    # Macro of "Ex umbra, in solem." & fan pads
│       └── microscope_solder_paste_qfn.jpg # Microscope view of VQFN-24 solder paste
└── 250W Bat/                       # KiCad design directory
    ├── 250W Bat.kicad_pro          # KiCad Project file (KiCad v10)
    ├── 250W Bat.kicad_sch          # Main schematic sheet
    ├── 250W Bat.kicad_pcb          # 6-layer board layout
    ├── Converter.step              # Full 3D STEP CAD assembly
    ├── board.step                  # Bare PCB 3D STEP file
    ├── bom/
    │   └── ibom.html               # Interactive HTML Bill of Materials
    └── production/                 # PCBA manufacturing outputs
        ├── 250W_Bat.zip            # Gerber & Excellon drill files
        ├── bom.csv                 # Production BOM with LCSC part numbers
        ├── positions.csv           # SMT pick-and-place centroid data
        └── netlist.ipc             # IPC-D-356 electrical test netlist
```

---

## Author & License

* **Author / Design:** Siddesh Jain
* **License:** Licensed under the permissive **[MIT License](LICENSE)**. You are free to use, modify, distribute, and manufacture this hardware for academic, hobbyist, and commercial projects.
