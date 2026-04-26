# Retail Sales & Stock Analytics Dashboard — Excel Power Pivot

**Stack:** Excel · Power Query (M) · Power Pivot · DAX · Pivot Charts  
**Dataset:** Multi-source retail data — transactions, warehouse stocks, in-store sales  
**Scope:** Full pipeline — raw multi-workbook data → cleaning → modeling → DAX measures → interactive dashboard

Drive Link : https://mega.nz/file/RA9CVTDD#veW7m0i5HOMzwa6wFH8ZB815Z-FwR5cZyoeEKcxdiy4 

---

## 📌 Project Overview

Three separate data sources, no common key format, no date dimension, no aggregated metrics — the raw data was unusable for analysis as-is.

**Raw data contained:**
- Duplicate and erroneous rows across all three workbooks
- Mismatched primary/foreign key formats between tables — preventing joins
- Stock split across 5 warehouses with no total column
- No pre-computed KPIs (revenue share, stock share, rankings)

**Result:** A fully interactive Excel dashboard tracking revenue trends, stock levels, and product rankings — dynamically filterable by article, category, month, quarter, and year.

---

## 📊 Dashboard KPIs

| Metric | Granularity |
|--------|------------|
| Monthly revenue evolution (2 years) | By month, year |
| Total revenue | By article, by category |
| Revenue share (% of total) | By article, by category |
| Top 10 best-selling articles | Dynamic — adapts to all active filters |
| Total available stock | By article, by category |
| Stock share (% of total) | By article, by category |

---

## 🧰 Tools & Techniques

| Layer | Tools / Techniques |
|-------|--------------------|
| **Data Cleaning** | Power Query — Remove Duplicates, Remove Errors |
| **Aggregation** | Power Query — Group By (primary key consolidation) |
| **Key Harmonization** | Power Query — Custom Column (type/format alignment) |
| **Stock Engineering** | Power Query — Custom Column (sum across 5 warehouses) |
| **Data Modeling** | Power Pivot — Add to Data Model, Diagram View, relationships |
| **Time Intelligence** | Calendar table — Year, Quarter, Month, Month Number |
| **DAX Measures** | SUM, SUMX, FILTER, COUNTROWS, CALCULATE, ALL, ALLSELECTED, RANKX |
| **Visualization** | Pivot Tables, Pivot Charts, Slicers |

---

## ⚙️ Power Query Pipeline

### Step 1 — Data Quality Cleanup

Applied to all three source workbooks:

```
Home → Remove Duplicates    → eliminates exact duplicate rows
Home → Remove Errors        → removes rows with data errors
```

> ⚠️ **Key lesson:** `Group By` requires a valid primary key column to aggregate correctly — always verify key uniqueness before grouping.

### Step 2 — Primary Key Consolidation (Group By)

After cleaning, rows were aggregated by primary key to ensure one row per entity:

```
Transform → Group By → [primary key column] → aggregate metrics
```

This step is critical — without it, relationships in Power Pivot produce incorrect fan-out results.

### Step 3 — Key Format Harmonization

**Problem identified:** `Data[Item_Code]` was formatted differently from `Stocks[Item_Code]` and `In_Store_Sales[Item_Code]` — joins were silently failing, producing aggregated totals instead of per-article values.

**Solution:** Created a new calculated column in Power Query to normalize the key format across all tables before loading into the model.

```
Add Column → Custom Column → [normalization formula]
```

> This was the root cause of incorrect measure results — all values returning the same grand total regardless of filter context.

### Step 4 — Stock Engineering (5 Warehouses → 1 Total)

`Stocks` originally had one column per warehouse (W1–W5) with no total:

```
Add Column → Custom Column:
Item Total Stock = [W1] + [W2] + [W3] + [W4] + [W5]
```

After adding this column:
1. Removed the old `Stocks` table from the data model
2. Re-added the updated table via **Power Pivot → Add to Data Model**

> The model must be refreshed after any Power Query structural change — simply editing the query is not enough.

---

## 🧩 Data Model — Power Pivot

### Loading Tables

```
Select table in Excel → Power Pivot → Add to Data Model
```

Repeated for all three tables: `Data`, `Stocks`, `In_Store_Sales` + `Calendar`.

### Relationships (Diagram View)

```
Calendar (1) ──▷ (*) Data (*) ──▷ (1) In_Store_Sales
                      │
                      └──▷ (1) Stocks
```

All relationships use `Item_Code` / `Code_Item` as the join key — harmonized in Step 3.

> **Filter direction:** `Calendar → Data` propagates date filters correctly. `Data → In_Store_Sales` and `Data → Stocks` required custom DAX patterns (see below) to propagate non-date filters.

### Calendar Table

Built directly in Power Pivot to enable time intelligence across all visuals:

