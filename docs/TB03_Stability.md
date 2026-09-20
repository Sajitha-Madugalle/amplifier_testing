[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | **[TB03: Stability](TB03_Stability.md)** | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB03 — Loop Gain & Stability

Characterization of closed-loop negative feedback systems using return-ratio loop probes to extract loop gain, phase margin, and gain margin without breaking DC operating points.

---

## 1. Purpose
- Measure true small-signal feedback loop gain $T(j\omega)$ in the intended closed-loop feedback topology.
- Determine the Unity-Loop-Gain Frequency ($\omega_u$ / $f_{ug}$).
- Measure Phase Margin (PM) to ensure stability against oscillation and excessive ringing.
- Measure Gain Margin (GM) to guarantee stability under loop parameter variations.
- Avoid erroneous DC bias point shifts associated with naive loop-breaking methods.

---

## 2. Parameters Measured
- **Loop Gain ($T_0$):** Low-frequency magnitude of $T(j\omega)$ in $\text{dB}$
- **Unity-Loop-Gain Frequency ($f_{ug}$):** Frequency where $|T(j\omega)| = 1$ ($0\,\text{dB}$)
- **Phase Margin (PM):** Phase margin in degrees
- **Gain Margin (GM):** Gain margin in $\text{dB}$ at the $-180^\circ$ phase crossover frequency ($f_{-180^\circ}$)

---

## 3. Applicable Amplifiers
- **Class C:** Single-stage, two-stage Miller, folded-cascode, and multistage op-amps in closed-loop feedback
- **Class D:** Fully differential op-amps configured in closed-loop differential feedback
- **Class B:** 5-T OTAs operated in buffer or closed-loop instrumentation configurations

---

## 4. Testbench Schematic

```text
               +----------------------+
Vin_CM ------->| +                    |
               |        DUT           |---- Vout ----+----> CL, RL
          +--->| -                    |              |
          |    +----------------------+              |
          |                |                         |
          |          Feedback Path                   |
          |                |                         |
          +---------[ Loop Probe ]<------------------+
                     (Middlebrook /
                      Tian Probe)
```

![TB03 Stability](assets/TB03_Stability.png)
*Figure 1: Closed-loop amplifier stability testbench with non-invasive loop probe.*

---

## 5. Circuit Configuration
1. **Operating Point Preservation:** Maintain DUT in its intended closed-loop feedback configuration (e.g. unity-gain follower, inverting gain, or capacitive divider).
2. **Loop Probe Insertion:** Insert a bilateral loop-probe element (such as a Middlebrook AC injection source or Cadence `stb` probe) in the feedback network where impedance is high on one side and low on the other.
3. **Common-Mode Bias:** Bias non-inverting input at nominal $V_{CM}$.

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>` or `.stb` (Stability Analysis directive in Spectre/Eldo).
- **Sweep Range:** `1 mHz` to `10 GHz` with at least 50 points/decade.

---

## 7. Simulation Procedure
```spice
* Middlebrook / Tian Loop Injection Method in SPICE
* In series with feedback wire between net_out and net_fb:
Vi net_out net_probe DC=0 AC=1
Ii net_probe 0 DC=0 AC=0

* Dual-run extraction or simulator-native stability engine (.stb)
.stb Vout probe_instance
```
1. Verify operating point solution matches nominal closed-loop state.
2. Run frequency response across the loop.
3. Compute loop return ratio $T(j\omega)$.

---

## 8. Calculations
### Loop Return Ratio
$$
T(j\omega) = - \frac{v_{\text{return}}(j\omega)}{v_{\text{injected}}(j\omega)}
$$

### Phase Margin (PM)
At the unity-loop-gain frequency $\omega_u$ where $|T(j\omega_u)| = 1$ ($0\,\text{dB}$):
$$
PM = 180^\circ + \angle T(j\omega_u)
$$
*Criteria:* Target $PM \ge 60^\circ$ for optimal settling with minimal overshoot.

### Gain Margin (GM)
At the phase crossover frequency $\omega_{-180^\circ}$ where $\angle T(j\omega_{-180^\circ}) = -180^\circ$:
$$
GM = - 20\log_{10}|T(j\omega_{-180^\circ})|
$$
*Criteria:* Target $GM \ge 10\,\text{dB}$.

---

## 9. Required Plots
- **Loop Gain Magnitude Bode Plot:** $20\log_{10}|T(j\omega)|$ vs. Frequency (log scale, Hz).
- **Loop Gain Phase Bode Plot:** $\angle T(j\omega)$ vs. Frequency (log scale, Hz) with $0\,\text{dB}$ and $-180^\circ$ crosshairs highlighted.
- **Nyquist Plot:** Imaginary vs. Real parts of $T(j\omega)$ to verify enclosure of the critical point $(-1, j0)$.

---

## 10. Measurements to Save
```spice
.meas AC fug WHEN vdb(loop_gain) = 0
.meas AC phase_ug FIND vp(loop_gain) WHEN vdb(loop_gain) = 0
.meas AC PM PARAM = '180 + phase_ug'
.meas AC f_180 WHEN vp(loop_gain) = -180
.meas AC mag_180 FIND vdb(loop_gain) WHEN vp(loop_gain) = -180
.meas AC GM PARAM = '-mag_180'
* Placeholder criteria:
* PM >= DESIGN_SPEC_PM
* GM >= DESIGN_SPEC_GM
```

---

## 11. PVT Considerations
- Under **fast corners (FF)**, higher transconductance pushes $f_{ug}$ higher, potentially eroding phase margin if secondary poles do not scale equally.
- Under **slow corners (SS) and high temperature**, lower $g_m$ reduces loop bandwidth and slows settling.

---

## 12. Post-Layout Considerations
- **Parasitic Capacitance at High-Impedance Nodes:** Parasitic routing capacitors directly add to internal nodes, pulling non-dominant poles down in frequency and reducing phase margin by $5^\circ$ to $20^\circ$.
- Compare:
  $$\Delta PM = PM_{\text{PEX}} - PM_{\text{schematic}}$$

---

## 13. References
- **R. D. Middlebrook**, "Measurement of Loop Gain in Feedback Systems," *Int. J. Electronics*, 1975. [DOI Link](https://doi.org/10.1080/00207217508920421)
- **H. Neag et al.**, "Comparative Analysis of Simulation-Based Methods for Deriving the Phase- and Gain-Margins of Feedback Circuits With Op-Amps," *IEEE Trans. Circuits Syst. I*, 2015. [DOI Link](https://doi.org/10.1109/TCSI.2014.2370151)

---

## 14. Related Testbenches
- **[TB01: AC Gain](TB01_AC_Gain.md)** / **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Open-loop foundations.
- **[TB10: Transient](TB10_Transient.md)** — Time-domain step response verifies PM via overshoot and ringing damping.
- **[TB15: Local Loop Stability](TB15_Local_Loop_Stability.md)** — Verifies nested/local feedback loops.
- **[TB16: Load Sweep](TB16_Load_Sweep.md)** — Evaluates PM vs. capacitive loading.

---

## 15. Navigation

| [<< TB02: Differential Gain](TB02_Differential_Gain.md) | [Docs Index](index.md) | [TB04: CMRR >>](TB04_CMRR.md) |
