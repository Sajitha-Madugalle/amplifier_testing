[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [References](references.md)  
**Testbenches:** [TB00](TB00_DC_Operating_Point.md) | [TB01](TB01_AC_Gain.md) | [TB02](TB02_Differential_Gain.md) | [TB03](TB03_Stability.md) | [TB04](TB04_CMRR.md) | [TB05](TB05_PSRR.md) | [TB06](TB06_OTA_Gm_Rout.md) | [TB07](TB07_Input_Offset.md) | [TB08](TB08_ICMR.md) | [TB09](TB09_Output_Swing.md) | [TB10](TB10_Transient.md) | [TB11](TB11_Noise.md) | [TB12](TB12_THD.md) | [TB13](TB13_FD_Differential_Mode.md) | [TB14](TB14_CMFB.md) | [TB15](TB15_Local_Loop_Stability.md) | [TB16](TB16_Load_Sweep.md) | [TB17](TB17_PVT.md) | [TB18](TB18_Monte_Carlo.md)

---

# Amplifier-to-Testbench Matrix & Post-Layout Workflow

This matrix cross-references amplifier architectures against the standardized characterization suite, indicating which tests are mandatory, conditional, or not applicable.

---

## Compatibility Matrix

| Amplifier Architecture | DC<br>[TB00](TB00_DC_Operating_Point.md) | AC<br>[TB01](TB01_AC_Gain.md) | Diff<br>[TB02](TB02_Differential_Gain.md) | Stab<br>[TB03](TB03_Stability.md) | CMRR<br>[TB04](TB04_CMRR.md) | PSRR<br>[TB05](TB05_PSRR.md) | Gm/Ro<br>[TB06](TB06_OTA_Gm_Rout.md) | VOS<br>[TB07](TB07_Input_Offset.md) | ICMR<br>[TB08](TB08_ICMR.md) | Swing<br>[TB09](TB09_Output_Swing.md) | Tran<br>[TB10](TB10_Transient.md) | Noise<br>[TB11](TB11_Noise.md) | THD<br>[TB12](TB12_THD.md) | FD-DM<br>[TB13](TB13_FD_Differential_Mode.md) | CMFB<br>[TB14](TB14_CMFB.md) | Booster<br>[TB15](TB15_Local_Loop_Stability.md) |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Cascode (Class A)** | ✓ | ✓ | — | — | — | ✓ | ○ | — | — | ✓ | ○ | ✓ | ✓ | — | — | — |
| **Wide-Swing Cascode (Class A)** | ✓ | ✓ | — | — | — | ✓ | ○ | — | — | ✓ | ○ | ✓ | ✓ | — | — | — |
| **Regulated Cascode (Class A)** | ✓ | ✓ | — | ○ | — | ✓ | ○ | — | — | ✓ | ○ | ✓ | ✓ | — | — | ✓ |
| **NMOS Diff Pair (Class B)** | ✓ | — | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ○ | ○ | ✓ | ✓ | — | — | — |
| **PMOS Diff Pair (Class B)** | ✓ | — | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ○ | ○ | ✓ | ✓ | — | — | — |
| **Active-Load Diff Pair (Class B)** | ✓ | — | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | — | — | — |
| **Complementary Diff Pair (Class B)** | ✓ | — | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | — | — | — |
| **5-T OTA (Class B/C)** | ✓ | — | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Current-Mirror OTA (Class B/C)** | ✓ | — | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Telescopic OTA (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Folded-Cascode OTA (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Gain-Boosted OTA (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | ✓ |
| **Single-Stage Op-Amp (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Two-Stage Miller Op-Amp (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | — |
| **Three-Stage Op-Amp (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | ○ |
| **Gain-Boosted Op-Amp (Class C)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | — | ✓ |
| **FD Telescopic (Class D)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ |
| **FD Folded Cascode (Class D)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ |
| **FD Two-Stage (Class D)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ |
| **FD Op-Amp + CMFB (Class D)** | ✓ | — | ✓ | ✓ | ✓ | ✓ | ○ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ○ |

### Legend
- `✓` — **Mandatory**: Core baseline characterization test for this architecture.
- `○` — **Conditional**: Required when specific operational features (e.g. feedback configurations, gain boosters, or reactive loads) are present.
- `—` — **Not Applicable**: Testbench is structurally incompatible or redundant for this topology.

*Note:* [TB16 (Load Sweeps)](TB16_Load_Sweep.md), [TB17 (PVT)](TB17_PVT.md), and [TB18 (Monte Carlo)](TB18_Monte_Carlo.md) are wrapper testbenches applied across all active testbenches.

---

## Post-Layout Characterization Workflow

Every characterization run must be executed across three netlist representations to track layout degradation:

1. **Schematic-Level Netlist (`_schematic`)**: Pre-layout baseline; ideal interconnects with intrinsic device models.
2. **LVS-Clean Extracted Netlist (`_LVS`)**: Includes layout geometry, proximity effects (WPE, PSE), and diffusion sharing.
3. **PEX Netlist (`_PEX`)**: Complete post-layout parasitic extraction containing parasitic resistances ($R$), capacitances ($C$), and optional high-frequency inductances ($L$).

### Standardized Parameter Naming Convention

All simulation outputs and script extraction tables should preserve netlist origin prefixes:

```text
A0_schematic          A0_LVS          A0_PEX
UGF_schematic         UGF_LVS         UGF_PEX
PM_schematic          PM_LVS          PM_PEX
CMRR_schematic        CMRR_LVS        CMRR_PEX
PSRR_schematic        PSRR_LVS        PSRR_PEX
Power_schematic       Power_LVS       Power_PEX
```

### Percentage Degradation Extraction

Quantify post-layout impact using relative deviation:

$$
\Delta X_{\%} = \frac{X_{\text{PEX}} - X_{\text{schematic}}}{X_{\text{schematic}}} \times 100\%
$$

> [!WARNING]
> For logarithmic figures such as Gain ($A_0$, dB), CMRR (dB), and PSRR (dB), compute differences linearly before percentage calculation, or report absolute dB drops:
> $$\Delta A_{0,\text{dB}} = A_{0,\text{PEX}}(\text{dB}) - A_{0,\text{schematic}}(\text{dB})$$

---

## Navigation

| [<< Amplifier Classes](amplifier_classes.md) | [References >>](references.md) |
