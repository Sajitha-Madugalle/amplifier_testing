[Home](../README.md) | [Classes](amplifier_classes.md) | [Testbench Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00](TB00_DC_Operating_Point.md) | [TB01](TB01_AC_Gain.md) | [TB02](TB02_Differential_Gain.md) | [TB03](TB03_Stability.md) | [TB04](TB04_CMRR.md) | [TB05](TB05_PSRR.md) | [TB06](TB06_OTA_Gm_Rout.md) | [TB07](TB07_Input_Offset.md) | [TB08](TB08_ICMR.md) | [TB09](TB09_Output_Swing.md) | [TB10](TB10_Transient.md) | [TB11](TB11_Noise.md) | [TB12](TB12_THD.md) | [TB13](TB13_FD_Differential_Mode.md) | [TB14](TB14_CMFB.md) | [TB15](TB15_Local_Loop_Stability.md) | [TB16](TB16_Load_Sweep.md) | [TB17](TB17_PVT.md) | [TB18](TB18_Monte_Carlo.md)

---

# CMOS Amplifier Characterization Documentation

Welcome to the standardized CMOS amplifier simulation testbench documentation suite. This library establishes a uniform, reproducible characterization methodology applicable across:

- **Schematic-level netlists** (pre-layout design phase)
- **LVS-clean extracted netlists** (device parasitical layout evaluation)
- **Post-layout PEX netlists** (full R+C/R+C+L parasitic extracted evaluation)
- **PVT Corner sweeps** (process, voltage, temperature)
- **Monte Carlo / statistical mismatch sweeps**

---

## Quick Navigation

| Document | Description |
|---|---|
| [Amplifier Classification](amplifier_classes.md) | Class A, B, C, and D topologies and their specific characterization requirements |
| [Testbench Matrix](testbench_matrix.md) | Complete topology-to-testbench cross-reference matrix and post-layout comparison rules |
| [Academic References](references.md) | Textbook, journal, and seminal paper bibliography with links |

---

## Testbench Library Index

| ID | Testbench Name | Key Parameters Measured | Primary Simulation |
|:---|:---|:---|:---|
| [TB00](TB00_DC_Operating_Point.md) | **DC Operating Point & Power** | $I_{DD}$, $I_{SS}$, $P_{DC}$, operating regions, $V_{DSAT}$, $g_m$, $g_{ds}$ | `.op` |
| [TB01](TB01_AC_Gain.md) | **Open-Loop AC Gain** | Low-frequency gain ($A_v$), -3 dB BW, unity-gain frequency (UGF), poles/zeros | `.ac` |
| [TB02](TB02_Differential_Gain.md) | **Differential AC Gain** | Differential gain ($A_d$), GBW, differential bandwidth, phase | `.ac` |
| [TB03](TB03_Stability.md) | **Loop Gain & Stability** | Loop gain $T(j\omega)$, Phase Margin (PM), Gain Margin (GM), $f_{ug}$ | `.ac` (Return-Ratio) |
| [TB04](TB04_CMRR.md) | **Common-Mode Rejection Ratio (CMRR)** | $A_d(f)$, $A_{CM}(f)$, $\text{CMRR}(f)$, DC CMRR | `.ac` |
| [TB05](TB05_PSRR.md) | **Power Supply Rejection Ratio (PSRR)** | $\text{PSRR}^+(f)$, $\text{PSRR}^-(f)$, $A_{VDD}(f)$, $A_{VSS}(f)$ | `.ac` |
| [TB06](TB06_OTA_Gm_Rout.md) | **OTA Transconductance & Output Resistance** | Transconductance $G_m(f)$, output impedance $R_o(f)$, intrinsic gain $G_m R_o$ | `.ac` |
| [TB07](TB07_Input_Offset.md) | **Input Offset Voltage** | Systematic offset $V_{OS,sys}$, mismatch offset $\mu$, $\sigma$, $3\sigma$ | `.dc` / `.mc` |
| [TB08](TB08_ICMR.md) | **Input Common-Mode Range (ICMR)** | $V_{ICMR,min}$, $V_{ICMR,max}$, linear buffer tracking, $G_m(V_{CM})$ | `.dc` / `.ac` |
| [TB09](TB09_Output_Swing.md) | **Output Voltage Swing** | $V_{out,min}$, $V_{out,max}$, peak-to-peak swing $V_{swing}$, compression | `.dc` |
| [TB10](TB10_Transient.md) | **Slew Rate & Settling Time** | Slew rate ($SR^+$, $SR^-$), settling time ($t_s$ at 1%, 0.1%), overshoot | `.tran` |
| [TB11](TB11_Noise.md) | **Noise Analysis** | Input-referred noise $e_{n,in}(f)$, thermal floor, $1/f$ corner, integrated RMS noise | `.noise` |
| [TB12](TB12_THD.md) | **Total Harmonic Distortion (THD)** | THD, HD2, HD3, spurious components, linearity vs. amplitude | `.tran` + FFT |
| [TB13](TB13_FD_Differential_Mode.md) | **Fully Differential DM Characterization** | Differential signal path gain, phase margin, transient response | `.ac` / `.tran` |
| [TB14](TB14_CMFB.md) | **CMFB Loop Characterization** | Common-mode loop gain $T_{CM}$, UGF$_{CM}$, PM$_{CM}$, GM$_{CM}$, $V_{OCM}$ error | `.ac` / `.tran` |
| [TB15](TB15_Local_Loop_Stability.md) | **Local Loop & Booster Stability** | Regulated cascode stability, booster amplifier loop gain, PM$_{local}$ | `.ac` (Return-Ratio) |
| [TB16](TB16_Load_Sweep.md) | **Load Characterization** | Stability and transient metrics vs. capacitive ($C_L$) and resistive ($R_L$) load | `.ac` / `.tran` sweep |
| [TB17](TB17_PVT.md) | **PVT Corner Characterization** | Performance envelopes over process (TT/FF/SS/FS/SF), $V_{DD}$, and temperature | Batch Corner Analysis |
| [TB18](TB18_Monte_Carlo.md) | **Monte Carlo & Statistical Mismatch** | Statistical yield, $\mu \pm 3\sigma$ distributions for offset, gain, and bandwidth | `.mc` Statistical Run |

---

## Characterization Flow Architecture

```text
Amplifier Design / Netlist (Schematic / LVS / PEX)
                    │
                    ▼
     Classify Amplifier Architecture (Class A / B / C / D)
                    │
                    ▼
         Select Applicable Testbenches
                    │
                    ▼
   ┌──────────────────────────────────────────────────┐
   │ Core Characterization Suite                      │
   │ ├── DC Operating Point (TB00)                    │
   │ ├── Small-Signal AC & Stability (TB01 - TB06)    │
   │ ├── Input / Output Dynamic Range (TB07 - TB09)   │
   │ ├── Large-Signal Transient & Noise (TB10 - TB12) │
   │ └── Architecture Loops: CMFB & Boosters (13 - 15)│
   └──────────────────────────────────────────────────┘
                    │
                    ▼
   ┌──────────────────────────────────────────────────┐
   │ Robustness & Environmental Sweeps                │
   │ ├── Reactive / Resistive Loading (TB16)          │
   │ ├── PVT Variations (TB17)                        │
   │ └── Monte Carlo Statistical Variations (TB18)   │
   └──────────────────────────────────────────────────┘
                    │
                    ▼
       Consolidated Characterization Dataset
                    │
                    ▼
       Post-Layout Degradation Analysis (PEX vs. Schematic)
```

---

[Next: Amplifier Classes >>](amplifier_classes.md)
