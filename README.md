# Quick-Commerce Inventory Analytics

A SQL project analyzing inventory availability, product pricing, and stockout patterns across 3,700+ retail products using PostgreSQL.

---

## Project Overview

In quick-commerce (10-15 minute delivery), managing store inventory and stock availability is critical to prevent lost sales. 

This project uses PostgreSQL to:
1. Set up and clean a real-world product catalog dataset.
2. Fix invalid prices and standardize currency units.
3. Identify out-of-stock products causing potential revenue loss.
4. Analyze category discounts and classify products by delivery weight.

---

## Database Schema

Table name: `zepto`

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

### Column Descriptions

- `sku_id`: Unique identifier for each product.
- `category`: Product category (e.g., Fruits and Vegetables, Snacks, Beverages).
- `name`: Product name.
- `mrp`: Maximum Retail Price (in Rupees).
- `discountPercent`: Discount percentage.
- `availableQuantity`: Units available in stock.
- `discountedSellingPrice`: Final selling price (in Rupees).
- `weightInGms`: Product weight in grams.
- `outOfStock`: Stock status (TRUE = out of stock, FALSE = in stock).
- `quantity`: Pack size or quantity.

---

## Data Cleaning

Before analysis, two cleaning steps were executed using SQL:

### 1. Delete rows with zero price
```sql
DELETE FROM zepto
WHERE mrp = 0;
```

### 2. Convert price from paise to rupees
```sql
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

---

## Key SQL Queries and Analysis

### 1. High-Value Out-of-Stock Products
Find expensive products (MRP > 300) that are currently out of stock:
```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```

### 2. Estimated Revenue per Category
Calculate total inventory value currently held in stock for each category:
```sql
SELECT 
    category,
    SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue DESC;
```

### 3. Top 5 Categories with Highest Average Discount
Identify categories offering the highest average discounts:
```sql
SELECT 
    category,
    ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

### 4. Value for Money (Price per Gram)
Calculate price per gram for items weighing 100 grams or more:
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

### 5. Delivery Weight Segmentation
Group products into weight buckets to help plan rider bag load:
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

## Key Findings

- **Revenue Loss from Stockouts**: Several high-priced products (above 300) were out of stock, resulting in missed sales.
- **Discount Patterns**: Snacks and Beverages had higher average discounts compared to fresh produce and dairy.
- **Delivery Planning**: Heavy items (over 5 kg) need special handling or multiple delivery trips to stay within rider weight limits.

---

## How to Run This Project

1. Clone this repository:
   ```bash
   git clone https://github.com/adarsh062/quick-commerce-inventory-analytics.git
   cd quick-commerce-inventory-analytics
   ```

2. Open PostgreSQL (pgAdmin or command line).
3. Run the table creation and cleaning queries from `Zepto_SQL_data_analysis.sql`.
4. Import `zepto_v2.csv` into the `zepto` table.
5. Run the analysis queries.

---

## License
MIT License
