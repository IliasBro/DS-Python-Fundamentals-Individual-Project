# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DS Python Fundamentals Individual Project (ZHAW). Topic D2: analysing patient outcomes across three hospitals to demonstrate confounding in observational data.

The core point of the project: a naive comparison ranks Hospital_A best and Hospital_B worst, but Hospital_B treats far more high-severity patients. Adjusting for severity removes most of that gap. It does NOT cleanly reverse the ranking, and that partial result is the intended lesson about the limits of simple adjustment. See "Analytical Result" below for the exact numbers. Do not restore any claim that the ranking fully reverses or that Hospital_B becomes the best after adjustment.

Submission deadline: 06.06.2026. The notebook must run top-to-bottom without errors and be reproducible.

## Environment Setup

```bash
# Activate virtual environment (Python 3.13, .venv committed in repo)
source .venv/bin/activate

# Or install from scratch
pip install -r requirements.txt
```

Dependencies: `pandas==3.0.2`, `numpy==2.4.4`, `matplotlib==3.10.8`, `seaborn==0.13.2`, `scipy==1.17.1`, `statsmodels==0.14.6`

## Running the Notebook

```bash
source .venv/bin/activate
jupyter notebook Analysis.ipynb
```

Run all cells top-to-bottom (`Kernel -> Restart & Run All`). The notebook creates `Plots/` via `os.makedirs(..., exist_ok=True)` and saves six figures there at `dpi=150`.

## Repository Structure

| Path | Purpose |
|---|---|
| `Analysis.ipynb` | Main analysis notebook (34 cells, see below) |
| `Data/topic_D2_hospital_outcomes_raw.csv` | Synthetic dataset, 1400 patients, 9 columns |
| `Data/CreateData.ipynb` | ZHAW-provided data generation script (seed 505) |
| `Plots/` | Output figures saved by the notebook |
| `requirements.txt` | Pinned Python dependencies |
| `README.md` | Project README |

Note: the notebook reads from `Data/` and writes to `Plots/`. If either folder is missing or the CSV is not in `Data/`, the notebook will fail at load time.

## Notebook Architecture

The notebook has 34 cells. The section structure (course requirement — preserve it):

1. Problem Framing: confounding trap, analytical approach
2. Data Understanding: shape, dtypes, missing values, patient counts; ends with a pairplot coloured by severity
2.5. Variable Associations: Spearman correlation matrix (heatmap + p-value table); mixed measurement levels encoded numerically
3. Method / Approach: stratified analysis rationale + note on ANCOVA and stepwise regression (both considered and rejected)
4. Implementation (stratified analysis): four steps in sequence
   - Naive comparison (raw recovery / stay / readmission rates)
   - Population analysis (severity mix, age, comorbidity per hospital)
   - Adjusted comparison (grouped by `hospital` and `severity`)
   - Confounding quantification (naive rank vs adjusted rank)
4.1 OLS Regression: predict `length_of_stay_days`; coefficient plot; verifies data-generation parameters
4.2 Logistic Regression on `recovered`: hierarchical blocks (confounders first, hospital second); LR test; odds-ratio table
4.3 Logistic Regression on `readmitted_30d`: same hierarchical block structure applied to the second binary quality outcome; LR test; odds-ratio table
5. Results, Validation & Robustness: findings from all analyses plus reproducibility note
6. Interpretation & Critical Reflection: why stratification falls short, what regression adds, limitations
7. AI Usage Documentation: transparent log of AI tool usage (required by course)

The course requires a well-structured scripting approach (functions, clear workflow). Object-oriented design is NOT required for this topic (that applies only to Focus A topics), so do not refactor the notebook into classes.

## Data Schema

```
patient_id            str    unique ID (P2xxxxx)
hospital              str    Hospital_A / Hospital_B / Hospital_C
age                   int    18 to 95
severity              str    low / medium / high  (key confounding variable)
comorbidity_score     int    count of additional conditions (residual confounder)
treatment_type        str    standard / advanced
recovered             int    1 = recovered
length_of_stay_days   float  days in hospital
readmitted_30d        int    1 = readmitted within 30 days
```

## Analytical Result (ground truth, verified by running the data)

Use these numbers as the reference. If an edit produces different figures, the edit is wrong, not these.

Naive recovery rate (overall):
- Hospital_A 81.3%, Hospital_C 74.6%, Hospital_B 68.1%

Recovery rate within each severity group:

| Severity | Hospital_A | Hospital_B | Hospital_C |
|---|---|---|---|
| low | 92.1% | 92.9% | 92.2% |
| medium | 72.2% | 76.5% | 71.8% |
| high | 55.9% | 39.3% | 43.1% |

Adjusted ranking (unweighted mean of the three severity-group rates):
- Hospital_A 73.4%, Hospital_B 69.5%, Hospital_C 69.0%

