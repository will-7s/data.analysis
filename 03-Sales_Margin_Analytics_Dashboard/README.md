# Sales & Margin Analytics Dashboard — Power BI (Sample Superstore)

**Stack:** Power BI Desktop · Power Query (M) · DAX · EDA App (Python/Dash)  
**Dataset:** Sample Superstore — 9,994 transactions · 21 columns · US retail market  
**Scope:** Full pipeline — EDA & statistical validation → selective data loading → DAX measures → interactive dashboard

**EDA App:** https://huggingface.co/spaces/will-7s/eda_app

Power BI link: https://app.powerbi.com/groups/me/reports/ff71f871-9238-40d1-ae1b-e202498178d0/e6bd3e980abaf9c9d19e?ctid=fd1df4e5-2eb0-410e-a611-12513030b133&experience=power-bi&bookmarkGuid=89f57e0c-e44e-457c-8655-e6686f0d221e 

Drive link: https://mega.nz/file/UUVgUL4K#gJpUTFsCkAnG8J0MYM8LnvzZ4UudRqqYoGtGh-_BW8c 

---

## 📌 Business Problem

A retail company wants to **understand the interactions between its products, regional performance, and margins** to optimize its commercial strategy.

**Key questions:**
- Which regions and product categories drive the most revenue and profit?
- Which sub-categories generate negative margins?
- How do sales and profit evolve over time?

---

## 📊 Dataset Profile

**Source:** `Sample - Superstore.csv` — 9,994 rows · 21 columns · 0 missing values

| Column | Type | Key Stats |
|--------|------|-----------|
| `Order ID` | Categorical | 5,009 distinct orders |
| `Ship Mode` | Categorical (4) | Standard Class 59.7% |
| `Customer ID` | Categorical | 793 distinct customers |
| `Segment` | Categorical (3) | Consumer 51.9% |
| `State` | Categorical | 49 distinct states |
| `Region` | Categorical (4) | West · East · Central · South |
| `Category` | Categorical (3) | Office Supplies 60.3% |
| `Sub-Category` | Categorical (17) | — |
| `Product Name` | Categorical | 1,850 distinct products |
| `Sales` | Continuous | Mean 230 · Median 55 · 11.7% outliers |
| `Quantity` | Integer [1–14] | Mean 3.8 |
| `Discount` | Continuous [0–0.8] | — |
| `Profit` | Continuous | Positive and negative values |

> `Country` contains only `United States` — excluded as non-informative. `Row ID`, `Postal Code`, `Ship Date` excluded as not relevant to the analysis.

---

## 🔬 Statistical Analysis (EDA App)

### Bivariate Analysis — Variables vs `Sales`

| Variable | Test | Result |
|----------|------|--------|
| `State` | Kruskal-Wallis | ✅ Significant |
| `Region` | Kruskal-Wallis | ✅ Significant |
| `Category` | Kruskal-Wallis | ✅ Significant |
| `Sub-Category` | Kruskal-Wallis | ✅ Significant |
| `Product ID` | Kruskal-Wallis | ⚠️ Significant but 1,862 modalities — too granular |
| `Product Name` | Kruskal-Wallis | ⚠️ Significant but 1,850 modalities — too granular |
| `Quantity` | Spearman + Kendall | ✅ Weak positive correlation |
| `Discount` | Spearman + Kendall | ✅ Very weak negative correlation |
| `Profit` | Spearman + Kendall | ✅ Moderate positive correlation |

### Bivariate Analysis — Variables vs `Profit`

| Variable | Test | Result |
|----------|------|--------|
| `State` | Kruskal-Wallis | ✅ Significant |
| `Region` | Kruskal-Wallis | ✅ Significant |
| `Category` | Kruskal-Wallis | ✅ Significant |
| `Sub-Category` | Kruskal-Wallis | ✅ Significant |
| `Customer ID` | Kruskal-Wallis | ⚠️ Significant but 793 modalities — trivial differences |
| `Customer Name` | Kruskal-Wallis | ⚠️ Significant but 793 modalities — trivial differences |
| `City` | Kruskal-Wallis | ⚠️ Significant but 531 modalities — too granular |
| `Product ID` | Kruskal-Wallis | ⚠️ Significant but 1,862 modalities — too granular |
| `Product Name` | Kruskal-Wallis | ⚠️ Significant but 1,850 modalities — too granular |
| `Sales` | Spearman + Kendall | ✅ Moderate positive correlation |
| `Quantity` | Spearman + Kendall | ✅ Very weak positive correlation |
| `Discount` | Spearman + Kendall | ✅ Weak negative correlation |

