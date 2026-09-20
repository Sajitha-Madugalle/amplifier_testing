[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | **[TB11: Noise](TB11_Noise.md)** | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB11 — Noise Analysis

Small-signal stochastic noise characterization extracting input-referred noise spectral density, thermal white noise floor, $1/f$ flicker noise corner, and total integrated RMS noise.

---

## 1. Purpose
- Extract input-referred noise voltage spectral density $e_{n,in}(f)$ in $\text{nV}/\sqrt{\text{Hz}}$.
- Measure output-referred noise voltage spectral density $e_{n,out}(f)$.
- Determine thermal white-noise floor and identify the $1/f$ flicker-noise corner frequency ($f_c$).
- Calculate total integrated input-referred RMS noise voltage over specified system bandwidth ($f_L$ to $f_H$).

---

## 2. Parameters Measured
- **Input-Referred Noise Spectral Density ($e_{n,in}$):** In $\text{nV}/\sqrt{\text{Hz}}$ at $1\,\text{kHz}, 10\,\text{kHz}, 100\,\text{kHz}, 1\,\text{MHz}$
- **Thermal White-Noise Floor ($S_{n,white}$):** Flat high-frequency asymptotic level
- **Flicker Noise Corner ($f_c$):** Frequency where $1/f$ noise equals thermal noise
- **Total Integrated RMS Noise ($V_{n,RMS}$):** In $\mu\text{V}_{\text{RMS}}$ integrated from $f_L$ to $f_H$

---

## 3. Applicable Amplifiers
- **Class A:** Low-noise cascode front-ends
- **Class B:** Differential pairs, 5-T OTAs
- **Class C:** Folded-cascode, telescopic, and two-stage Miller op-amps
- **Class D:** Fully differential low-noise amplifiers

---

## 4. Testbench Schematic

```text
               VDD
                │
                ▼
        ┌───────────────┐
VIN+ ───┤+            + ├─── VOUTP ───┬─── CL = CL,nom
(AC=1)  │   Amplifier   │             │
VIN- ───┤-            - ├─── VOUTM ───┴─── Output Noise: en,out(f)
(AC=0)  └───────┬───────┘
                │
                ▼
               VSS
Input-Referred Noise: en,in(f) = en,out(f) / |Ad(f)|
```

![TB11 Noise](assets/TB11_Noise.png)
*Figure 1: Standardized small-signal noise analysis testbench.*

---

## 5. Circuit Configuration
1. **Quiescent Conditions:** Bias the circuit at nominal operating point ($V_{DD}, V_{SS}, V_{CM}$).
2. **Noise Reference Source:** Nominate input differential voltage source $V_{id}$ as the input noise reference.
3. **Output Sense:** Define output differential pair `(VOUTP, VOUTM)` (or single-ended output) as the noise measurement port.
4. **Load:** Connect specified load capacitance $C_L$ and load resistance $R_L$.

---

## 6. Simulation Type
- **SPICE Directive:** `.noise V(outp, outm) VIN <dec> <f_start> <f_stop>`
- **Frequency Range:** Typically `0.1 Hz` to `100 MHz` to capture both deep sub-audio flicker noise and thermal noise floor.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB11
VINP inp 0 DC=0.9 AC=1.0
VINM inm 0 DC=0.9 AC=0.0
XDUT inp inm outp outm vdd vss amplifier_core
CLOAD outp outm 2pF

* Noise directive
.noise V(outp, outm) VINP dec 50 0.1 100Meg
```
1. Run DC operating point.
2. Calculate device noise contributions (thermal noise of channel, flicker noise, gate resistance noise).
3. Compute total output noise and reflect back to input by dividing by transfer function.

---

## 8. Calculations
### Input-Referred Noise Spectral Density
$$
e_{n,in}(f) = \frac{e_{n,out}(f)}{|A_d(f)|} \quad \left[\frac{\text{V}}{\sqrt{\text{Hz}}}\right]
$$

### Integrated Input-Referred RMS Noise
$$
V_{n,RMS} = \sqrt{\int_{f_L}^{f_H} e_{n,in}^2(f) \, df} \quad [\text{V}_{\text{RMS}}]
$$
*Mandatory Rule:* Always record the integration bandwidth limits $[f_L, f_H]$ alongside the numerical result (e.g. $0.1\,\text{Hz}$ to $10\,\text{Hz}$ for biomedical, $20\,\text{Hz}$ to $20\,\text{kHz}$ for audio).

### Flicker Corner Frequency ($f_c$)
Defined at the intersection:
$$
e_{n,\text{flicker}}^2(f_c) = e_{n,\text{thermal}}^2
$$

---

## 9. Required Plots
- **Input-Referred Noise Bode Plot:** $e_{n,in}(f)$ in $\text{nV}/\sqrt{\text{Hz}}$ vs. logarithmic frequency.
- **Noise Contribution Breakdown:** Bar chart or pie chart of top contributing transistors ($M_1, M_2, M_3\dots$) at $1\,\text{kHz}$ and $1\,\text{MHz}$.
- **Cumulative Integrated Noise:** $\sqrt{\int_{f_L}^f e_{n,in}^2(\xi) d\xi}$ vs. frequency.

---

## 10. Measurements to Save
```spice
.meas NOISE en_in_1k FIND inoise AT=1k
.meas NOISE en_in_floor FIND inoise AT=10Meg
.meas NOISE vn_rms INTEG inoise FROM=10 TO=100k
* Placeholder criteria:
* en_in_1k <= DESIGN_SPEC_NOISE_1K
* vn_rms <= DESIGN_SPEC_INTEG_NOISE
```

---

## 11. PVT Considerations
- **Thermal Noise:** Proportional to absolute temperature ($4kT\gamma / g_m$). At **$125^\circ\text{C}$**, thermal noise increases substantially.
- **Flicker Noise:** Dependent on process oxide thickness $t_{ox}$ and gate area $W \cdot L$. In **SS corners**, $1/f$ noise often shifts upward due to trap density variations.

---

## 12. Post-Layout Considerations
- **Gate Resistance ($R_g$):** Un-interdigitated transistor layouts introduce significant distributed polysilicon gate resistance thermal noise ($4kT R_g$). Multi-finger transistor layouts are mandatory for low-noise designs.
- **Well & Substrate Taps:** High-impedance substrate contacts pick up digital switching noise that appears as output noise.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 7: Noise (Thermal Noise, Flicker Noise, Input-Referred Noise). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)
- **P. R. Gray and R. G. Meyer**, "MOS Operational Amplifier Design — A Tutorial Overview," *IEEE J. Solid-State Circuits*, 1982.

---

## 14. Related Testbenches
- **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Transfer function used for input noise referral.
- **[TB12: THD](TB12_THD.md)** — Establishes the dynamic range ($DR = \frac{V_{swing,RMS}}{V_{n,RMS}}$) alongside noise.
- **[TB18: Monte Carlo](TB18_Monte_Carlo.md)** — Evaluates noise floor variations across device mismatch.

---

## 15. Navigation

| [<< TB10: Transient](TB10_Transient.md) | [Docs Index](index.md) | [TB12: THD >>](TB12_THD.md) |
