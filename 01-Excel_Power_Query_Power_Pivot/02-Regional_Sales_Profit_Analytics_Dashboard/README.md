# Regional Sales & Profit Analytics Dashboard — Excel Power Pivot (Superstore)

**Stack:** Excel · Power Query (M) · Power Pivot · DAX · Pivot Charts · EDA App (Python/Dash)  
**Dataset:** Superstore — 9,994 transactions · 21 columns · US retail market · 4 years  
**Scope:** Full pipeline — EDA & statistical validation → selective data loading → Calendar modeling → DAX measures → interactive multi-KPI dashboard

**EDA App:** https://huggingface.co/spaces/will-7s/eda_app

Drive Link : https://mega.nz/file/5Q9CjawR#R0yqASIQULwXpErRLiyq_VOHbSc_gkjt7j2MLj-4wNo 

---

## 📌 Business Problem

A retail company needs to **track sales performance by region and product category** to identify underperforming areas and guide strategic decisions.

**Key questions:**
- Which regions generate the most revenue and profit?
- Which product categories and sub-categories are most profitable?
- How do sales and profit evolve monthly, quarterly, and annually?
- What are the top 10 best-selling products?

---

## 📊 Dataset Profile

**Source:** `Sample_Superstore.csv` — 9,994 rows · 21 columns · 0 missing values

| Column | Description | Cardinality |
|--------|-------------|-------------|
| `Order ID` | Unique order identifier | 5,009 distinct |
| `Order Date` | Purchase date | 1,237 distinct |
| `Ship Mode` | Shipping method (Standard, Second, First, Same Day) | 4 |
| `Customer ID` | Unique customer identifier | 793 distinct |
| `Segment` | Customer segment | 3 (Consumer · Corporate · Home Office) |
| `Region` | US sales region | 4 (West · East · Central · South) |
| `Category` | Product category | 3 (Office Supplies · Furniture · Technology) |
| `Sub-Category` | Product sub-category | 17 (Binders, Phones, Chairs, Copiers…) |
| `Sales` | Transaction revenue | 11.7% potential outliers |
| `Profit` | Net profit / loss per transaction | 18.8% potential outliers |
| `Discount` | Discount rate applied | Continuous |
| `Quantity` | Units ordered | Integer |

> Columns retained for analysis: `Order Date`, `Region`, `Category`, `Sub-Category`, `Product Name`, `Sales`, `Profit` — all others dropped in Power Query to reduce model size and improve performance.

---

## 🔬 Exploratory Data Analysis (EDA App)

Before building the dashboard, a rigorous statistical analysis was conducted using the **EDA web app** to validate which dimensions actually explain Sales and Profit variability.

### Bivariate Analysis Results — Kruskal-Wallis Tests

| Variable | Dimension | Test | Result | p-value |
|----------|-----------|------|--------|---------|
| `Profit` | `Region` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Profit` | `Category` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Profit` | `Sub-Category` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Profit` | `Segment` | Kruskal-Wallis | ❌ Not significant | p = 0.1123 |
| `Sales` | `Region` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Sales` | `Category` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Sales` | `Sub-Category` | Kruskal-Wallis | ✅ Significant | < 0.05 |
| `Sales` | `Segment` | Kruskal-Wallis | ❌ Not significant | p = 0.7103 |

> **Key insight from EDA:** `Segment` has no statistically significant impact on either Sales or Profit — it was excluded from the dashboard dimensions. `Region` and `Category` are the primary drivers of performance differences.

> **Note on Sales × Region:** ANOVA suggests no significant difference (p = 0.4933) but normality assumption is violated — Kruskal-Wallis is the appropriate test here, confirming significant differences exist.

---

## 🧰 Tools & Techniques

| Layer | Tools / Techniques |
|-------|--------------------|
| **EDA & Validation** | EDA App — univariate profiling, Kruskal-Wallis tests, bivariate analysis |
| **Selective Loading** | Power Query — column pruning, error removal, deduplication |
| **Time Intelligence** | Power Pivot — native Calendar table (Design → Date Table → New) |
| **Data Modeling** | Power Pivot — Add to Data Model, Diagram View, Calendar → Data relationship |
| **DAX Measures** | SUM, CALCULATE, ALL, ALLSELECTED, RANKX, IF |
| **Visualization** | Pivot Tables, Combo Charts (Column + Line), Horizontal Bar, Gauge Charts, Slicers |

---

## ⚙️ Power Query Pipeline

### Design Principle — Load Only What's Needed

> *"Reducing the model to only relevant columns improves performance, reduces file size, and clarifies business logic."*

```
Data → Get Data → From File → From Text/CSV → Transform Data
```

### Transformations Applied

| Step | Action | Reason |
|------|--------|--------|
| **Column pruning** | Kept only `Order Date`, `Region`, `Category`, `Sub-Category`, `Product Name`, `Sales`, `Profit` | Remove noise, reduce model size |
| **Remove Errors** | `Home → Remove Errors` | Prevent calculation errors downstream |
| **Remove Duplicates** | `Home → Remove Duplicates` | Ensure row-level integrity |
| **Type validation** | Verified `Sales` and `Profit` as Decimal, `Order Date` as Date | Required for DAX aggregations |

---

## 🧩 Data Model — Power Pivot

### Calendar Table

Created natively in Power Pivot — no external file needed:

```
Power Pivot → Manage → Design → Date Table → New
```

Auto-generates a complete date dimension with Year, Quarter, Month, Day — linked automatically to `Order Date`.

### Model Structure

```
Calendar (1) ──▷ (*) Sample_Superstore
                      │
                      ├── Order Date
                      ├── Region
                      ├── Category / Sub-Category
                      ├── Product Name
                      ├── Sales
                      └── Profit
