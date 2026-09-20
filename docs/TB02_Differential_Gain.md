[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | **[TB02: Diff](TB02_Differential_Gain.md)** | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB02 — Differential AC Gain

Frequency-domain small-signal characterization of differential-input operational transconductance amplifiers and op-amps.

---

## 1. Purpose
- Extract small-signal open-loop differential voltage gain ($A_d$ or $A_{dm}$).
- Measure differential Gain-Bandwidth Product (GBW) and Unity-Gain Frequency (UGF).
- Determine differential dominant pole frequency ($f_{-3\text{dB}}$) and secondary high-frequency pole/zero locations.
- Support both single-ended and balanced fully differential output topographies.

---

## 2. Parameters Measured
- **Differential DC Gain ($A_{d0}$):** In $\text{V/V}$ and $\text{dB}$
- **Differential $-3\,\text{dB}$ Bandwidth ($f_{-3\text{dB},diff}$):** Corner frequency
- **Unity-Gain Frequency ($f_{u,diff}$):** Frequency where $|A_d(f)| = 1$ ($0\,\text{dB}$)
- **Phase at UGF ($\angle A_d(f_u)$):** Phase margin indicator
- **Gain-Bandwidth Product (GBW):** Calculated small-signal product

---

## 3. Applicable Amplifiers
- **Class B:** Active-load differential pairs, complementary differential pairs, 5-T OTAs
- **Class C:** Folded-cascode OTAs, telescopic OTAs, two-stage Miller op-amps
- **Class D:** Fully differential OTAs and op-amps (with active CMFB network initialized)

---

## 4. Testbench Schematic

```text
               VDD
                │
                ▼
        ┌───────────────┐
Vin+ ───┤+            + ├─── Vout+ (or Vout) ─── Load
(AC=+0.5)│   Amplifier   │
Vin- ───┤-            - ├─── Vout- (Fully Diff)── Load
(AC=-0.5)└───────┬───────┘
                 │
                 ▼
                VSS
```

![TB02 Differential Gain](assets/TB02_Differential_Gain.png)
*Figure 1: Balanced differential AC excitation testbench configuration.*

---

## 5. Circuit Configuration
1. **Common-Mode Biasing:** Inputs are biased at nominal input common-mode voltage $V_{CM}$:
   $$V_{in+} = V_{CM} + \frac{v_d}{2}, \quad V_{in-} = V_{CM} - \frac{v_d}{2}$$
2. **AC Excitations:** Set balanced anti-phase small-signal sources:
   ```text
   VIN+ : DC = VCM, AC = +0.5
   VIN- : DC = VCM, AC = -0.5
   ```
   This ensures a normalized differential input of $v_{id} = V_{in+} - V_{in-} = 1.0\,\text{V}_{AC}$.
3. **Load:** Connect nominal load capacitors and resistors from output(s) to AC ground or differential load.

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>`
- **Sweep Range:** `0.1 Hz` to `10 GHz` with 50 points/decade.

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB02
.param VCM=0.9
VINP inp 0 DC=VCM AC=0.5 0
VINM inm 0 DC=VCM AC=0.5 180

* Device under test
XDUT inp inm outp outm vdd vss diff_opamp

* Terminations
CLP outp 0 2pF
CLM outm 0 2pF

.ac dec 50 0.1 10G
```
1. Verify quiescent operating conditions (`.op`).
2. Run AC frequency response across the spectrum.
3. Extract differential voltage transfer functions.

---

## 8. Calculations
### Single-Ended Output Amplifier
$$
A_d(f) = \frac{V_{out}(f)}{V_{in+}(f) - V_{in-}(f)} = V_{out}(f)
$$

### Fully Differential Output Amplifier
$$
A_d(f) = \frac{V_{out+}(f) - V_{out-}(f)}{V_{in+}(f) - V_{in-}(f)} = V_{out+}(f) - V_{out-}(f)
$$

### Gain in Decibels
$$
A_{d,\text{dB}}(f) = 20\log_{10}|A_d(f)|
$$

---

## 9. Required Plots
- **Differential Gain Bode Plot:** $20\log_{10}|A_d(f)|$ vs. Frequency (log scale, Hz).
- **Differential Phase Bode Plot:** $\angle A_d(f)$ vs. Frequency (log scale, Hz).
- **High-Frequency Phase Roll-off:** Visual inspection of secondary pole/zero doublets.

---

## 10. Measurements to Save
```spice
.meas AC Ad0 MAX vdb(outp, outm)
.meas AC f_3db WHEN vdb(outp, outm) = 'Ad0 - 3'
.meas AC UGF WHEN vdb(outp, outm) = 0
.meas AC phase_ugf FIND vp(outp, outm) WHEN vdb(outp, outm) = 0
* Placeholder criteria:
* Ad0 >= DESIGN_SPEC_GAIN
* UGF >= DESIGN_SPEC_UGF
```

---

## 11. PVT Considerations
- Differential gain is sensitive to output impedance ($r_o$), which degrades at high temperatures and in the **FF** corner due to channel-length modulation ($\lambda$).
- Bandwidth extends under FF corners but contracts under SS corners; check minimum $GBW$ at SS / low-$V_{DD}$ / high-$T$.

---

## 12. Post-Layout Considerations
- **Differential Routing Symmetry:** Parasitic routing mismatch between `INP`/`INM` and `OUTP`/`OUTM` degrades phase matching at high frequencies.
- **Interconnect Capacitance:** Inter-wire capacitances between adjacent metal runs introduce undesired capacitive feedthrough or pole shifts.

---

## 13. References
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 4: Differential Amplifiers; Chapter 9: Operational Amplifiers. [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)

---

## 14. Related Testbenches
- **[TB04: CMRR](TB04_CMRR.md)** — Combines differential gain $A_d$ with common-mode gain $A_{CM}$.
- **[TB03: Stability](TB03_Stability.md)** — Evaluates closed-loop loop gain and phase margin.
- **[TB13: FD DM Mode](TB13_FD_Differential_Mode.md)** — Dedicated fully differential characterization suite.

---

## 15. Navigation

| [<< TB01: AC Gain](TB01_AC_Gain.md) | [Docs Index](index.md) | [TB03: Stability >>](TB03_Stability.md) |
