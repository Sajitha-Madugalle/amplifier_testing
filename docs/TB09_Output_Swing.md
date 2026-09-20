[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | **[TB09: Swing](TB09_Output_Swing.md)** | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB09 — Output Voltage Swing

Characterization of maximum and minimum output voltage limits ($V_{out,max}$, $V_{out,min}$) and linear output dynamic range before saturation or compression occurs.

---

## 1. Purpose
- Extract the maximum positive output voltage level ($V_{out,max}$).
- Extract the minimum negative output voltage level ($V_{out,min}$).
- Determine total linear output swing span ($V_{swing} = V_{out,max} - V_{out,min}$).
- Identify output stage overdrive limitations and gain compression points ($1\,\text{dB}$ compression).

---

## 2. Parameters Measured
- **Maximum Output Voltage ($V_{out,max}$):** Upper saturation boundary
- **Minimum Output Voltage ($V_{out,min}$):** Lower saturation boundary
- **Peak-to-Peak Output Swing ($V_{swing,pp}$):** Full voltage range
- **Headroom to Rails:** $V_{DD} - V_{out,max}$ and $V_{out,min} - V_{SS}$

---

## 3. Applicable Amplifiers
- **Class A:** Single-ended cascode, wide-swing cascode stages
- **Class B/C:** Operational amplifiers with unbuffered or push-pull output stages
- **Class D:** Fully differential operational amplifiers (differential swing $V_{od,swing} = 2 \times V_{swing,single}$)

---

## 4. Testbench Schematic

```text
               Closed-Loop Inverting Configuration (ACL = -10)
                          RF = 100k
                       ┌───/\/\/\───┐
                       │            │
             R1 = 10k  │   ┌────┐   │
   Vin_DC ────/\/\/\───┴───┤-   │   │
                           │DUT ├───┴──── Vout ──── RL, CL
   Vref ───────────────────┤+   │
                           └────┘
```

![TB09 Output Swing](assets/TB09_Output_Swing.png)
*Figure 1: High closed-loop gain test configuration driving output stage into saturation.*

---

## 5. Circuit Configuration
1. **Feedback Configuration:** Configure amplifier in a closed-loop inverting gain configuration with gain $|A_{CL}| \gg 1$ (e.g. $A_{CL} = -R_F/R_1 = -10$).
   *Rationale:* High closed-loop gain forces the output to reach its physical headroom limits while keeping the input terminal within a very small common-mode range.
2. **Reference:** Bias non-inverting input at nominal reference voltage $V_{ref} = V_{CM,nom}$.
3. **Sweep Source:** Sweep input DC voltage $V_{in}$ across a range calculated to drive the output rail-to-rail.
4. **Load:** Connect specified minimum resistive load $R_{L,min}$ and nominal capacitive load $C_L$.

---

## 6. Simulation Type
- **SPICE Directive:** `.dc VIN <v_start> <v_stop> <v_step>`
- **Derivative Extraction:** Compute $\frac{dV_{out}}{dV_{in}}$ to detect compression slope.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB09
.param VCM=0.9
VREF ref 0 DC=VCM
VIN in 0 DC=VCM
R1 in nin 10k
RF nin out 100k
RLOAD out 0 25k
CLOAD out 0 5pF

XDUT ref nin out vdd vss opamp_core

* Sweep input around nominal point
.dc VIN 0.7 1.1 0.5m
```
1. Perform DC sweep.
2. Differentiate output: $A_{gain}(V_{in}) = \frac{d(V_{out})}{d(V_{in})}$.
3. Define swing limits at the point where incremental gain drops by $10\%$ or $1\,\text{dB}$ from nominal $|A_{CL}|$.

---

## 8. Calculations
### Total Peak-to-Peak Output Swing
$$
V_{swing} = V_{out,max} - V_{out,min}
$$

### Fully Differential Equivalent Differential Swing
$$
V_{swing,diff} = 2 \times \left(V_{out,max} - V_{out,min}\right)
$$

### Rail Headroom Margins
$$
\Delta V_{headroom,high} = V_{DD} - V_{out,max}
$$
$$
\Delta V_{headroom,low} = V_{out,min} - V_{SS}
$$

---

## 9. Required Plots
- **DC Voltage Transfer Curve:** $V_{out}$ vs. $V_{in}$, clearly showing top and bottom clipping.
- **Incremental Gain Curve:** $\frac{dV_{out}}{dV_{in}}$ vs. $V_{out}$, illustrating linear plateau and compression knees.
- **Transistor Overdrive vs. Swing:** Tracking $V_{DS} - V_{DSAT}$ of output pull-up and pull-down devices.

---

## 10. Measurements to Save
```spice
.meas DC Vout_min MIN v(out)
.meas DC Vout_max MAX v(out)
.meas DC Vswing PARAM = 'Vout_max - Vout_min'
.meas DC Headroom_high PARAM = '1.8 - Vout_max'
.meas DC Headroom_low PARAM = 'Vout_min - 0.0'
* Placeholder criteria:
* Vswing >= DESIGN_SPEC_SWING
```

---

## 11. PVT Considerations
- In cascode output stages, $V_{out,min} = V_{DSAT1} + V_{DSAT2}$. At the **SS corner and low temperature**, higher $V_{th}$ and lower mobility widen required saturation voltages, shrinking $V_{swing}$.
- Under heavy resistive loading ($R_L$), output stage current capability limits the voltage swing before device saturation limits are reached.

---

## 12. Post-Layout Considerations
- **Output Metal Widths:** High current output stages driving resistive loads cause voltage drops along narrow metal routes, directly reducing effective rail-to-rail swing.
- Ensure sufficient via arrays between metal layers on output stage drains.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 3: Cascode Stages (Output Swing Limitations); Chapter 9: Operational Amplifiers (Output Stage Design). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB08: ICMR](TB08_ICMR.md)** — Evaluates complementary input voltage range.
- **[TB10: Transient](TB10_Transient.md)** — Verifies dynamic slewing across the full swing.
- **[TB12: THD](TB12_THD.md)** — Quantifies distortion near the swing limits.

---

## 15. Navigation

| [<< TB08: ICMR](TB08_ICMR.md) | [Docs Index](index.md) | [TB10: Transient >>](TB10_Transient.md) |
