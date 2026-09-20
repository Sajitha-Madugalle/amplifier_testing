[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | **[TB07: Offset](TB07_Input_Offset.md)** | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB07 — Input Offset Voltage

Characterization of systematic offset caused by asymmetric circuit biasing and random offset resulting from transistor threshold and geometry mismatch.

---

## 1. Purpose
- Extract the systematic input-referred DC offset voltage ($V_{OS,sys}$).
- Evaluate random mismatch offset through statistical Monte Carlo analysis.
- Calculate the offset mean ($\mu_{VOS}$), standard deviation ($\sigma_{VOS}$), and $3\sigma$ bounds ($V_{OS,3\sigma}$).
- Detect design-level asymmetry in differential pairs and active mirror loads.

---

## 2. Parameters Measured
- **Systematic Input Offset ($V_{OS,sys}$):** In millivolts ($\text{mV}$) or microvolts ($\mu\text{V}$)
- **Statistical Offset Mean ($\mu_{VOS}$):** Center of Gaussian distribution
- **Offset Standard Deviation ($\sigma_{VOS}$):** Scatter metric
- **Three-Sigma Offset ($V_{OS,3\sigma}$):** $3 \times \sigma_{VOS}$ (yield bound)
- **Offset Temperature Drift ($TC_{VOS}$):** Drift in $\mu\text{V}/^\circ\text{C}$

---

## 3. Applicable Amplifiers
- **Class B:** All differential-input amplifiers, active loads, 5-T OTAs
- **Class C:** Telescopic, folded-cascode, and multistage op-amps
- **Class D:** Fully differential operational amplifiers and OTAs

---

## 4. Testbench Schematic

```text
               VDD
                │
                ▼
        ┌───────────────┐
VIN+ ───┤+            + ├─── VOUTP ───┐
        │   Amplifier   │             ├─── Sense: Vod = VOUTP - VOUTM = 0
VIN- ───┤-            - ├─── VOUTM ───┘
        └───────┬───────┘
                │
                ▼
               VSS
VIN+ = VCM + VOS_test/2
VIN- = VCM - VOS_test/2
```

![TB07 Input Offset](assets/TB07_Input_Offset.png)
*Figure 1: High-resolution DC sweep testbench for input offset voltage extraction.*

---

## 5. Circuit Configuration
1. **Inputs:** Apply balanced DC differential excitation centered at the nominal common-mode voltage:
   $$V_{in+} = V_{CM} + \frac{V_{OS,test}}{2}, \quad V_{in-} = V_{CM} - \frac{V_{OS,test}}{2}$$
2. **DC Sweep:** Sweep $V_{OS,test}$ across a fine window around zero (e.g. $-20\,\text{mV}$ to $+20\,\text{mV}$ in steps of $1\,\mu\text{V}$).
3. **Outputs:** Connect nominal DC load or high-impedance sense node.

---

## 6. Simulation Type
- **SPICE Directive:** `.dc <sweep_var> <start> <stop> <step>`
- **Monte Carlo Directive:** `.mc <num_runs> dc ...` for statistical mismatch extraction.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB07
.param VCM=0.9
.param VOS=0
VINP inp 0 DC={VCM + VOS/2}
VINM inm 0 DC={VCM - VOS/2}

XDUT inp inm outp outm vdd vss diff_amp

* Fine DC sweep to identify zero-crossing
.dc VOS -20m 20m 10u
```
1. Run DC sweep.
2. Locate the input voltage $V_{OS}$ where differential output voltage $V_{od} = V_{out+} - V_{out-} = 0\,\text{V}$ (or nominal single-ended output reference).
3. Under Monte Carlo mode, repeat across 200+ statistical iterations.

---

## 8. Calculations
### Systematic Offset
$$
V_{OS,sys} = V_{id}\Big|_{V_{od}=0} = \left(V_{in+} - V_{in-}\right)\Big|_{V_{od}=0}
$$

### Statistical Extraction
Given $N$ Monte Carlo sample runs:
$$
\mu_{VOS} = \frac{1}{N}\sum_{k=1}^N V_{OS,k}
$$
$$
\sigma_{VOS} = \sqrt{\frac{1}{N-1}\sum_{k=1}^N \left(V_{OS,k} - \mu_{VOS}\right)^2}
$$
$$
V_{OS,3\sigma} = 3 \times \sigma_{VOS}
$$

---

## 9. Required Plots
- **DC Transfer Characteristic:** Differential output $V_{od}$ vs. $V_{OS,test}$, showing zero-crossing.
- **Monte Carlo Offset Histogram:** Distribution of $V_{OS}$ with fitted Gaussian bell curve.
- **Cumulative Probability Plot:** Quantifying yield percentage within design specification limits.

---

## 10. Measurements to Save
```spice
.meas DC Vos_sys WHEN v(outp, outm) = 0
* Placeholder criteria:
* abs(Vos_sys) <= DESIGN_SPEC_VOS_SYS
* Vos_3sigma <= DESIGN_SPEC_VOS_3SIGMA
```

---

## 11. PVT Considerations
- Systematic offset changes across PVT if non-ideal current mirror $V_{DS}$ matching drifts with temperature.
- In rail-to-rail complementary input stages, offset varies significantly across the common-mode range due to the transition between NMOS and PMOS pairs.

---

## 12. Post-Layout Considerations
- **Common-Centroid Matching:** Input differential pairs and current-mirror loads must use cross-quad / common-centroid layout with dummy devices to minimize gradient-induced offset.
- **Stress & Proximity:** Asymmetry in STI (shallow trench isolation) or well edges creates catastrophic threshold mismatch ($\sigma_{Vth} \propto \frac{A_{Vth}}{\sqrt{WL}}$).

---

## 13. References
- **P. R. Gray and R. G. Meyer**, "MOS Operational Amplifier Design — A Tutorial Overview," *IEEE J. Solid-State Circuits*, 1982. [DOI Link](https://doi.org/10.1109/JSSC.1982.1051851)
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: Input Offset Measurement.

---

## 14. Related Testbenches
- **[TB00: DC Operating Point](TB00_DC_Operating_Point.md)** — Sets quiescent operating baseline.
- **[TB08: ICMR](TB08_ICMR.md)** — Evaluates offset variations across the input common-mode range.
- **[TB18: Monte Carlo](TB18_Monte_Carlo.md)** — Statistical wrapper for offset yield estimation.

---

## 15. Navigation

| [<< TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [Docs Index](index.md) | [TB08: ICMR >>](TB08_ICMR.md) |
