# credit-risk-analytics
Financial risk analysis using PostgreSQL and Power BI |  Credit default prediction | 32K loans | 8 SQL analyses | 3-page dashboard
# Financial Risk Analytics Dashboard
### Credit Risk Analysis | PostgreSQL + Power BI | Real Dataset

---

## Business Problem

A lending institution is facing an elevated default rate of **21.81%** — above the industry safe threshold of 15%. The CFO needs answers to three critical questions:
- Which borrower profiles are defaulting most?
- What is our total financial exposure at risk?
- What credit policy changes can reduce defaults?

This project delivers a complete end-to-end analysis — from raw data cleaning in PostgreSQL to an executive Power BI dashboard — answering all three questions with data.

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| PostgreSQL 18 | Database, data cleaning, SQL analysis |
| Power BI Desktop | 3-page interactive dashboard |
| DAX | KPI measures and dynamic calculations |
| Python | Dataset validation and verification |

---

## Dataset

**Source:** Credit Risk Dataset (Kaggle — real credit bureau data)
- **32,509 loans** after cleaning
- **12 columns** including grade, income, DTI, home ownership, default status
- Loan grades A through G (A = safest, G = most risky)

---

## Data Cleaning (PostgreSQL)

Identified and fixed 5 data quality issues before analysis:

```sql
-- 1. Removed impossible ages (144 year old borrowers)
DELETE FROM credit_risk WHERE person_age > 90;

-- 2. Removed impossible employment lengths (123 years)
DELETE FROM credit_risk WHERE person_emp_length > 60;

-- 3. Capped extreme incomes at $200K (99th percentile)
UPDATE credit_risk SET person_income = 200000 WHERE person_income > 200000;

-- 4. Filled missing employment length with median
UPDATE credit_risk SET person_emp_length = 4.0 WHERE person_emp_length IS NULL;

-- 5. Filled missing interest rates with grade-specific median
UPDATE credit_risk AS cr
SET loan_int_rate = sub.median_rate
FROM (
    SELECT loan_grade,
           PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY loan_int_rate) AS median_rate
    FROM credit_risk WHERE loan_int_rate IS NOT NULL GROUP BY loan_grade
) sub
WHERE cr.loan_int_rate IS NULL AND cr.loan_grade = sub.loan_grade;
```

---

## SQL Analysis — 8 Business Queries

### Key Findings

| Analysis | Finding |
|---|---|
| Overall Default Rate | **21.81%** — above 15% safe threshold |
| Worst Grade | Grade G = **98.44%** default rate |
| Financial Exposure | **$76.8M = 24.65%** of total portfolio at risk |
| Riskiest Purpose | Medical Loan = **19.35%** default rate |
| Past Default Impact | Prior defaulters = **37.56%** vs clean history **18.08%** |
| DTI Danger Zone | DTI > 40% = **74.05%** default rate |
| Home Ownership | Renters default at **4x** rate of homeowners (31.6% vs 7.5%) |

### Advanced SQL Concepts Used

```sql
-- Window Function: Top 3 largest defaulted loans per grade
WITH defaulted_ranked AS (
    SELECT loan_grade, loan_amnt, loan_intent, loan_int_rate,
           ROW_NUMBER() OVER (
               PARTITION BY loan_grade
               ORDER BY loan_amnt DESC
           ) AS row_num
    FROM credit_risk
    WHERE loan_status = 1
)
SELECT * FROM defaulted_ranked WHERE row_num <= 3
ORDER BY loan_grade, row_num;
```

---

## 3 Reusable SQL Views

```sql
-- View 1: Grade-level risk summary (used in Power BI Page 1)
CREATE VIEW vw_grade_risk_summary AS ...

-- View 2: Customer risk profile by DTI + home ownership
CREATE VIEW vw_customer_risk_profile AS ...

-- View 3: Default rate by loan purpose and grade
CREATE VIEW vw_intent_risk AS ...
```

Views allow Power BI to connect to pre-aggregated data — 
faster dashboard refresh, cleaner data model.

---

## Power BI Dashboard — 3 Pages

### Page 1 — Portfolio Risk Overview (CFO View)
- Default rate **Gauge** showing 21.81% vs 15% safe threshold
- **Waterfall chart** showing default rate progression Grade A → G
- **Stacked bar** — loan volume by purpose with default composition
- **DTI band bar chart** — shows 74% default rate at DTI > 40%

### Page 2 — Customer Risk Profile (Risk Manager View)
- **Scatter plot** — income vs loan amount colored by default status
- **Home ownership** bar chart — RENT vs OWN vs MORTGAGE default rates
- **Prior default history** comparison card
- **Key Influencers** AI visual — automated root cause of defaults

### Page 3 — Grade Drill-Through (Credit Team View)
- Right-click any grade on Page 1 → opens grade-specific detail
- Shows that grade's intent breakdown, income profile, interest rates
- **Back button** returns to Page 1

---

## Key Business Recommendations

1. **Reject Grade F/G + DEBTCONSOLIDATION combination** — 100% default rate across 186 loans. No exceptions.
2. **Hard DTI cap at 40%** — borrowers above this threshold default at 74%. Clear policy rule.
3. **Flag prior defaulters for enhanced review** — they default at 2x rate of clean-history borrowers.
4. **Renters require additional collateral** — 31.6% default vs 7.5% for homeowners.

---

## SQL Concepts Demonstrated

`SELECT` `WHERE` `GROUP BY` `ORDER BY` `CASE WHEN` `JOIN` `CTE` `Window Functions` `ROW_NUMBER()` `RANK()` `PARTITION BY` `Views` `PERCENTILE_CONT` `EXPLAIN ANALYZE` `Indexing` `DELETE` `UPDATE` `Subqueries`

## Power BI Concepts Demonstrated

`Live PostgreSQL connection` `Star schema` `DAX measures` `CALCULATE` `DIVIDE` `SUMX` `Gauge chart` `Waterfall chart` `Drill-through navigation` `Conditional formatting` `Scatter plot` `Key Influencers visual` `Top N filters`

---

## Project Structure

```
credit-risk-analytics/
├── 01_Dataset/
│   └── credit_risk_dataset.csv
├── 02_SQL/
│   ├── 01_schema_creation.sql
│   ├── 02_data_cleaning.sql
│   ├── 03_business_analysis.sql
│   ├── 04_views.sql
│   └── 05_indexing.sql
├── 03_PowerBI/
│   └── Credit_Risk_Dashboard.pbix
├── 04_Screenshots/
│   ├── page1_portfolio_overview.png
│   ├── page2_customer_profile.png
│   └── page3_grade_drillthrough.png
└── README.md
```

---

*Project 3 of 3 in Data Analytics Portfolio | SQL + Power BI*
*Previous projects: ShopEase E-Commerce Analytics | HR Attrition Analytics*
