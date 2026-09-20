[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | [TB05: PSRR](TB05_PSRR.md) | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | **[TB16: Load](TB16_Load_Sweep.md)** | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB16 — Load Characterization (Capacitive & Resistive Sweeps)

Parametric sensitivity characterization evaluating amplifier gain, stability, slew rate, and settling behavior across wide variations of capacitive ($C_L$) and resistive ($R_L$) loading.

---

## 1. Purpose
- Determine amplifier stability boundaries as a function of capacitive loading ($C_L$).
- Quantify phase margin degradation or enhancement across $C_L$ decades (differentiating unbuffered single-stage OTAs from two-stage Miller op-amps).
- Measure output current delivery and gain compression under heavy resistive loading ($R_L$).
- Identify maximum allowable load capacitance ($C_{L,max}$) before phase margin drops below minimum acceptable specifications.

---

## 2. Parameters Measured
- **Phase Margin vs. Load ($PM(C_L)$):** In degrees across $C_L$
- **Unity-Gain Frequency vs. Load ($UGF(C_L)$):** Bandwidth scaling
- **Slew Rate vs. Load ($SR(C_L)$):** Dynamic slewing capability
- **Settling Time vs. Load ($t_s(C_L)$):** Small- and large-signal recovery
- **DC Gain vs. Resistive Load ($A_0(R_L)$):** Output loading degradation

---

## 3. Applicable Amplifiers
- **Class B / C / D:** OTAs, single-stage op-amps, two-stage Miller op-amps, and fully differential amplifiers driving off-chip or on-chip routing loads

---

## 4. Testbench Schematic

```text
               Load Parameter Sweep Testbench
               VDD
                │
                ▼
        ┌───────────────┐
Vin ────┤+              │
        │      DUT      ├──── Vout ───┬─── [ CL Sweep: 100fF to 100pF ]
   ┌────┤-              │             │
   │    └───────┬───────┘             └─── [ RL Sweep: 1k to 10Meg ]
   │            │
   │            ▼
   │           VSS
   └──────────────────────────────────┘
```

![TB16 Load Sweep](assets/TB16_Load_Sweep.png)
*Figure 1: Closed-loop parametric sweep setup across load impedance arrays.*

---

## 5. Circuit Configuration
1. **Feedback Setup:** Place amplifier in standard closed-loop buffer or target application configuration.
2. **Capacitive Sweep Range:** Sweep $C_L$ logarithmically over at least 3 decades:
   $$C_L \in [100\,\text{fF}, 500\,\text{fF}, 1\,\text{pF}, 5\,\text{pF}, 20\,\text{pF}, 100\,\text{pF}]$$
3. **Resistive Sweep Range:** For amplifiers driving resistive loads (e.g. output buffers), sweep $R_L$:
   $$R_L \in [1\,\text{k}\Omega, 10\,\text{k}\Omega, 100\,\text{k}\Omega, 1\,\text{M}\Omega, \infty]$$

---

## 6. Simulation Type
- **SPICE Directives:**
  - `.step param CL LIST 100f 500f 1p 2p 5p 10p 20p 50p 100p` with nested `.ac`
  - `.step param CL ...` with nested `.tran`

---

## 7. Simulation Procedure
```spice
* Standard SPICE netlist syntax for TB16
.param CL_VAL=2p
VIN in 0 DC=0.9 AC=1.0
XDUT in out out vdd vss opamp_core
CLOAD out 0 {CL_VAL}

* Sweep CL across decades
.step param CL_VAL LIST 0.1p 0.5p 1p 2p 5p 10p 20p 50p 100p
.ac dec 50 1 10G
```
1. Run nested small-signal AC sweeps for each discrete $C_L$ value.
2. Extract $UGF$ and $PM$ at each step.
3. Switch source to step pulse and run nested transient sweeps to extract $SR$ and $t_s$.

---

## 8. Calculations
### Architecture-Dependent Load Scaling
- **Single-Stage OTAs (Telescopic / Folded-Cascode):**
  $$f_p = \frac{1}{2\pi R_o C_L}, \quad UGF \approx \frac{G_m}{2\pi C_L}$$
  *Behavior:* Increasing $C_L$ lowers $UGF$ and **increases** phase margin (more stable).
- **Two-Stage Miller Op-Amps:**
  $$p_1 \approx \frac{1}{2\pi g_{m2} R_1 R_2 C_c}, \quad p_2 \approx \frac{g_{m2}}{2\pi C_L}$$
  *Behavior:* Increasing $C_L$ pulls the secondary pole $p_2$ lower in frequency, **reducing** phase margin (risking instability).

---

## 9. Required Plots
- **Phase Margin vs. Load Capacitance:** $PM$ (degrees) vs. $\log_{10}(C_L)$.
- **UGF vs. Load Capacitance:** $UGF$ (Hz) vs. $\log_{10}(C_L)$.
- **Settling Time vs. Load Capacitance:** $t_s$ vs. $C_L$ showing the optimal damping minimum.
- **Gain Compression vs. Resistive Load:** $A_0$ (dB) vs. $R_L$.

---

## 10. Measurements to Save
```spice
.meas AC PM_at_1p FIND PM_val WHEN CL_VAL=1p
.meas AC PM_at_10p FIND PM_val WHEN CL_VAL=10p
.meas AC PM_at_50p FIND PM_val WHEN CL_VAL=50p
* Placeholder criteria:
* PM_at_CL_MAX >= DESIGN_SPEC_PM
```

---

## 11. PVT Considerations
- In two-stage op-amps under **SS corners**, $g_{m2}$ drops, pushing $p_2$ down in frequency and creating worst-case phase margin degradation at heavy $C_L$.
- Under fast corners (FF), higher bandwidth increases sensitivity to package lead inductance ($L_{lead}$) in series with $C_L$.

---

## 12. Post-Layout Considerations
- **Load Routing Parasitic Resistance ($ESR$):** Physical trace resistance in series with $C_L$ creates an intentional or unintentional zero $z_{ESR} = \frac{1}{2\pi R_{ESR} C_L}$ that can assist or degrade stability.
- Include parasitic inductance and series resistance in the load models for off-chip loads.

---

## 13. References
- **B. K. Ahuja**, "An Improved Frequency Compensation Technique for CMOS Operational Amplifiers," *IEEE J. Solid-State Circuits*, 1983. [DOI Link](https://doi.org/10.1109/JSSC.1983.1052012)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 10: Stability and Frequency Compensation (Capacitive Loading).

---

## 14. Related Testbenches
- **[TB03: Stability](TB03_Stability.md)** — Core stability metric extracted at nominal load.
- **[TB10: Transient](TB10_Transient.md)** — Time-domain step response evaluated during load sweeps.
- **[TB17: PVT](TB17_PVT.md)** — Combines load extremes with environmental corners.

---

## 15. Navigation

| [<< TB15: Local Loops](TB15_Local_Loop_Stability.md) | [Docs Index](index.md) | [TB17: PVT >>](TB17_PVT.md) |
