[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | **[TB14: CMFB](TB14_CMFB.md)** | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB14 — Common-Mode Feedback (CMFB) Characterization

Stability, loop gain, regulation precision, and transient settling characterization of Common-Mode Feedback (CMFB) networks in fully differential amplifiers.

---

## 1. Purpose
- Measure quiescent output common-mode voltage ($V_{OCM}$) and static common-mode error ($V_{CM,error} = V_{OCM} - V_{CMREF}$).
- Extract small-signal CMFB loop gain $T_{CM}(j\omega)$, unity-gain bandwidth ($UGF_{CM}$), phase margin ($PM_{CM}$), and gain margin ($GM_{CM}$).
- Characterize common-mode transient recovery and settling time when disturbed by large common-mode steps.
- Ensure the CMFB loop bandwidth is comparable to the differential-mode bandwidth to prevent common-mode ringing.

---

## 2. Parameters Measured
- **Static Output Common-Mode Voltage ($V_{OCM}$):** In volts ($\text{V}$)
- **Static Common-Mode Error ($V_{CM,error}$):** In millivolts ($\text{mV}$)
- **CMFB Low-Frequency Loop Gain ($T_{CM0}$):** In decibels ($\text{dB}$)
- **CMFB Unity-Gain Frequency ($UGF_{CM}$):** In $\text{MHz}$
- **CMFB Phase Margin ($PM_{CM}$):** Phase margin in degrees
- **CMFB Settling Time ($t_{s,CM}$):** Transient response to common-mode step

---

## 3. Applicable Amplifiers
- **Class D:** Fully differential telescopic, folded-cascode, and two-stage amplifiers with continuous-time (resistive/MOS-based) or switched-capacitor CMFB networks

---

## 4. Testbench Schematic

```text
               CMFB Loop Stability Testbench
               VDD
                │
                ▼
        ┌───────────────┐
VCM_IN ─┤+            + ├─── VOUTP ───┬───┐
        │  Fully Diff   │             │   ├─── [ CMFB Sensing ]
VCM_IN ─┤-  Amplifier - ├─── VOUTM ───┴───┘            │
        │               │                              ▼
        │   Bias Ctrl   │<── [ Loop Probe ] ─── [ CMFB Error Amp ]
        └───────┬───────┘                              ▲
                │                                      │
                ▼                                   VCMREF
               VSS
```

![TB14 CMFB](assets/TB14_CMFB.png)
*Figure 1: Non-invasive loop probe insertion inside the common-mode feedback control loop.*

---

## 5. Circuit Configuration
1. **Differential Inputs:** Tie differential input terminals together to the nominal input common-mode voltage ($v_{in+} = v_{in-} = V_{CM,nom}$) to suppress differential mode excitation.
2. **Loop Probe Insertion:** Break the common-mode feedback loop at the high-impedance control point (e.g. at the gate of the tail current source or error amplifier output) with an AC loop probe or Middlebrook injection element.
3. **Reference Voltage:** Connect $V_{CMREF}$ to target output common-mode voltage ($V_{DD}/2$).
4. **Load:** Connect nominal single-ended capacitive loads from $V_{OUTP}$ and $V_{OUTM}$ to ground.

---

## 6. Simulation Type
- **Directives:**
  - `.op` (Static common-mode error extraction)
  - `.ac DEC <points_per_decade> <f_start> <f_stop>` (CMFB loop stability)
  - `.tran <t_step> <t_stop>` (Common-mode transient step)

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB14
VCMREF vcmref 0 DC=0.9
VIN_CM inp 0 DC=0.9 AC=0
VIN_CMM inm 0 DC=0.9 AC=0

* Break control wire with AC injection source
Vicmfb cmfb_sense cmfb_ctrl DC=0 AC=1

XDUT inp inm outp outm cmfb_ctrl vcmref vdd vss fd_amplifier

.ac dec 50 1 10G
```
1. Solve DC operating point and record node voltages `V(outp)` and `V(outm)`.
2. Execute AC sweep to measure the return ratio of the CMFB loop.
3. Apply a $200\,\text{mV}$ step pulse to $V_{CMREF}$ in transient analysis to observe settling.

---

## 8. Calculations
### Output Common-Mode Voltage
$$
V_{OCM} = \frac{V_{out+} + V_{out-}}{2}
$$

### Static Common-Mode Regulation Error
$$
V_{CM,error} = V_{OCM} - V_{CMREF}
$$

### CMFB Phase Margin
At frequency $\omega_{u,CM}$ where $|T_{CM}(j\omega)| = 1$ ($0\,\text{dB}$):
$$
PM_{CM} = 180^\circ + \angle T_{CM}(j\omega_{u,CM})
$$
*Criteria:* Target $PM_{CM} \ge 60^\circ$ to prevent common-mode ringing during large-signal transients.

---

## 9. Required Plots
- **CMFB Loop Gain Bode Plot:** Magnitude $20\log_{10}|T_{CM}(j\omega)|$ and Phase $\angle T_{CM}(j\omega)$ vs. frequency.
- **Common-Mode Transient Step:** $V_{OCM}(t)$ following a step change on $V_{CMREF}$.
- **DC Output Common-Mode vs. Supply Voltage:** $V_{OCM}$ vs. $V_{DD}$ showing regulation across supply variations.

---

## 10. Measurements to Save
```spice
.meas OP Vocm_dc PARAM = '(v(outp) + v(outm))/2'
.meas OP Vcm_err PARAM = 'Vocm_dc - 0.9'
.meas AC fug_cm WHEN vdb(loop_cm) = 0
.meas AC PM_cm PARAM = '180 + vp(loop_cm)'
* Placeholder criteria:
* abs(Vcm_err) <= DESIGN_SPEC_VCM_ERR
* PM_cm >= DESIGN_SPEC_PM_CM
```

---

## 11. PVT Considerations
- In two-stage fully differential amplifiers, the CMFB loop often contains two gain stages in the common-mode path, requiring dedicated common-mode compensation capacitors.
- In **SS corners**, CMFB transconductance drops, eroding common-mode bandwidth and slowing down CM recovery.

---

## 12. Post-Layout Considerations
- **Parasitic Capacitance on Control Node:** The high-impedance gate node controlling the tail current source is vulnerable to routing capacitance, which introduces an unexpected dominant pole into the CMFB loop and reduces phase margin.
- Verify parasitic extraction on the `cmfb_ctrl` net.

---

## 13. References
- **P. J. Hurst and S. H. Lewis**, "Determination of Stability Using Return Ratios in Balanced Fully Differential Feedback Circuits," *IEEE Trans. Circuits Syst. II*, 1995. [DOI Link](https://doi.org/10.1109/82.476178)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 9: Operational Amplifiers (Common-Mode Feedback). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB13: FD DM Mode](TB13_FD_Differential_Mode.md)** — Evaluates the complementary differential loop.
- **[TB04: CMRR](TB04_CMRR.md)** — Relies directly on CMFB regulation gain.
- **[TB09: Output Swing](TB09_Output_Swing.md)** — Output swing is centered around $V_{OCM}$.

---

## 15. Navigation

| [<< TB13: FD DM Mode](TB13_FD_Differential_Mode.md) | [Docs Index](index.md) | [TB15: Local Loops >>](TB15_Local_Loop_Stability.md) |
