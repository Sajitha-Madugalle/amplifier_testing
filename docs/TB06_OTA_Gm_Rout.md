[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | **[TB06: Gm/Rout](TB06_OTA_Gm_Rout.md)** | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB06 — OTA Transconductance & Output Resistance

Direct extraction of small-signal short-circuit transconductance $G_m(f)$, small-signal output resistance $R_o(f)$, and intrinsic low-frequency gain $A_0 \approx G_m R_o$.

---

## 1. Purpose
- Directly measure the short-circuit small-signal transconductance $G_m(f)$ of OTAs and gain stages.
- Extract the dynamic output impedance $Z_o(f)$ and low-frequency output resistance $R_o$.
- Verify the fundamental product relation $A_0 = G_m R_o$.
- Determine the pole location formed by $R_o$ and internal/load capacitances.

---

## 2. Parameters Measured
- **Low-Frequency Transconductance ($G_{m0}$):** In Siemens ($\text{S}$ or $\text{mA/V}$)
- **Transconductance Bandwidth ($f_{-3\text{dB},Gm}$):** Frequency where $G_m$ drops by $3\,\text{dB}$
- **DC Output Resistance ($R_o$):** In Ohms ($\Omega$ or $\text{M}\Omega$)
- **Output Capacitance ($C_o$):** Extracted from high-frequency $Z_o(f)$ roll-off
- **Intrinsic DC Voltage Gain ($A_0$):** $G_m R_o$ product

---

## 3. Applicable Amplifiers
- **Class B:** 5-T OTAs, symmetrical current-mirror OTAs, differential pairs
- **Class C:** Telescopic OTAs, folded-cascode OTAs, gain-boosted OTAs
- **Class D:** Fully differential OTAs (differential $G_m$ and $R_{od}$)

---

## 4. Testbench Schematic

```text
       (1) Gm Setup (Short-Circuit Output)       (2) Rout Setup (Current Injection)
         ┌───────────────┐                          ┌───────────────┐
Vid ─────┤+            + ├─── I_out                 │+            + ├─── Vo
(AC=1V)  │      OTA      │      │            Inputs │      OTA      │    │
         │-            - ├───   ▼ V_out_nom   AC=0  │-            - ├─── ▲ Itest = 1A_AC
         └───────────────┘                          └───────────────┘
          Measure: I_out                             Measure: Vo -> Ro(f)
```

![TB06 OTA Gm Rout](assets/TB06_OTA_Gm_Rout.png)
*Figure 1: Direct extraction schemes for transconductance and output impedance.*

---

## 5. Circuit Configuration
1. **Transconductance Setup ($G_m$):**
   - Apply normalized differential input: $V_{id} = 1.0\,\text{V}_{AC}$.
   - Hold output pin(s) at nominal DC operating voltage using an ideal DC voltage source ($V_{\text{sense}}$) acting as an AC short circuit to ground.
   - Measure AC current $I(V_{\text{sense}})$.
2. **Output Resistance Setup ($R_o$):**
   - AC-ground all input signal pins ($V_{id,AC} = 0$).
   - Connect an ideal test current source at the output: $I_{test} = 1.0\,\text{A}_{AC}$ in parallel with an AC-choke inductor or DC bias source.
   - Measure resulting AC voltage $V_o$.

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>`
- **Sweep Range:** `0.1 Hz` to `10 GHz` with 50 points/decade.

---

## 7. Simulation Procedure
```spice
* Gm Extraction
VINP inp 0 DC=0.9 AC=0.5
VINM inm 0 DC=0.9 AC=0.5 180
VSENSE outp_gm 0 DC=0.9 AC=0.0
XDUT_GM inp inm outp_gm outm_dummy vdd vss ota_core

* Rout Extraction
VINP2 inp2 0 DC=0.9 AC=0.0
VINM2 inm2 0 DC=0.9 AC=0.0
ITEST 0 outp_ro DC=0.0 AC=1.0
XDUT_RO inp2 inm2 outp_ro outm_dummy vdd vss ota_core

.ac dec 50 0.1 10G
```
1. Verify operating points are identical in both simulations.
2. Run AC frequency sweep.
3. Extract currents and voltages.

---

## 8. Calculations
### Transconductance
$$
G_m(f) = \frac{I_{out}(f)}{V_{id}(f)}
$$
When $V_{id} = 1\,\text{V}_{AC}$, numerically $G_m(f) = |I_{out}(f)|$. Low-frequency value: $g_m = G_m(0)$.

### Output Resistance
$$
R_o(f) = \frac{V_o(f)}{I_{test}(f)}
$$
When $I_{test} = 1\,\text{A}_{AC}$, numerically $R_o(f) = |V_o(f)|$.

### Intrinsic Gain Verification
$$
A_{0,\text{calc}} = G_m(0) \times R_o(0)
$$
Compare with $A_{d0}$ measured directly in [TB02](TB02_Differential_Gain.md).

---

## 9. Required Plots
- **Transconductance Frequency Response:** $G_m(f)$ in $\text{mA/V}$ vs. logarithmic frequency.
- **Output Impedance Magnitude:** $|Z_o(f)|$ in $\text{dB}\Omega$ or $\text{k}\Omega$ vs. logarithmic frequency.
- **Transconductance vs. Input Common-Mode:** $G_m(V_{CM})$ swept across input common-mode range (especially for complementary input pairs).

---

## 10. Measurements to Save
```spice
.meas AC Gm0 FIND mag(i(vsense)) AT=1
.meas AC Ro0 FIND mag(v(outp_ro)) AT=1
.meas AC A0_product PARAM = 'Gm0 * Ro0'
.meas AC A0_product_dB PARAM = '20 * log10(A0_product)'
* Placeholder criteria:
* Gm0 >= DESIGN_SPEC_GM
* Ro0 >= DESIGN_SPEC_RO
```

---

## 11. PVT Considerations
- In cascode OTAs, $R_o \approx g_{m,casc} r_{o,casc} r_{o,main}$. In the **FF corner and at high temperatures**, $r_o$ decreases drastically due to lower channel resistance, lowering $R_o$ by up to $50\%$.
- $G_m$ decreases monotonically with temperature due to mobility degradation ($\mu \propto T^{-1.5}$).

---

## 12. Post-Layout Considerations
- **Output Routing Resistance:** Series metal trace resistance adds directly to load lines, but is negligible compared to high $R_o$ in OTAs.
- **Parasitic Output Capacitance ($C_{par}$):** Layout parasitics at the output node appear directly in parallel with $R_o$, lowering the output pole frequency $f_p = \frac{1}{2\pi R_o (C_L + C_{par})}$.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 4: Differential Amplifiers; Chapter 9: Operational Amplifiers (OTA topologies). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Relates to $A_d = G_m R_o$.
- **[TB08: ICMR](TB08_ICMR.md)** — Evaluates $G_m(V_{CM})$ across the common-mode window.
- **[TB16: Load Sweep](TB16_Load_Sweep.md)** — Determines dominant pole sensitivity to external $C_L$.

---

## 15. Navigation

| [<< TB05: PSRR](TB05_PSRR.md) | [Docs Index](index.md) | [TB07: Offset >>](TB07_Input_Offset.md) |
