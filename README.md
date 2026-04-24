
---

# TradeZone E-Commerce Data Analysis (2023-2024)

## Project Overview
This project involves cleaning and analyzing a database for **TradeZone**, a Nigerian e-commerce platform. The goal was to identify operational bottlenecks, seller efficiency, and revenue trends to assist the Head of Growth and Head of Seller Operations in 2025 planning.

## Tools Used
* **Database:** MySQL
* **Interface:** MySQL Workbench
* **Language:** SQL

---

## Part A: Data Cleaning & Preparation

### 1. Cleaning and Formatting
I disabled safe updates to allow for global changes, removed incomplete records, and standardized city/state naming conventions to ensure data consistency across the platform. This ensures variations like "Lagos " and "LAGOS" are merged into a single entity.

```sql
SET SQL_SAFE_UPDATES = 0;

DELETE FROM orders WHERE order_id IS NULL OR customer_id IS NULL;
DELETE FROM order_items WHERE order_id IS NULL OR product_id IS NULL;

UPDATE customers SET 
    city = TRIM(CONCAT(UPPER(LEFT(city, 1)), LOWER(SUBSTRING(city, 2)))),
    state = UPPER(TRIM(state));

UPDATE sellers SET 
    city = TRIM(CONCAT(UPPER(LEFT(city, 1)), LOWER(SUBSTRING(city, 2)))),
    state = UPPER(TRIM(state));
```

### 2. Financial Integrity Check
I verified that the `total_amount` recorded in the orders table matches the actual sum of individual line items in the `order_items` table. Differences greater than ₦10 were flagged as discrepancies.

```sql
SELECT 
    o.order_id, 
    o.total_amount AS reported_total, 
    SUM(oi.unit_price * oi.quantity) AS calculated_total,
    ABS(o.total_amount - SUM(oi.unit_price * oi.quantity)) AS discrepancy
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.total_amount
HAVING discrepancy > 10;
```

**Results:**
<img width="1917" height="989" alt="image" src="https://github.com/user-attachments/assets/b749b9b1-2a97-4cd9-b57c-c31a9f75ae75" />


---

## Part B: Key Business Insights

### 1. Customer Acquisition & Conversion
I analyzed 2024 sign-ups by state to calculate the 30-day conversion rate. This determines what percentage of new customers made a purchase within their first 30 days of joining the platform, helping measure onboarding effectiveness.

```sql
SELECT 
    state,
    COUNT(customer_id) AS total_new_customers,
    SUM(CASE WHEN DATEDIFF(first_purchase, signup_date) <= 30 THEN 1 ELSE 0 END) AS converted_customers,
    ROUND((SUM(CASE WHEN DATEDIFF(first_purchase, signup_date) <= 30 THEN 1 ELSE 0 END) / COUNT(customer_id)) * 100, 2) AS conversion_rate_pct
FROM (
    SELECT c.customer_id, c.state, c.signup_date, MIN(o.order_date) AS first_purchase
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    WHERE YEAR(c.signup_date) = 2024
    GROUP BY c.customer_id, c.state, c.signup_date
) AS signup_summary
GROUP BY state
ORDER BY total_new_customers DESC LIMIT 5;
```
<img width="1912" height="936" alt="image" src="https://github.com/user-attachments/assets/88921123-42d3-4346-8f04-fd03606c15c8" />


### 2. Product Performance
I identified the top 10 products by revenue in 2024. This ranking helps the platform understand which high-value categories, specifically Electronics and Sports equipment, are the primary revenue drivers.

```sql
SELECT 
    p.product_name, 
    p.category, 
    SUM(oi.line_total) AS total_revenue,
    COUNT(oi.order_id) AS total_orders
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE YEAR(o.order_date) = 2024
GROUP BY p.product_id, p.product_name, p.category
ORDER BY total_revenue DESC LIMIT 10;
```

**Results:**
![Product Performance](image_a56edb.png)

### 3. Seller Fulfillment Efficiency
I calculated the average time in hours between order placement and delivery. This identifies our most efficient sellers. Note that currently, no sellers have met the 20-order threshold, indicating a need for merchant scaling.

```sql
SELECT 
    s.seller_name,
    COUNT(o.order_id) AS completed_orders,
    ROUND(AVG(TIMESTAMPDIFF(HOUR, o.order_date, o.delivery_date)), 1) AS avg_delivery_hours,
    ROUND(AVG(r.rating), 2) AS avg_seller_rating
FROM sellers s
JOIN orders o ON s.seller_id = o.seller_id
LEFT JOIN reviews r ON o.order_id = r.order_id
WHERE o.order_status = 'Delivered' 
  AND o.delivery_date IS NOT NULL
GROUP BY s.seller_id, s.seller_name
HAVING completed_orders >= 1 
ORDER BY avg_delivery_hours ASC LIMIT 20;
```

**Results:**
<img width="1917" height="969" alt="image" src="https://github.com/user-attachments/assets/33b1994e-22d1-4cda-8ebb-7cbd436019aa" />


