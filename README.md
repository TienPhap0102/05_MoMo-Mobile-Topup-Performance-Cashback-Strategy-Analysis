# MoMo Mobile Topup Performance & Cashback Strategy Analysis

[View the live Dashboard](https://app.powerbi.com/view?r=eyJrIjoiOTlhMjI1ZTktOTI0Ny00YmVjLTg0NzQtZGMzMmFlYzY3ODJmIiwidCI6IjM3MGZiM2I4LTMzMDYtNDg5MC05MDYzLWNjMDhiZTc4ODI1NyIsImMiOjEwfQ%3D%3D](https://app.powerbi.com/view?r=eyJrIjoiZmViZTA5YmEtZDI4OC00ZjNlLWIwNzEtN2RmZDBjOWUyNjkyIiwidCI6IjM3MGZiM2I4LTMzMDYtNDg5MC05MDYzLWNjMDhiZTc4ODI1NyIsImMiOjEwfQ%3D%3D)) 

An end-to-end analytics project built with **SQL Server** and **Power BI** to evaluate Mobile Topup performance, identify growth drivers across time, merchants, and customer segments, and assess the financial sustainability of a proposed cashback strategy.

> **Core decision:** Growth should be scaled only when incremental contribution offsets the additional cashback cost.

---

## Table of Contents

- [Business Context](#business-context)
- [Business Objectives](#business-objectives)
- [Business Requirements](#business-requirements)
- [Data Scope](#data-scope)
- [Analytical Approach](#analytical-approach)
- [Data Processing with SQL](#data-processing-with-sql)
- [Data Model](#data-model)
- [Core KPI Definitions](#core-kpi-definitions)
- [Descriptive Analysis](#descriptive-analysis)
- [Power BI Report](#power-bi-report)
- [Key Findings](#key-findings)
- [Recommendations](#recommendations)
- [Cashback Experiment Framework](#cashback-experiment-framework)
- [Repository Structure](#repository-structure)
- [Limitations](#limitations)
- [Author](#author)

---

## 🧭 Business Context

MoMo's Mobile Topup service enables users to recharge their own mobile balance or purchase mobile cards for others. The service is strategically important because of its broad reach, recurring transaction potential, and contribution to MoMo's ecosystem revenue.

The Vietnamese e-wallet market is highly competitive, with providers using aggressive cashback programmes to attract and retain users. MoMo is therefore considering increasing cashback rates across telecommunications merchants.

Before implementing this strategy, the business needs to understand:

- The current performance of the Mobile Topup service.
- The main drivers of GMV, commission revenue, and user activity.
- The differences in scale and monetisation across merchants.
- The acquisition, value, and transaction behaviour of customer segments.
- Whether incremental GMV can offset the additional cashback cost.

This project uses historical transaction data from **January to December 2020**, customer demographic information, and merchant commission rates.

---

## 🎯 Business Objectives

### 🔭 Overall objective

Develop an end-to-end analytical solution using SQL Server and Power BI that monitors Mobile Topup performance, identifies key growth drivers, and evaluates the financial sustainability of MoMo's proposed cashback strategy.

### ✅ Specific objectives

1. Measure monthly performance through transaction, customer, and financial metrics.
2. Identify the main drivers behind changes in GMV and commission revenue.
3. Understand customer demographics and transaction behaviour, especially differences between new and current users.
4. Evaluate merchant-level scale, monetisation, and contribution.
5. Quantify the financial impact of proposed cashback rates.
6. Estimate the GMV growth required to preserve current contribution.
7. Provide actionable recommendations for Marketing, Product, Partnership, and Finance teams.

---

## 📋 Business Requirements

The analytical solution must allow stakeholders to:

- Monitor orders, active users, GMV, gross revenue, contribution, AOV, ARPU, and purchase frequency.
- Analyse performance by month and telecommunications merchant.
- Compare self-use transactions with purchases made for other users.
- Track new-user acquisition and changes in the new-user rate.
- Compare merchant GMV share, revenue share, commission rate, and contribution.
- Analyse customer value by age group, gender, location, and lifecycle segment.
- Simulate different GMV uplift assumptions under the proposed cashback rates.
- Calculate merchant-level and portfolio-level break-even thresholds.
- Identify offers with financially impossible break-even conditions.
- Translate analytical results into clear decision guidance.

---

## 🗂️ Data Scope

| Data domain | Main fields | Analytical purpose |
|---|---|---|
| Transactions | Transaction date, value, user, merchant, purchase type | Orders, GMV, AOV, trends, and purchase behaviour |
| Customers | User ID, age, gender, location, customer status | Acquisition, demographics, and customer value |
| Merchants | Merchant name and commission rate | Revenue, contribution, and merchant economics |
| Date | Calendar date, month, and reporting attributes | Time intelligence and monthly comparison |
| Scenario inputs | Current cashback, proposed cashback, assumed GMV lift | Financial simulation and break-even analysis |

---

## 🔎 Analytical Approach

1. **Data profiling** — Reviewed source structure, data types, missing values, and business rules.
2. **Data preparation** — Standardised dates, merchant names, customer attributes, and transaction values in SQL Server.
3. **Business logic** — Created reusable calculations for GMV, commission revenue, cashback cost, and contribution.
4. **Data modelling** — Built a reporting model using date, customer, and merchant dimensions.
5. **DAX development** — Created governed measures for KPIs, time comparison, segmentation, and scenario analysis.
6. **Validation** — Reconciled portfolio, monthly, and merchant-level totals.
7. **Business interpretation** — Converted descriptive results into actions for Product, Marketing, Partnership, and Finance teams.

---

## 🧹 Data Processing with SQL

SQL Server was used to build a three-layer data pipeline before loading the analytical model into Power BI:

```mermaid
flowchart LR
    A[Raw Layer] --> B[Staging Layer]
    B --> C[Data Warehouse]
    C --> D[Power BI]
```

| Layer | Schema | Responsibility |
|---|---|---|
| Raw | `raw` | Preserve source data in its original structure for traceability |
| Staging | `stg` | Clean, standardise, validate, deduplicate, and derive reusable fields |
| Data Warehouse | `dw` | Store dimensional tables and the transaction fact used by Power BI |

This design separates ingestion, transformation, and reporting logic. Raw data remains unchanged, cleaning rules are auditable in staging, and Power BI connects only to governed warehouse objects.

### Creating the three SQL schemas

```sql
--- 1. CREATE DATABASE 
CREATE DATABASE DB_MoMo_Topup 
;
--- 2. CREATE SCHEMA RAW 
CREATE SCHEMA raw
;
--- 2. CREATE SCHEMA STAGING 
CREATE SCHEMA stg
;
--- 2. CREATE SCHEMA DATA WAREHOUSE
CREATE SCHEMA dw
;
```

### Layer 1 — Raw data

The raw layer receives the source datasets without applying business transformations. Its purpose is to preserve the original values and provide a reproducible starting point for the pipeline.

Example raw tables:

- `raw.data_transaction`
- `raw.data_user_info`
- `raw.data_commission`

Rules applied to this layer:

- Keep original column names and source values.
- Do not overwrite incorrect or inconsistent records.
- Add load metadata where available, such as `source_file_name` and `loaded_at`.
- Restrict Power BI from reading raw tables directly.

### Layer 2 — Staging and data cleaning

The staging layer converts raw source values into clean and consistent business fields. Data-quality flags are retained so invalid records can be reviewed instead of silently removed.

#### Data-quality assessment

| Table | Column | Data problem | Preprocessing solution | Standardised output |
|---|---|---|---|---|
| `data_transaction` | `Date` | Mixed formats such as `YYYY-MM-DD` and `DD-MM-YYYY` | Parse both recognised formats with `TRY_CONVERT` | SQL `DATE` in `YYYY-MM-DD` format |
| `Data User_Info` | `First_tran_date` | Invalid values such as `9920-12-12` | Convert valid dates and flag impossible years for review | Clean date or data-quality flag |
| `Data User_Info` | `Location` | Multiple labels for the same location | Standardise equivalent values with `CASE` | `Ho Chi Minh City`, `Ha Noi`, `Other Cities` |
| `Data User_Info` | `Gender` | Inconsistent spelling and language, such as `Femal`, `FEMALE`, `F`, and `Nữ` | Normalise text, then map equivalent values | `Female`, `Male`, or `Unknown` |

#### Derived fields created in staging

| Derived field | Purpose |
|---|---|
| `transaction_date` | Consistent transaction-level date analysis |
| `month_start_date` | Monthly aggregation and Power BI date relationships |
| `month_year` | User-friendly reporting label |
| `standardised_location` | Consistent geographic grouping |
| `standardised_gender` | Consistent demographic analysis |
| `customer_status` | Separation of new and current users |
| `purchase_status` | Separation of self-use and purchase-for-others transactions |
| `age_group` | Demographic comparison by age band |
| `gross_revenue` | Transaction value multiplied by merchant commission rate |
| `current_contribution` | Gross revenue after current cashback cost |

#### Staging output

The staging layer produces clean, reusable datasets such as:

- `stg.transaction`
- `stg.user_info`
- `stg.merchant`

These objects become the controlled source for the Data Warehouse layer.

### Layer 3 — Data Warehouse

The Data Warehouse applies a star schema optimised for analysis. Dimensions provide descriptive attributes, while the fact table stores transaction-level measures and foreign keys.

| Warehouse object | Grain | Main purpose |
|---|---|---|
| `dw.dim_date` | One row per calendar date | Time intelligence and consistent date filtering |
| `dw.dim_user` | One row per user | Demographics, location, acquisition, and customer status |
| `dw.dim_merchant` | One row per merchant | Merchant name, commission rate, and commercial attributes |
| `dw.fact_transaction` | One row per completed transaction | Orders, GMV, revenue, cashback cost, and contribution |

#### Creating `dw.dim_date`

The date dimension covers the complete analysis period and provides attributes required for Power BI time intelligence.

```sql
--- Create dim_date
CREATE TABLE dw.dim_date
(
    full_date          DATE          NOT NULL,
    day_number         TINYINT       NOT NULL,
    weekday_number     TINYINT       NOT NULL,
    weekday_name       NVARCHAR(10)  NOT NULL,
    month_number       TINYINT       NOT NULL,
    month_name         NVARCHAR(10)  NOT NULL,
    quarter_number     TINYINT       NOT NULL,
    quarter_name       CHAR(2)       NOT NULL,
    year_number        SMALLINT      NOT NULL,
    year_month         CHAR(7)       NOT NULL,
    year_month_sort    INT           NOT NULL,
    is_weekend         BIT           NOT NULL,
    CONSTRAINT PK_dw_dim_date
        PRIMARY KEY (full_date)
);

DECLARE @start_date DATE;
DECLARE @end_date DATE;
SELECT
    @start_date = MIN(transaction_date),
    @end_date   = MAX(transaction_date)
FROM staging.topup_transaction;
;WITH date_series AS
(
    SELECT @start_date AS full_date
    UNION ALL
    SELECT DATEADD(DAY, 1, full_date)
    FROM date_series
    WHERE full_date < @end_date
),
date_calculation AS
(
    SELECT
        full_date,
        CAST(DATEDIFF(DAY, '19000107', full_date) % 7 + 1  AS TINYINT) AS weekday_number
    FROM date_series
)
INSERT INTO dw.dim_date
(
    full_date,
    day_number,
    weekday_number,
    weekday_name,
    month_number,
    month_name,
    quarter_number,
    quarter_name,
    year_number,
    year_month,
    year_month_sort,
    is_weekend
)
SELECT
    full_date,
    DAY(full_date),
    weekday_number,
    CHOOSE(
        weekday_number,
        N'Sunday',
        N'Monday',
        N'Tuesday',
        N'Wednesday',
        N'Thursday',
        N'Friday',
        N'Saturday'
    ),
    MONTH(full_date),
    CHOOSE(
        MONTH(full_date),
        N'January',
        N'February',
        N'March',
        N'April',
        N'May',
        N'June',
        N'July',
        N'August',
        N'September',
        N'October',
        N'November',
        N'December'
    ),
    DATEPART(QUARTER, full_date),
    CONCAT('Q', DATEPART(QUARTER, full_date)),
    YEAR(full_date),
    CONVERT(CHAR(7), full_date, 126),
    YEAR(full_date) * 100 + MONTH(full_date),
    CASE
        WHEN weekday_number IN (1, 7) THEN 1
        ELSE 0
    END
FROM date_calculation
OPTION (MAXRECURSION 0)
```

`day_of_week_number` is calculated independently of the SQL Server `DATEFIRST` setting, with Monday represented by 1 and Sunday by 7.

Power BI connects to the `dw` schema rather than the raw or staging layers. This keeps report logic consistent and prevents data-cleaning rules from being duplicated in Power Query or DAX.

---

## 🧩 Data Model

The Power BI model follows a star-schema approach:

```mermaid
flowchart TB
    DD[dw.dim_date] --> FT[dw.fact_transaction]
    DU[dw.dim_user] --> FT
    DM[dw.dim_merchant] --> FT
    SP[dw.vw_cashback_scenario_compare] -.-> FT
```

Key modelling principles:

- One-to-many relationships from `dw.dim_date`, `dw.dim_user`, and `dw.dim_merchant` to `dw.fact_transaction`.
- `dw.dim_date[full_date]` filters transaction activity through the warehouse date key.
- `month_start_date`, month, quarter, and year fields come from the governed date dimension.
- Measures used for rates and totals instead of directly aggregating raw rate columns.
- Disconnected parameter tables used for scenario simulation and sensitivity analysis.
- Power BI imports only Data Warehouse objects; Raw and Staging remain outside the semantic model.

---

## 📐 Core KPI Definitions

| KPI | Definition |
|---|---|
| Total GMV | Total value of completed Mobile Topup transactions |
| Gross Revenue | Commission revenue earned from merchants before cashback cost |
| Current Cashback Cost | GMV multiplied by the current cashback rate |
| Current Contribution | Gross Revenue minus Current Cashback Cost |
| AOV | GMV divided by Total Orders |
| Gross ARPU | Gross Revenue divided by Active Users |
| Purchase Frequency | Total Orders divided by Active Users |
| New-user Rate | New Users divided by Total Active Users |
| Scenario Contribution | Proposed contribution adjusted by the selected assumed GMV lift |
| Contribution Gap | Scenario Contribution minus Current Contribution |
| Required GMV Lift | Current Contribution divided by Proposed Contribution, minus one |

> The **Required GMV Lift** is a mathematical break-even threshold, not a growth forecast. The **Assumed GMV Lift** is a scenario input, not an observed causal effect.

---

## 📊 Descriptive Analysis

SQL descriptive analysis was completed before dashboard development to answer the initial business requirements and establish validated benchmark figures.

 **OVER PERFORMANCE** 
 ```sql
SELECT 
        COUNT(DISTINCT order_id) AS Total_orders,
        COUNT(DISTINCT user_id) AS Total_customers,
        SUM(amount) AS Total_GMV,
        ROUND(SUM(amount) * 1.00 / COUNT(DISTINCT order_id), 2) AS AOV,
        SUM(revenue) AS Total_commission_revenue
FROM dw.fact_topup_transaction;
```
 Total_orders | Total_customers | Total_GMV | AOV | Total_commission_revenue |
|---|---|---|---|---|
 13496 | 13391 | 696604234 | 51615 | 18752727 |

### Requirement 1 — What was the gross revenue in January 2020?

```sql
SELECT
    SUM(gross_revenue) AS january_gross_revenue
FROM reporting.vw_topup_performance
WHERE transaction_date >= '2020-01-01'
  AND transaction_date <  '2020-02-01';
```

**Answer:** January generated approximately **1,409,827 VND in gross revenue**.

### Requirement 2 — Which month generated the highest business performance?

```sql
SELECT TOP (1)
    month_start_date,
    SUM(gmv) AS total_gmv,
    SUM(gross_revenue) AS gross_revenue
FROM reporting.vw_topup_performance
GROUP BY month_start_date
ORDER BY gross_revenue DESC;
```

**Answer:** **September 2020** recorded the highest gross revenue. It also generated **65.4M VND in GMV** and the highest monthly AOV of approximately **55K VND**.

### Requirement 3 — How did revenue vary by day of the week?

```sql
SELECT
    DATENAME(weekday, transaction_date) AS weekday_name,
    DATEPART(weekday, transaction_date) AS weekday_number,
    SUM(gross_revenue) AS gross_revenue,
    COUNT(DISTINCT transaction_id) AS total_orders
FROM reporting.vw_topup_performance
GROUP BY
    DATENAME(weekday, transaction_date),
    DATEPART(weekday, transaction_date)
ORDER BY weekday_number;
```

This query provides the weekday revenue pattern used to identify operational peaks. The repository should report the final weekday ranking only after validating the server's `DATEFIRST` setting.

### Requirement 4 — How many new users were acquired each month?

```sql
WITH monthly_user_acquisition AS
(
    SELECT
        d.year_month,
        d.year_month_sort,
        COUNT(DISTINCT f.user_id)
            AS total_transacting_users,
        COUNT(
            DISTINCT CASE
                WHEN f.user_type = N'New'
                THEN f.user_id
            END
        ) AS new_users,
        COUNT(
            DISTINCT CASE
                WHEN f.user_type = N'Current'
                THEN f.user_id
            END
        ) AS current_users,
        COUNT(
            DISTINCT CASE
                WHEN f.user_type = N'Unknown'
                THEN f.user_id
            END
        ) AS unknown_users
    FROM dw.fact_topup_transaction AS f
    INNER JOIN dw.dim_date AS d
        ON f.transaction_date = d.full_date
    GROUP BY
        d.year_month,
        d.year_month_sort
)
SELECT
    year_month,
    total_transacting_users,
    new_users,
    current_users,
    unknown_users,
    CAST(
        new_users * 100.0
        / NULLIF(total_transacting_users, 0)
        AS DECIMAL(10,2)
    ) AS new_user_rate_pct
FROM monthly_user_acquisition
ORDER BY
    year_month_sort;
-- New user in month 12: 76, 6.16%
```

**Answer:** New-user acquisition peaked in **March at approximately 113 users** and declined to **76 users in December**. The new-user rate fell from **10.29% to 6.16%** over the same comparison.

### Requirement 5 — Which purchase behaviour generated more value?

| Purchase type | Order share | GMV share | AOV |
|---|---:|---:|---:|
| Self-use | 83.44% | 63.62% | 39.36K VND |
| Purchase for others | 16.56% | 36.38% | 113.39K VND |

Although purchase-for-others transactions represented only **16.6% of orders**, they generated **36.4% of GMV** and an AOV approximately **2.9 times** higher than self-use transactions.

### Descriptive-analysis conclusion

The initial SQL analysis established three business themes that shaped the Power BI report:

1. Performance peaked in September, but the underlying drivers needed to be separated by merchant and purchase behaviour.
2. Acquisition weakened toward year-end while transaction frequency remained low.
3. High transaction scale did not necessarily produce high contribution because merchant commission rates differed.

---

## 📈 Power BI Report

The report contains five pages that follow a decision-oriented analytical flow.

### 🏠 1. Introduction

Explains the business problem, analytical objectives, headline findings, and recommended strategic direction.

![Introduction Page](<img width="1544" height="882" alt="image" src="https://github.com/user-attachments/assets/6c0ec156-7c7a-4177-b1f6-ebaf1f032342" />)

### 📊 2. Business Overview

Provides an executive view of Mobile Topup scale, financial performance, monthly trends, and purchase behaviour.

Main components:

- Total GMV, Gross Revenue, Current Contribution, Orders, and Users.
- Monthly performance trends.
- Self-use versus purchase-for-others behaviour.
- AOV, Gross ARPU, and Purchase Frequency.

![Business Overview](assets/images/02-business-overview.png)

### 🏪 3. Merchant Performance

Separates merchant scale from monetisation efficiency and financial contribution.

Main components:

- Merchant ranking by GMV, revenue, and contribution.
- Monthly performance by merchant.
- GMV share versus revenue share.
- Customer value map using active users and AOV.
- Merchant operating table.

![Merchant Performance](assets/images/03-merchant-performance.png)

### 👥 4. Customer Insight

Evaluates acquisition health, demographics, geographic performance, customer value, and RFM-based lifecycle opportunities.

Main components:

- Total Users, New Users, New-user Rate, GMV per User, and Revenue per User.
- New versus current users by month.
- Performance by gender, location, and age group.
- Demographic and RFM segmentation views.

![Customer Insight](assets/images/04-customer-insight.png)

### 🎚️ 5. Cashback Scenario

Tests whether incremental GMV can recover the contribution sacrificed by the proposed cashback rates.

Main components:

- Current Contribution, Scenario Contribution, and Contribution Gap.
- Assumed GMV Lift parameter.
- Current versus scenario contribution by merchant.
- Merchant-level break-even requirements.
- Contribution sensitivity analysis.
- Dynamic decision guidance and scenario details.

![Cashback Scenario](assets/images/05-cashback-scenario.png)

---

## 💡 Key Findings

### 📈 Business performance

- Total GMV reached **696.60M VND**.
- Gross Revenue reached **18.75M VND**.
- Current Contribution reached **11.79M VND**.
- September was the peak month, generating **65.4M VND in GMV** and the highest monthly AOV of approximately **55K VND**.
- Purchase Frequency remained close to **1.01 order per active user**, showing limited repeat usage.

### 🛒 Purchase behaviour

- Purchase-for-others transactions represented only **16.6% of orders** but generated **36.4% of GMV**.
- Purchase-for-others AOV was approximately **113.39K VND**, compared with **39.36K VND** for self-use transactions.
- This behaviour represents a high-value growth opportunity despite its lower transaction volume.

### 🏪 Merchant performance

| Merchant | GMV | GMV share | Commission | Revenue | Contribution |
|---|---:|---:|---:|---:|---:|
| Viettel | 359.62M | 51.62% | 2% | 7.19M | 3.60M |
| Mobifone | 191.90M | 27.55% | 3% | 5.76M | 3.84M |
| Vinaphone | 123.32M | 17.70% | 4% | 4.93M | 3.70M |
| Vietnamobile | 21.68M | 3.11% | 4% | 0.87M | 0.65M |

- Viettel generated **51.6% of GMV** but only **38.4% of gross revenue** because its commission rate was limited to 2%.
- Mobifone generated the highest current contribution at approximately **3.84M VND**.
- Vinaphone generated **26.3% of gross revenue from 17.7% of GMV**, demonstrating strong monetisation efficiency.

### 👥 Customer insight

- The new-user rate declined from **10.29% in March to 6.16% in December**.
- March recorded approximately **113 new users**, compared with only **76 in December**.
- Customers aged **23–27** generated the highest total revenue and provided scale.
- Customers aged **over 37** recorded the highest Gross ARPU and provided higher individual value.
- Other Cities generated approximately **9.5M VND in revenue**, ahead of Ho Chi Minh City and Ha Noi.

### 💰 Cashback scenario

| Metric | Result |
|---|---:|
| Current Contribution | 11.79M VND |
| Proposed Contribution at 0% uplift | 2.41M VND |
| Contribution Gap | -9.38M VND |
| Contribution Reduction | 79.5% |
| Required Portfolio GMV Lift | Approximately 389% |

- The proposed policy would reduce contribution by **79.5%** if it produced no incremental GMV.
- The portfolio would require approximately **389% GMV uplift** to preserve current contribution.
- Viettel has no finite break-even point because its proposed 2% cashback rate equals its 2% commission rate.
- Mobifone requires approximately **300% uplift**.
- Vinaphone and Vietnamobile each require approximately **200% uplift**.
- At an assumed **310% uplift**, scenario contribution remains approximately **16.2% below** current contribution.

---

## 🚀 Recommendations

### 🧭 Portfolio strategy

- Do not proceed with a blanket cashback rollout.
- Replace the uniform policy with merchant-specific, segment-aware tests.
- Use contribution and incremental contribution as decision KPIs instead of GMV alone.

### 🤝 Merchant strategy

- **Viettel:** Reduce the proposed cashback rate below the commission rate or secure merchant funding.
- **Mobifone:** Protect contribution leadership and test whether additional volume can be acquired efficiently.
- **Vinaphone:** Prioritise controlled growth because monetisation is strong relative to GMV share.
- **Vietnamobile:** Use smaller targeted tests because its current scale and total financial impact are limited.

### 🎯 Customer strategy

- Separate acquisition, first-transaction activation, and repeat-use campaigns.
- Build second-purchase journeys to improve the low transaction frequency.
- Target customers aged 23–27 with frequency and retention propositions.
- Target customers aged over 37 with higher-value and convenience-focused propositions.
- Develop family, gifting, and purchase-for-others use cases.
- Use RFM segmentation to prioritise high-value, at-risk, and reactivation audiences.

---

## 🧪 Cashback Experiment Framework

### 🧬 Recommended design

1. Randomly assign eligible users to treatment and control groups within each merchant.
2. Keep campaign eligibility and communication consistent except for cashback exposure.
3. Cap cashback per user and at campaign level.
4. Measure outcomes during the campaign and through a post-campaign retention period.
5. Evaluate results separately by merchant and priority customer segment.

### 🏆 Primary success metric

```text
Incremental Contribution
= Incremental Gross Revenue - Incremental Cashback Cost
```

### 🚦 Decision rules

- **Scale:** Incremental contribution is positive and repeat behaviour persists.
- **Iterate:** GMV increases but contribution remains slightly negative; reduce the rate, narrow eligibility, or seek merchant funding.
- **Stop:** Incremental cashback cost exceeds incremental gross revenue or the campaign reaches its stop-loss threshold.
- **Reject:** The post-cashback contribution rate is zero or negative.

---

## 📁 Repository Structure

```text
MoMo-Mobile-Topup-Analysis/
│
├── README.md
├── data/
│   └── data-dictionary.xlsx
│
├── sql/
│   ├── 01-create-schemas.sql
│   ├── 02-load-raw-data.sql
│   ├── 03-build-staging-layer.sql
│   ├── 04-build-dim-date.sql
│   ├── 05-build-data-warehouse.sql
│   ├── 06-descriptive-analysis.sql
│   ├── 07-business-overview.sql
│   ├── 08-merchant-performance.sql
│   ├── 09-customer-insight.sql
│   └── 10-cashback-scenario.sql
│
├── power-bi/
│   └── momo-mobile-topup-analysis.pbix
│
├── documentation/
│   └── MoMo_Mobile_Topup_Analysis_Documentation.docx
│
└── assets/
    ├── icons/
    └── images/
        ├── 01-introduction.png
        ├── 02-business-overview.png
        ├── 03-merchant-performance.png
        ├── 04-customer-insight.png
        └── 05-cashback-scenario.png
```

Rename the folders and files above if the repository uses different names, then update the links in this README.

---

## ⚠️ Limitations

- The analysis uses historical data from January to December 2020 and may not reflect current market conditions or merchant contracts.
- Break-even uplift is a mathematical threshold, not a forecast of achievable growth.
- The scenario assumes proposed cashback rates remain fixed while GMV changes proportionally.
- Historical trends and segment differences are descriptive and do not establish causal campaign impact.
- Unknown demographic and location values may reduce segment precision.
- Merchant-funded promotions or renegotiated commission rates would change the scenario economics.

---

## 👤 Author

**[Your Name]**  
Data Analyst  

- LinkedIn: [Add your LinkedIn URL](https://www.linkedin.com/)
- Portfolio: [Add your portfolio URL](https://github.com/)
- Email: `your.email@example.com`

---

If you found this project useful, consider giving the repository a star.
