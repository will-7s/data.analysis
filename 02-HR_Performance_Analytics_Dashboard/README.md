# HR Performance Analytics Dashboard — Power BI (HRDataset)

**Stack:** Power BI Desktop · Power Query (M) · DAX · EDA App (Python/Dash)  
**Dataset:** HRDataset — 311 employees · 36 columns · no documentation provided  
**Scope:** Full pipeline — undocumented dataset decoding → EDA → statistical validation → feature engineering → 3-page interactive dashboard

**EDA App:** https://huggingface.co/spaces/will-7s/eda_app

Power BI Link: https://app.powerbi.com/groups/me/reports/c1c05a56-6001-496e-9da0-99315fade642/cbacbdb0ab278ce42257?ctid=fd1df4e5-2eb0-410e-a611-12513030b133&experience=power-bi&bookmarkGuid=c9000760-a7b4-4c7a-92f0-130e64802514 

Drive Link: https://mega.nz/file/RZVnCKqa#uYwOccc99NXudwBrJBriuS0PqceiS9WgURfOoLX6To4 

---

## 📌 Business Problem

A company wants to **analyze the distribution of employee performance scores**, understand performance gaps, and identify the behavioral and organizational factors that explain them.

**Key questions:**
- How are performance scores distributed across the workforce?
- What is the relationship between engagement, satisfaction, and performance?
- Do behavioral indicators (tardiness, absenteeism) correlate with performance levels?
- Does tenure or time since last review explain performance differences?

---

## 🔍 Dataset Decoding — Working Without Documentation

The dataset was provided **with no column descriptions** — a realistic enterprise scenario where data arrives from a legacy HR system with no data dictionary.

The approach used was systematic inference from univariate statistics :

| Signal | Deduction method |
|--------|-----------------|
| `nunique() == nrows` | Unique identifier → exclude from analysis |
| 2 distinct values (0/1) | Binary flag → decode as Yes/No |
| 5 ordered numeric values | Ordinal categorical → decode by cross-referencing labeled column |
| Distributions matching exactly | Redundant columns → keep labeled version, drop ID version |
| Empty values at 66.6% | Not missing data — employees still active (confirmed by `TermReason`) |
| Mixed case (`Yes`/`yes`) | Data entry inconsistency → standardize before analysis |

> **Key insight:** `PerfScoreID` and `PerformanceScore` were confirmed identical via Cramér's V = 0.9737 — one was dropped. Same for `GenderID`/`Sex`, `DeptID`/`Department`, `MaritalStatusID`/`MaritalDesc`.

---

## 📊 Dataset Profile

**Source:** From Kaggle - `HRDataset_v14.csv` — 311 rows · 36 columns · 0 missing values

| Column | Type | Key Stats |
|--------|------|-----------|
| `PerformanceScore` | Categorical (4 levels) | Fully Meets 78.1% · Exceeds 11.9% · Needs Improvement 5.8% · PIP 4.2% |
| `EngagementSurvey` | Continuous [1.2 – 5] | Mean 4.11 · Median 4.28 |
| `EmpSatisfaction` | Ordinal [1 – 5] | Mode = 3 (34.7%) |
| `DaysLateLast30` | Integer [0 – 6] | 89.4% = 0 (no tardiness) |
| `DateofHire` | Date | Converted to `Tenure` in days |
| `LastPerformanceReview_Date` | Date | Converted to `DaysSinceReview` in days |
| `Salary` | Continuous [$45k – $250k] | Mean $69k · Median $62.8k · 9.3% outliers |

---

## 🔬 Statistical Analysis (EDA App)

### Bivariate Analysis — Variables vs `PerformanceScore`

| Variable | Test | Result | Effect Size |
|----------|------|--------|-------------|
| `PerfScoreID` | Cramér's V | ✅ Identical | V = 0.9737 → dropped |
| `EngagementSurvey` | Kruskal-Wallis | ✅ Significant | — |
| `EmpSatisfaction` | Chi-2 + Fisher | ✅ Significant | V = 0.3984 |
| `LastPerformanceReview_Date` | Chi-2 + Fisher | ✅ Significant | V = 0.6735 |
| `DaysLateLast30` | Chi-2 + Fisher | ✅ Significant | V = 0.6171 |
| `DateofHire` (Tenure) | Chi-2 + Fisher | ✅ Significant | V = 0.6150 |
| `Salary` | Kruskal-Wallis | ❌ Not significant | p = 0.4156 |
| `SpecialProjectsCount` | Chi-2 + Fisher | ❌ Not significant | — |
| `Absences` | Kruskal-Wallis | ❌ Not significant | — |
| `Department` | Chi-2 + Fisher | ❌ Not significant | p = 0.6017 |
| `Manager Name` | Chi-2 + Fisher | ❌ Not significant | — |
| `Position` | Chi-2 + Fisher | ❌ Not significant | — |
| `RecruitmentSource` | Chi-2 + Fisher | ❌ Not significant | — |
| `FromDiversityJobFairID` | Chi-2 + Fisher | ❌ Not significant | — |

> **Variables retained for dashboard:** `EngagementSurvey`, `EmpSatisfaction`, `DaysLateLast30`, `DateofHire` → `Tenure`, `LastPerformanceReview_Date` → `DaysSinceReview`

> **Variables excluded despite intuition:** `Salary`, `Absences`, `Department`, `Manager`, `Position` — no statistically significant association with performance confirmed by tests.

---

## ⚙️ Power Query Pipeline

### Column Pruning

Kept only the 6 analytically relevant columns — all others dropped to reduce model size and clarify business logic:

```
PerformanceScore · EngagementSurvey · EmpSatisfaction
DaysLateLast30 · DateofHire · LastPerformanceReview_Date
```

