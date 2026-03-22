# Mem Element Emulator (LTspice Simulation)

This is a semester project we did in our 5th semester ECE. The goal was to understand how memristive behaviour works by building emulator circuits in LTspice — since actual memristive devices aren't something you can just get in a lab.

The project is based on the paper by Biolek et al. (IEEE TCAS-II, 2025) and some related works listed at the bottom. We didn't invent the circuits — we studied, implemented, and analysed them ourselves in simulation.

---

## Why this project

Real memristors need special nanoscale materials to fabricate. Most of us don't have access to that. But the interesting thing is — you can actually recreate memristive behaviour using normal components like diodes, capacitors, inductors, and JFETs. That's what an emulator does.

We wanted to understand *why* the hysteresis loop forms, how different memory elements (capacitor vs inductor vs JFET) change the shape of the loop, and whether synthetic inductance using a GIC can replace bulky physical inductors.

---

## What we simulated

### 1. Graetz bridge + Capacitor

A standard diode bridge (D1N4148) with a 95 pF capacitor across it, driven by a low-frequency sine wave.

The capacitor stores charge during one half-cycle and that stored charge affects the next — that's what gives it memory. At 30 Hz, you get a clear pinched hysteresis loop in the V–I plot.

```spice
V1 Vin 0 SIN(0 1 30)
D1 Vin 2 D1N4148
D2 3 Vin D1N4148
D3 3 0 D1N4148
D4 0 2 D1N4148
C1 2 3 95pF
.model D1N4148 D(IS=2.52e-9 N=1.9 BV=100 IBV=0.1 CJO=4.0e-12 M=0.333 TT=4e-9)
.tran 0 0.3 0.05 50u
```

---

### 2. Graetz bridge + Inductor

Same bridge, but the capacitor is swapped for a 1 mH inductor. Inductors store magnetic flux instead of charge, and since current through an inductor can't change instantly, the memory effect is stronger.

The loop gets noticeably wider here compared to the capacitor case. The tradeoff is you need large inductance values to see this clearly at low frequencies, which is impractical for hardware.

```spice
V1 Vin 0 SIN(0 2.4 12.15)
D1 Vin 2 1N4007
D2 3 Vin 1N4007
D3 3 0 1N4007
D4 0 2 1N4007
L1 2 3 1m
.model 1N4007 D()
.tran 0 0.3 0.05 50
```

---

### 3. JFET-based emulator

This one doesn't use a bridge at all. A J310 JFET has a channel resistance that varies with gate voltage. Under sinusoidal excitation, this resistance changes gradually with the signal history — which is exactly what gives mem-resistive behaviour.

The loop here is smoother than the passive stages, and the nice thing is you can tune the behaviour by changing the gate bias.

```spice
V1 Vin 0 SIN(0 3.2 120)
Rsen Vin 1 10
R1 1 2 1Meg
RG 2 G 1Meg
C1 2 0 1.48n
J1 1 G 0 J310_MODEL
.model J310_MODEL NJF (VTO=-2.5 BETA=2m IS=1e-12 LAMBDA=0.02 CGS=2p CGD=2p)
.tran 0 40m 0
```

---

### 4. Floating GIC (synthetic inductor)

To avoid the bulky inductor problem, we also designed a floating Generalized Impedance Converter using two UA741 op-amps. It creates equivalent inductance using only resistors and capacitors.

The formula we used:

```
Leq = (R1 * R3 * C4 / R2) * R5
```

With R = 10kΩ, C = 100pF, Rb = 1kΩ → Leq = 1 mH

We tested this separately in a low-pass filter to verify it behaves like a real inductor. The frequency response matched. Integrating it with the Graetz bridge is left as future work.

---

## Results summary

All three main configurations produced a pinched hysteresis loop passing through the origin — which is the standard signature of mem-element behaviour.

- Capacitor loop: moderate width, clearly frequency-dependent
- Inductor loop: wider, stronger memory effect
- JFET loop: smooth, tunable, no passive storage element needed

---

## How to run

1. Install LTspice (free, from Analog Devices)
2. Open any `.asc` file from the `netlists/` folder
3. Hit Run
4. To get the V–I plot: right click the waveform → Add trace → X-axis: `V(2,3)`, Y-axis: `I(V1)`

---

## Based on

- D. Biolek, Z. Kolka, V. Biolkova, Z. Biolek — "Modeling and Emulation of Extended Memristors: Two-Port Approach Revisited," IEEE TCAS-II, 2025
- F. Corinto, A. Ascoli — "Memristive diode bridge with LCR filter," Electronics Letters, 2012
- J. Sadecki, W. Marszalek — "Analysis of a memristive diode bridge rectifier," Electronics Letters, 2019
- R. Senani — "New single-capacitor simulations of floating inductors," 1982
- L. O. Chua, S. M. Kang — "Memristive Devices and Systems," Proceedings of the IEEE, 1976

---

Done as part of B.Tech ECE Semester V at Faculty of Technology, University of Delhi, under the supervision of Prof. Raj Senani and Dr. Khushwant Sehra.
