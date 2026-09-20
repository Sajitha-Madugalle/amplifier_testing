[Home](../README.md) | [Index](index.md) | [Classes](amplifier_classes.md) | [Matrix](testbench_matrix.md)  
**Testbenches:** [TB00](TB00_DC_Operating_Point.md) | [TB01](TB01_AC_Gain.md) | [TB02](TB02_Differential_Gain.md) | [TB03](TB03_Stability.md) | [TB04](TB04_CMRR.md) | [TB05](TB05_PSRR.md) | [TB06](TB06_OTA_Gm_Rout.md) | [TB07](TB07_Input_Offset.md) | [TB08](TB08_ICMR.md) | [TB09](TB09_Output_Swing.md) | [TB10](TB10_Transient.md) | [TB11](TB11_Noise.md) | [TB12](TB12_THD.md) | [TB13](TB13_FD_Differential_Mode.md) | [TB14](TB14_CMFB.md) | [TB15](TB15_Local_Loop_Stability.md) | [TB16](TB16_Load_Sweep.md) | [TB17](TB17_PVT.md) | [TB18](TB18_Monte_Carlo.md)

---

# Primary Academic & Technical References

A curated compilation of standard textbooks, seminal IEEE journal papers, and industry guides supporting the testbench architectures and characterization methodologies.

---

## Standard Reference Textbooks

