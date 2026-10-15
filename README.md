# 🛒 Quick-Commerce Inventory & Pricing Analytics

An end-to-end SQL data analytics project analyzing product inventory, pricing strategies, and stockout patterns across 3,700+ SKUs from a quick-commerce retail platform.

---

## 📌 Project Overview

Quick-commerce businesses operate under tight delivery SLAs (10-15 minutes) and rely heavily on optimized dark-store inventory. This project explores real-world catalog and inventory data using **PostgreSQL** to identify revenue leakage from stockouts, assess discount distribution across product categories, and analyze delivery payload constraints.

### Key Objectives:
- **Database Setup & Modeling**: Design a clean relational schema to store multi-category product catalog data.
- **In-Database Data Cleansing**: Standardize corrupted price points and unit metrics using SQL DML (`UPDATE`, `DELETE`).
- **Exploratory Data Analysis (EDA)**: Detect stock availability rates, multi-SKU product variations, and missing data points.
- **Business Insights**: Formulate analytical SQL queries answering inventory health, pricing, and category profitability questions.

---

## 📁 Dataset & Schema

The dataset represents retail dark-store inventory containing **3,734 records** across various grocery and consumer categories.

### Schema Definition (`zepto` table):
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

### Data Dictionary:
| Column | Type | Description |
| :--- | :--- | :--- |
| `sku_id` | `SERIAL PRIMARY KEY` | Unique SKU identifier |
| `category` | `VARCHAR(120)` | Product category (e.g., Fruits & Vegetables, Snacks, Beverages) |
| `name` | `VARCHAR(150)` | Product title |
| `mrp` | `NUMERIC(8,2)` | Maximum Retail Price (converted to INR) |
| `discountPercent` | `NUMERIC(5,2)` | Applied discount percentage |
| `availableQuantity`| `INTEGER` | Current on-shelf stock count in dark store |
| `discountedSellingPrice` | `NUMERIC(8,2)` | Actual customer selling price (converted to INR) |
| `weightInGms` | `INTEGER` | Item weight in grams |
| `outOfStock` | `BOOLEAN` | Current availability status (`TRUE` / `FALSE`) |
| `quantity` | `INTEGER` | Pack size or unit count |

---

## 🧹 Data Cleaning (SQL DML)

1. **Purged Zero-Value Records**:
   ```sql
   DELETE FROM zepto
   WHERE mrp = 0;
   ```

2. **Currency Unit Standardization (Paise to INR)**:
   ```sql
   UPDATE zepto
   SET mrp = mrp / 100.0,
       discountedSellingPrice = discountedSellingPrice / 100.0;
   ```

---

## 📊 Key Business Analysis & Queries

### 1. High-Value Lost Revenue (Stockout Analysis)
Identify premium products (MRP > ₹300) currently out of stock to flag immediate restocking priorities:
```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```

### 2. Category-Level Inventory Valuation
Calculate total estimated on-shelf revenue currently held in stock per product category:
```sql
SELECT 
    category,
    SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue DESC;
```

### 3. Promotional Distribution by Category
Identify categories with the highest average discounts to understand promotional loss leaders:
```sql
SELECT 
    category,
    ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

### 4. True Value Analysis (Price per Gram)
Find cost-efficiency across products weighing at least 100g:
```sql
SELECT DISTINCT 
    name, 
    weightInGms, 
    discountedSellingPrice,
    ROUND(discountedSellingPrice / weightInGms, 2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram ASC;
```

### 5. Delivery Payload Segmentation
Segment products into weight classes to help rider dispatch manage payload limits:
```sql
SELECT DISTINCT 
    name, 
    weightInGms,
    CASE 
        WHEN weightInGms < 1000 THEN 'Low'
        WHEN weightInGms < 5000 THEN 'Medium'
        ELSE 'Bulk'
    END AS weight_category
FROM zepto;
```

---

## 📈 Key Findings & Insights

- **Stockout Impact**: Several high-MRP items (> ₹300) in personal care and packaged food had frequent stockouts, creating direct revenue leakage.
- **Discount Patterns**: Packaged snacks and beverages exhibited the highest promotional discounts (~15-18%) to drive repeat order frequency.
- **Inventory Weight Concentration**: Heavy staple categories (e.g., Atta, Rice, Beverages) accounted for the majority of warehouse storage load, requiring dedicated rider allocation for bulk deliveries.

---

## 🛠️ How to Run Locally

1. **Clone the repository**:
   ```bash
   git clone https://github.com/<your-username>/quick-commerce-inventory-analytics.git
   cd quick-commerce-inventory-analytics
   ```

2. **Database Setup**:
   - Open PostgreSQL (pgAdmin / DBeaver / psql CLI).
   - Execute the table creation script in `Zepto_SQL_data_analysis.sql`.
   - Import `zepto_v2.csv` into the `zepto` table.

3. **Execute Analysis**:
   - Run the data cleaning updates followed by the analytical queries in `Zepto_SQL_data_analysis.sql`.

---

## 📜 License
MIT License
