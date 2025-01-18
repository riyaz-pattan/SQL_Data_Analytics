# 📊 Retail Sales Analysis

This project demonstrates an end-to-end data analysis workflow for retail sales data. The dataset was cleaned, transformed, and analyzed using Python and SQL. The insights focus on product performance, regional trends, and time-based sales growth. 

---

## 🚀 **Key Features**

### **1. Data Cleaning and Transformation**
- **Python libraries**: Used `pandas` for preprocessing.
- Handled missing values (`NaN`) with domain-specific replacements.
- Renamed columns for consistency: Converted to lowercase and replaced spaces with underscores.
- Derived new metrics:
  - **Profit**: `sale_price - cost_price`
  - **Discount**: Computed from `list_price` and `discount_percent`.

### **2. SQL-Based Analysis**
- **Insights Generated**:
  1. **Top 10 Revenue-Generating Products**: Products that contributed the most to overall sales.
  2. **Top 5 Selling Products by Region**: Regional-level product insights.
  3. **Month-over-Month Sales Growth**: Sales comparison between 2022 and 2023.
  4. **Category with Highest Monthly Sales**: Identified peak sales months per category.
  5. **Subcategory Growth**: Subcategories with the largest profit increase in 2023 over 2022.

### **3. Data Storage**
- **Database**: Data loaded into an SQL Server for querying and analysis.


---
🛠️ Technologies Used
Python: Data preprocessing, cleaning, and deriving insights.
SQL: Advanced querying for complex business problems.


📊 SQL Insights
1.Top Revenue-Generating Products
SQL query:
```
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;
```
2.Highest Selling Products by Region
SQL query:
```
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;
```

3.Month-Over-Month Sales Growth
SQL query:
```
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
       SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
       SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;
```
4.Peak Sales Month by Category
SQL query:
```
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn = 1;
```
5.Highest Growth Subcategory (Profit)
SQL query:
```
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
),
cte2 AS (
    SELECT sub_category,
           SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
           SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *, (sales_2023 - sales_2022) AS growth
FROM cte2
ORDER BY growth DESC;
```
