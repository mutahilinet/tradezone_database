# tradezone_database
SQL-based data analysis and cleaning project for TradeZone e-commerce platform (2023-2024), focusing on seller efficiency, customer conversion, and revenue growth.

## Project Overview
This project involves cleaning and analyzing a database for **TradeZone**, a Nigerian e-commerce platform. The goal was to identify operational bottlenecks, seller efficiency, and revenue trends to assist the Head of Growth and Head of Seller Operations in 2025 planning.

## Tools Used
- **Database:** MySQL
- **Interface:** MySQL Workbench
- **Language:** SQL

---

## Part A: Data Cleaning & Validation
In this phase, I handled several data quality issues including normalization of city/state names and ensuring financial integrity by flagging order discrepancies greater than ₦10.

### Financial Integrity Check
```sql
-- Disable Safe Updates
SET SQL_SAFE_UPDATES = 0;

/*******************************************************************************
PART A: DATA CLEANING & PREPARATION
*******************************************************************************/

-- 1. HANDLING MISSING VALUES
DELETE FROM orders WHERE order_id IS NULL OR customer_id IS NULL;
DELETE FROM order_items WHERE order_id IS NULL OR product_id IS NULL;

-- 2. INCONSISTENT FORMATTING (City and Date)
-- We do this before checking duplicates to ensure 'Lagos ' and 'Lagos' are merged
UPDATE customers SET 
    city = TRIM(CONCAT(UPPER(LEFT(city, 1)), LOWER(SUBSTRING(city, 2)))),
    state = UPPER(TRIM(state));

UPDATE sellers SET 
    city = TRIM(CONCAT(UPPER(LEFT(city, 1)), LOWER(SUBSTRING(city, 2)))),
    state = UPPER(TRIM(state));

-- 3. DATA VALIDATION: Ratings check
SELECT * FROM reviews WHERE rating < 1 OR rating > 5;
<img width="1901" height="935" alt="image" src="https://github.com/user-attachments/assets/65688995-8130-46c9-80b2-2810f434b34b" />


-- 4. FINANCIAL VALIDATION
-- Flagging discrepancies > ₦10
## Financial Integrity Check
I verified that the total_amount recorded in the orders table matches the actual sum of individual line items in the order_items table. Differences greater than ₦10 were flagged as discrepancies.
SELECT 
    o.order_id, 
    o.total_amount AS reported_total, 
    SUM(oi.unit_price * oi.quantity) AS calculated_total,
    ABS(o.total_amount - SUM(oi.unit_price * oi.quantity)) AS discrepancy
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.total_amount
HAVING discrepancy > 10;
<img width="1896" height="920" alt="image" src="https://github.com/user-attachments/assets/b0afcd79-dfe4-4482-88a5-df9ecdeeb859" />
## Part B: Key Business Insights
1. Customer Acquisition & Conversion
I analyzed 2024 sign-ups by state to calculate the 30-day conversion rate. This determines what percentage of new customers made a purchase within their first 30 days of joining the platform.
/* BUSINESS QUESTION 1: Customer Acquisition & Conversion
Analysis: Filter for 2024 sign-ups and calculate the percentage of users 
who converted to a paid order within 30 days of registration.
*/


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
<img width="1899" height="989" alt="image" src="https://github.com/user-attachments/assets/b962df99-d5f7-41a8-bd16-c2e02ebcc6dc" />

## Seller Fulfillment Efficiency
I calculated the average time (in hours) between order placement and delivery for sellers who have completed at least 1 order.

Note: The analysis revealed that no sellers currently meet a 20-order threshold, suggesting an opportunity to help merchants scale their volume.
