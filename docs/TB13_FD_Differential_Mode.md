[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | **[TB13: FD-DM](TB13_FD_Differential_Mode.md)** | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB13 — Fully Differential Differential-Mode Characterization

Dedicated characterization of the pure differential signal path in balanced fully differential operational amplifiers and OTAs with active common-mode feedback (CMFB).

---

## 1. Purpose
- Extract small-signal differential-mode loop gain $T_{DM}(j\omega)$, unity-gain frequency ($f_{u,DM}$), and phase margin ($PM_{DM}$).
- Verify closed-loop differential transient step response ($V_{od}(t) = V_{out+} - V_{out-}$).
- Confirm that common-mode feedback circuitry does not disturb the differential signal transfer or introduce parasitic pole-zero doublets into the differential path.
- Monitor output common-mode stability ($V_{ocm}$) during large-signal differential swings.

---

## 2. Parameters Measured
- **Differential-Mode DC Gain ($A_{d0}$):** Open-loop or closed-loop differential gain
- **Differential Unity-Gain Frequency ($f_{u,DM}$):** High-frequency gain intercept
- **Differential Phase Margin ($PM_{DM}$):** Phase margin of the differential loop
- **Differential Settling Time ($t_{s,diff}$):** Large- and small-signal step response
- **Dynamic Common-Mode Perturbation ($\Delta V_{ocm}$):** Transient variation in output CM

---

## 3. Applicable Amplifiers
- **Class D:** Fully differential telescopic, folded-cascode, and two-stage amplifiers with continuous-time or switched-capacitor CMFB networks

---

## 4. Testbench Schematic

```text
               Fully Differential Closed-Loop Testbench
               VDD
                │
                ▼
        ┌───────────────┐
VIP ────┤+            + ├──── VOP ───┬─── [ Diff Loop Probe ] ─── Feedback
(AC=+0.5)│ Fully Diff   │            │
VIN ────┤-   Amplifier- ├──── VON ───┴─── [ Diff Loop Probe ] ─── Feedback
(AC=-0.5)│   with CMFB  │
        └───────┬───────┘
                │
                ▼
               VSS
Measure: Vod = VOP - VON    Monitor: Vocm = (VOP + VON) / 2
```

![TB13 Fully Differential](assets/TB13_FD_Differential_Mode.png)
*Figure 1: Fully differential signal-path characterization setup with differential loop probe.*

---

## 5. Circuit Configuration
1. **Differential Feedback Network:** Configure DUT with matched symmetrical differential feedback networks (e.g. resistive feedback $R_2/R_1$ or capacitive feedback $C_1/C_2$).
2. **Differential Injection Probe:** Insert a balanced differential return-ratio probe (Hurst & Lewis probe) across both feedback lines simultaneously to measure pure differential loop gain.
3. **Common-Mode Control:** Connect CMFB reference terminal $V_{CMREF}$ to target output common-mode voltage ($V_{DD}/2$).
4. **Balanced Loading:** Terminate outputs with balanced differential load capacitors $C_{L,diff}$ and single-ended load capacitors to AC ground.

---

## 6. Simulation Type
- **SPICE Directives:**
  - `.ac DEC <points_per_decade> <f_start> <f_stop>` (Differential AC loop gain)
  - `.tran <t_step> <t_stop>` (Differential step response)

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB13
.param VCM=0.9
VIP inp 0 DC=VCM AC=0.5
VIN inm 0 DC=VCM AC=0.5 180
VCMREF vcmref 0 DC=0.9

* Fully differential amplifier instantiation
XDUT inp inm outp outm vcmref vdd vss fd_opamp

* Balanced differential load
CLP outp 0 2pF
CLM outm 0 2pF
CL_DIFF outp outm 5pF

.ac dec 50 1 10G
```
1. Verify operating points for both signal paths and the CMFB control path.
2. Measure differential open-loop and closed-loop transfer responses.
3. Confirm that output common-mode voltage remains locked at $V_{CMREF}$ while inputs are swept differentially.

---

## 8. Calculations
### Differential Output Signal
$$
V_{od}(t) = V_{out+}(t) - V_{out-}(t)
$$

### Output Common-Mode Monitoring
$$
V_{ocm}(t) = \frac{V_{out+}(t) + V_{out-}(t)}{2}
$$

### Differential Phase Margin
At frequency $\omega_{u,DM}$ where $|T_{DM}(j\omega)| = 1$:
$$
PM_{DM} = 180^\circ + \angle T_{DM}(j\omega_{u,DM})
$$

---

## 9. Required Plots
- **Differential Loop Gain Bode Plot:** $|T_{DM}(j\omega)|$ (dB) and $\angle T_{DM}(j\omega)$ vs. frequency.
- **Differential Transient Step:** $V_{od}(t)$ responding to positive and negative differential input steps.
- **Dynamic Common-Mode Cross-Coupling:** $V_{ocm}(t)$ plotted on the same time scale to verify rejection of differential transients.

---

## 10. Measurements to Save
```spice
.meas AC Tdm_dc FIND vdb(loop_gain_dm) AT=1
.meas AC f_ug_dm WHEN vdb(loop_gain_dm) = 0
.meas AC PM_dm PARAM = '180 + vp(loop_gain_dm)'
.meas TRAN Vocm_drift MAX abs(v(ocm) - 0.9)
* Placeholder criteria:
* PM_dm >= DESIGN_SPEC_PM
* Vocm_drift <= DESIGN_SPEC_VOCM_TOL
```

---

## 11. PVT Considerations
- In two-stage fully differential amplifiers, separate Miller capacitors are required on both branches. Mismatch between branches in corner simulations can produce odd-mode/even-mode phase skew.
- Verify that common-mode feedback devices do not leave saturation during extreme PVT corners.

---

## 12. Post-Layout Considerations
- **Layout Symmetry & Cross-Coupling:** The physical layout of the positive and negative half-circuits must be mirrors or translated replicas. Routing capacitance imbalances will convert differential signals into common-mode transients.
- Check extracted netlist parasitic capacitance: $|C_{par,outp} - C_{par,outm}| \le 2\%$.

---

## 13. References
- **P. J. Hurst and S. H. Lewis**, "Determination of Stability Using Return Ratios in Balanced Fully Differential Feedback Circuits," *IEEE Trans. Circuits Syst. II*, 1995. [DOI Link](https://doi.org/10.1109/82.476178)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 9: Operational Amplifiers (Fully Differential Topologies).

---

## 14. Related Testbenches
- **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Basic small-signal differential response.
- **[TB14: CMFB](TB14_CMFB.md)** — Complementary characterization of the common-mode loop.
- **[TB12: THD](TB12_THD.md)** — Evaluates HD2 cancellation in fully differential operation.

---

## 15. Navigation

| [<< TB12: THD](TB12_THD.md) | [Docs Index](index.md) | [TB14: CMFB >>](TB14_CMFB.md) |
