[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | **[TB04: CMRR](TB04_CMRR.md)** | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB04 — Common-Mode Rejection Ratio (CMRR)

Characterization of differential versus common-mode transmission across frequency to quantify the rejection of common-mode input interference.

---

## 1. Purpose
- Measure the small-signal differential voltage gain $A_d(f)$.
- Measure the small-signal common-mode to differential conversion gain $A_{CM}(f)$ (or $A_{CM \rightarrow DM}(f)$).
- Calculate the Common-Mode Rejection Ratio $\text{CMRR}(f)$ as a function of frequency.
- Determine low-frequency DC CMRR and the $-3\,\text{dB}$ CMRR roll-off corner.

---

## 2. Parameters Measured
- **DC Common-Mode Rejection Ratio ($\text{CMRR}_0$):** In decibels ($\text{dB}$)
- **CMRR Roll-Off Corner Frequency ($f_{CMRR}$):** Frequency where CMRR degrades by $3\,\text{dB}$
- **High-Frequency CMRR:** Value at key application frequencies (e.g. $10\,\text{kHz}$, $1\,\text{MHz}$)
- **Common-Mode Gain ($A_{CM}$):** In $\text{dB}$ across frequency

---

## 3. Applicable Amplifiers
- **Class B:** All differential-input amplifiers and OTAs
- **Class C:** Single-ended operational amplifiers
- **Class D:** Fully differential operational amplifiers and OTAs

---

## 4. Testbench Schematic

```text
       (1) Differential Mode                      (2) Common Mode
         ┌───────────────┐                       ┌───────────────┐
VIN+ ────┤+            + ├── VOUTP       VIN ────┤+            + ├── VOUTP
(AC=+0.5)│   Amplifier   │               (AC=1.0)│   Amplifier   │
VIN- ────┤-            - ├── VOUTM       VIN ────┤-            - ├── VOUTM
(AC=-0.5)└───────────────┘               (AC=1.0)└───────────────┘
          Measure: Ad(f)                          Measure: ACM(f)
```

![TB04 CMRR](assets/TB04_CMRR.png)
*Figure 1: Dual-simulation AC scheme for differential-mode and common-mode gain extraction.*

---

## 5. Circuit Configuration
Characterization requires two successive or parallel AC simulations under identical bias conditions:
1. **Differential-Mode Excitation:**
   ```text
   VIN+ : DC = VCM, AC = +0.5
   VIN- : DC = VCM, AC = -0.5
   ```
2. **Common-Mode Excitation:**
   ```text
   VIN+ : DC = VCM, AC = 1.0
   VIN- : DC = VCM, AC = 1.0
   ```

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>`
- **Sweep Range:** `1 Hz` to `1 GHz` with 50 points/decade.

---

## 7. Simulation Procedure
```spice
* Run 1: Differential Mode
VINP inp 0 DC=0.9 AC=0.5
VINM inm 0 DC=0.9 AC=0.5 180
XDUT1 inp inm outp outm vdd vss diff_amp

* Run 2: Common Mode
VCM_IN cm_in 0 DC=0.9 AC=1.0
XDUT2 cm_in cm_in outp_cm outm_cm vdd vss diff_amp

.ac dec 50 1 1G
```
1. Solve operating points for both setups.
2. Sweep frequency across decades.
3. Compute $A_d(f)$ and $A_{CM}(f)$ transfer responses.

---

## 8. Calculations
### Single-Ended Output Gain Definitions
$$
A_d(f) = \frac{V_{out}(f)}{V_{in+}(f) - V_{in-}(f)}, \quad A_{CM}(f) = \frac{V_{out}(f)}{V_{CM}(f)}
$$

### Fully Differential Output Definitions
$$
A_d(f) = \frac{V_{out+}(f) - V_{out-}(f)}{V_{in+}(f) - V_{in-}(f)}, \quad A_{CM \rightarrow DM}(f) = \frac{V_{out+}(f) - V_{out-}(f)}{V_{CM}(f)}
$$

### CMRR in Decibels
$$
\text{CMRR}(f) = 20\log_{10}\left|\frac{A_d(f)}{A_{CM}(f)}\right| = 20\log_{10}|A_d(f)| - 20\log_{10}|A_{CM}(f)|
$$

---

## 9. Required Plots
- **Composite Bode Plot:** Overlay $A_{d,\text{dB}}(f)$ and $A_{CM,\text{dB}}(f)$ on the same axis.
- **CMRR Bode Plot:** $\text{CMRR}(f)$ in $\text{dB}$ vs. logarithmic frequency.
- **CMRR Phase Response:** Phase difference between differential and common-mode signal paths.

---

## 10. Measurements to Save
```spice
.meas AC Ad_dc FIND vdb(outp, outm) AT=1
.meas AC Acm_dc FIND vdb(outp_cm, outm_cm) AT=1
.meas AC CMRR_dc PARAM = 'Ad_dc - Acm_dc'
.meas AC f_cmrr_3db WHEN vdb(cmrr_trace) = 'CMRR_dc - 3'
* Placeholder criteria:
* CMRR_dc >= DESIGN_SPEC_CMRR
```

---

## 11. PVT Considerations
- Under nominal conditions, symmetric topologies offer theoretically infinite CMRR without mismatch.
- In practical PVT sweeps with systematic offsets and temperature-dependent tail source output resistances ($r_{o,tail}$), CMRR degrades at elevated temperatures due to lower tail impedance.

---

## 12. Post-Layout Considerations
- **Layout Asymmetry:** Layout trace routing capacitance imbalances between positive and negative signal paths significantly degrade high-frequency CMRR by creating common-mode to differential mode conversion.
- **Tail Node Parasitics:** Parasitic capacitance at the common-source tail node ($C_{tail}$) bypasses the tail impedance at high frequencies, degrading CMRR as $\omega C_{tail} r_{o,tail} > 1$.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 4: Differential Amplifiers (Common-Mode Rejection). [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: CMRR Measurement Techniques.

---

## 14. Related Testbenches
- **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Provides the differential transmission channel.
- **[TB05: PSRR](TB05_PSRR.md)** — Evaluates rejection of power-rail common-mode disturbances.
- **[TB08: ICMR](TB08_ICMR.md)** — Identifies the linear input range over which CMRR remains valid.
- **[TB18: Monte Carlo](TB18_Monte_Carlo.md)** — Essential to evaluate mismatch-induced CMRR degradation.

---

## 15. Navigation

| [<< TB03: Stability](TB03_Stability.md) | [Docs Index](index.md) | [TB05: PSRR >>](TB05_PSRR.md) |
