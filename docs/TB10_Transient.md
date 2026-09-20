[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | **[TB10: Transient](TB10_Transient.md)** | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB10 — Transient Response, Slew Rate & Settling Time

Large-signal and small-signal time-domain characterization to extract positive/negative slew rates, linear settling times, overshoot, and ringing damping.

---

## 1. Purpose
- Measure positive ($SR^+$) and negative ($SR^-$) large-signal slew rates.
- Determine small-signal and large-signal settling times ($t_s$) to defined error bands ($1\%$, $0.1\%$, $0.01\%$).
- Quantify transient step metrics: rise time ($t_r$), fall time ($t_f$), percentage overshoot ($OS_{\%}$), and ringing damping.
- Validate that internal bias circuits do not experience slew-induced recovery delays or latch-up.

---

## 2. Parameters Measured
- **Positive Slew Rate ($SR^+$):** Maximum rising slope $\frac{dV_{out}}{dt}$ in $\text{V}/\mu\text{s}$
- **Negative Slew Rate ($SR^-$):** Maximum falling slope $|\frac{dV_{out}}{dt}|$ in $\text{V}/\mu\text{s}$
- **Settling Time ($t_{s,1\%}, t_{s,0.1\%}, t_{s,0.01\%}$):** Time to enter and remain within error envelope
- **Percentage Overshoot ($OS_{\%}$):** Peak excursion beyond steady state
- **Rise & Fall Times ($t_r, t_f$):** $10\%$-to-$90\%$ transition duration

---

## 3. Applicable Amplifiers
- **Class C:** Operational amplifiers in closed-loop follower or inverting setups
- **Class D:** Fully differential amplifiers (differential transient response $V_{od}(t)$)
- **Class B:** 5-T and symmetrical OTAs driving capacitive loads

---

## 4. Testbench Schematic

```text
               Unity-Gain Buffer Follower Transient Setup
               VDD
                │
                ▼
        ┌───────────────┐
Vin(t) ─┤+              │
(Pulse) │      DUT      ├──── Vout(t) ───┬─── CL = CL,nom
   ┌────┤-              │                │
   │    └───────┬───────┘                └─── RL = RL,nom
   │            │
   │            ▼
   │           VSS
   └─────────────────────────────────────┘
```

![TB10 Transient](assets/TB10_Transient.png)
*Figure 1: Pulse response testbench for slew rate and settling time characterization.*

---

## 5. Circuit Configuration
1. **Feedback Topology:** Configure amplifier in a closed-loop unity-gain buffer configuration (or specified closed-loop application configuration).
2. **Pulse Generator:** Apply a fast square-wave pulse $V_{in}(t)$ to the non-inverting input:
   - **Large-Signal Test (Slew Rate):** Pulse amplitude spanning $80\%$ of the linear dynamic range (e.g. $0.4\,\text{V}$ to $1.4\,\text{V}$ on a $1.8\,\text{V}$ rail) with rise/fall times much faster than amplifier capability ($t_{edge} \le 100\,\text{ps}$).
   - **Small-Signal Test (Settling & Phase Margin):** Step amplitude of $50\,\text{mV}$ to isolate linear dynamics without saturating the input stage.
3. **Load:** Connect nominal load capacitor $C_L$ and resistor $R_L$.

---

## 6. Simulation Type
- **SPICE Directive:** `.tran <t_step> <t_stop>`
- **Time Step Control:** Use tight maximum step size (e.g. `t_step <= 0.1 / (10 * UGF)`) to prevent numerical integration damping artifacts.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB10
VIN in 0 PULSE(0.4 1.4 10n 100p 100p 100n 200n)
XDUT in out out vdd vss opamp_core
CLOAD out 0 5pF
RLOAD out 0 100k

* Transient simulation for two complete cycles
.tran 10p 250n
```
1. Run transient solver.
2. Differentiate output: $SR(t) = \frac{d(V_{out})}{dt}$.
3. Locate final settled value $V_{final}$ and compute tolerance boundaries.

---

## 8. Calculations
### Slew Rate
$$
SR^+ = \max\left(\frac{dV_{out}}{dt}\right)
$$
$$
SR^- = \left|\min\left(\frac{dV_{out}}{dt}\right)\right|
$$

### Settling Error Envelope
$$
e(t) = V_{out}(t) - V_{final}
$$
Settling time $t_s$ is the earliest time $t$ such that:
$$
|e(\tau)| \le \epsilon \times |V_{step}| \quad \forall \, \tau \ge t
$$
where $\epsilon = 0.01$ ($1\%$) or $\epsilon = 0.001$ ($0.1\%$).

### Percentage Overshoot
$$
OS_{\%} = \frac{V_{peak} - V_{final}}{|V_{final} - V_{initial}|} \times 100\%
$$

---

## 9. Required Plots
- **Large-Signal Transient Waveform:** $V_{in}(t)$ and $V_{out}(t)$ on the same axis over two full pulse periods.
- **Derivative Slope Plot:** $\frac{dV_{out}}{dt}$ vs. time, highlighting constant-slope slewing regions.
- **Small-Signal Step Zoom:** High-resolution zoom around the settling edge with $\pm 1\%$ and $\pm 0.1\%$ tolerance bands.

---

## 10. Measurements to Save
```spice
.meas TRAN SR_plus MAX deriv(v(out))
.meas TRAN SR_minus MIN deriv(v(out))
.meas TRAN Overshoot MAX v(out)
.meas TRAN ts_1pct WHEN v(out) = 'Vfinal * 0.99' CROSS=LAST
* Placeholder criteria:
* SR_plus >= DESIGN_SPEC_SR
* SR_minus >= DESIGN_SPEC_SR
* ts_1pct <= DESIGN_SPEC_SETTLING
```

---

## 11. PVT Considerations
- Slew rate is governed by tail current and compensation capacitance: $SR \approx \frac{I_{tail}}{C_c}$. In **SS corners and at high temperatures**, bias currents shrink, causing worst-case (lowest) slew rate.
- In two-stage op-amps, right-half-plane zeros and pole-zero doublets cause slow long-term settling tails ("sluggish settling").

---

## 12. Post-Layout Considerations
- **Parasitic Output Capacitance:** Layout wiring capacitance adds directly to $C_L$, reducing slew rate.
- **Current Supply Inductance (L):** Rapid current transients during slewing excite bond-wire / package inductance, creating rail bounce ringing.

---

## 13. References
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: Slew Rate and Settling-Time Measurement. [Course Notes Link](https://pallen.ece.gatech.edu/Academic/ECE_6412/Spring_2003/L240-Sim%26MeasofOpAmps%282UP%29.pdf)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 9: Operational Amplifiers (Slew Rate Mechanisms).

---

## 14. Related Testbenches
- **[TB03: Stability](TB03_Stability.md)** — Small-signal phase margin directly dictates overshoot $OS_{\%}$.
- **[TB09: Output Swing](TB09_Output_Swing.md)** — Defines the voltage boundaries for large-signal steps.
- **[TB16: Load Sweep](TB16_Load_Sweep.md)** — Evaluates slew rate and settling vs. load capacitance $C_L$.

---

## 15. Navigation

| [<< TB09: Swing](TB09_Output_Swing.md) | [Docs Index](index.md) | [TB11: Noise >>](TB11_Noise.md) |
