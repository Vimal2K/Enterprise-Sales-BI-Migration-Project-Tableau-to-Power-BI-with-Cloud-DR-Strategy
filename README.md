# Enterprise Sales BI Migration Project – Tableau to Power BI with Cloud & DR Strategy

## Short Description

This project simulates a **real-world enterprise BI modernization journey** where a growing company migrates from **Excel-based reporting to SSMS-driven analytics**, adopts **Tableau for initial visualization**, and later transitions to **Power BI with cloud migration to BigQuery**, using **MySQL as an on‑prem Disaster Recovery (DR)** system.

The project emphasizes **strong business storytelling, SQL-first modeling, BI tool migration, KPI parity validation, and DR readiness**, making it highly relevant for **Data Analyst / BI Engineer roles**.

---

## Company & Business Context

The company started as a **budget-constrained small enterprise** relying on Excel-based sales tracking. As revenue and data volume grew, leadership required:

* Centralized data storage
* Reliable KPI definitions
* Interactive dashboards for decision-makers
* Cloud scalability and disaster recovery

This led to a **phased analytics maturity journey** rather than a big-bang transformation.

---

## End-to-End Architecture Overview

```
Excel (Single File → 4 CSVs)
        ↓
SSMS (On‑Prem SQL Server)
        ↓
Tableau Dashboards (Initial BI)
        ↓
Power BI Migration & Enhancements
        ↓
BigQuery (Cloud Data Warehouse)
        ↓
MySQL (On‑Prem Disaster Recovery)
        ↓
Power BI Dataflow (Semantic Layer)
```
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ee65eb3f-3355-407f-b498-b53ff6e65678" />

---

## Phase 1 – Excel to SSMS (Foundation Layer)

### Why This Step Was Required

**Business Reason**

* Excel was error-prone and not scalable
* Multiple sheets caused inconsistent KPIs

**Technical Reason**

* SQL enables data type control, joins, and reusable logic

---

## Data Modeling – Star Schema

The data was split into **four datasets**, forming a classic **star schema**:

### Fact Table

* Fact_Table (Sales transactions)

### Dimension Tables

* Dim_Customers
* Dim_Product
* Dim_Promotion

This modeling approach improved:

* Query performance
* BI tool compatibility
* KPI consistency

---

## SQL Data Preparation & Transformation (SSMS)

### Data Type Standardization

**Dim_Customers**

* Customer_ID, Name, City, State, Email → NVARCHAR
* Phone_Number, Pincode → INT

**Dim_Product**

* Product_ID, Name, Line → TEXT
* Price_INR → INT

**Fact_Table**

* Units_Sold, Total_Sales, Price_Per_Unit → INT
* Discount_Value, Net_Sales → FLOAT

---

### Column Standardization

```sql
EXEC sp_rename 'Dim_Customers.EmailID', 'Email_ID', 'COLUMN';
EXEC sp_rename 'Dim_Product.ProductID', 'Product_ID', 'COLUMN';
EXEC sp_rename 'Dim_Promotion.PromotionID', 'Promotion_ID', 'COLUMN';
```

---

### Business Rule Implementation – Promotions

```sql
ALTER TABLE Dim_Promotion ADD Percentage INT;

UPDATE Dim_Promotion
SET Percentage = CASE
  WHEN Price_Reduction_Type LIKE '%Buy 1 Get 1%' THEN 50
  WHEN Price_Reduction_Type LIKE '%off'
    THEN CAST(REPLACE(REPLACE(Price_Reduction_Type, ' off', ''), '%', '') AS INT)
  ELSE NULL
END;
```

---

### Fact Table Enrichment

```sql
UPDATE f SET Price_Per_Unit = d.Price_INR
FROM Fact_Table f JOIN Dim_Product d ON f.Product_ID = d.Product_ID;

UPDATE Fact_Table SET Total_Sales = Units_Sold * Price_Per_Unit;

UPDATE f SET Discount_Percentage = ISNULL(p.Percentage,0)
FROM Fact_Table f LEFT JOIN Dim_Promotion p ON f.Promotion_ID = p.Promotion_ID;

UPDATE Fact_Table SET Discount_Value = (Total_Sales * Discount_Percentage) / 100.0;
UPDATE Fact_Table SET Net_Sales = Total_Sales - Discount_Value;
UPDATE Fact_Table SET Profit = Net_Sales * 0.10;
```