```

Single-table fact model — no complex joins needed given the dataset structure.

---

## 📐 DAX Measures — Full Reference

### Core KPIs

```dax
Total Sales := SUM(Sample___Superstore[Sales])

Total Profit := SUM(Sample___Superstore[Profit])
```

Filter-aware — respond dynamically to all slicers (Region, Category, Year, Month).

---

### Global Denominators (for share calculations)

```dax
Total Sales Global :=
CALCULATE(
    SUM(Sample___Superstore[Sales]),
    ALL(Sample___Superstore),
    ALL(Calendar)
)

Total Profit Global :=
CALCULATE(
    SUM(Sample___Superstore[Profit]),
    ALL(Sample___Superstore),
    ALL(Calendar)
)
```

`ALL()` removes all active filters — these measures return the unconditional total regardless of slicer selection, used as denominators for % share KPIs.

---

### Dynamic Top 10 Products

```dax
Top 10 Sales :=
IF(
    RANKX(
        ALLSELECTED(Sample___Superstore[Product Name]),
        [Total Sales],
        ,
        DESC,
        DENSE
    ) <= 10,
    [Total Sales]
)
```

**`ALLSELECTED` vs `ALL`:**
- `ALL` → ranks across all products ignoring filters — static Top 10
- `ALLSELECTED` → ranks only among currently visible products — **the Top 10 adapts to every active filter**

Filter by West region → Top 10 of West. Filter by 2017 → Top 10 of 2017.

---

## 📈 Dashboard Visuals

| Visual | Chart Type | Measures Used | Notes |
|--------|-----------|---------------|-------|
| Monthly Sales & Profit | Combo — Column + Line | `Total Sales` + `Total Profit` | Profit on secondary axis |
| Top 10 Products | Horizontal Bar | `Top 10 Sales` | Dynamic — updates with all filters |
| Annual Sales Share | Gauge | `Total Sales` / `Total Sales Global` | Styled as speedometer gauge |
| Annual Profit Share | Gauge | `Total Profit` / `Total Profit Global` | Same gauge style |
| Slicers | — | Year · Month · Region · Category | Connected to all Pivot Tables |

### Combo Chart Setup (Sales + Profit)

```
Insert → Combo Chart
  → Sales   : Clustered Column  (primary axis — left)
  → Profit  : Line              (secondary axis — right)
```

The secondary axis is critical — Sales and Profit have different scales; without it, the Profit line would be visually crushed by the Sales bars.

---

## 📌 Key Business Findings

### By Region
- **West** is the top-performing region in both Sales and Profit
- **East** is a close second
- **Central** and **South** underperform — priority areas for improvement

### By Category
- **Technology** generates the highest sales volume overall
- **Furniture** delivers the highest profit overall
- **Exception — 2017:** Technology led in both Sales and Profit that year

### By Sub-Category (Technology)
- Highest sales: **Phones**
- Highest profit: **Copiers** — high-margin product despite lower volume

### Segment — Excluded
- `Segment` (Consumer / Corporate / Home Office) shows **no statistically significant difference** in Sales or Profit — confirmed by Kruskal-Wallis (p = 0.71 for Sales, p = 0.11 for Profit)

---

## 💡 Key Skills Demonstrated

| Skill | Application |
|-------|-------------|
| **Statistical validation before dashboarding** | Kruskal-Wallis tests confirmed which dimensions drive Sales/Profit — Segment excluded based on evidence |
| **Selective data loading** | Power Query column pruning — only 7 of 21 columns loaded into the model |
| **Native Calendar table** | Power Pivot Design → Date Table → New — clean time intelligence without external files |
| **Global denominator pattern** | `CALCULATE + ALL()` for filter-independent totals used in share KPIs |
| **Dynamic Top 10** | `RANKX + ALLSELECTED` — Top 10 adapts to every active slicer combination |
| **Combo chart design** | Secondary axis configuration for Sales (bars) + Profit (line) on the same visual |
| **Gauge chart styling** | Custom speedometer-style gauge for annual share KPIs |
| **EDA-to-dashboard workflow** | Statistical analysis directly informed dashboard design — no wasted dimensions |

---

## 📁 Project Structure

```
📁 Project/
├── Sample_Superstore.csv        ← Raw data — 9,994 rows · 21 columns
├── Regional_Sales_Profit_Analytics_Dashboard.xlsx               ← Power Query + Power Pivot + DAX + Charts + Slicers
└── README.md
```

---

*This project demonstrates a rigorous analytics workflow where statistical analysis (EDA) drives dashboard design decisions — not the other way around. Dimensions without explanatory power (Segment) are excluded before modeling, resulting in a cleaner, faster, and more meaningful dashboard.*
