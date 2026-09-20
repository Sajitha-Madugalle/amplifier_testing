[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00: DC](TB00_DC_Operating_Point.md) | [TB01: AC](TB01_AC_Gain.md) | [TB02: Diff](TB02_Differential_Gain.md) | [TB03: Stability](TB03_Stability.md) | [TB04: CMRR](TB04_CMRR.md) | **[TB05: PSRR](TB05_PSRR.md)** | [TB06: Gm/Rout](TB06_OTA_Gm_Rout.md) | [TB07: Offset](TB07_Input_Offset.md) | [TB08: ICMR](TB08_ICMR.md) | [TB09: Swing](TB09_Output_Swing.md) | [TB10: Transient](TB10_Transient.md) | [TB11: Noise](TB11_Noise.md) | [TB12: THD](TB12_THD.md) | [TB13: FD-DM](TB13_FD_Differential_Mode.md) | [TB14: CMFB](TB14_CMFB.md) | [TB15: Local Loops](TB15_Local_Loop_Stability.md) | [TB16: Load](TB16_Load_Sweep.md) | [TB17: PVT](TB17_PVT.md) | [TB18: Monte Carlo](TB18_Monte_Carlo.md)

---

# TB05 — Power Supply Rejection Ratio (PSRR)

Characterization of supply ripple isolation across frequency for positive ($V_{DD}$) and negative ($V_{SS}$) rails.

---

## 1. Purpose
- Quantify amplifier immunity to supply noise, switching converter ripple, and rail bounce.
- Measure positive supply rejection $\text{PSRR}^+(f)$ and negative supply rejection $\text{PSRR}^-(f)$ across frequency.
- Identify supply noise coupling paths through compensation capacitors, cascode bias generators, and bulk/substrate nodes.

---

## 2. Parameters Measured
- **DC Positive PSRR ($\text{PSRR}^+_0$):** In $\text{dB}$
- **DC Negative PSRR ($\text{PSRR}^-_0$):** In $\text{dB}$
- **Supply Gain ($A_{VDD}(f), A_{VSS}(f)$):** Direct supply-to-output gain
- **PSRR Roll-Off Frequency:** $-3\,\text{dB}$ degradation point

---

## 3. Applicable Amplifiers
- **Class A:** Single-ended cascodes, regulated cascodes
- **Class B:** Differential pairs, 5-T OTAs
- **Class C:** Folded-cascode, two-stage Miller, and multistage op-amps
- **Class D:** Fully differential amplifiers

---

## 4. Testbench Schematic

```text
       PSRR+ Test Setup                           PSRR- Test Setup
    VDD = VDD_DC + 1V_AC                       VDD = VDD_DC (AC=0)
           │                                          │
           ▼                                          ▼
   ┌───────────────┐                          ┌───────────────┐
   │      DUT      ├── VOUT                   │      DUT      ├── VOUT
   └───────┬───────┘                          └───────┬───────┘
           │                                          │
           ▼                                          ▼
    VSS = VSS_DC (AC=0)                        VSS = VSS_DC + 1V_AC
    (Inputs AC Grounded)                       (Inputs AC Grounded)
```

![TB05 PSRR](assets/TB05_PSRR.png)
*Figure 1: AC test setup for positive and negative power supply rejection characterization.*

---

## 5. Circuit Configuration
1. **Signal Inputs:** AC-ground all signal inputs ($v_{in+,AC} = 0$, $v_{in-,AC} = 0$) while maintaining nominal DC common-mode voltages.
2. **PSRR+ Setup:**
   - Supply $V_{DD}$: $V_{DD,DC} + 1.0\,\text{V}_{AC}$
   - Supply $V_{SS}$: $V_{SS,DC} + 0.0\,\text{V}_{AC}$
3. **PSRR- Setup:**
   - Supply $V_{DD}$: $V_{DD,DC} + 0.0\,\text{V}_{AC}$
   - Supply $V_{SS}$: $V_{SS,DC} + 1.0\,\text{V}_{AC}$
4. **Output:** Connect nominal output RC load.

---

## 6. Simulation Type
- **SPICE Directive:** `.ac DEC <points_per_decade> <f_start> <f_stop>`
- **Sweep Range:** `0.1 Hz` to `1 GHz` with 50 points/decade.

---

