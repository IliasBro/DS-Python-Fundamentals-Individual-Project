# Analysing Patient Outcomes Across Hospitals
**DS Python Fundamentals — Individual Project (Topic D2)**

## Overview
This project analyses patient outcome data from three hospitals and asks whether a naive cross-hospital comparison can be trusted as a quality ranking.

The core finding: **Hospital_B looks worst at first glance, but treats roughly three times as many high-severity patients as Hospital_A.** After adjusting for severity within strata, most of the gap between hospitals disappears. After adjusting for severity *and* comorbidity in a hierarchical logistic regression, no statistically detectable hospital effect on recovery remains. The naive ranking is therefore misleading, and no defensible hospital quality ranking can be extracted from this observational data once case mix is properly controlled.

---

## Repository Structure

```
├── Analysis.ipynb                              # Main analysis notebook
├── Data
│   ├── CreateData.ipynb                        # Data generation script provided by ZHAW
│   └── topic_D2_hospital_outcomes_raw.csv      # Raw dataset (1400 patients)
├── Plots
│   ├── plot_adjusted_recovery.png
│   ├── plot_correlation_heatmap.png
│   ├── plot_naive_recovery.png
│   ├── plot_ols_coefficients.png
│   ├── plot_pairplot.png
│   └── plot_severity_mix.png
├── README.md
└── requirements.txt                            # Python dependencies
```

The analysis treats the CSV in `Data/` as the only input. The `CreateData.ipynb` file is part of the course materials and is not used as a source of analytical conclusions.

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
jupyter notebook Analysis.ipynb
```

Run all cells from top to bottom (`Kernel → Restart & Run All`).

---

## Dataset

The dataset contains 1400 patient records across three hospitals.

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
   - 2.1 Variable Distributions — Pairplot
   - 2.2 Variable Associations — Spearman correlations
3. **Method / Approach** — Stratified analysis plus regression, method choice guided by the [UZH Methodenberatung decision tree](https://www.methodenberatung.uzh.ch/de.html)
4. **Implementation** — Naive comparison, population analysis, severity-adjusted comparison, confounding check
   - 4.1 What Drives Length of Stay? — OLS Regression
   - 4.2 Does Hospital Choice Matter? — Hierarchical Logistic Regression
5. **Results, Validation & Robustness** — Key findings and validation checks
   - 5.1 Bootstrap Confidence Intervals for Stratified Recovery Rates
   - 5.2 Outlier Sensitivity for the OLS Length-of-Stay Model
6. **Interpretation & Critical Reflection** — Limitations and takeaways
7. **AI Usage Documentation** — Transparent documentation of AI tool usage

---

## Key Result

| Hospital | Naive recovery rate | Severity-adjusted recovery rate (unweighted mean) |
|---|---|---|
| Hospital_A | 81.3% | 73.4% |
| Hospital_B | 68.1% | 69.5% |
| Hospital_C | 74.6% | 69.0% |

> Severity confounds the naive comparison. Adjusting for severity shrinks the gap from over 13 percentage points to under 5. A hierarchical logistic regression that controls for severity and comorbidity finds no statistically detectable hospital effect on recovery (LR test p = 0.87).

---

## Dependencies

```
pandas==3.0.2
numpy==2.4.4
matplotlib==3.10.8
seaborn==0.13.2
scipy==1.17.1
statsmodels==0.14.6
```

Python 3.13
