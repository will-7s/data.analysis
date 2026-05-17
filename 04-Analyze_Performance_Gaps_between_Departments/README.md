# Employee Performance Gap Analysis

Exploratory Data Analysis (EDA) on an HR dataset of 100,000 employees, aiming to identify factors associated with performance and job satisfaction through statistical testing.

---

## Business Context

A company seeks to analyze performance gaps across its departments by exploring relationships between qualitative variables (department, satisfaction) and quantitative variables (performance scores, hours worked). The goal is to statistically validate observed differences.

**Dataset**: `Extended_Employee_Performance_and_Productivity_Data` — available on [Kaggle](https://www.kaggle.com/)
**Dimensions**: 100,000 rows × 20 columns

---

## Project Structure

```
├── performance_gaps.ipynb    # Analysis notebook  
└── README.md                 # This file
```

---

## Tech Stack

| Library | Usage |
|---------|-------|
| `numpy` | Data loading (`genfromtxt`), array operations, descriptive statistics |
| `scipy.stats` | Normality tests, Chi-2, Kruskal-Wallis, Levene, ANOVA, correlation |
| `statsmodels` | Lilliefors test (`lilliefors`) |
| `pandas` | Contingency tables (`crosstab`) |
| `seaborn` / `matplotlib` | Boxplots, heatmaps, histograms, Q-Q plots |

---

## Dataset Overview

| Variable | Type | Description | Distribution |
|----------|------|-------------|--------------|
| `Employee_ID` | Numeric | Unique identifier | 1 – 100,000 |
| `Department` | Categorical | Employee's department | 9 departments, ~11% each |
| `Gender` | Categorical | Employee's gender | Male / Female ~48% each, Other ~4% |
| `Age` | Numeric | Employee's age | 22 – 60, mean = median = 41, uniform |
| `Job_Title` | Categorical | Role | 7 titles, ~14.3% each |
| `Hire_Date` | Date | Recruitment date | 3,650 distinct dates |
| `Years_At_Company` | Numeric | Tenure in years | 0 – 10, mean = 4.48, median = 4 |
| `Education_Level` | Categorical | Highest degree | Bachelor 50%, High School 30%, Master 15%, PhD 5% |
| `Performance_Score` | Ordinal | Performance rating | 1 – 5, uniform distribution |
| `Monthly_Salary` | Numeric | Monthly salary (USD) | 3,850 – 9,000, mean = 6,403, median = 6,500 |
| `Work_Hours_Per_Week` | Numeric | Weekly working hours | 30 – 60, mean = median = 45, uniform |
| `Projects_Handled` | Numeric | Total projects managed | 0 – 49, mean = 24.43, uniform |
| `Overtime_Hours` | Numeric | Yearly overtime hours | 0 – 29, mean = 14.5, uniform |
| `Sick_Days` | Numeric | Sick leave days | 0 – 14, mean = median = 7, uniform |
| `Remote_Work_Frequency` | Categorical | % of remote work | 0 / 25 / 50 / 75 / 100%, uniform |
| `Team_Size` | Numeric | Team headcount | 1 – 19, mean = median = 10, uniform |
| `Training_Hours` | Numeric | Training hours | 0 – 99, mean = 49.5, quasi-uniform |
| `Promotions` | Ordinal | Number of promotions | 0 / 1 / 2, uniform |
| `Employee_Satisfaction_Score` | Ordinal | Satisfaction rating | 1 – 5, mean = median = 3, uniform |
| `Resigned` | Binary | Resignation status | False 90%, True 10% |

---

## Univariate Analysis — `Employee_Satisfaction_Score`

The notebook performs a detailed univariate analysis on `Employee_Satisfaction_Score` as a worked example of the methodology applied to numeric variables.

**Descriptive statistics**

| Metric | Value |
|--------|-------|
| Mean | 3.0 |
| Median | 3.0 |
| Min / Max | 1 / 5 |
| Outliers (IQR × 1.5) | 0% |
| Skewness | ≈ 0 (symmetric) |
| Kurtosis (Pearson) | ≈ 1.809 (vs 3 for a normal distribution) |

**Normality assessment** — all five tests conclude non-normality:

| Test | Conclusion |
|------|------------|
| Shapiro-Wilk | Non-normal |
| Kolmogorov-Smirnov | Non-normal |
| Anderson-Darling | Non-normal |
| Lilliefors | Non-normal |
| D'Agostino-Pearson | Non-normal |

The Q-Q plot confirms systematic deviation from the theoretical line. The uniform-like distribution makes normality structurally impossible regardless of sample size.

---

## Bivariate Analysis

Two target variables were studied: `Performance_Score` and `Employee_Satisfaction_Score`.

### Target: `Performance_Score`

**Test selection per pair type**
- *Categorical × Categorical*: Chi-2 + Cramér's V + Tschuprow's T for effect size
- *Numeric × Categorical*: Levene (variance homogeneity) → ANOVA if assumptions met, Kruskal-Wallis otherwise; normality tested per group with the 5-test battery

| Variable | Test | p-value | Conclusion |
|----------|------|---------|------------|
| `Department` | Chi-2 | 0.5763 | No association (V = 0.009) |
| `Gender` | Chi-2 | 0.0473 | Significant but negligible effect (V = 0.009, T = 0.007) |
| `Age` | ANOVA / Kruskal-Wallis | 0.6968 / 0.6990 | No difference |
| `Job_Title` | Chi-2 | 0.5441 | No association |
| `Years_At_Company` | ANOVA / Kruskal-Wallis | 0.4343 / 0.4374 | No difference |
| `Education_Level` | Chi-2 | 0.0668 | No association |
| **`Monthly_Salary`** | **Kruskal-Wallis** | **≈ 0.000** | **Significant differences ✓** |
| `Work_Hours_Per_Week` | ANOVA / Kruskal-Wallis | 0.4141 / 0.4151 | No difference |
| `Projects_Handled` | Kruskal-Wallis | 0.5436 | No difference |
| `Overtime_Hours` | ANOVA / Kruskal-Wallis | 0.4502 / 0.4509 | No difference |
| `Sick_Days` | ANOVA / Kruskal-Wallis | 0.0672 / 0.0674 | No difference |
| `Remote_Work_Frequency` | Chi-2 | 0.6240 | No association |
| **`Team_Size`** | **ANOVA / Kruskal-Wallis** | **0.0250 / 0.0252** | **Significant differences ✓** |
| `Training_Hours` | ANOVA / Kruskal-Wallis | 0.6861 / 0.6825 | No difference |
| `Promotions` | Chi-2 | 0.0874 | No association |
| `Employee_Satisfaction_Score` | ANOVA / Kruskal-Wallis | 0.8582 / 0.8570 | No difference |
| `Resigned` | Chi-2 | 0.5258 | No association |

> **Note on ANOVA applicability**: Levene's test rejected equal variance for `Monthly_Salary` and `Projects_Handled`, making ANOVA inappropriate. For `Team_Size`, Levene confirmed equal variance but the per-group normality battery rejected normality across all 5 groups — Kruskal-Wallis remains the valid choice.

**Deep dive — `Monthly_Salary` × `Performance_Score`**

Boxplots show visible salary differences across the 5 performance groups. Kruskal-Wallis confirms significance (p ≈ 0). This is the strongest and most actionable finding in the dataset.

**Deep dive — `Team_Size` × `Performance_Score`**

Kruskal-Wallis confirms significant differences (p = 0.025). Effect size is modest. Per-group normality tests (all 5 tests, all 5 groups) unanimously reject normality, ruling out ANOVA despite equal variances.

### Target: `Employee_Satisfaction_Score`

| Variable | Test | Statistic | p-value | Conclusion |
|----------|------|-----------|---------|------------|
| **`Department`** | **ANOVA / Kruskal-Wallis** | — | **0.0055 / 0.0056** | **Significant differences ✓** |
| `Gender` | Kruskal-Wallis | — | 0.3288 | No difference |
| `Age` | Spearman | ρ = −0.0001 | 0.9669 | No correlation |
| `Job_Title` | ANOVA | — | 0.7483 | No difference |
| `Years_At_Company` | Spearman | ρ = −0.0032 | 0.3126 | No correlation |
| `Education_Level` | ANOVA | — | 0.7542 | No difference |
| `Monthly_Salary` | Spearman | ρ = 0.0013 | 0.6835 | No correlation |
| `Work_Hours_Per_Week` | Spearman | ρ = 0.0005 | 0.8656 | No correlation |
| `Projects_Handled` | Spearman | ρ = 0.0062 | 0.0516 | No correlation |
| `Overtime_Hours` | Spearman | ρ = 0.0010 | 0.7417 | No correlation |
| `Sick_Days` | Spearman | ρ = −0.0009 | 0.7737 | No correlation |
| `Remote_Work_Frequency` | ANOVA | — | 0.2291 | No difference |
| `Team_Size` | Spearman | ρ = 0.0009 | 0.7658 | No correlation |
| `Training_Hours` | Spearman | ρ = −0.0015 | 0.6351 | No correlation |
| `Promotions` | ANOVA | — | 0.0766 | No difference |
| `Resigned` | ANOVA / t-test / Mann-Whitney | — | 0.3709 / 0.3709 / 0.3740 | No difference |

> The 5-test normality battery was applied to all 9 department groups. All groups failed normality — ANOVA assumptions are not met and Kruskal-Wallis is the valid test for the `Department` finding.

### Target: `Work_Hours_Per_Week`

No variable in the dataset shows a significant relationship with weekly working hours. All p-values are well above 0.05 across ANOVA, Kruskal-Wallis, and Spearman tests.

---

## Key Findings

| Finding | Variables | Test | p-value |
|---------|-----------|------|---------|
| Salary varies with performance level | `Monthly_Salary` → `Performance_Score` | Kruskal-Wallis | ≈ 0.000 |
| Satisfaction differs by department | `Department` → `Employee_Satisfaction_Score` | Kruskal-Wallis | 0.006 |
| Team size marginally linked to performance | `Team_Size` → `Performance_Score` | Kruskal-Wallis | 0.025 |

The `Gender` × `Performance_Score` association is statistically significant (p = 0.047) but practically negligible — Cramér's V = 0.009 and Tschuprow's T = 0.007 indicate no meaningful effect. This is a common large-sample artefact: with n = 100,000, even trivial differences become detectable.

**Structural observation**: the near-uniform distribution of almost every variable strongly limits the detection of associations. The three findings above stand out precisely because they are exceptions to this pattern.

---

## Statistical Testing Methodology

### Normality — 5-test battery (applied per variable and per group)

| Test | Min n | Strength |
|------|-------|----------|
| Shapiro-Wilk | 3 | Most powerful for n ≤ 5,000 |
| Kolmogorov-Smirnov | 3 | Conservative when parameters estimated from sample |
| Anderson-Darling | 7 | Emphasises tail deviations |
| Lilliefors | 4 | Corrects KS bias via Monte Carlo |
| D'Agostino-Pearson | 8 | Omnibus K² = Z²(skew) + Z²(kurt) |

### Test selection logic

```
Categorical × Categorical  →  Chi-2  +  Cramér's V  +  Tschuprow's T
                               Fisher's Exact if table is small or min(expected) < 5

Numeric × Categorical      →  Levene (variance homogeneity)
                               + Normality per group (5-test battery)
                               → ANOVA if normality ✓ and equal variances ✓
                               → Kruskal-Wallis otherwise

Numeric × Numeric          →  Pearson  (if both normal)
                               Spearman (non-normal, or robust preference)
                               Kendall  (small n or many ties)
```

### Large-sample considerations (n = 100,000)

At this scale, even negligible effects reach statistical significance. Effect size measures — Cramér's V, Spearman's ρ, η² — are essential complements to p-values for assessing practical relevance. The `Gender` × `Performance_Score` case is a direct illustration of this.