### Behzad Razavi — Design of Analog CMOS Integrated Circuits
- **Citation:** B. Razavi, *Design of Analog CMOS Integrated Circuits*, 2nd ed., McGraw-Hill Education, 2016.
- **Resource Link:** [McGraw-Hill Catalog](https://www.mheducation.com/highered/product/design-of-analog-cmos-integrated-circuits-razavi)
- **Applicable Testbenches:**
  - [TB01 (AC Gain)](TB01_AC_Gain.md) — Single-stage gain, bandwidth, frequency poles and zeros
  - [TB02 (Diff Gain)](TB02_Differential_Gain.md) & [TB04 (CMRR)](TB04_CMRR.md) — Differential pairs and common-mode analysis
  - [TB05 (PSRR)](TB05_PSRR.md) — Power supply rejection mechanisms in cascode and differential topologies
  - [TB06 (Gm & Rout)](TB06_OTA_Gm_Rout.md) — Intrinsic transconductance, output resistance, and cascode impedance
  - [TB08 (ICMR)](TB08_ICMR.md) & [TB09 (Swing)](TB09_Output_Swing.md) — Saturation boundaries, headroom constraints, and rail limits
  - [TB10 (Transient)](TB10_Transient.md) — Large-signal slewing mechanisms and compensation
  - [TB11 (Noise)](TB11_Noise.md) & [TB12 (THD)](TB12_THD.md) — MOS thermal/flicker noise, non-linearity, and harmonic distortion
  - [TB14 (CMFB)](TB14_CMFB.md) & [TB15 (Local Loops)](TB15_Local_Loop_Stability.md) — Common-mode feedback networks and gain-boosting stability

---

### Phillip E. Allen & Douglas R. Holberg — CMOS Analog Circuit Design
- **Citation:** P. E. Allen and D. R. Holberg, *CMOS Analog Circuit Design*, 3rd ed., Oxford University Press, 2011.
- **Course Notes & Measurement Guide:** [Allen Op-Amp Simulation & Measurement Lecture Notes](https://pallen.ece.gatech.edu/Academic/ECE_6412/Spring_2003/L240-Sim%26MeasofOpAmps%282UP%29.pdf)
- **Applicable Testbenches:**
  - [TB00 (DC Operating Point)](TB00_DC_Operating_Point.md) — Quiescent power, bias current extraction, and device operating points
  - [TB04 (CMRR)](TB04_CMRR.md) & [TB05 (PSRR)](TB05_PSRR.md) — Balanced injection schemes for high-gain op-amps
  - [TB07 (Input Offset)](TB07_Input_Offset.md) — High-gain closed-loop offset extraction schemes
  - [TB08 (ICMR)](TB08_ICMR.md) — Unity-gain buffer follower sweep methodology
  - [TB10 (Transient)](TB10_Transient.md) — Pulse slewing and settling time definitions (1%, 0.1%, 0.01%)

---

## Seminal Papers & IEEE Journals

### Gray & Meyer — MOS Operational Amplifier Design: A Tutorial Overview (1982)
- **Citation:** P. R. Gray and R. G. Meyer, "MOS Operational Amplifier Design — A Tutorial Overview," *IEEE Journal of Solid-State Circuits*, vol. 17, no. 6, pp. 969-982, Dec. 1982.
- **DOI / Link:** [10.1109/JSSC.1982.1051851](https://doi.org/10.1109/JSSC.1982.1051851)
- **Applicable Topics:** Foundational MOS differential stages, folded-cascode topologies, input offset analysis, and noise limitations.

---

### R. D. Middlebrook — Measurement of Loop Gain in Feedback Systems (1975)
- **Citation:** R. D. Middlebrook, "Measurement of Loop Gain in Feedback Systems," *International Journal of Electronics*, vol. 38, no. 4, pp. 485-512, 1975.
- **DOI / Link:** [10.1080/00207217508920421](https://doi.org/10.1080/00207217508920421)
- **Applicable Topics:** Return-ratio loop breaking, closed-loop DC preservation, bidirectional AC injection, and feedback loop gain extraction used in [TB03 (Stability)](TB03_Stability.md).

---

### Neag et al. — Simulation Methods for Deriving Phase and Gain Margins (2014)
- **Citation:** H. Neag, M. Neag, et al., "Comparative Analysis of Simulation-Based Methods for Deriving the Phase- and Gain-Margins of Feedback Circuits With Op-Amps," *IEEE Transactions on Circuits and Systems I*, vol. 62, no. 2, pp. 367-377, Feb. 2015.
- **DOI / Link:** [10.1109/TCSI.2014.2370151](https://doi.org/10.1109/TCSI.2014.2370151)
- **Applicable Topics:** Rigorous assessment of Middlebrook, Tian, and replica-loop methods; high-frequency loading corrections for stability testing in [TB03](TB03_Stability.md).

---

### Hurst & Lewis — Balanced Fully Differential Stability (1995)
- **Citation:** P. J. Hurst and S. H. Lewis, "Determination of Stability Using Return Ratios in Balanced Fully Differential Feedback Circuits," *IEEE Transactions on Circuits and Systems II*, vol. 42, no. 12, pp. 805-817, Dec. 1995.
- **DOI / Link:** [10.1109/82.476178](https://doi.org/10.1109/82.476178)
- **Applicable Topics:** Separation and simultaneous stability verification of differential-mode signal loops and common-mode feedback loops in [TB13](TB13_FD_Differential_Mode.md) and [TB14](TB14_CMFB.md).

---

### Ribner & Copeland — Cascoded CMOS Op-Amps with Improved PSRR (1984)
- **Citation:** D. B. Ribner and M. A. Copeland, "Design Techniques for Cascoded CMOS Op Amps with Improved PSRR and Common-Mode Input Range," *IEEE Journal of Solid-State Circuits*, vol. 19, no. 6, pp. 919-925, Dec. 1984.
- **DOI / Link:** [10.1109/JSSC.1984.1052246](https://doi.org/10.1109/JSSC.1984.1052246)
- **Applicable Topics:** Power-supply coupling paths through compensation capacitors and bias mirrors evaluated in [TB05](TB05_PSRR.md).

---

### B. K. Ahuja — Improved Frequency Compensation for CMOS Op-Amps (1983)
- **Citation:** B. K. Ahuja, "An Improved Frequency Compensation Technique for CMOS Operational Amplifiers," *IEEE Journal of Solid-State Circuits*, vol. 18, no. 6, pp. 629-633, Dec. 1983.
- **DOI / Link:** [10.1109/JSSC.1983.1052012](https://doi.org/10.1109/JSSC.1983.1052012)
- **Applicable Topics:** Cascode Miller compensation, right-half-plane zero elimination, and large capacitive load stability in [TB16](TB16_Load_Sweep.md).

---

### Tian et al. — Striving for Small-Signal Stability (2001)
- **Citation:** M. Tian, V. Visvanathan, J. Hantgan, and K. Kundert, "Striving for Small-Signal Stability," *IEEE Circuits and Devices Magazine*, vol. 17, no. 1, pp. 31-41, Jan. 2001.
- **Resource Link:** [Kundert Consulting CD2001-01 PDF](https://kenkundert.com/docs/cd2001-01.pdf)
- **Applicable Topics:** Bilateral loop gain extraction and local feedback loops in gain-boosted cascodes in [TB15](TB15_Local_Loop_Stability.md).

---

## Navigation

| [<< Testbench Matrix](testbench_matrix.md) | [TB00: DC Operating Point >>](TB00_DC_Operating_Point.md) |
