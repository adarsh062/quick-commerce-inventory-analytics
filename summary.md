# Zepto E-Commerce Inventory SQL Analysis: Project Summary & Blueprint

---

## 1. Executive Summary

This project is a **Beginner-Friendly, Pure SQL Data Analytics Portfolio Project** based on scraped product catalog and inventory data from **Zepto** (a leading Indian quick-commerce platform). 

Unlike multi-stage Python + SQL pipelines, this project focuses entirely on **SQL fundamentals** within PostgreSQL / pgAdmin:
- Database DDL & Schema definition
- In-database Data Exploration & Data Cleaning (`DELETE`, `UPDATE`)
- Business-oriented descriptive queries (Pricing, Discounts, Inventory weights, Out-of-Stock loss)

### Core Workflow:
```
Raw Dataset (zepto_v2.csv)
       │
       ▼
PostgreSQL Table Creation & Direct CSV Import (pgAdmin / \copy)
       │
       ▼
In-Database Data Cleaning (SQL DML)
  ├── Remove Zero MRP items (DELETE WHERE mrp = 0)
  └── Convert Paise to Rupees (UPDATE SET mrp = mrp / 100.0)
       │
       ▼
Business Analytics Queries (Zepto_SQL_data_analysis.sql)
  ├── Inventory Availability & Stockout Analysis
  ├── Discount & Best-Value Product Identification
  ├── Category-wise Potential Revenue & Total Weight
  └── Weight Bucketing (CASE WHEN: Low / Medium / Bulk)
```

---

## 2. File & Component Inventory

| File | Type | Purpose & Details |
| :--- | :--- | :--- |
| **`zepto_v2.csv`** | Dataset | Scraped inventory dataset containing 3,734 rows and 9 columns (MRP in paise, weights, stock flags). |
| **`Zepto_SQL_data_analysis.sql`** | SQL Script | 134-line PostgreSQL script containing table DDL, exploration, cleaning, and 8 business problem queries. |
| **`README.md`** | Documentation | Guide detailing project objectives, Kaggle source, schema description, and video walkthrough links. |
| **`LICENSE`** | Legal | MIT License. |

---

## 3. Database Schema & Data Dictionary

### Target Table: `zepto`
```sql
CREATE TABLE zepto (
  sku_id SERIAL PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp NUMERIC(8,2),
  discountPercent NUMERIC(5,2),
  availableQuantity INTEGER,
  discountedSellingPrice NUMERIC(8,2),
  weightInGms INTEGER,
  outOfStock BOOLEAN,
  quantity INTEGER
);
```

| Column | Type | Description | Notes |
| :--- | :--- | :--- | :--- |
| `sku_id` | `SERIAL PRIMARY KEY` | Auto-incrementing synthetic primary key | Generated during import |
| `category` | `VARCHAR(120)` | Product category (e.g. Fruits & Vegetables, Snacks) | Used for grouping |
| `name` | `VARCHAR(150)` | Product name | Multiple rows can have same name (different weights/SKUs) |
| `mrp` | `NUMERIC(8,2)` | Maximum Retail Price | Scraped in paise; converted to ₹ by dividing by 100 |
| `discountPercent` | `NUMERIC(5,2)` | Percentage discount | e.g. 15.00 for 15% |
| `availableQuantity`| `INTEGER` | Stock quantity available in dark store | Used for inventory valuation |
| `discountedSellingPrice` | `NUMERIC(8,2)` | Final customer selling price | Scraped in paise; converted to ₹ |
| `weightInGms` | `INTEGER` | Item weight in grams | Used for price per gram & weight bucketing |
| `outOfStock` | `BOOLEAN` | Stock status (`TRUE` / `FALSE`) | High-MRP out-of-stock items flag lost revenue |
| `quantity` | `INTEGER` | Units / pack quantity | Packaging unit count |

---

## 4. SQL Tasks & Business Problems Breakdown

### A. Data Exploration
1. **Row Count Check**: `SELECT COUNT(*) FROM zepto;`
2. **Data Sampling**: `SELECT * FROM zepto LIMIT 10;`
3. **Null Check**: Multi-column `OR` check to detect missing values across all critical attributes.
4. **Distinct Categories**: Identifying unique product categories across the catalog.
5. **Stockout Split**: Comparing count of active (`FALSE`) vs out-of-stock (`TRUE`) items.
6. **Multi-SKU Detection**: `GROUP BY name HAVING count(sku_id) > 1` to find products listed in multiple variants/pack sizes.