Naive 30-day readmission rate (overall):
- Hospital_A 6.9%, Hospital_C 7.8%, Hospital_B 11.7%

Readmission rate within each severity group (%):

| Severity | Hospital_A | Hospital_B | Hospital_C |
|---|---|---|---|
| low | 1.9 | 0.6 | 2.6 |
| medium | 9.3 | 12.3 | 8.1 |
| high | 23.7 | 19.9 | 18.1 |

High-severity share of patients: Hospital_A about 12%, Hospital_B about 35%, Hospital_C about 19%.

True hospital quality effects from `CreateData.ipynb` (`hospital_effect`): A -0.01, B +0.03, C +0.01, so the true order is B > C > A. The severity-only adjustment does NOT recover this order, because comorbidity still differs within severity bands (for example, in the high group B has comorbidity 4.51 vs A 3.78) and because Hospital_A's high-severity group is small (n = 59) and noisy. This unrecovered gap is a deliberate finding, not an error to fix.

OLS regression on `length_of_stay_days` (R-squared 0.749):
- Severity high vs low: +7.14 days
- Severity medium vs low: +2.65 days
- Comorbidity score: +0.36 days per unit
- Hospital B vs A: +0.12 days (not significant, p = 0.32)
- Hospital C vs A: +0.15 days (not significant, p = 0.25)

Logistic regression on `recovered` (hierarchical):
- Block 1 pseudo-R2 (McFadden): 0.1721
- Block 2 pseudo-R2 (McFadden): 0.1722
- LR test p = 0.87 — hospital does NOT significantly improve fit after severity + comorbidity
- Hospital_B vs A: OR = 0.947 (95% CI 0.679–1.320), p = 0.75
- Hospital_C vs A: OR = 0.906 (95% CI 0.631–1.301), p = 0.59

Logistic regression on `readmitted_30d` (hierarchical, Section 4.3):
- Events: 126 / 1400 readmitted
- Block 1 pseudo-R2 (McFadden): 0.1101
- Block 2 pseudo-R2 (McFadden): 0.1108
- LR test p = 0.75 — hospital does NOT significantly improve fit after severity + comorbidity
- Severity high vs low: OR = 11.82 (p < 0.001)
- Severity medium vs low: OR = 5.86 (p < 0.001)
- Comorbidity: OR = 1.063 per unit, p = 0.26 (NOT significant for readmission — unlike for recovery)
- Hospital_B vs A: OR = 0.981 (95% CI 0.614–1.568), p = 0.94
- Hospital_C vs A: OR = 0.834 (95% CI 0.487–1.429), p = 0.51
- The descriptive within-high-severity flip (Hospital_A worst at 23.7%) does NOT survive joint adjustment for severity + comorbidity. This is a deliberate result, not an error to fix.

## Key Design Decisions

- Stratified analysis is the main story (Sections 2–4). Regression (Sections 4b–4c) extends and formalises the findings.
- ANCOVA was considered but rejected: it assumes a continuous DV; `recovered` is binary. For `length_of_stay_days` it would work but OLS is equivalent and more flexible.
- Stepwise regression was rejected: inflates Type I error and produces unstable variable sets. Hierarchical block entry is used instead.
- Three helper functions defined once (`recovery_rate`, `readmission_rate`, `avg_length_of_stay`) and reused throughout.
- Plots saved to `Plots/` at `dpi=150`. The directory is created with `os.makedirs("Plots", exist_ok=True)` near the start of the notebook.
- All results are reproducible: the dataset was generated with `np.random.seed(505)`.

## Code Style Conventions (follow these strictly)

This notebook is meant to be presented and explained live to a class. Keep it readable, not clever.

- Keep code as simple as possible. No unnecessary abstractions.
- One task per cell.
- Comments in English, short and explanatory.
- No fancy one-liners. Readable is better than compact.
- When several approaches exist, choose the simplest.
- No emojis anywhere.
- No decorative separators in output or comments (no lines of `===`, no `# --- ... ---`, no arrow glyphs). Use plain labels in `print` statements.
- In any German text, use proper umlauts (a, o, u with diaeresis) rather than ae/oe/ue, and write ss instead of the sharp-s character.
- Markdown cells and code cells stay separate; do not merge explanation into code or vice versa.

## When Editing

- Preserve the section structure listed in Notebook Architecture above.
- Keep the notebook fully executable from top to bottom.
- If you change any analysis, recompute and confirm the figures match the Analytical Result section above; if they legitimately change, update that section in the same edit.
- Do not reintroduce the "ranking fully reverses / Hospital_B is best after adjustment" claim.
- The logistic regression finding (hospital not significant after controlling for severity + comorbidity, LR p = 0.87) is a deliberate result, not an error. Do not change the model specification to force significance.