---

## Phase 2 – Tableau BI Implementation

### Tableau Dashboards Built

1. **Overview Dashboard**

   * Net Sales Trend
   * Sales vs Profit Scatterplot
   * Promotion vs Average Discount

2. **Top & Bottom 5 Analysis Dashboard**

   * Products by Net Sales
   * Units Sold
   * Profit

---

## Tableau Calculated Fields (Geo Fixes)

### Issue Identified

Tableau failed to detect certain cities/states due to incorrect naming.

### Fix Applied – Calculated Fields

```tableau
IF [City] = "Delhi" THEN "New Delhi" ELSE [City] END
```

```tableau
CASE [City]
WHEN "Banglore" THEN "Bengaluru"
WHEN "Bangalore" THEN "Bengaluru"
ELSE [City]
END
```

**Why This Matters**

* Shows real-world data quality issues
* Demonstrates BI debugging skills

## Tableau Dashboard

**Dashboard 1: Overview**

<img width="1918" height="856" alt="image" src="https://github.com/user-attachments/assets/d94bfe53-6de5-49b3-87f5-765fa55d1399" />

**Dashboard 2: Top Bottom Analysis**

<img width="1876" height="850" alt="image" src="https://github.com/user-attachments/assets/bb7b1274-ba60-4537-af6a-9f9d1aec90d4" />

---

## Phase 3 – BI Tool Migration (Tableau → Power BI)

### Business Reason

* Power BI adoption aligned with company growth
* Better Microsoft ecosystem integration
* Lower licensing cost

### Migration Strategy

* Migrate **5 core KPIs first**
* Enhance UI/UX
* Add slicers and drill-down capability

## Power BI Dashboard with enhancement

**Dashboard 1: Overview**

<img width="1163" height="728" alt="image" src="https://github.com/user-attachments/assets/13ee1e3f-13d1-4387-a1ec-eb9f7af1c2cd" />

**Dashboard 2: Top Bottom Analysis**

<img width="1172" height="663" alt="image" src="https://github.com/user-attachments/assets/52dbc02c-fc71-4e9c-b835-2dfee0589b37" />

## Modeling Issue & Fix in Power BI

### Issue

* Promotion names showing same value

### Root Cause

* Missing relationship
* Data type mismatch (numeric vs text)

### Fix

* Correct datatype
* Create one-to-many relationship
* Filter blanks

---

## Phase 4 – SSMS to BigQuery Migration

### Migration Method

* Manual CSV export
* Dataset recreation in BigQuery

### Validation Strategy

```sql
-- Row Count Validation
SELECT COUNT(*) FROM Fact_Table;

-- Aggregation Validation
SELECT SUM(Net_Sales), MIN(Net_Sales), MAX(Net_Sales) FROM Fact_Table;
```

Same logic executed on **SSMS and BigQuery** to ensure parity.

---

## Phase 5 – MySQL Disaster Recovery

### Why MySQL

* On‑prem reliability
* Works during cloud outages
* Lightweight auditing access

Migration performed using **MySQL Migration Wizard**.

Validation repeated using:

* Row counts
* Distinct counts
* Aggregations

---

## Phase 6 – Dataflow & Final Power BI Setup

* Dataflow connected to BigQuery
* Data types standardized
* Naming conflicts resolved
* Scheduled refresh enabled

SSMS datasource **fully decommissioned**.

---

## KPI Parity Validation

* Same KPIs
* Same filters
* Same visuals
* Same results across:

  * SSMS
  * BigQuery
  * MySQL

---

## Future Improvements

1. **Automated Migration Pipelines**

   * Use managed ETL tools

2. **Dashboard Polishing**

   * Advanced UX patterns
   * Performance tuning

3. **CI/CD for Analytics**

   * Version control dashboards
   * Automated deployment

These steps align with the company’s transition from **low-level to mid-level analytics maturity**.

---

## Final Note

This project demonstrates:

* Enterprise BI thinking
* Strong SQL & DAX skills
* Real migration challenges
* Tool-agnostic analytics design

✅ **This document is GitHub-ready and interview-ready without further edits.**
