[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | **[TB08: ICMR](TB08_ICMR.md)** | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB08 — Input Common-Mode Range (ICMR)

Characterization of the allowable input common-mode voltage range ($V_{ICMR,min}$ to $V_{ICMR,max}$) over which the amplifier maintains linear operation and proper transistor saturation.

---

## 1. Purpose
- Determine the minimum allowable input common-mode voltage ($V_{ICMR,min}$).
- Determine the maximum allowable input common-mode voltage ($V_{ICMR,max}$).
- Quantify tracking error and gain compression as input approaches supply rails.
- For complementary (rail-to-rail) input stages, evaluate the transconductance handover $G_m(V_{CM})$ profile.

---

## 2. Parameters Measured
- **Lower ICMR Limit ($V_{ICMR,min}$):** Voltage below which devices leave saturation or gain collapses
- **Upper ICMR Limit ($V_{ICMR,max}$):** Voltage above which tail sources crush or differential pair triodes
- **Linear Dynamic Window ($\Delta V_{ICMR}$):** $V_{ICMR,max} - V_{ICMR,min}$
- **Handover Variation ($\Delta G_m / G_{m,nom}$):** Peak-to-peak ripple across the transition zone (rail-to-rail pairs)

---

## 3. Applicable Amplifiers
- **Class B:** Differential pairs, active-load diff-pairs, complementary input OTAs
- **Class C:** Operational amplifiers, folded-cascode and telescopic topologies
- **Class D:** Fully differential amplifiers

---

## 4. Testbench Schematic

```text
       Closed-Loop Unity-Gain Follower Method (Op-Amps)
               VDD
                │
                ▼
        ┌───────────────┐
Vin ────┤+              │
        │      DUT      ├──── Vout ───┬─── RL, CL
   ┌────┤-              │             │
   │    └───────┬───────┘             │
   │            │                     │
   │            ▼                     │
   │           VSS                    │
   └──────────────────────────────────┘
```

![TB08 ICMR](assets/TB08_ICMR.png)
*Figure 1: Unity-gain buffer follower test configuration for ICMR extraction.*

---

## 5. Circuit Configuration
### Method A: Unity-Gain Follower (Operational Amplifiers)
1. Configure DUT in unity-gain negative feedback: connect inverting input ($V_{IN-}$) directly to output ($V_{OUT}$).
2. Connect non-inverting input ($V_{IN+}$) to an independent DC sweep source $V_{IN}$.
3. Sweep $V_{IN}$ from $V_{SS} - 0.1\,\text{V}$ to $V_{DD} + 0.1\,\text{V}$.

### Method B: Small-Signal AC vs. Common-Mode Sweep (OTAs)
1. Apply small differential AC signal ($V_{id} = 10\,\text{mV}_{AC}$).
2. Sweep the common-mode DC level $V_{CM}$ from $V_{SS}$ to $V_{DD}$.
3. Measure differential gain $A_d(V_{CM})$ or transconductance $G_m(V_{CM})$.

---

## 6. Simulation Type
- **Method A:** `.dc VIN <v_start> <v_stop> <v_step>`
- **Method B:** `.dc VCM <v_start> <v_stop> <v_step>` with nested `.ac`

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB08 (Method A)
VIN in 0 DC=0.9
XDUT in out out vdd vss opamp_core
RLOAD out 0 100k
CLOAD out 0 2pF

.dc VIN 0 1.8 1m
```
1. Run DC voltage transfer sweep.
2. Calculate tracking error voltage $V_{error} = V_{out} - V_{in}$.
3. Define error threshold $\epsilon_{tol}$ (e.g. $1\,\text{mV}$ or $1\%$ gain compression).
4. Identify boundary points $V_{ICMR,min}$ and $V_{ICMR,max}$.

---

## 8. Calculations
### Tracking Error
$$
V_{error}(V_{in}) = V_{out}(V_{in}) - V_{in}
$$

### Small-Signal Derivative
$$
A_{CL}(V_{in}) = \frac{dV_{out}}{dV_{in}}
$$

### Valid ICMR Boundary Condition
$$
V_{ICMR,min} \le V_{in} \le V_{ICMR,max} \quad \text{where} \quad |A_{CL}(V_{in}) - 1.0| \le \delta_{spec}
$$

---

## 9. Required Plots
- **DC Transfer Characteristic:** $V_{out}$ vs. $V_{in}$ across entire rail-to-rail span.
- **Buffer Error Curve:** $(V_{out} - V_{in})$ vs. $V_{in}$ (magnified $\text{mV}$ scale).
- **Transconductance Profile:** $G_m(V_{CM})$ vs. $V_{CM}$ showing handover flat-zone.

---

## 10. Measurements to Save
```spice
.meas DC Vicmr_min WHEN v(error) = -1m FALL=1
.meas DC Vicmr_max WHEN v(error) = 1m RISE=1
.meas DC Vicmr_span PARAM = 'Vicmr_max - Vicmr_min'
* Placeholder criteria:
* Vicmr_min <= DESIGN_SPEC_VICMR_MIN
* Vicmr_max >= DESIGN_SPEC_VICMR_MAX
```

---

## 11. PVT Considerations
- In NMOS-input pairs, $V_{ICMR,min} = V_{SS} + V_{DSAT,tail} + V_{GS,in}$. At the **SS corner and low temperature**, threshold voltage $V_{th}$ rises, shifting $V_{ICMR,min}$ up and shrinking the allowable range.
- At low $V_{DD}$, complementary pairs may exhibit a dead-zone where neither NMOS nor PMOS input pair is fully turned on.

---

## 12. Post-Layout Considerations
- **Body Effect ($V_{SB}$):** If input pair substrates are tied to global rails rather than isolated source wells, threshold voltages shift dynamically with $V_{CM}$, altering the linear range.
- Ensure symmetric layout dummy bars around tail transistors to prevent $V_{DSAT}$ mismatch.

---

## 13. References
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: Input Common-Mode Range Measurements. [Course Notes Link](https://pallen.ece.gatech.edu/Academic/ECE_6412/Spring_2003/L240-Sim%26MeasofOpAmps%282UP%29.pdf)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 4: Differential Amplifiers (Common-Mode Range Limitations).

---

## 14. Related Testbenches
- **[TB04: CMRR](TB04_CMRR.md)** — Relies on operating within the valid ICMR window.
- **[TB06: Gm/Rout](TB06_OTA_Gm_Rout.md)** — Measures $G_m(V_{CM})$ directly.
- **[TB09: Output Swing](TB09_Output_Swing.md)** — Complementary metric for output dynamic range.

---

## 15. Navigation

| [<< TB07: Offset](TB07_Input_Offset.md) | [Docs Index](index.md) | [TB09: Swing >>](TB09_Output_Swing.md) |