## 7. Simulation Procedure
```spice
* Run 1: PSRR+ Setup
VDD vdd 0 DC=1.8 AC=1.0
VSS vss 0 DC=0.0 AC=0.0
VINP inp 0 DC=0.9 AC=0.0
VINM inm 0 DC=0.9 AC=0.0
XDUT1 inp inm outp outm vdd vss amplifier_core

* Run 2: PSRR- Setup
VDD2 vdd2 0 DC=1.8 AC=0.0
VSS2 vss2 0 DC=0.0 AC=1.0
XDUT2 inp inm outp2 outm2 vdd2 vss2 amplifier_core

.ac dec 50 0.1 1G
```
1. Solve DC operating points.
2. Perform frequency sweep for both rails.
3. Combine with differential gain $A_d(f)$ from [TB02](TB02_Differential_Gain.md).

---

## 8. Calculations
### Supply-to-Output Gain
$$
A_{VDD}(f) = \frac{V_{out}(f)}{V_{DD,AC}(f)}, \quad A_{VSS}(f) = \frac{V_{out}(f)}{V_{SS,AC}(f)}
$$

### PSRR Calculation
$$
\text{PSRR}^+(f) = 20\log_{10}\left|\frac{A_d(f)}{A_{VDD}(f)}\right| = A_{d,\text{dB}}(f) - 20\log_{10}|A_{VDD}(f)|
$$
$$
\text{PSRR}^-(f) = 20\log_{10}\left|\frac{A_d(f)}{A_{VSS}(f)}\right| = A_{d,\text{dB}}(f) - 20\log_{10}|A_{VSS}(f)|
$$

---

## 9. Required Plots
- **PSRR+ Bode Plot:** $\text{PSRR}^+(f)$ in $\text{dB}$ vs. logarithmic frequency.
- **PSRR- Bode Plot:** $\text{PSRR}^-(f)$ in $\text{dB}$ vs. logarithmic frequency.
- **Direct Gain Overlay:** Overlay $A_d(f)$, $A_{VDD}(f)$, and $A_{VSS}(f)$ to identify coupling frequencies.

---

## 10. Measurements to Save
```spice
.meas AC PSRR_plus_dc FIND vdb(psrr_plus) AT=1
.meas AC PSRR_minus_dc FIND vdb(psrr_minus) AT=1
.meas AC PSRR_plus_100k FIND vdb(psrr_plus) AT=100k
.meas AC PSRR_plus_1Meg FIND vdb(psrr_plus) AT=1Meg
* Placeholder criteria:
* PSRR_plus_dc >= DESIGN_SPEC_PSRR
* PSRR_minus_dc >= DESIGN_SPEC_PSRR
```

---

## 11. PVT Considerations
- Under **SS corners and high temperatures**, output resistance drops, degrading low-frequency PSRR.
- In two-stage Miller amplifiers, high-frequency $\text{PSRR}^+$ typically drops severely because the compensation capacitor $C_c$ couples supply ripple directly into the second stage gate.

---

## 12. Post-Layout Considerations
- **Substrate Noise Injection:** Substrate contacts and guard-ring taps must be connected directly to clean ground lines; layout parasitic resistance in ground taps reduces $\text{PSRR}^-$.
- **Capacitive Coupling:** Metal routing overlaps between supply buses and internal high-impedance nodes will bypass cascode shielding and degrade high-frequency PSRR.

---

## 13. References
- **D. B. Ribner and M. A. Copeland**, "Design Techniques for Cascoded CMOS Op Amps with Improved PSRR and Common-Mode Input Range," *IEEE J. Solid-State Circuits*, 1984. [DOI Link](https://doi.org/10.1109/JSSC.1984.1052246)
- **Behzad Razavi**, *Design of Analog CMOS Integrated Circuits*, Chapter 9: Operational Amplifiers (PSRR).

---

## 14. Related Testbenches
- **[TB02: Diff Gain](TB02_Differential_Gain.md)** — Supplies the numerator $A_d(f)$.
- **[TB04: CMRR](TB04_CMRR.md)** — Sister metric for input common-mode rejection.
- **[TB11: Noise](TB11_Noise.md)** — Complements supply noise analysis.

---

## 15. Navigation

| [<< TB04: CMRR](TB04_CMRR.md) | [Docs Index](index.md) | [TB06: Gm/Rout >>](TB06_OTA_Gm_Rout.md) |