### Feature Engineering

Reference date used: **April 26, 2019** (dataset snapshot date is about seven years ago)

```m
// Tenure — days since hire
Tenure = Duration.Days(#date(2019, 04, 26) - [DateofHire])
```

```m
// DaysSinceReview — days since last performance review
DaysSinceReview = Duration.Days(#date(2019, 04, 26) - [LastPerformanceReview_Date])
```

Both original date columns dropped after transformation — only engineered numeric columns loaded into the model.

---

## 📐 DAX Measures — Full Reference

```dax
-- Distribution
Count Performance := COUNTROWS(HRDataset_v14)

-- Engagement
Avg Engagement := AVERAGE(HRDataset_v14[EngagementSurvey])

-- Satisfaction
Avg Satisfaction := AVERAGE(HRDataset_v14[EmpSatisfaction])

-- % employees with at least 1 late day in last 30
% Late Employees :=
DIVIDE(
    COUNTROWS(FILTER(HRDataset_v14, HRDataset_v14[DaysLateLast30] > 0)),
    COUNTROWS(HRDataset_v14),
    0
)

-- Average tenure in years
Avg Tenure := AVERAGE(HRDataset_v14[Tenure]) / 365.25

-- Average tardiness
Avg Days Late := AVERAGE(HRDataset_v14[DaysLateLast30])

-- Average days since last performance review
Avg Days Since Review := AVERAGE(HRDataset_v14[DaysSinceReview])
```

All measures are **fully filter-aware** — they adapt automatically to any active slicer on `PerformanceScore` or any other dimension.

---

## 📈 Dashboard Structure — 3 Pages

### Page 1 — Performance Overview

| Visual | Measure / Field | Type |
|--------|----------------|------|
| Performance distribution | `Count Performance` + `PerformanceScore` | Donut chart |
| Average engagement | `Avg Engagement` | KPI Card |
| Average satisfaction | `Avg Satisfaction` | KPI Card |
| % employees with tardiness | `% Late Employees` | KPI Card (gauge) |
| Average tenure | `Avg Tenure` | KPI Card |

---

### Page 2 — Performance & Engagement / Satisfaction

| Visual | Variables | Type |
|--------|-----------|------|
| Engagement by performance level | `PerformanceScore` + `Avg Engagement` | Clustered bar chart |
| Avg engagement vs avg satisfaction by performance | `Avg Engagement` + `Avg Satisfaction` + `PerformanceScore` | Scatter plot (4 points) |
| Satisfaction by performance level | `PerformanceScore` + `Avg Satisfaction` | Clustered bar chart |

> The scatter plot positions each performance level according to its average engagement (X) and average satisfaction (Y) — immediately reveals the correlation between the two dimensions across performance groups.

---

### Page 3 — Performance & Behavior

| Visual | Variables | Type |
|--------|-----------|------|
| Tardiness by performance level | `PerformanceScore` + `Avg Days Late` | Clustered bar chart |
| Tenure by performance level | `PerformanceScore` + `Avg Tenure` | Clustered bar chart |
| Days since review by performance level | `PerformanceScore` + `Avg Days Since Review` | Clustered bar chart |

---

## 📌 Key Findings

### Performance Distribution
- **78%** of employees are at **Fully Meets** — objectives are broadly achieved
- **10%** are underperforming — **6% Needs Improvement** + **4% PIP** (Performance Improvement Plan)
- Only **12%** exceed expectations

### Engagement & Satisfaction
- Employees at **Fully Meets** and **Exceeds** show **high engagement and satisfaction scores**
- Employees at **Needs Improvement** and **PIP** show significantly **lower engagement and satisfaction**
- Average satisfaction and average engagement are **strongly correlated** across performance levels

### Behavioral Indicators
- Employees at **Needs Improvement** and **PIP** are late on average **4 days out of 30** — a strong behavioral signal
- Employees at **Fully Meets** and **Exceeds** show **zero tardiness**
- `DaysLateLast30` (Cramér's V = 0.617) is one of the **strongest predictors** of performance level

### Non-significant factors
- **Tenure** — length of service does not explain performance differences
- **Days since last review** — review recency does not explain performance
- **Salary**, **Department**, **Manager**, **Absences** — no statistically significant association with performance

---

## 💡 Key Skills Demonstrated

| Skill | Application |
|-------|-------------|
| **Undocumented dataset decoding** | Inferred role of all 36 columns from distributions, cardinalities, and cross-referencing — no data dictionary available |
| **Redundancy detection** | Identified 6 pairs of duplicate columns via Cramér's V and distribution matching |
| **Statistical validation** | Kruskal-Wallis + Chi-2 + Fisher tests — 14 variables tested, 6 retained based on evidence |
| **Counterintuitive findings** | Salary, Absences, Department, Manager — excluded despite common assumptions |
| **Feature engineering** | `Tenure` and `DaysSinceReview` computed in Power Query from raw dates using a reference date |
| **DAX filter-aware measures** | All 7 measures adapt dynamically to slicer context |
| **3-page dashboard architecture** | Overview → Engagement/Satisfaction → Behavior — progressive analytical narrative |
| **EDA-to-dashboard workflow** | Statistical analysis directly drove variable selection and visual design |

---

## 📁 Project Structure

```
📁 Project/
├── HRDataset_v14.csv          ← Raw HR data — 311 employees · 36 columns · no documentation
├── employee_performances.pbix             ← Power BI report — Power Query + DAX + 3-page dashboard
└── README.md
```

---

*This project demonstrates that rigorous exploratory analysis — including the ability to decode undocumented datasets and exclude variables based on statistical evidence rather than intuition — is the foundation of meaningful BI dashboards.*