> **Variables excluded despite significance:** `Customer ID`, `Customer Name`, `City`, `Product ID`, `Product Name` — Kruskal-Wallis detects trivial differences at high cardinality (793 to 1,862 groups on 9,994 observations). Not analytically meaningful as dashboard dimensions.

> **Variables retained:** `Region`, `State`, `Category`, `Sub-Category`, `Sales`, `Quantity`, `Discount`, `Profit`

---

## ⚙️ Power Query Pipeline

### Column Pruning

```
View → Column Quality → 0 errors, 0 nulls confirmed across all columns
```

Removed non-analytical columns — keeping only the 10 relevant fields:

```
Order Date · Region · State · Category · Sub-Category
Product Name · Sales · Quantity · Discount · Profit
```

> Power Query auto-detected the correct data type for every column — no manual type correction needed.

Query named `Sample - Superstore` and loaded into Power BI model.

---

## 📐 DAX Measures — Full Reference

```dax
-- Total revenue
Total Sales := SUM('Sample - Superstore'[Sales])

-- Total profit
Total Profit := SUM('Sample - Superstore'[Profit])

-- Average margin
Avg Margin := AVERAGE('Sample - Superstore'[Profit])

-- % transactions with negative profit
% Negatives :=
VAR Total = COUNTROWS('Sample - Superstore')
VAR Neg    = CALCULATE(
                COUNTROWS('Sample - Superstore'),
                'Sample - Superstore'[Profit] < 0
             )
RETURN DIVIDE(Neg, Total) * 100
```

All measures are **fully filter-aware** — they adapt automatically to any active slicer on `Region`, `Category`, `Sub-Category`, or `Order Date`.

---

## 📈 Dashboard Structure - Performance Overview

| Visual | Measure / Field | Type |
|--------|----------------|------|
| Total revenue | `Total Sales` | KPI Card |
| Total profit | `Total Profit` | KPI Card |
| Average margin | `Avg Margin` | KPI Card |
| % negative margin transactions | `% Negatives` | KPI Card |
| Revenue breakdown | `Total Sales` + `Category` or `Region` | Donut chart + Field Parameter |
| Monthly sales & profit trend | `Total Sales` + `Total Profit` + `Order Date` | Combo chart (bar + line) |

> **Field Parameter** on the donut chart allows switching between `Category` and `Region` breakdown without duplicating visuals.

**Slicer:** `Sub-Category` — enables filtering all visuals down to a specific product line, including isolating negative-margin sub-categories.

---

## 💡 Key Skills Demonstrated

| Skill | Application |
|-------|-------------|
| **High-cardinality variable exclusion** | `Customer ID`, `City`, `Product Name` — significant by test but analytically meaningless at 500–1,800 modalities |
| **Dual bivariate analysis** | Tested variables against both `Sales` and `Profit` independently — different variable sets emerged |
| **Selective data loading** | Power Query column pruning — 10 of 21 columns retained |
| **Data quality validation** | `View → Column Quality` confirmed 0 errors and 0 nulls before loading |
| **DAX % negative margin** | `VAR` + `CALCULATE` + `DIVIDE` pattern for conditional row percentage |
| **Field Parameter** | Dynamic dimension selector on donut chart — `Category` vs `Region` in one visual |
| **Combo chart** | `Total Sales` (bars) + `Total Profit` (line) on dual axis for monthly trend |
| **EDA-to-dashboard workflow** | Statistical analysis directly drove variable selection and dashboard design |

---

## 📁 Project Structure

```
📁 Project/
├── Sample - Superstore.csv     ← Raw data — 9,994 transactions · 21 columns
├── sales_margin.pbix   ← Power BI report — Power Query + DAX + dashboard
└── README.md
```

---

*This project demonstrates that effective commercial analytics requires first identifying which dimensions genuinely explain revenue and margin variance — and deliberately excluding high-cardinality variables that produce statistically significant but analytically meaningless results.*
