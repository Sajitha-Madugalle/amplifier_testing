[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | **[TB01: AC](TB01_AC_Gain.md)** | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB01 — Open-Loop AC Gain

Small-signal frequency-domain characterization of single-ended gain stages, cascode architectures, and open-loop response.

---

## 1. Purpose
- Extract the low-frequency DC voltage gain ($A_0$ or $A_{v0}$).
- Identify the dominant pole frequency and determine the $-3\,\text{dB}$ open-loop bandwidth ($f_{-3\text{dB}}$).
- Determine the Unity-Gain Frequency (UGF / $f_u$).
- Characterize high-frequency secondary poles ($p_2, p_3$) and zeros ($z_1$) in single-ended amplification stages.

---

## 2. Parameters Measured
- **DC Voltage Gain ($A_0$):** In $\text{V/V}$ and decibels ($\text{dB}$)
- **$-3\,\text{dB}$ Bandwidth ($f_{-3\text{dB}}$):** Half-power cut-off frequency
- **Unity-Gain Frequency (UGF / $f_u$):** Frequency where $|A_v(f)| = 0\,\text{dB}$
- **Phase at UGF ($\angle A_v(f_u)$):** Open-loop phase roll-off
- **Gain-Bandwidth Product (GBW):** $A_0 \times f_{-3\text{dB}}$ (for single-pole roll-off systems)

---

## 3. Applicable Amplifiers
- **Class A:** Single-ended cascode, wide-swing cascode, regulated cascode stages
- **Class B/C:** Open-loop unbuffered gain blocks where feedback loop is evaluated separately

---

## 4. Testbench Schematic

```text
               VDD
                │
                ▼
        ┌───────────────┐
        │  Single-Ended │
Vin ────┤     Stage     ├──── Vout ───┬─── CL = CL,nom
        │   Amplifier   │             │
        └───────┬───────┘             └─── RL = RL,nom
                │
                ▼
               VSS
```

![TB01 AC Gain](assets/TB01_AC_Gain.png)
*Figure 1: Single-ended open-loop AC test configuration with output RC loading.*

---

## 5. Circuit Configuration
- **Input Source:** $V_{in} = V_{bias} + v_{ac}$, where $v_{ac} = 1\,\text{V}$ (for direct normalization in SPICE).
- **DC Bias:** Maintain precisely the nominal gate bias established in [TB00](TB00_DC_Operating_Point.md).
- **Output Termination:** Connect specified nominal load:
  $$C_L = C_{L,nom}, \quad R_L = R_{L,nom}$$

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>`
- **Sweep Range:** Typically `1 Hz` to `10 GHz` with at least 50 points/decade to resolve resonant peaking or narrow pole-zero doublets.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB01
VIN in 0 DC=0.75V AC=1.0V
CLOAD out 0 1pF
RLOAD out 0 100k

* Subcircuit call
X1 in out vdd vss cascode_stage

* AC Sweep Directive
.ac dec 50 1 10G
```
1. Verify operating point solution before AC linearization.
2. Sweep frequency across logarithmic decade intervals.
3. Record magnitude vector `vdb(out)` and phase vector `vp(out)`.

---

## 8. Calculations
### Open-Loop Voltage Gain
$$
A_v(f) = \frac{V_{out}(f)}{V_{in}(f)}
$$
When $V_{in,AC} = 1\,\text{V}$:
$$
A_v(f) = V_{out}(f), \quad A_{v,\text{dB}}(f) = 20\log_{10}|V_{out}(f)|
$$

### $-3\,\text{dB}$ Bandwidth
Find frequency $f_{-3\text{dB}}$ satisfying:
$$
20\log_{10}|A_v(f_{-3\text{dB}})| = A_{0,\text{dB}} - 3.01\,\text{dB}
$$

### Unity-Gain Frequency
Find frequency $f_u$ satisfying:
$$
20\log_{10}|A_v(f_u)| = 0\,\text{dB}
$$

---

## 9. Required Plots
- **Gain Bode Plot:** Magnitude $20\log_{10}|A_v(f)|$ (dB) vs. Frequency (log scale, Hz).
- **Phase Bode Plot:** Phase $\angle A_v(f)$ (degrees) vs. Frequency (log scale, Hz).
- **Poles & Zeros Extraction Map:** Root locus or list of extracted s-domain roots.

---

## 10. Measurements to Save
```spice
.meas AC A0 MAX vdb(out)
.meas AC f_3db WHEN vdb(out) = 'A0 - 3'
.meas AC UGF WHEN vdb(out) = 0
.meas AC phase_ugf FIND vp(out) WHEN vdb(out) = 0
* Placeholder criteria:
* A0 >= DESIGN_SPEC_GAIN
* UGF >= DESIGN_SPEC_UGF
```

---

## 11. PVT Considerations
- Under **SS (Slow-Slow)** corner and **high temperature ($125^\circ\text{C}$)**, carrier mobility ($\mu$) and transconductance ($g_m$) decrease, lowering $A_0$ and shifting $f_u$ downwards.
- Output junction capacitances increase at low $V_{DD}$, pulling the dominant pole to lower frequencies.

---

## 12. Post-Layout Considerations
- **Parasitic Capacitances ($C_{gd}, C_{db}$):** Extracted routing capacitance at the high-impedance output node substantially reduces $f_{-3\text{dB}}$ and UGF compared to schematic predictions.
- Compare schematic vs. PEX:
  $$\Delta UGF_{\%} = \frac{UGF_{\text{PEX}} - UGF_{\text{schematic}}}{UGF_{\text{schematic}}} \times 100\%$$

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 3: Single-Stage Amplifiers; Chapter 6: Frequency Response of Amplifiers. [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB00: DC Operating Point](TB00_DC_Operating_Point.md)** — Sets input bias voltage.
- **[TB02: Differential Gain](TB02_Differential_Gain.md)** — Differential counterpart of AC gain.
- **[TB03: Stability](TB03_Stability.md)** — Evaluates closed-loop feedback stability using loop gain.
- **[TB16: Load Sweep](TB16_Load_Sweep.md)** — AC gain sensitivity across load variations ($C_L$).

---

## 15. Navigation

| [<< TB00: DC Operating Point](TB00_DC_Operating_Point.md) | [Docs Index](index.md) | [TB02: Differential Gain >>](TB02_Differential_Gain.md) |
