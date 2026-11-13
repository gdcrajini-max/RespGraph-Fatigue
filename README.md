# RespGraph-Fatigue: Multi-Sensor Respiratory State Transition Graph Modeling and Stability Quantification Dataset (v1.0 / v12)

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Platform: ESP32-WROOM-32](https://img.shields.io/badge/Edge%20Hardware-ESP32--WROOM--32-blue.svg)]()
[![Framework: PyTorch 2.2 | TFLite-Micro v2.14](https://img.shields.io/badge/Framework-PyTorch%20%7C%20TFLite--Micro-orange.svg)]()

## Overview

This repository contains the synchronized multi-sensor respiratory dataset, extracted Markovian **Respiratory State Transition Graph (RSTG)** and **Respiratory Transition Stability Index (RTSI)** feature matrices, 24-fold Leave-One-Subject-Out (LOSO) cross-validation logs, and hysteresis re-arming early-warning event records supporting the research study:

> **"RespGraph-Fatigue: Multi-Sensor Respiratory State Transition Graph Modeling and Stability Quantification for Real-Time Physical Fatigue Detection and Early Onset Warning"**

---

## Experimental & Hardware Specifications

* **Ethics & Cohort:** Human-subject recordings approved by the Institutional Human Ethics Committee (**Protocol Approval No. IHEC/BME/2026/014**) across **$N = 24$ healthy adult participants** (`P01`–`P24`; 14 males, 10 females; age $26.4 \pm 4.8\text{ years}$; BMI $23.1 \pm 2.6\text{ kg/m}^2$) with written informed consent.
* **Wearable Multi-Sensor Hardware:** Synchronized **$50\text{ Hz}$** ($20.0\text{ ms}$ hardware timer ISR) chest-worn belt interfaced with an **ESP32-WROOM-32** dual-core microcontroller ($240\text{ MHz}$, $520\text{ KB}$ SRAM):
  * **6-Axis IMU (`MPU6050`):** Xiphoid-mounted tri-axial accelerometer ($A_x, A_y, A_z$, $\pm 2g$) and gyroscope ($G_x, G_y, G_z$, $\pm 250^\circ/\text{s}$) via $400\text{ kHz}$ Fast-Mode $\text{I}^2\text{C}$.
  * **Piezoresistive Force Sensor (`FlexiForce A201`):** Mid-axillary radial chest-belt contact force proxy ($P$, $0\text{–}4.4\text{ N}$, $R^2 = 0.994$ dead-weight calibration) conditioned via an `MCP6001` unity-gain op-amp buffer and passive RC anti-aliasing filter ($f_c = 15.9\text{ Hz}$) into a 12-bit SAR ADC.
* **Graded Exertion Protocol & Ground Truth:** Supervised **$40.0\text{-minute}$ ($2,400\text{ s}$)** four-stage protocol (Minutes `0–1` dedicated resting calibration; Minutes `1–40` [$39.0\text{ min}$] post-calibration evaluation) annotated every $2.0\text{ min}$ (21 query points) using the 15-point **Borg Rating of Perceived Exertion (RPE)** scale mapped into four mutually exclusive intervals:
  * **Normal:** $\text{RPE} \in [6.0, 8.5]$
  * **Mild Fatigue:** $\text{RPE} \in (8.5, 11.5]$
  * **Moderate Fatigue:** $\text{RPE} \in (11.5, 15.5]$
  * **Severe Fatigue:** $\text{RPE} \in (15.5, 20.0]$

---

## Repository File Structure

| File Name | Format / Dimensions | Description |
| :--- | :---: | :--- |
| `01_Participant_QC_and_Window_Accounting_N24.csv` | CSV ($24 \times 14$) | Participant-wise demographics, `11,208` candidate $5\text{-s}$ windows (`467`/subject), `456` QC rejections ($4.07\%$), `10,752` clean instances (`2,714` Normal / `2,675` Mild / `2,677` Moderate / `2,686` Severe), and `10,080` balanced instances (`105`/class/subject). |
| `02_24Fold_LOSO_Results_and_Transformer_Comparison.csv` | CSV ($24 \times 10$) | Subject-wise 24-fold LOSO accuracy, macro precision/recall/F1, and paired fold comparison between **RespGraph-Fatigue** ($97.20\% \pm 0.77\%$) and **Time-Series Transformer** ($95.22\% \pm 1.39\%$, Wilcoxon $W^+ = 294, p = 1.67 \times 10^{-6}$). |
| `03_Participant_Early_Warning_Onset_Alert_Logs.csv` | CSV ($24 \times 14$) | Per-participant Borg RPE onset timestamps, prospective alert trigger timestamps, lead times, event-free monitoring minutes, and hysteresis re-arming logs ($\theta_{\text{rearm}} = \theta_{\text{trig}} + 0.5\sigma_{\text{cal}}$). |
| `04_Balanced_RSTG_RTSI_22D_Feature_Dataset_N10080.csv` | CSV ($10,080 \times 34$) | Complete balanced evaluation dataset containing causal sub-frame boundary $b(t_m) = 2\lfloor t_m/2 \rfloor$, external workload covariates, the 22-D topological-stability vector $h_m = [\text{vec}(A_m), \pi, H(A_m), \text{RTSI}]$, ground-truth labels, and held-out LOSO predictions. |
| `05_Sample_50Hz_7Channel_Raw_Sensor_Stream_P01.csv` | CSV ($120,000 \times 10$) | Representative $40.0\text{-minute}$ ($50\text{ Hz}$) synchronized 7-channel sensor time series (`Ax_g`–`Gz_dps`, `Belt_Force_P_N`) with interpolated Borg RPE and ground-truth stage labels for Participant `P01`. |
| `RespGraph_Complete_Datasets_and_Paper_Audit_v12.xlsx` | XLSX (6 Tabs) | Master Excel workbook combining all tables, the aggregated $4 \times 4$ confusion matrix, comparison charts, and live Excel formulas auditing 100% agreement with the manuscript. |

---

## Feature Schema (`04_Balanced_RSTG_RTSI_22D_Feature_Dataset_N10080.csv`)

* **Metadata & Covariates:** `Instance_ID`, `Participant_ID` (`P01`–`P24`), `Timestamp_t_s` ($t_m \in [70.0, 2400.0]\text{ s}$), `Boundary_b_tm_s` ($b(t_m) = 2\lfloor t_m/2 \rfloor$), `Treadmill_Speed_kmh`, `Instant_RR_BPM`, `Interpolated_Borg_RPE`.
* **Labels & LOSO Predictions:** `True_Fatigue_Label` (`0`–`3`), `True_Fatigue_Class`, `LOSO_Predicted_Label` (`0`–`3`), `LOSO_Predicted_Class`, `Correct_Prediction` (`1` = correct, `0` = error; sum = `9,798 / 10,080`).
* **22-D Topological-Stability Vector ($h_m \in \mathbb{R}^{22}$):**
  * `RTSI`: Baseline-referenced Respiratory Transition Stability Index $\in [0, 1]$ normalized by $1/(2K)$ with $2K = 8$.
  * `Transition_Entropy_H`: Shannon transition entropy $H(A_m) = -\sum_{i,j} \pi_i a_{ij} \log_2(a_{ij})$.
  * `pi_S1_Stable`, `pi_S2_Deep`, `pi_S3_Irregular`, `pi_S4_Fatigue`: Unique Perron–Frobenius stationary state distribution $\pi \in \mathbb{R}^4$.
  * `a_11` ... `a_44`: Laplace-smoothed ($\alpha = 10^{-3}$) $4 \times 4$ row-stochastic Markov transition probabilities $\text{vec}(A_m) \in \mathbb{R}^{16}$.

---

## Summary of Benchmark Results (Reconciled with Manuscript)

| Evaluation Metric | Value |
| :--- | :--- |
| **24-Fold LOSO Classification Accuracy** | **$97.20\% \pm 0.77\%$** ($95\%\text{ CI: } [96.88\%, 97.53\%]$) |
| **24-Fold LOSO Macro F1-Score** | **$97.20\% \pm 0.76\%$** ($9,798 / 10,080$ correct windows) |
| **Mean RTSI Trajectory (Rest $\rightarrow$ Severe)** | **$0.93 \pm 0.02 \rightarrow 0.81 \pm 0.03 \rightarrow 0.68 \pm 0.04 \rightarrow 0.57 \pm 0.05$** ($p < 0.0001$) |
| **Workload-Controlled Partial Correlation** | **$\text{partial } r = -0.74$** ($p < 0.0001$, controlling for speed & RR) |
| **Moderate Onset Early Warning ($\text{RPE} > 11.5$)** | **$91.67\%$ sensitivity** ($22/24$), **$2.8 \pm 0.8\text{ min}$ lead time**, **$0.172\text{ FA/hr}$** |
| **Severe Onset Early Warning ($\text{RPE} > 15.5$)** | **$95.83\%$ sensitivity** ($23/24$), **$4.2 \pm 1.1\text{ min}$ lead time**, **$0.103\text{ FA/hr}$** |
| **On-Device INT8 Inference (`ESP32-WROOM-32`)** | **$14.8 \pm 0.6\text{ ms}$ latency**, **$42.6\text{ KB}$ peak SRAM**, **$68.40\text{ mA}$ avg current** |

---

## Quick Start (Python)

```python
import pandas as pd

# Load the 10,080-instance 22-D topological-stability dataset
df = pd.read_csv("04_Balanced_RSTG_RTSI_22D_Feature_Dataset_N10080.csv")

# Verify overall 24-fold LOSO accuracy
acc = df["Correct_Prediction"].mean() * 100
print(f"Pooled 24-Fold LOSO Accuracy: {acc:.2f}% ({df['Correct_Prediction'].sum()}/{len(df)})")

# Inspect RTSI progression across fatigue classes
print(df.groupby("True_Fatigue_Class")["RTSI"].agg(["count", "mean", "std"]))