### B. Data Cleaning
1. **Invalid Price Removal**: 
   ```sql
   DELETE FROM zepto WHERE mrp = 0;
   ```
2. **Currency Standardization (Paise to Rupees)**:
   ```sql
   UPDATE zepto
   SET mrp = mrp / 100.0,
       discountedSellingPrice = discountedSellingPrice / 100.0;
   ```

### C. Business Analytics (8 Key Questions)
| # | Question / Metric | SQL Clause & Method | Business Insight |
| :-: | :--- | :--- | :--- |
| **Q1** | **Top 10 Best-Value Products** | `ORDER BY discountPercent DESC LIMIT 10` | Identifies headline discount items used to drive consumer app opens. |
| **Q2** | **High-MRP Out of Stock Items** | `WHERE outOfStock = TRUE AND mrp > 300 ORDER BY mrp DESC` | Flags critical revenue leakage on premium items due to stockouts. |
| **Q3** | **Estimated Category Revenue** | `SUM(discountedSellingPrice * availableQuantity) GROUP BY category` | Measures current on-shelf inventory valuation per category. |
| **Q4** | **Expensive Low-Discount Products** | `WHERE mrp > 500 AND discountPercent < 10` | High-margin staple goods that sell at full price without heavy promotions. |
| **Q5** | **Top 5 Discounted Categories** | `ROUND(AVG(discountPercent), 2) GROUP BY category LIMIT 5` | Shows which categories rely on aggressive discounting to move volume. |
| **Q6** | **Price per Gram Analysis** | `ROUND(discountedSellingPrice / weightInGms, 2) WHERE weightInGms >= 100` | Highlights true value-for-money items regardless of pack size. |
| **Q7** | **Weight Category Bucketing** | `CASE WHEN weightInGms < 1000 THEN 'Low' ... END` | Segments items into delivery weight tiers (crucial for 10-minute delivery rider payload limits). |
| **Q8** | **Total Inventory Weight per Category** | `SUM(weightInGms * availableQuantity) GROUP BY category` | Dark-store warehouse storage capacity planning. |

---

## 5. Difficulty & Complexity Assessment

### Overall Level: **Easy (Beginner)**
- **Prerequisites**: Basic SQL syntax (`SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `CASE WHEN`, `UPDATE`, `DELETE`).
- **No Complex Features**:
  - ❌ No Window Functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`, `LEAD`, `LAG`)
  - ❌ No CTEs (`WITH` clauses)
  - ❌ No Table Joins (Single-table queries only)
  - ❌ No Subqueries
  - ❌ No Python / CLI pipeline setup required

### Comparison with Walmart Project:
| Dimension | Walmart Project (`Walmart_SQL_Python`) | Zepto Project (`zepto-SQL-data-analysis-project`) |
| :--- | :--- | :--- |
| **Difficulty Level** | **Intermediate** (Advanced Beginner) | **Easy** (Pure Beginner) |
| **Technology Stack** | Python (Pandas, SQLAlchemy) + MySQL / PostgreSQL | Pure PostgreSQL / SQL |
| **ETL Complexity** | Python script handling regex `$`, dates, types | Direct SQL `UPDATE` / `DELETE` |
| **SQL Advancedness** | Window Functions, CTEs, Self-Joins, Shift categorization | Basic Aggregations (`SUM`, `AVG`), `CASE WHEN`, `LIMIT` |
| **Best For** | Resume standout project for job interviews | Day 1 learning / First-time SQL practice |

---

## 6. Recommendations to Elevate This Project

To make this project interview-worthy for a Business Analyst role:
1. **Add Window Functions**: E.g., Rank the top 3 highest-discount products *within each category* using `RANK() OVER (PARTITION BY category ORDER BY discountPercent DESC)`.
2. **Add Multi-condition Stock Health**: Calculate stock turnover ratio or dark-store storage utilization percentages.
3. **Build a Quick Dashboard**: Connect this table to Power BI or Tableau to build an *E-commerce Inventory & Stockout Monitoring Dashboard*.