| Column | Formula |
|--------|---------|
| `Year` | `YEAR(Calendar[Date])` |
| `Quarter` | `"Q" & ROUNDUP(MONTH(Calendar[Date])/3,0)` |
| `Month` | `FORMAT(Calendar[Date],"MMMM")` |
| `Month Number` | `MONTH(Calendar[Date])` |

---

## 📐 DAX Measures — Full Reference

### Core Revenue

```dax
Total Sales := SUM(Data[Sales])
```
Fully filter-aware — responds to all slicers (date, category, supplier, article) via the `Calendar → Data` relationship.

---

### Revenue Grand Total (filter-independent denominator)

```dax
Total Sales Global :=
CALCULATE(
    SUM(Data[Sales]),
    ALL(Stocks),
    ALL(Data),
    ALL(Calendar)
)
```
Ignores all active filters — used as the denominator for revenue share calculations.

---

### Stock Total (cross-table filter via COUNTROWS pattern)

```dax
Total Stock :=
SUMX(
    FILTER(
        Stocks,
        COUNTROWS(
            FILTER(
                Data,
                Data[Code_Item] = Stocks[Item_Code]
            )
        ) > 0
    ),
    Stocks[Item Total Stock]
)
```

**Why this pattern:** `Stocks` has no date column — standard relationship filtering doesn't propagate from `Calendar` to `Stocks`. This measure manually checks which articles are visible in `Data` under the current filter context, then sums their stocks.

---

### Stock Grand Total (filter-independent denominator)

```dax
Total Stocks Global :=
CALCULATE(
    SUM(Stocks[Item Total Stock]),
    ALL(Stocks),
    ALL(Data),
    ALL(Calendar)
)
```

---

### Dynamic Top 10 (adapts to all active filters)

```dax
Top 10 Sales :=
IF(
    RANKX(
        ALLSELECTED(Data[Item_Label]),
        [Total Sales],
        ,
        DESC,
        DENSE
    ) <= 10,
    [Total Sales]
)
```

**Key design choice — `ALLSELECTED` vs `ALL`:**

| Function | Behavior |
|----------|---------|
| `ALL` | Ranks across all articles, ignores all filters — static Top 10 |
| `ALLSELECTED` | Ranks only among currently visible articles — **dynamic Top 10** |

`ALLSELECTED` ensures the Top 10 recalculates within whatever filter context is active — filter by category → Top 10 of that category; filter by month → Top 10 of that month.

---

## 📈 Dashboard Visuals

| Visual | Source | Configuration |
|--------|--------|--------------|
| Line chart — monthly revenue | Pivot Chart from Pivot Table | `Calendar[Month]` on axis, `Total Sales` as value, year as series |
| KPI Cards | Pivot Table | `Total Sales`, `Total Stock` with slicer context |
| Revenue share | Pivot Table | `Total Sales` / `Total Sales Global` |
| Stock share | Pivot Table | `Total Stock` / `Total Stocks Global` |
| Top 10 bar chart | Pivot Chart | `Top 10 Sales` measure, `Item_Label` on axis |
| Slicers | Connected to all Pivot Tables | Category, Supplier, Year, Month, Quarter |

---

## 💡 Key Skills Demonstrated

| Skill | Application |
|-------|-------------|
| **Root cause diagnosis** | Identified key format mismatch as the source of all incorrect measure results |
| **Power Query engineering** | Deduplication, error removal, Group By aggregation, custom column for key harmonization |
| **Stock consolidation** | Summed 5 warehouse columns into a single `Item Total Stock` via custom column |
| **Model refresh workflow** | Understood that structural Power Query changes require table removal + re-add in Power Pivot |
| **Cross-table DAX pattern** | `SUMX + FILTER + COUNTROWS` to propagate filters across tables with no direct date relationship |
| **ALLSELECTED vs ALL** | Applied correct scoping for dynamic Top 10 that respects active filter context |
| **Global denominators** | Used `CALCULATE + ALL(...)` to create filter-independent totals for share calculations |
| **Calendar table** | Built and linked a date dimension to enable clean time intelligence across all visuals |
| **End-to-end ownership** | Raw multi-source Excel data → cleaned → modeled → DAX → interactive dashboard |

---

## 📁 Project Structure

```
📁 Project/
├── Data.xlsx              ← Sales transactions — Item_Code, Date, Sales, Category, Supplier
├── Stocks.xlsx            ← Warehouse stocks — W1 to W5 per article + computed Item Total Stock
├── In_Store_Sales.xlsx    ← In-store sales — Sales_B1 to Sales_B4 + Item Sales per article
└── Dashboard.xlsx         ← Power Pivot model + DAX measures + Pivot Charts + Slicers
```

---

*This project demonstrates that Excel Power Pivot, when used with rigorous data preparation and well-architected DAX, can deliver professional-grade analytics dashboards — without Power BI or any external BI tool.*
