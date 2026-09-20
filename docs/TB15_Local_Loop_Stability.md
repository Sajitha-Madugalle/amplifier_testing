[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | **[TB15: Local Loops](TB15_Local_Loop_Stability.md)** | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB15 — Local Loop & Gain-Booster Stability

Stability, phase margin, and frequency response characterization of internal auxiliary loops, gain-boosting amplifiers, and regulated cascode stages.

---

## 1. Purpose
- Characterize the local feedback loop gain $T_{local}(j\omega)$ of auxiliary gain-boosting amplifiers and regulated cascode topologies.
- Extract the local unity-gain frequency ($UGF_{local}$), local phase margin ($PM_{local}$), and local gain margin ($GM_{local}$).
- Prevent high-frequency parasitic oscillation or pole-zero doublet peaking within gain-boosted cascodes.
- Ensure the booster bandwidth is engineered to avoid slowing down main amplifier transient settling ("slow settling doublet" effect).

---

## 2. Parameters Measured
- **Local Low-Frequency Loop Gain ($T_{local,0}$):** In decibels ($\text{dB}$)
- **Local Unity-Loop-Gain Frequency ($UGF_{local}$):** In $\text{MHz}$ or $\text{GHz}$
- **Local Phase Margin ($PM_{local}$):** Phase margin in degrees
- **Local Gain Margin ($GM_{local}$):** In $\text{dB}$
- **Doublet Separation:** Frequency offset between booster bandwidth and main amplifier UGF

---

## 3. Applicable Amplifiers
- **Class A:** Regulated cascode amplifiers
- **Class C:** Gain-boosted telescopic and folded-cascode OTAs, three-stage nested-feedback op-amps
- **Class D:** Fully differential gain-boosted OTAs and op-amps

---

## 4. Testbench Schematic

```text
               Regulated Cascode Local Loop Probe Insertion
                             VDD
                              │
                              ▼
                         [ Cascode FET ]
                              ▲
                              │ Gate
                              ├─── [ Local Loop Probe ] ──┐
                              │                            │
                              │ Drain                     │
                         [ Main FET ]                      │
                              │ Source                     │
                              ├─────────► [ Booster ] ────┘
                              │           (Aux Amp)
                              ▼
                             VSS
```

![TB15 Local Loop Stability](assets/TB15_Local_Loop_Stability.png)
*Figure 1: Internal return-ratio probe insertion at the auxiliary booster output node.*

---

## 5. Circuit Configuration
1. **Operating Point Preservation:** Biasing conditions of the main amplifier must remain completely intact.
2. **Probe Location:** Insert an AC return-ratio probe (Tian / Middlebrook probe) between the booster amplifier output and the gate of the cascode transistor.
   *Rationale:* The high-impedance gate of the cascode device provides an optimal insertion point with minimal reverse loading error.
3. **Multi-Loop Rule:** If multiple gain boosters are present (e.g. NMOS booster and PMOS booster), characterize each independent local loop in separate test instances.

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>` or `.stb`
- **Sweep Range:** `100 Hz` to `50 GHz` (local booster loops often have bandwidths exceeding multi-GHz).

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB15
* Break gate net of cascode transistor M_CASC
Vprobe aux_out casc_gate DC=0 AC=1

* Run AC stability sweep
.ac dec 50 100 50G
```
1. Run DC operating point to confirm all cascode and booster devices are in saturation.
2. Execute small-signal return-ratio sweep.
3. Extract local loop gain magnitude and phase.

---

## 8. Calculations
### Local Loop Return Ratio
$$
T_{local}(j\omega) = - \frac{v_{\text{return}}(j\omega)}{v_{\text{injected}}(j\omega)}
$$

### Local Phase Margin
At frequency $\omega_{u,local}$ where $|T_{local}(j\omega_{u,local})| = 1$:
$$
PM_{local} = 180^\circ + \angle T_{local}(j\omega_{u,local})
$$
*Criteria:* Target $PM_{local} \ge 65^\circ$. Lower phase margin causes severe peaking in overall amplifier AC gain and prolonged ringing in transient step response.

### Local Gain Margin
$$
GM_{local} = - 20\log_{10}|T_{local}(j\omega_{-180^\circ})|
$$

---

## 9. Required Plots
- **Local Loop Bode Plot:** $20\log_{10}|T_{local}|$ and $\angle T_{local}$ vs. logarithmic frequency.
- **Overall Open-Loop Gain Overlay:** Overlaying overall amplifier gain [TB01](TB01_AC_Gain.md) to inspect for pole-zero doublet dips or peaks near $UGF_{local}$.
- **Transient Step Zoom:** Output settling response showing any secondary ringing matching the booster frequency.

---

## 10. Measurements to Save
```spice
.meas AC fug_local WHEN vdb(loop_local) = 0
.meas AC PM_local PARAM = '180 + vp(loop_local)'
.meas AC GM_local FIND vdb(loop_local) WHEN vp(loop_local) = -180
* Placeholder criteria:
* PM_local >= DESIGN_SPEC_PM_LOCAL
* GM_local >= DESIGN_SPEC_GM_LOCAL
```

---

## 11. PVT Considerations
- In **fast corners (FF)**, the auxiliary amplifier's bandwidth increases, pushing $UGF_{local}$ closer to secondary poles and potentially degrading phase margin.
- In **slow corners (SS)**, reduced booster bandwidth can drop below the main amplifier's UGF, creating an active pole-zero doublet that severely degrades settling time.

---

## 12. Post-Layout Considerations
- **Interconnect Capacitance on Cascode Gate:** Long routing lines between the booster output and cascode gate introduce heavy parasitic capacitance $C_{par}$, lowering $UGF_{local}$ and degrading phase margin unless budgeted in design.
- Always place auxiliary booster cells immediately adjacent to their corresponding cascode transistors in layout.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 9: Operational Amplifiers (Gain Boosting Architectures). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)
- **M. Tian, V. Visvanathan, J. Hantgan, and K. Kundert**, "Striving for Small-Signal Stability," *IEEE Circuits and Devices Mag.*, 2001. [PDF Link](https://kenkundert.com/docs/cd2001-01.pdf)

---

## 14. Related Testbenches
- **[TB03: Stability](TB03_Stability.md)** — Evaluates main closed-loop feedback stability.
- **[TB06: Gm/Rout](TB06_OTA_Gm_Rout.md)** — Quantifies output resistance boost $R_o \approx g_{m,casc} r_{o,casc} r_{o,main} \times A_{booster}$.
- **[TB10: Transient](TB10_Transient.md)** — Settling response directly exhibits doublet sluggishness.

---

## 15. Navigation

| [<< TB14: CMFB](TB14_CMFB.md) | [Docs Index](index.md) | [TB16: Load Sweep >>](TB16_Load_Sweep.md) |
