# Freddie Mac Small Balance Loan Survival Analysis

An end-to-end credit-risk project that models the timing of delinquency and default in Freddie Mac Small Balance Loan (SBL) securitizations while accounting for payoff as a competing event.

The analysis converts quarterly loan-performance records into a multi-state survival framework, estimates portfolio-level default and payoff incidence, identifies risk factors, and produces exploratory risk rankings for active loans.

## Project objective

Traditional default classifiers reduce each loan to a binary outcome and ignore when the event occurs. This project uses survival analysis to answer three questions:

1. How does default risk evolve as loans season?
2. How should default be measured when loans can pay off before defaulting?
3. Which underwriting and performance characteristics are associated with elevated risk?

## Dataset

The analysis covers quarterly Freddie Mac SBL performance records from 2014 through 2023.

| Measure | Value |
|---|---:|
| Quarterly loan records | 202,285 |
| Unique loans | 14,778 |
| SB deals | 104 |
| Observed defaults | 51 |
| Paid-off loans | 5,238 |
| Right-censored loans | 9,489 |
| Median follow-up | 15 quarters |
| Maximum follow-up | 33 quarters |

Default is defined as the first observation of 90+ days delinquent or foreclosure. Paid-off loans are treated as a competing event.

During validation, mortgage status code `500` was corrected from `REO` to `Paid Off`. Its frequency and liquidation balances align with the program's reported prepayment volume rather than its much smaller REO population. The original status field remains preserved for auditability.

## Methodology

### Data preparation

- Standardized column names and data types
- Removed blank Excel columns
- Validated unique loan-quarter records
- Repaired century-shifted maturity and interest-only dates
- Created quarterly start-stop survival intervals
- Engineered loan age, UPB ratio, prior delinquency and transition features
- Removed outcome-revealing variables from model predictors
- Truncated modeling histories at the first default or payoff

### Survival analysis

- **Kaplan-Meier:** descriptive cause-specific default-free survival
- **Aalen-Johansen:** cumulative incidence of default and payoff
- **Subgroup analysis:** LTV, DCR, origination vintage and Census region
- **Log-rank tests:** exploratory cause-specific group comparisons
- **Time-varying Cox regression:** penalized and limited to four predictors
- **Random Survival Forest:** constrained nonlinear challenger model

### Model evaluation

- Harrell's concordance index
- Time-dependent AUC
- Integrated Brier score
- Five-fold sensitivity analysis
- Risk-group calibration
- Permutation feature importance
- Chronological holdout feasibility assessment

## Key results

| Result | Estimate |
|---|---:|
| 32-quarter default cumulative incidence | 0.66% |
| 32-quarter payoff cumulative incidence | 81.07% |
| RSF holdout C-index | 0.637 |
| Mean time-dependent AUC | 0.648 |
| Integrated Brier score | 0.0053 |
| Five-fold mean C-index | 0.767 |

Key observations:

- Kaplan-Meier estimated an eight-year cause-specific default probability of approximately 1.03%, compared with a 0.66% Aalen-Johansen default incidence. The difference shows why payoff should be modeled as a competing event.
- Higher LTV and lower DCR groups experienced greater default incidence.
- Prior delinquency was the clearest time-varying Cox predictor, with an estimated hazard ratio of approximately 1.50.
- LTV produced the largest positive permutation importance in the Random Survival Forest.
- Model discrimination weakened at longer horizons, with dynamic AUC declining from approximately 0.71 to 0.60.
- No origination-year cutoff contained enough defaults on both sides for credible out-of-time validation.

## Repository structure

```text
.
├── README.md
├── Freddie_Mac_SBL_End_to_End_Survival_Analysis.ipynb
├── data/
│   └── raw/
│       └── Sheet4.xlsx          # optional; subject to distribution rights
├── outputs/                     # generated preprocessing outputs
└── analysis_outputs/            # generated tables and charts
```

The notebook contains the complete workflow. The generated output folders are optional and do not need to be committed.

## Running the project

### Google Colab

1. Upload `Freddie_Mac_SBL_End_to_End_Survival_Analysis.ipynb` to Google Colab.
2. Run the installation cell.
3. Upload `Sheet4.xlsx` when prompted.
4. Select **Runtime → Run all**.

### Local Jupyter environment

Clone the repository and place the workbook at `data/raw/Sheet4.xlsx`.

```bash
git clone <your-repository-url>
cd <your-repository-name>
```

Install the required packages:

```bash
pip install "numpy<2.4" "pandas<3" openpyxl pyarrow lifelines scikit-survival seaborn shap jupyter
```

Start Jupyter:

```bash
jupyter notebook
```

Open the consolidated notebook and run all cells.

## Technology stack

- Python
- pandas and NumPy
- lifelines
- scikit-survival
- scikit-learn
- Matplotlib and Seaborn
- Jupyter Notebook / Google Colab

## Limitations

- Only 51 defaults are observed, limiting model complexity and statistical power.
- Random cross-validation mixes vintages and may overstate expected future performance.
- No credible out-of-time validation sample is available.
- The Random Survival Forest and Cox outputs should be interpreted as relative-risk signals, not production-grade probabilities of default.
- Regional differences may reflect portfolio composition, vintage and underwriting mix rather than causal geography effects.
- Protected-class attributes are unavailable, so demographic fairness cannot be evaluated.
- Macroeconomic and updated property-level operating variables are not included.

## Recommended use

This project is best used as an exploratory early-warning and portfolio-monitoring framework. A production implementation would require additional default history, later-vintage validation, macroeconomic covariates, updated property performance data and formal model-risk governance.

## Data and references

The analysis was developed using Freddie Mac Small Balance Loan performance data and the following supporting publications:

- *Small Balance Loan Prepayments* — July 2024
- *Small Balance Loan Program Handout*
- *Small Balance Loan Performance Data* — February 2025

Confirm applicable data licensing and distribution requirements before committing the raw workbook to a public repository.

## Author

Ved Bhanderi
