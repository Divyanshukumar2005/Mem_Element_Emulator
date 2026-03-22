# Mem Element Emulator (LTspice Simulation)

Semester V project on simulating mem-element behaviour using standard circuit components in LTspice. Based on the work by Biolek et al. (IEEE TCAS-II, 2025) and related papers — we implemented the circuits, ran the simulations, and tried to understand *why* the hysteresis forms the way it does, not just that it does.

---

## Background

The core idea is simple: a real memristor is hard to fabricate — it needs nanoscale titanium dioxide structures or similar. But you can get the same electrical behaviour (a pinched hysteresis loop in the V-I plane) using a nonlinear resistive two-port loaded with a capacitor or inductor. That's an emulator.

The paper by Biolek et al. goes deeper than just showing the loop — it derives the exact conditions a two-port must satisfy for the emulated element to qualify as a proper extended memristor (the zero-crossing property). We used this framework to understand each of our circuits.

---

## What we built

### 1. Graetz bridge + Capacitor

Four 1N4148 diodes in a bridge configuration, with a 95 pF capacitor across the output port. Driven at 30 Hz.

The diode bridge is the nonlinear two-port. The capacitor is the memory element — it stores charge from one half-cycle and that stored charge modifies the next. At low frequencies, this produces a pinched hysteresis loop.

One thing worth noting from the paper: the zero-crossing property (loop pinching exactly at the origin) holds only when all four diodes are identical. With mismatched diodes, the pinch point shifts. We used D1N4148 throughout to keep it symmetric.

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

Same bridge, capacitor replaced with a 1 mH inductor. Driven at 12.15 Hz with 1N4007 diodes this time.

Inductors store magnetic flux rather than charge. Since current through an inductor resists sudden change, the memory effect is stronger — the loop gets wider. The state variable here is the inductor flux, not charge.

Practical limitation: you need very high inductance to see clear low-frequency behaviour. That's what motivated the GIC stage.

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

A J310 JFET with supporting resistors and a capacitor. No diode bridge here.

The JFET's channel resistance varies with gate voltage. Under a sinusoidal input, the capacitor voltage (which controls the gate) changes gradually — giving the device a history-dependent resistance. That's the memory.

Technically, the paper shows that the JFET circuit doesn't strictly satisfy the zero-crossing condition the way the diode bridge does. But the deviation is in the picoampere range in practice — immeasurably small. The loop still looks pinched at the origin, and all the fingerprints of memristive behaviour are present.

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

The problem with physical inductors is that getting 1 mH or more in a compact form isn't easy — parasitics, size, cost. A Generalized Impedance Converter built from two UA741 op-amps solves this by synthesizing inductance electronically.

The equivalent inductance comes out to:

```
Leq = (R1 * R3 * C4) / R2  *  R5
```

With R = 10 kΩ, C = 100 pF, Rb = 1 kΩ, this gives Leq = 1 mH — same as the physical inductor in stage 2, without the bulk.

We verified this independently using a low-pass filter — the frequency response matched what you'd expect from a real inductor. Connecting the GIC directly to the Graetz bridge to complete the emulator is left as future work.

---

## Results

| Configuration | Loop shape | Memory mechanism |
|---|---|---|
| Bridge + Capacitor | Moderate width, frequency-dependent | Charge storage (state = qC) |
| Bridge + Inductor | Wider, stronger | Flux storage (state = φL) |
| JFET | Smooth, tunable | Channel resistance modulation |

All three produced a pinched hysteresis loop — the standard fingerprint of mem-element behaviour described in Chua's original work and verified in the Biolek paper.

---

## How to run the simulations

1. Download LTspice (free, Analog Devices)
2. Open any `.asc` file from `netlists/`
3. Press Run (F5)
4. For the V-I hysteresis plot: right-click waveform viewer → Add Trace → set X-axis to `V(2,3)`, Y-axis to `I(V1)`

---

## References

- D. Biolek, Z. Kolka, V. Biolkova, Z. Biolek, Z. Kohl — "Modeling and Emulation of Extended Memristors: Two-Port Approach Revisited," *IEEE Trans. Circuits Syst. II*, vol. 72, no. 1, Jan. 2025
- F. Corinto, A. Ascoli — "Memristive diode bridge with LCR filter," *Electronics Letters*, 2012
- J. Sadecki, W. Marszalek — "Analysis of a memristive diode bridge rectifier," *Electronics Letters*, 2019
- R. Senani — "New single-capacitor simulations of floating inductors," *Electrocomponent Science and Technology*, 1982
- L. O. Chua, S. M. Kang — "Memristive Devices and Systems," *Proc. IEEE*, 1976
- S. P. Adhikari et al. — "Three fingerprints of memristor," *IEEE Trans. Circuits Syst. I*, 2013

---

B.Tech ECE, Semester V — Faculty of Technology, University of Delhi
Supervised by Prof. Raj Senani and Dr. Khushwant Sehra
