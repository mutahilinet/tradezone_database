# tradezone_database
SQL-based data analysis and cleaning project for TradeZone e-commerce platform (2023-2024), focusing on seller efficiency, customer conversion, and revenue growth
# TradeZone E-Commerce Data Analysis (2023-2024)

## Project Overview
This project involves cleaning and analyzing a database for **TradeZone**, a Nigerian e-commerce platform. The goal was to identify operational bottlenecks, seller efficiency, and revenue trends to assist the Head of Growth and Head of Seller Operations in 2025 planning.

## Tools Used
- **Database:** MySQL
- **Interface:** MySQL Workbench
- **Language:** SQL

## Part A: Data Cleaning & Validation
In this phase, I handled several data quality issues:
- **Standardization:** Normalized city and state names (e.g., merging "Lagos " and "LAGOS").
- **Integrity Checks:** Identified orders where the reported total did not match the sum of line items.
- **Formatting:** Converted product categories to Title Case and standardized date formats.

## Part B: Key Business Insights
### 1. Customer Acquisition & Conversion
Analyzed 2024 sign-ups by state. 
- **Finding:** [Insert your Top State here] had the highest sign-up volume.
- **Conversion:** Calculated the percentage of users making a purchase within 30 days.

### 2. Seller Fulfillment Efficiency
Identified fulfillment times by calculating the difference between order and delivery dates.
- **Metric:** Focused on average hours per seller.
- **Note:** Discovered that no sellers currently meet a 20-order threshold, suggesting a need for merchant scaling.

### 3. Revenue Trends
Evaluated quarterly performance across 2023 and 2024 to identify seasonal growth patterns and Average Order Value (AOV).
