[Home](../README.md) | [Index](index.md) | [Matrix](testbench_matrix.md) | [References](references.md)  
**Testbenches:** [TB00](TB00_DC_Operating_Point.md) | [TB01](TB01_AC_Gain.md) | [TB02](TB02_Differential_Gain.md) | [TB03](TB03_Stability.md) | [TB04](TB04_CMRR.md) | [TB05](TB05_PSRR.md) | [TB06](TB06_OTA_Gm_Rout.md) | [TB07](TB07_Input_Offset.md) | [TB08](TB08_ICMR.md) | [TB09](TB09_Output_Swing.md) | [TB10](TB10_Transient.md) | [TB11](TB11_Noise.md) | [TB12](TB12_THD.md) | [TB13](TB13_FD_Differential_Mode.md) | [TB14](TB14_CMFB.md) | [TB15](TB15_Local_Loop_Stability.md) | [TB16](TB16_Load_Sweep.md) | [TB17](TB17_PVT.md) | [TB18](TB18_Monte_Carlo.md)

---

# Amplifier Classification

To standardize characterization, amplifiers are categorized into four distinct architectural classes based on their terminal interfaces, signal paths, and feedback topologies.

---

## Class A — Single-Input / Single-Output Gain Stages

Single-ended input and single-ended output amplifiers typically used as basic gain cells, pre-amplifiers, or buffer building blocks.

### Architectures Covered
- Standard Cascode amplifier (telescopic cascode)
- Wide-swing cascode amplifier
- Regulated cascode amplifier (gain-boosted cascode)

### Mandatory Testbenches
- **[TB00](TB00_DC_Operating_Point.md)** — DC Operating Point & Power
- **[TB01](TB01_AC_Gain.md)** — Open-Loop AC Gain
- **[TB05](TB05_PSRR.md)** — PSRR
- **[TB09](TB09_Output_Swing.md)** — Output Swing
- **[TB11](TB11_Noise.md)** — Noise
- **[TB12](TB12_THD.md)** — Total Harmonic Distortion (THD)

### Topology-Specific Requirements
- **Regulated Cascode Stages**: Must additionally be characterized using **[TB15](TB15_Local_Loop_Stability.md)** (Local Loop Stability) to verify the feedback stability of the auxiliary boosting amp.

---

## Class B — Differential-Input / Single-Ended Amplifiers

Circuits with a differential input pair and a single-ended output. These include basic input stages and unbuffered operational transconductance amplifiers (OTAs).

### Architectures Covered
- NMOS differential pair with resistive/current-source loads
- PMOS differential pair with resistive/current-source loads
- Active-load (current-mirror) differential pair
- Complementary (rail-to-rail) differential pair
- 5-transistor OTA (5-T OTA)
- Symmetric current-mirror OTA

### Mandatory Testbenches
- **[TB00](TB00_DC_Operating_Point.md)** — DC Operating Point & Power
- **[TB02](TB02_Differential_Gain.md)** — Differential AC Gain
- **[TB04](TB04_CMRR.md)** — Common-Mode Rejection Ratio (CMRR)
- **[TB05](TB05_PSRR.md)** — PSRR
- **[TB06](TB06_OTA_Gm_Rout.md)** — OTA Transconductance & Output Resistance
- **[TB07](TB07_Input_Offset.md)** — Input Offset
- **[TB08](TB08_ICMR.md)** — Input Common-Mode Range
- **[TB11](TB11_Noise.md)** — Noise
- **[TB12](TB12_THD.md)** — THD & Linearity

### Topology-Specific Requirements
- **Complementary (Rail-to-Rail) Differential Pairs**: Must plot transconductance handover $G_m(V_{CM})$ across the full supply range to evaluate NMOS/PMOS transition flat-zone variation.

---

## Class C — OTAs and Single-Ended Op-Amps

High-gain operational transconductance amplifiers and operational amplifiers intended for closed-loop feedback operation with single-ended output.

### Architectures Covered
- Telescopic OTA
- Folded-cascode OTA
- Gain-boosted telescopic/folded-cascode OTA
- Single-stage op-amp
- Two-stage Miller-compensated op-amp
- Three-stage nested-feedback op-amp
- Gain-boosted multi-stage op-amp

### Mandatory Testbenches
- **[TB00](TB00_DC_Operating_Point.md)** — DC Operating Point & Power
- **[TB02](TB02_Differential_Gain.md)** — Differential AC Gain
- **[TB03](TB03_Stability.md)** — Loop Gain, Phase Margin & Gain Margin
- **[TB04](TB04_CMRR.md)** — CMRR
- **[TB05](TB05_PSRR.md)** — PSRR
- **[TB07](TB07_Input_Offset.md)** — Input Offset
- **[TB08](TB08_ICMR.md)** — Input Common-Mode Range
- **[TB09](TB09_Output_Swing.md)** — Output Swing
- **[TB10](TB10_Transient.md)** — Slew Rate & Settling Time
- **[TB11](TB11_Noise.md)** — Noise
- **[TB12](TB12_THD.md)** — THD & Linearity

### Topology-Specific Requirements
- **Gain-Boosted Amplifiers**: Require **[TB15](TB15_Local_Loop_Stability.md)** for booster loop stability.
- **Three-Stage Op-Amps**: Require internal nested-loop stability analysis.

---

## Class D — Fully Differential Amplifiers

Differential-input, differential-output architectures requiring common-mode feedback (CMFB) circuitry to define and stabilize the output common-mode level.

### Architectures Covered
- Fully differential telescopic amplifier
- Fully differential folded-cascode amplifier
- Fully differential two-stage amplifier
- Fully differential amplifier with continuous-time or switched-capacitor CMFB

### Mandatory Testbenches
- **[TB00](TB00_DC_Operating_Point.md)** — DC Operating Point & Power
- **[TB02](TB02_Differential_Gain.md)** — Differential AC Gain
- **[TB04](TB04_CMRR.md)** — CMRR ($A_{CM \rightarrow DM}$)
- **[TB05](TB05_PSRR.md)** — PSRR
- **[TB07](TB07_Input_Offset.md)** — Input Offset
- **[TB08](TB08_ICMR.md)** — Input Common-Mode Range
- **[TB09](TB09_Output_Swing.md)** — Differential Output Swing
- **[TB10](TB10_Transient.md)** — Large-Signal Differential Slew Rate & Settling
- **[TB11](TB11_Noise.md)** — Noise
- **[TB12](TB12_THD.md)** — THD
- **[TB13](TB13_FD_Differential_Mode.md)** — Fully Differential DM Signal Path & Stability
- **[TB14](TB14_CMFB.md)** — CMFB Loop Gain, Bandwidth, and Stability

### Dual-Loop Verification Rule
For all Class D circuits, two independent feedback loops must be characterized and confirmed stable under identical loading:
1. **Differential-Mode Loop**: Governs differential signal transfer and differential phase margin.
2. **Common-Mode Feedback Loop**: Governs output common-mode stability, CMRR, and common-mode rejection.

---

## Navigation

| [<< Docs Index](index.md) | [Testbench Matrix >>](testbench_matrix.md) |
