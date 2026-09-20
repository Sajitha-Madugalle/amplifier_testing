[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | **[TB17: PVT](TB17_PVT.md)** | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB17 — Process, Voltage & Temperature (PVT) Characterization

A comprehensive automation wrapper executing the core electrical testbench suite across semiconductor process corners, supply voltage tolerances, and ambient temperature extremes.

---

## 1. Purpose
- Establish the operational envelope and guarantee worst-case circuit specifications across environmental variations.
- Verify stability, gain, dynamic range, and bandwidth under combined stress corners.
- Identify design sensitivities to cross-corners (e.g. Fast-NMOS / Slow-PMOS causing current mirror imbalance).
- Compile characterization tables (`_min`, `_typ`, `_max`) for formal design reviews and data sheets.

---

## 2. Parameters Measured
Every primary metric is recorded across the 3D corner matrix ($P \times V \times T$):
- **DC Gain ($A_0$):** Typical and minimum values
- **Unity-Gain Frequency ($UGF$) / Bandwidth:** Typical and minimum values
- **Phase Margin ($PM$):** Minimum across all corners
- **Power Dissipation ($P_{DC}$):** Maximum under worst-case leakage/overvoltage
- **CMRR & PSRR:** Minimum over operating frequency bands
- **Slew Rate ($SR^+$ and $SR^-$):** Slowest corner values
- **Dynamic Headroom ($V_{ICMR}, V_{swing}$):** Tightest boundaries

---

## 3. Applicable Amplifiers
- **All Classes (A, B, C, D):** Mandatory verification gate for all tapeout candidates.

---

## 4. Testbench Schematic

```text
                  PVT Automation Wrapper
                  
      Process Corners (P)          Supply Voltage (V)       Temperature (T)
   ┌───────────────────────┐     ┌───────────────────┐    ┌─────────────────┐
   │ TT (Typical-Typical)  │     │ VDD_MIN (-10%)    │    │ T_MIN (-40°C)   │
   │ FF (Fast-Fast)        │  x  │ VDD_NOM (1.8V)    │  x │ T_NOM (+27°C)   │
   │ SS (Slow-Slow)        │     │ VDD_MAX (+10%)    │    │ T_MAX (+125°C)  │
   │ FS (Fast-N / Slow-P)  │     └───────────────────┘    └─────────────────┘
   │ SF (Slow-N / Fast-P)  │
   └───────────────────────┘
                                   │
                                   ▼
             [ Execute TB00 through TB15 on Netlist ]
                                   │
                                   ▼
          Output Matrix: [Corner ID, Parameter, Min, Typ, Max]
```

*Figure 1: Multidimensional PVT corner sweep execution workflow.*

---

## 5. Circuit Configuration
1. **Process Corner Libraries:** Load foundry corner models:
   - `TT`: Nominal threshold, mobility, and tox
   - `FF`: High mobility, low threshold (maximum speed and leakage)
   - `SS`: Low mobility, high threshold (slowest speed)
   - `FS` & `SF`: Skewed cross-corners for asymmetric device pairs
2. **Supply Voltage Range:**
   $$V_{DD,min} = V_{DD,nom} \times 0.90, \quad V_{DD,max} = V_{DD,nom} \times 1.10$$
3. **Temperature Range:**
   $$T \in [-40^\circ\text{C}, +27^\circ\text{C}, +125^\circ\text{C}]$$ (Automotive grade: up to $+150^\circ\text{C}$)

---

## 6. Simulation Type
- **Automated Scripting:** Batch execution script (Python / Skill / Ocean) invoking SPICE engine across all 45 discrete corner combinations ($5 \times 3 \times 3$).

---

## 7. Simulation Procedure
```spice
* Standard SPICE corner invocation syntax
.lib 'foundry_models.lib' ss
.param VDD_VAL=1.62
.temp 125

VDD vdd 0 DC={VDD_VAL}
* ... Execute testbench directives (.op, .ac, .tran)
```
1. Run nominal condition (`TT`, $V_{DD,nom}$, $27^\circ\text{C}$) to establish the baseline `_typ` vector.
2. Iterate through all corner combinations.
3. Collate extracted scalar measurements into a consolidated tabular dataset.

---

## 8. Calculations
For each extracted performance metric $X$:
$$
X_{\text{typ}} = X(\text{TT}, V_{DD,nom}, 27^\circ\text{C})
$$
$$
X_{\text{min}} = \min_{\forall P, V, T} \left(X(P, V, T)\right)
$$
$$
X_{\text{max}} = \max_{\forall P, V, T} \left(X(P, V, T)\right)
$$

---

## 9. Required Plots
- **Corner Overlay Bode Plots:** 45 curves of $|A(f)|$ and $\angle A(f)$ on a single chart showing gain/phase spread.
- **Kite / Radar Plot:** Visualizing performance envelope across Power, Speed, Gain, and Linearity.
- **Metric vs. Temperature:** Parameter trajectories plotted across temperature for each corner.

---

## 10. Measurements to Save
Consolidated Characterization Dataset:
```text
A0_typ:      [VAL] dB        A0_min:      [VAL] dB
UGF_typ:     [VAL] MHz       UGF_min:     [VAL] MHz
PM_typ:      [VAL] deg       PM_min:      [VAL] deg
CMRR_typ:    [VAL] dB        CMRR_min:    [VAL] dB
PSRR_typ:    [VAL] dB        PSRR_min:    [VAL] dB
Power_typ:   [VAL] mW        Power_max:   [VAL] mW
SR_typ:      [VAL] V/us      SR_min:      [VAL] V/us
```

---

## 11. PVT Considerations
- **Worst-Case Corners for Key Specs:**
  - **Bandwidth & Slew Rate:** Worst at `SS` / $V_{DD,min}$ / $125^\circ\text{C}$
  - **Phase Margin:** Worst at `FF` / $V_{DD,max}$ / $-40^\circ\text{C}$
  - **Static Power:** Worst at `FF` / $V_{DD,max}$ / $125^\circ\text{C}$
  - **Gain & Headroom:** Worst at `SS` / $V_{DD,min}$ / $125^\circ\text{C}$

---

## 12. Post-Layout Considerations
- Run PVT on the **PEX netlist** to ensure interconnect parasitics (whose resistances have positive temperature coefficients, $+0.4\%/^\circ\text{C}$) do not violate timing or stability margins at elevated temperatures.

---

## 13. References
- **Phillip E. Allen & Douglas R. Holberg**, *CMOS Analog Circuit Design*, Chapter 6: Practical Op-Amp Design and Characterization.
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 17: Layout and Packaging (Process and Temperature Effects).

---

## 14. Related Testbenches
- **[TB00 to TB15](index.md)** — All underlying electrical testbenches called by TB17.
- **[TB18: Monte Carlo](TB18_Monte_Carlo.md)** — Complements PVT by exploring intra-die random mismatch.
- **[Testbench Matrix](testbench_matrix.md)** — Defines PEX post-layout comparison formulas.

---

## 15. Navigation

| [<< TB16: Load Sweep](TB16_Load_Sweep.md) | [Docs Index](index.md) | [TB18: Monte Carlo >>](TB18_Monte_Carlo.md) |
