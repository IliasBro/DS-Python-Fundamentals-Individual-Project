# D2 — Analysing Patient Outcomes Across Hospitals
**DS Python Fundamentals — Individual Project**

## Overview
This project analyses patient outcome data from three hospitals and investigates whether naive comparisons between hospitals are valid and meaningful.

The core finding: **Hospital_B looks worst at first glance — but treats twice as many high-severity patients as Hospital_A.** After adjusting for patient severity, the picture reverses. This is a textbook example of confounding in observational data.

---

## Repository Structure

```
├── Analysis.ipynb                              # Main analysis notebook
├── Data
│   ├── CreateData.ipynb                        # Data generation script (provided by ZHAW)
│   └── topic_D2_hospital_outcomes_raw.csv      # Raw dataset (1400 patients)
├── Plots
│   ├── plot_adjusted_recovery.png
│   ├── plot_naive_recovery.png
│   └── plot_severity_mix.png
├── README.md
└── requirements.txt                            # Python dependencies
```

---

## How to Run

**1. Clone the repository**
```bash
git clone https://github.com/IliasBro/DS-Python-Fundamentals-Individual-Project
cd DS-Python-Fundamentals-Individual-Project
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Open the notebook**
```bash
jupyter notebook D2_Hospital_Outcomes.ipynb
```

Run all cells from top to bottom (`Kernel → Restart & Run All`).

---

## Dataset

The dataset was synthetically generated using `CreateData.ipynb` (fixed seed `np.random.seed(505)` — results are fully reproducible).

| Column | Description |
|---|---|
| `patient_id` | Unique patient identifier |
| `hospital` | Hospital_A, Hospital_B, or Hospital_C |
| `age` | Patient age (18–95) |
| `severity` | Illness severity: low / medium / high |
| `comorbidity_score` | Number of additional conditions |
| `treatment_type` | standard or advanced |
| `recovered` | 1 = recovered, 0 = did not recover |
| `length_of_stay_days` | Days in hospital |
| `readmitted_30d` | 1 = readmitted within 30 days |

---

## Notebook Structure

1. **Problem Framing** — Define the task and the confounding trap
2. **Data Understanding** — Dataset structure and key characteristics
3. **Method / Approach** — Stratified analysis explained
4. **Implementation** — Naive comparison → population analysis → adjusted comparison
5. **Results, Validation & Robustness** — Key findings and validation checks
6. **Interpretation & Critical Reflection** — Limitations and takeaways
7. **AI Usage Documentation** — Transparent documentation of AI tool usage

---

## Key Result

| | Naive Ranking | Adjusted Ranking |
|---|---|---|
| 🥇 | Hospital_A | Hospital_B |
| 🥈 | Hospital_C | Hospital_C |
| 🥉 | Hospital_B | Hospital_A |

> Severity is a confounding variable. Without adjusting for it, the hospital ranking is misleading.

---

## Dependencies

```
pandas==3.0.2
numpy==2.4.4
matplotlib==3.10.8
seaborn==0.13.2
```

Python 3.12