[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** **[TB00: DC](TB00_DC_Operating_Point.md)** | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB00 — DC Operating Point & Power

Characterization of quiescent biasing conditions, static current consumption, power dissipation, and transistor inversion regions.

---

## 1. Purpose
- Establish and verify the static DC operating point for all internal nodes.
- Quantify quiescent supply currents ($I_{DD}$, $I_{SS}$) and total static power consumption ($P_{DC}$).
- Verify that all active transistors operate in their intended region (saturation/pinch-off for gain transistors, triode/linear for degenerated elements).
- Detect systematic design flaws (e.g., bias starvation, forward-biased junctions, floating gates).

---

## 2. Parameters Measured
- **Supply Currents:** $I_{DD}$ (positive rail current), $I_{SS}$ (negative rail current)
- **Static Power:** $P_{DC}$
- **Device Operating Parameters:** $V_{GS}$, $V_{DS}$, $V_{DSAT}$, $V_{th}$, $g_m$, $g_{ds}$, $g_{mb}$, $c_{gg}$, $c_{dd}$
- **Operating Region:** Saturation status ($V_{DS} \ge V_{GS} - V_{th}$ with adequate headroom $V_{DS} > V_{DSAT} + \Delta V_{margin}$)

---

## 3. Applicable Amplifiers
- **Class A:** Single-ended cascode, wide-swing cascode, regulated cascode
- **Class B:** NMOS/PMOS differential pairs, active-load diff-pairs, 5-T OTAs
- **Class C:** Telescopic, folded-cascode, two-stage Miller, three-stage op-amps
- **Class D:** Fully differential OTAs, two-stage FD op-amps, CMFB networks
*(Mandatory across all amplifier architectures).*

---

## 4. Testbench Schematic

```text
               VDD = VDD_nom
                 │
                 ▼
         ┌───────────────┐
         │   DUT Core    │
V_INP ───┤+             +├──── V_OUTP ─── Load (C_L, R_L)
         │   Amplifier   │
V_INM ───┤-             -├──── V_OUTM ─── Load (C_L, R_L)
         │               │
         └───────┬───────┘
                 │
                 ▼
               VSS = VSS_nom
```

![TB00 DC Operating Point](assets/TB00_DC_Operating_Point.png)
*Figure 1: Standardized DC testbench configuration with nominal terminal terminations.*

---

## 5. Circuit Configuration
1. **Supplies:** Connect $V_{DD} = V_{DD,nom}$ and $V_{SS} = V_{SS,nom}$ (ground or negative rail).
2. **Inputs:** Tie input terminals to the nominal common-mode voltage ($V_{CM,nom}$). For single-ended Class A stages, set $V_{IN} = V_{bias,nom}$.
3. **Outputs:** Terminate outputs with nominal capacitive and resistive loads ($C_{L,nom}$, $R_{L,nom}$) connected to nominal output reference level.
4. **Bias Enable:** Enable all master bias currents and bandgap-derived reference pins.

---

## 6. Simulation Type
- **SPICE Directive:** `.op` (Operating Point Analysis)
- **Auxiliary:** `.dc` (DC sweep over supply voltage $V_{DD}$ or temperature $T$)

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB00
VDD vdd 0 DC=1.8V
VSS vss 0 DC=0.0V
VCM vcm 0 DC=0.9V
VINP inp 0 DC=0.9V
VINM inm 0 DC=0.9V

* Instantiate DUT
XDUT inp inm outp outm vdd vss amplifier_core

* Analysis directive
.op
.save all
```
1. Run `.op` at nominal temperature ($27^\circ\text{C}$) and typical-typical (TT) corner.
2. Inspect output `.out` file or raw operating point vector.
3. Extract terminal currents through supply voltage sources: `I(VDD)` and `I(VSS)`.

---

## 8. Calculations
### Static DC Power
$$
P_{DC} = V_{DD} \cdot I_{DD} + |V_{SS} \cdot I_{SS}|
$$

### Saturation Margin Check
For each active MOSFET $M_i$:
$$
V_{margin,i} = V_{DS,i} - V_{DSAT,i}
$$
*Criteria:* $V_{margin,i} \ge 50\,\text{mV}$ across expected signal excursions.

---

## 9. Required Plots
- **Supply Current vs. Supply Voltage:** $I_{DD}$ vs. $V_{DD}$ ($V_{DD,min}$ to $V_{DD,max}$).
- **Quiescent Current vs. Temperature:** $I_{DD}(T)$ from $-40^\circ\text{C}$ to $+125^\circ\text{C}$.
- **Transistor Operating Map:** Bar plot or tabular view of $V_{DS}/V_{DSAT}$ across critical current mirrors and cascode branches.

---

## 10. Measurements to Save
```spice
.meas OP idd_meas AVG I(VDD)
.meas OP iss_meas AVG I(VSS)
.meas OP pdc_meas PARAM = '(1.8 * abs(idd_meas))'
* Placeholder criteria:
* Power <= DESIGN_SPEC_POWER
```

---

## 11. PVT Considerations
- Under **FF / high-VDD / high-temperature** conditions, current mirrors may exhibit significant increase in quiescent current, risking thermal overstress.
- Under **SS / low-VDD / low-temperature** conditions, cascode transistors approach the triode boundary ($V_{DS} \approx V_{DSAT}$), causing drastic degradation in open-loop gain.

---

## 12. Post-Layout Considerations
- **Parasitic Resistance ($R$):** Significant metal trace IR drop on power rails ($V_{DD}$, $V_{SS}$) alters internal gate and drain bias voltages.
- **Well Proximity & Stress Effects:** WPE and PSE shift $V_{th}$ by tens of millivolts if guard rings and dummy transistors are not symmetrically placed.
- **Verification:** Ensure `abs(P_DC_PEX - P_DC_schematic) / P_DC_schematic <= 5%`.

---

## 13. References
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: Op-Amp Simulation and DC Measurement. [Course Notes Link](https://pallen.ece.gatech.edu/Academic/ECE_6412/Spring_2003/L240-Sim%26MeasofOpAmps%282UP%29.pdf)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 3: Basic Current Mirrors and Biasing Techniques.

---

## 14. Related Testbenches
- **[TB01: AC Gain](TB01_AC_Gain.md)** — Uses the established DC bias for small-signal linearization.
- **[TB07: Input Offset](TB07_Input_Offset.md)** — Measures systematic shift from the nominal DC operating point.
- **[TB17: PVT](TB17_PVT.md)** — Sweeps DC operating point across process and voltage corners.

---

## 15. Navigation

| [<< Docs Index](index.md) | [Library Home](../README.md) | [TB01: AC Gain >>](TB01_AC_Gain.md) |
