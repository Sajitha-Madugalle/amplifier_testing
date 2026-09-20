[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | **[TB18: Monte Carlo](TB18_Monte_Carlo.md)**

---

# TB18 — Monte Carlo & Statistical Mismatch Characterization

Statistical yield and robustness analysis evaluating intra-die device mismatch and inter-die process parameter dispersion.

---

## 1. Purpose
- Extract statistical distributions (mean $\mu$, standard deviation $\sigma$, and $3\sigma$ design bounds) for key amplifier figures of merit.
- Determine yield percentages against hard design specification thresholds.
- Quantify mismatch-dependent parameters: input offset voltage ($V_{OS}$), finite CMRR, and finite PSRR.
- Identify parameter cross-correlations (e.g. correlation between quiescent power and UGF).

---

## 2. Parameters Measured
Statistical vectors $(\mu, \sigma, 3\sigma, \text{Min}, \text{Max})$ for:
- **Input Offset Voltage ($V_{OS}$):** Primary mismatch parameter
- **Low-Frequency Gain ($A_0$):** Gain variance caused by $g_m$ and $r_o$ mismatch
- **Gain-Bandwidth Product ($GBW$) & Phase Margin ($PM$):** Dynamic stability variance
- **Common-Mode Rejection Ratio ($\text{CMRR}$):** Floor set by pair mismatch
- **Power Supply Rejection Ratio ($\text{PSRR}$):** Degradation from symmetry loss
- **Output Common-Mode Voltage ($V_{OCM}$):** Mismatch in CMFB sensing networks
- **Quiescent Supply Current ($I_{DD}$):** Current mirror ratio variations

---

## 3. Applicable Amplifiers
- **All Classes (A, B, C, D):** Crucial for differential and high-precision analog architectures.

---

## 4. Testbench Schematic

```text
                  Monte Carlo Statistical Framework
                                  
      Foundry Statistical Model Cards (Mismatch & Process Dist.)
                                  │
                                  ▼
      ┌───────────────────────────────────────────────────────┐
      │ Monte Carlo Engine (N >= 200 to 1000 Iterations)      │
      │ ├── Pseudo-Random Device Geometry Variations (ΔW, ΔL) │
      │ ├── Threshold Voltage Mismatch (ΔVth)                 │
      │ └── Flat-band and Mobility Variations (Δtox, Δu0)     │
      └───────────────────────────────────────────────────────┘
                                  │
                                  ▼
                  [ Execute Selected Testbench ]
                                  │
                                  ▼
               Statistical Histograms & Yield Report
```

*Figure 1: Monte Carlo simulation workflow across statistical device variation distributions.*

---

## 5. Circuit Configuration
1. **Mismatch Enable:** Enable foundry statistical Monte Carlo model cards with mismatch parameters (`mismatch=1`, `process=1`).
2. **Sample Size:** Use a statistically significant sample size:
   - $N \ge 200$ for preliminary screening and $3\sigma$ estimations
   - $N \ge 1000$ for formal yield certification ($> 99.7\%$ confidence)
3. **Environment:** Run at nominal supply voltage and room temperature ($27^\circ\text{C}$), or at worst-case PVT corners.

---

## 6. Simulation Type
- **SPICE Directive:** `.mc <num_runs> <analysis_type> <measurement_var> ...`
- **Cadence / Eldo:** Monte Carlo analysis engine with Gaussian/uniform parameter distribution sampling.

---

## 7. Simulation Procedure
```spice
* Standard SPICE Monte Carlo syntax for TB18
.lib 'foundry_models.lib' mc_models

* Run 500 iterations of DC sweep for offset extraction
.mc 500 DC Vos_meas
```
1. Initialize pseudo-random seed for reproducibility.
2. Execute batch simulation across all $N$ runs.
3. Compute summary statistics ($\mu$, $\sigma$) for each output variable.

---

## 8. Calculations
### Statistical Distribution Metrics
For $N$ iterations:
$$
\mu = \frac{1}{N}\sum_{k=1}^N X_k
$$
$$
\sigma = \sqrt{\frac{1}{N-1}\sum_{k=1}^N \left(X_k - \mu\right)^2}
$$
$$
3\sigma_{\text{bound}} = 3 \times \sigma
$$

### Yield Calculation
$$
\text{Yield}_{\%} = \frac{N_{\text{passed}}}{N_{\text{total}}} \times 100\%
$$
where $N_{\text{passed}}$ represents runs meeting all defined constraints:
```text
A0 >= DESIGN_SPEC_GAIN
PM >= DESIGN_SPEC_PM
CMRR >= DESIGN_SPEC_CMRR
abs(VOS) <= DESIGN_SPEC_VOS
```

---

## 9. Required Plots
- **Parameter Histograms:** Frequency distribution with overlaid Gaussian curve.
- **Cumulative Distribution Function (CDF):** Sigmoidal probability curve showing yield vs. specification threshold.
- **Scatter / Correlation Plots:** Bi-variate scatter (e.g. Gain vs. Bandwidth, Offset vs. Power).
- **Quantile-Quantile (Q-Q) Plot:** Confirming Gaussian normality of extracted offsets.

---

## 10. Measurements to Save
```text
VOS_mean:        [VAL] uV        VOS_sigma:       [VAL] mV
VOS_3sigma:      [VAL] mV        Yield_Spec:      [VAL] %
A0_mean:         [VAL] dB        A0_sigma:        [VAL] dB
PM_mean:         [VAL] deg       PM_min:          [VAL] deg
CMRR_mean:       [VAL] dB        CMRR_min_3sigma: [VAL] dB
```

---

## 11. PVT Considerations
- Standard Monte Carlo runs vary mismatch at nominal $27^\circ\text{C}$. For high-reliability designs, run **Corner + Monte Carlo** (e.g. 200 runs at SS / $125^\circ\text{C}$) to observe worst-case mismatch under elevated thermal conditions.

---

## 12. Post-Layout Considerations
- Run Monte Carlo on **PEX netlists** whenever possible. Layout parasitic resistances and capacitances exhibit slight variations that can worsen matching if routing asymmetry exists.
- Pelgrom's matching coefficient:
  $$\sigma_{\Delta Vth} = \frac{A_{Vth}}{\sqrt{W \cdot L}}$$
  Verify that device sizes are large enough to meet required $3\sigma$ bounds.

---

## 13. References
- **P. R. Gray and R. G. Meyer**, "MOS Operational Amplifier Design — A Tutorial Overview," *IEEE J. Solid-State Circuits*, 1982. [DOI Link](https://doi.org/10.1109/JSSC.1982.1051851)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 13: Nonlinearity and Mismatch (Offset and Asymmetry).

---

## 14. Related Testbenches
- **[TB07: Input Offset](TB07_Input_Offset.md)** — Core testbench wrapped by Monte Carlo analysis.
- **[TB04: CMRR](TB04_CMRR.md)** & **[TB05: PSRR](TB05_PSRR.md)** — Evaluates mismatch-limited rejection floors.
- **[TB17: PVT](TB17_PVT.md)** — Global process variation counterpart.

---

## 15. Navigation

| [<< TB17: PVT](TB17_PVT.md) | [Docs Index](index.md) | [Library Home >>](../README.md) |
