# Mem Element Emulator

> LTspice-based simulation study of memristive behaviour using Graetz bridge, JFET, and floating GIC circuits.

**B.Tech Project | Semester V | Electronics and Communication Engineering**  
Faculty of Technology, University of Delhi (2025–2026)  
*Supervised by Prof. Raj Senani | Co-supervised by Dr. Khushwant Sehra*

---

## Overview

Mem-elements are circuit components whose behaviour depends not only on the present input but also on the history of past excitation. This property — known as memory — makes them fundamental to neuromorphic computing, adaptive analog hardware, and non-volatile memory design.

Since physical memristive devices require specialised fabrication techniques unavailable in most laboratories, this project builds and analyses **emulator circuits** that reproduce memristive behaviour using standard, easily available components — all simulated in **LTspice**.

---

## Project Stages

| Stage | Circuit | Memory Type | Key Component |
|-------|---------|-------------|---------------|
| 1 | Graetz bridge + Capacitor | Charge-based | 95 pF capacitor |
| 2 | Graetz bridge + Inductor | Flux-based | 1 mH inductor |
| 3 | JFET-based emulator | Channel modulation | J310 JFET |
| 4 | Floating GIC | Synthetic inductance | UA741 op-amps |

---

## Circuit Descriptions

### Stage 1: Capacitor-Based Mem-Element Emulator

A Graetz diode bridge (D1N4148) driven by a sinusoidal source with a 95 pF capacitor connected across the bridge nodes.

- The capacitor stores charge during each half cycle
- Stored charge influences the next cycle → history-dependent response
- Produces a **narrow pinched hysteresis loop** at low frequencies

**LTspice Netlist:**
```spice
* Graetz Bridge with Capacitor - Extended Memristor
V1 Vin 0 SIN(0 1 30)
D1 Vin 2 D1N4148
D2 3 Vin D1N4148
D3 3 0 D1N4148
D4 0 2 D1N4148
C1 2 3 95pF
.model D1N4148 D(IS=2.52e-9 N=1.9 BV=100 IBV=0.1 CJO=4.0e-12 M=0.333 TT=4e-9)
.tran 0 0.3 0.05 50u
.end
```

---

### Stage 2: Inductor-Based Mem-Element Emulator

The capacitor is replaced with a **1 mH inductor**. Inductors store energy as magnetic flux, producing stronger and longer-lasting memory.

- Current cannot change instantaneously → stronger history-dependence
- Produces a **wider, more pronounced hysteresis loop** compared to Stage 1

**LTspice Netlist:**
```spice
* Graetz Bridge with Inductor - Hysteresis Test
V1 Vin 0 SIN(0 2.4 12.15)
D1 Vin 2 1N4007
D2 3 Vin 1N4007
D3 3 0 1N4007
D4 0 2 1N4007
L1 2 3 1m
.model 1N4007 D()
.tran 0 0.3 0.05 50
.PROBE
.END
```

---

### Stage 3: JFET-Based Mem-Resistor Emulator

A **J310 JFET** is used as the memory element. The JFET's channel resistance varies with gate voltage, and under sinusoidal excitation, this resistance changes in a history-dependent manner.

- No special materials required
- Compact and tunable via gate bias
- Produces a **smooth pinched hysteresis loop** through channel modulation

**LTspice Netlist:**
```spice
* Passive memristor emulator (JFET)
V1 Vin 0 SIN(0 3.2 120)
Rsen Vin 1 10
R1 1 2 1Meg
RG 2 G 1Meg
C1 2 0 1.48n
J1 1 G 0 J310_MODEL
.model J310_MODEL NJF (VTO=-2.5 BETA=2m IS=1e-12 LAMBDA=0.02 CGS=2p CGD=2p)
.tran 0 40m 0
.end
```

---

### Stage 4: Floating GIC for Synthetic Inductance

A **Floating Generalized Impedance Converter (GIC)** is designed using UA741 op-amps to generate synthetic inductance — eliminating the need for bulky physical inductors.

**Key formula:**
$$Z_{eq} = \frac{Z_1 Z_3 Z_5}{Z_2 Z_4}$$

For inductive GIC (Z4 = capacitor, rest = resistors):
$$L_{eq} = \frac{R_1 R_3 C_4}{R_2} \cdot R_5$$

**Design values used:**
- R = 10 kΩ, C = 100 pF, Rb = 1 kΩ → **Leq = 1 mH**

Validated using a low-pass filter configuration; frequency response matched a real inductor.

---

## Simulation Results

### V–I Hysteresis Loops

| Stage | Loop Type | Observation |
|-------|-----------|-------------|
| Capacitor | Narrow, pinched | Charge-driven memory, reduces at higher frequency |
| Inductor | Wide, pronounced | Flux-controlled memory, stronger and longer-lasting |
| JFET | Smooth, pinched | Semiconductor channel modulation, tunable |

**Key finding:** All three configurations produce a **pinched hysteresis loop passing through the origin** — the fundamental signature of mem-element behaviour.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| LTspice XVII (Analog Devices) | Schematic, simulation, V–I plots |
| 1N4148, 1N4007 | Diode bridge nonlinearity |
| J310 JFET | Semiconductor mem-resistive emulation |
| UA741 Op-Amp | Floating GIC construction |

---

## How to Run Simulations

1. Download and install **LTspice XVII** from [Analog Devices](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html)
2. Open any `.asc` file from the `LTspice_Netlists/` folder
3. Press **Run** (F5)
4. To plot V–I hysteresis: right-click on plot → Add trace → set X-axis to `V(2,3)`, Y-axis to `I(V1)`

---

## References

1. D. Biolek et al., "Modeling and Emulation of Extended Memristors: Two-Port Approach Revisited," *IEEE TCAS-II*, vol. 72, no. 1, Jan. 2025.
2. F. Corinto and A. Ascoli, "Memristive diode bridge with LCR filter," *Electronics Letters*, vol. 48, no. 14, 2012.
3. J. Sadecki and W. Marszalek, "Analysis of a memristive diode bridge rectifier," *Electronics Letters*, vol. 55, no. 3, 2019.
4. R. Senani, "New single-capacitor simulations of floating inductors," *Electrocomponent Science and Technology*, vol. 10, 1982.
5. L. O. Chua and S. M. Kang, "Memristive Devices and Systems," *Proceedings of the IEEE*, 1976.

---

## Team

| Name | Roll No. |
|------|----------|
| Kanishk Nagar | 23294917003 |
| Divyanshu Kumar | 23294917079 |
| Jay Prakash | 23294917152 |
| Manish Kumar | 23294917153 |
| Aman Kumar Mukhiya | 23294917158 |

**B.Tech ECE, Faculty of Technology, University of Delhi**

---

*Similarity Index: 7% (verified via Turnitin)*
