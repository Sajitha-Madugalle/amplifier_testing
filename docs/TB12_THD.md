[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | **[TB12: THD](TB12_THD.md)** | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB12 — Total Harmonic Distortion (THD) & Linearity

Large-signal sinusoidal dynamic linearity characterization extracting harmonic distortion components (HD2, HD3, $\dots$, HDn), THD, and Spurious-Free Dynamic Range (SFDR).

---

## 1. Purpose
- Quantify non-linear distortion under sinusoidal excitation.
- Measure Total Harmonic Distortion ($\text{THD}$) across input amplitude and frequency.
- Extract individual harmonic components: second harmonic ($\text{HD2}$), third harmonic ($\text{HD3}$), and higher order terms.
- Determine maximum linear signal handling capacity before clipping ($1\%$ or $0.1\%$ THD limit).
- Assess the harmonic suppression benefit of fully differential balanced structures.

---

## 2. Parameters Measured
- **Total Harmonic Distortion ($\text{THD}$):** Expressed in $\%$ or decibels ($\text{dB}$)
- **Second Harmonic Distortion ($\text{HD2}$):** $20\log_{10}(V_2/V_1)$ in $\text{dBc}$
- **Third Harmonic Distortion ($\text{HD3}$):** $20\log_{10}(V_3/V_1)$ in $\text{dBc}$
- **Spurious-Free Dynamic Range ($\text{SFDR}$):** In $\text{dBc}$
- **Signal-to-Noise-and-Distortion Ratio ($\text{SINAD}$):** Combined metric

---

## 3. Applicable Amplifiers
- **Class A:** Single-ended cascodes (dominated by even-order HD2)
- **Class B:** Single-ended OTAs and buffers
- **Class C:** Operational amplifiers in closed-loop configurations
- **Class D:** Fully differential operational amplifiers (even-order harmonics theoretically canceled)

---

## 4. Testbench Schematic

```text
               Closed-Loop Linearity Test Setup
               VDD
                │
                ▼
        ┌───────────────┐
Vin(t) ─┤+              │
(Sine)  │      DUT      ├──── Vout(t) ───┬─── FFT Spectral Analysis
   ┌────┤-              │                │    (HD1, HD2, HD3... HDn)
   │    └───────┬───────┘                └─── Load (CL, RL)
   │            │
   │            ▼
   │           VSS
   └─────────────────────────────────────┘
```

![TB12 THD](assets/TB12_THD.png)
*Figure 1: Closed-loop transient setup with coherent sinusoidal excitation for spectral analysis.*

---

## 5. Circuit Configuration
1. **Feedback Setup:** Place amplifier in nominal closed-loop configuration (e.g. unity-gain buffer or non-inverting gain $A_{CL} = +2$).
2. **Sinusoidal Source:** Apply low-distortion sinusoidal excitation:
   $$V_{in}(t) = V_{CM} + A_{in}\sin(2\pi f_{in} t)$$
3. **Coherent Sampling Criterion:** To prevent spectral leakage without windowing, ensure:
   $$\frac{f_{in}}{f_{sample}} = \frac{M}{N}$$
   where $N$ is the number of FFT points (power of 2) and $M$ is an integer number of prime cycles.
4. **Transient Settling:** Run transient simulation for several initial cycles to flush startup transients before collecting data.

---

## 6. Simulation Type
- **SPICE Directive:** `.tran <t_step> <t_stop>` followed by `.four <f_in> <num_harmonics> V(out)` or FFT post-processing.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB12
.param FIN=10k
.param AMP=0.5
VIN in 0 SIN(0.9 {AMP} {FIN})
XDUT in out out vdd vss opamp_core
CLOAD out 0 5pF
RLOAD out 0 50k

* Run for 12 cycles, sample last 8 cycles with 2048 points
.tran 0.1u 1.2m 0.4m 50n
.four 10k 9 v(out)
```
1. Run transient analysis to steady-state periodic regime.
2. Sample waveform uniformly.
3. Compute Fourier decomposition extracting fundamental $V_1$ and harmonics $V_2, V_3, \dots, V_9$.

---

## 8. Calculations
### Harmonic Voltages
$$V_1 = \text{Fundamental magnitude at } f_{in}$$
$$V_k = \text{Harmonic magnitude at } k \cdot f_{in} \quad (k \ge 2)$$

### Total Harmonic Distortion (THD)
$$
\text{THD} = \frac{\sqrt{\sum_{k=2}^n V_k^2}}{V_1}
$$
In decibels:
$$
\text{THD}_{\text{dB}} = 20\log_{10}(\text{THD}) = 20\log_{10}\left(\frac{\sqrt{\sum_{k=2}^n V_k^2}}{V_1}\right)
$$

### Individual Harmonics (dBc)
$$
\text{HD2} = 20\log_{10}\left(\frac{V_2}{V_1}\right), \quad \text{HD3} = 20\log_{10}\left(\frac{V_3}{V_1}\right)
$$

---

## 9. Required Plots
- **FFT Power Spectrum:** Magnitude (dBV / dBFS) vs. Frequency (linear or log scale) with harmonic bins highlighted.
- **THD vs. Input Amplitude:** THD (dB) vs. $A_{in}$ ($10\,\text{mV}$ to near-rail swing).
- **HD2 and HD3 vs. Output Swing:** Tracking how third-order distortion overtakes second-order distortion in balanced designs.

---

## 10. Measurements to Save
```spice
.meas FOUR thd_meas THD v(out)
.meas FOUR hd2_meas HD2 v(out)
.meas FOUR hd3_meas HD3 v(out)
* Placeholder criteria:
* thd_meas <= DESIGN_SPEC_THD
* hd3_meas <= DESIGN_SPEC_HD3
```

---

## 11. PVT Considerations
- Distortion worsens dramatically near supply rails as transistors transition from saturation toward the linear region.
- In **SS corners and low temperatures**, higher threshold voltages decrease effective overdrive $(V_{GS}-V_{th})$, leading to earlier non-linear gain compression.

---

## 12. Post-Layout Considerations
- **Non-Linear Junction Capacitance:** Variable depletion capacitances of large drain diffusion regions in post-layout models increase voltage-dependent phase modulation and harmonic distortion.
- **Differential Symmetry:** Imperfect layout matching between positive and negative signal paths leaks second harmonic (HD2) into fully differential outputs.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 13: Nonlinearity and Mismatch. [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB09: Output Swing](TB09_Output_Swing.md)** — Establishes upper clipping limits.
- **[TB11: Noise](TB11_Noise.md)** — Combines with THD to compute SINAD.
- **[TB13: FD DM Mode](TB13_FD_Differential_Mode.md)** — Fully differential linearity analysis.

---

## 15. Navigation

| [<< TB11: Noise](TB11_Noise.md) | [Docs Index](index.md) | [TB13: FD DM Mode >>](TB13_FD_Differential_Mode.md) |
