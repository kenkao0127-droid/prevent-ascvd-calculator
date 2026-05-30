# PREVENT ASCVD Risk Calculator

Build a single-page HTML/CSS/JS calculator implementing the AHA PREVENT 2023 equations.

## Requirements
- Single `index.html` file (inline CSS + JS, no external dependencies)
- Traditional Chinese UI (繁體中文)
- Mobile-friendly responsive design
- Beautiful medical-grade UI with clean typography

## Input Parameters
- Age (30-79)
- Sex (Male/Female)
- Total Cholesterol (mg/dL)
- HDL Cholesterol (mg/dL)
- Systolic Blood Pressure (mmHg)
- BMI (18.5-39.9)
- eGFR (15-140) — with optional CKD-EPI calculator (age, sex, creatinine)
- Diabetes (Yes/No)
- Current Smoker (Yes/No)
- On Antihypertensive (Yes/No)
- On Statin (Yes/No)

## Optional Parameters
- HbA1c (%)
- Urine Albumin-Creatinine Ratio (UACR, mg/g)

## Output
- 10-year ASCVD Risk (%)
- 10-year Heart Failure Risk (%)
- 10-year Total CVD Risk (%)
- Risk category (Low <3%, Borderline 3-5%, Intermediate 5-10%, High ≥10%)
- Visual risk gauge/bar

## PREVENT Equation Implementation

The formula uses logistic regression:
risk = exp(log_odds) / (1 + exp(log_odds)) * 100

### Variable Transformations in calcRiskLogit(n):
```
RL = coeff[0][n] * (age - 55) / 10 +
     coeff[1][n] * ((age - 55) / 10)^2 +
     coeff[2][n] * ((total_chol - hdl_chol) * 0.02586 - 3.5) +
     coeff[3][n] * (hdl_chol * 0.02586 - 1.3) / 0.3 +
     coeff[4][n] * (min(sbp, 110) - 110) / 20 +
     coeff[5][n] * (max(sbp, 110) - 130) / 20 +
     coeff[6][n] * diabetes +
     coeff[7][n] * smoker +
     coeff[8][n] * (min(bmi, 30) - 25) / 5 +
     coeff[9][n] * (max(bmi, 30) - 30) / 5 +
     coeff[10][n] * (min(egfr, 60) - 60) / -15 +
     coeff[11][n] * (max(egfr, 60) - 90) / -15 +
     coeff[12][n] * on_bp_med +
     coeff[13][n] * on_statin +
     coeff[14][n] * on_bp_med * (max(sbp, 110) - 130) / 20 +
     coeff[15][n] * on_statin * ((total_chol - hdl_chol) * 0.02586 - 3.5) +
     coeff[16][n] * (age - 55) / 10 * ((total_chol - hdl_chol) * 0.02586 - 3.5) +
     coeff[17][n] * (age - 55) / 10 * (hdl_chol * 0.02586 - 1.3) / 0.3 +
     coeff[18][n] * (age - 55) / 10 * (max(sbp, 110) - 130) / 20 +
     coeff[19][n] * (age - 55) / 10 * diabetes +
     coeff[20][n] * (age - 55) / 10 * smoker +
     coeff[21][n] * (max(bmi, 30) - 30) / 5 +
     coeff[22][n] * (age - 55) / 10 * (min(egfr, 60) - 60) / -15 +
     coeff[31][n];  // intercept
```

### Optional factors (added to RL):
- If no HbA1c: a1c_factor = coeff[30][n]
- If HbA1c + diabetes: a1c_factor = (hba1c - 5.3) * coeff[28][n]
- If HbA1c + no diabetes: a1c_factor = (hba1c - 5.3) * coeff[29][n]
- If no UACR: uacr_factor = coeff[27][n]
- If UACR: uacr_factor = ln(uacr) * coeff[26][n]

### Model Selection (index n into coefficient arrays):
- Full model (all optional): n = 0+sex for CVD, 2+sex for ASCVD, 4+sex for HF
- HbA1c only: n = 12+sex, 14+sex, 16+sex
- UACR only: n = 24+sex, 26+sex, 28+sex
- Base model (no optionals): n = 48+sex, 50+sex, 52+sex

### eGFR Calculation (CKD-EPI 2021):
- Female: alpha=-0.241, kappa=0.7, SexFactor=1
- Male: alpha=-0.302, kappa=0.9, SexFactor=1
- eGFR = 142 * min(creatinine/kappa, 1)^alpha * max(creatinine/kappa, 1)^-1.2 * 0.9938^age * SexFactor
- For females (not black, since PREVENT is race-free): multiply by 1.012

## Risk Categories (2026 ACC/AHA)
- Low: <3%
- Borderline: 3% to <5%
- Intermediate: 5% to <10%
- High: ≥10%

## Design Guidelines
- Use a heart/cardiology theme (red accents on white background)
- Show results with a visual gauge or progress bar
- Include a "Calculate" button
- Show interpretation text based on risk category
- Include disclaimer that this is for educational purposes
- Reference: Khan SS et al. Circulation. 2024;149(6):430-449
