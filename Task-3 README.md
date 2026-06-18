# 🗃️ E-Commerce Sales Analysis using SQL

![SQL](https://img.shields.io/badge/Language-SQL-blue?style=for-the-badge&logo=postgresql)
![Database](https://img.shields.io/badge/Database-PostgreSQL-336791?style=for-the-badge&logo=postgresql)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-Beginner--Intermediate-orange?style=for-the-badge)

---

## 📖 Project Overview

**Project Title:** E-Commerce Sales Analysis
**Database Name:** `ecommerce_analysis`
**Table Name:** `data_order`
**Tool Used:** PostgreSQL

This project uses pure SQL to extract business intelligence from a raw e-commerce orders dataset. Rather than relying on a BI tool, every metric in this project — revenue, customer ranking, channel performance, coupon effectiveness, and monthly growth — is computed directly through SQL queries.

**Why SQL was used:** SQL is the closest language to how relational business data actually lives — in rows and tables inside a database. Writing the analysis directly in SQL demonstrates the ability to filter, aggregate, rank, and segment data using nothing but structured queries, which is the core skill expected of a Data Analyst working with company databases.

**Business value:** This project answers some of the most common questions a business stakeholder asks — which products make the most money, which marketing channel is worth more budget, which customers are most valuable, and how much revenue is being lost to cancellations and returns. The result is a set of SQL queries that double as a reusable reporting toolkit.

---

## 🎯 Objectives

- Design a relational table structure for e-commerce order data
- Perform SQL-based data quality checks (nulls, duplicates, unique IDs)
- Validate computed fields and numeric ranges using SQL
- Answer 17 distinct business questions using SQL aggregation, window functions, and CTEs
- Rank and segment customers based on spending behavior
- Measure revenue lost to cancelled and returned orders
- Identify the most effective marketing channels and coupon strategies

---

## 📂 Dataset Information

| Detail | Value |
|---|---|
| Number of Rows | 1,200 |
| Number of Columns | 14 |
| Primary Key | `orderid` |
| Foreign Key | None (single-table dataset) |
| Dataset Size | ~132 KB (CSV) |
| File Format | CSV |

**Features & Data Types:**

| Column | Data Type | Description |
|---|---|---|
| `orderid` | VARCHAR(20) | Unique identifier for each order |
| `date` | DATE | Date the order was placed |
| `customerid` | VARCHAR(20) | Unique identifier for the customer |
| `product` | VARCHAR(100) | Product purchased |
| `quantity` | INT | Units purchased |
| `unitprice` | NUMERIC(10,2) | Price per unit |
| `shippingaddress` | TEXT | Delivery address |
| `paymentmethod` | VARCHAR(50) | Payment mode used |
| `orderstatus` | VARCHAR(50) | Status of the order |
| `trackingnumber` | VARCHAR(30) | Shipment tracking code |
| `itemsincart` | INT | Number of items in cart at checkout |
| `couponcode` | VARCHAR(30) | Coupon applied to the order (NULL if none) |
| `referralsource` | VARCHAR(50) | Channel through which the customer arrived |
| `totalprice` | NUMERIC(10,2) | Final order value |

---

## 🗄️ Database Information

| Detail | Value |
|---|---|
| Database Name | `ecommerce_analysis` |
| Table Name | `data_order` |
| SQL Environment | PostgreSQL (via pgAdmin) |
| Schema Type | Single flat table (no joins/foreign keys) |

The schema was kept intentionally flat — all order-level attributes live in one table — since the source data is a single transactional export rather than a normalized multi-table system.

---

## 🛠️ Tools & Technologies

- PostgreSQL
- pgAdmin
- SQL
- Excel (for source dataset)
- GitHub

---

## 🧩 SQL Concepts Used

| Concept | Used In |
|---|---|
| `SELECT`, `WHERE` | Data cleaning checks, filtered queries (Q3, Q4) |
| `GROUP BY`, `ORDER BY` | Almost every aggregation query |
| `HAVING` | Duplicate `orderid` detection |
| `CASE WHEN` | Coupon status labeling (Q11), customer segmentation (Q17) |
| `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` | Aggregation across all business queries |
| `DISTINCT` | Unique customer and order counts |
| `LIMIT` | Top-N queries (best month, top 20 customers, top customers by frequency) |
| Subqueries | Revenue percentage calculations (Q2, Q6) |
| CTEs (`WITH`) | Top-spending customers (Q15), Month-over-Month growth (Q17) |
| Window Functions (`RANK()`, `LAG()`) | Customer ranking (Q10, Q15), MoM growth comparison (Q17) |

---

## 🔄 Project Workflow

```
Raw Dataset
   ↓
Database Creation
   ↓
Table Creation
   ↓
Import Dataset
   ↓
SQL Data Cleaning Checks
   ↓
Business Questions
   ↓
Insights
   ↓
Recommendations
```

---

## 🏗️ Database Creation

```sql
CREATE DATABASE ecommerce_analysis;

CREATE TABLE data_order (
    orderid VARCHAR(20),
    date DATE,
    customerid VARCHAR(20),
    product VARCHAR(100),
    quantity INT,
    unitprice NUMERIC(10,2),
    shippingaddress TEXT,
    paymentmethod VARCHAR(50),
    orderstatus VARCHAR(50),
    trackingnumber VARCHAR(30),
    itemsincart INT,
    couponcode VARCHAR(30),
    referralsource VARCHAR(50),
    totalprice NUMERIC(10,2)
);
```

A dedicated database, `ecommerce_analysis`, was created to isolate this project's data. Inside it, a single table, `data_order`, was defined with explicit data types for every column — `VARCHAR` for identifiers and categorical text, `NUMERIC(10,2)` for currency fields, `INT` for whole-number fields, and `DATE` for the order date. The raw CSV dataset was then imported into this table for analysis.

---

## 🧹 Data Cleaning Checks (Pre-Analysis Validation)

Before answering any business question, the dataset was validated directly in SQL:

| Check | Query Logic | Result |
|---|---|---|
| Total record count | `SELECT COUNT(*) FROM data_order` | 1,200 records |
| Null check on key fields | `WHERE orderid IS NULL OR customerid IS NULL OR quantity IS NULL OR unitprice IS NULL OR totalprice IS NULL` | 0 null values found |
| Duplicate `orderid` check | `GROUP BY orderid HAVING COUNT(*) > 1` | 0 duplicate order IDs |
| Unique order count | `SELECT COUNT(DISTINCT orderid)` | 1,200 unique orders |
| Date range check | `SELECT MIN(date), MAX(date)` | 2023-01-01 to 2025-06-30 |
| TotalPrice formula check | `WHERE totalprice <> quantity * unitprice` | 0 mismatches — all values consistent |
| ItemsInCart range check | `MIN()`, `MAX()`, `AVG(itemsincart)` | Range 1–10, average 5.49 |
| Coupon usage check | `GROUP BY couponcode` | 309 orders with no coupon (NULL), 3 active coupon types |

This confirmed the dataset was already structurally sound — no duplicate orders, no broken `TotalPrice` calculations, and the only data quality consideration was the 309 orders with no coupon applied, which were handled explicitly as `"Without Coupon"` in the business analysis rather than treated as missing data.

---

## 📊 SQL Analysis

### 💰 Revenue & Business Health

#### Query 1 — What is the overall health of the business?

<img src="images/query01.png" width="900">
<img src="images/output01.png" width="900">

**Business Question:** What do the core business metrics look like at a glance — total orders, customers, revenue, and average spend?

**SQL Objective:** Calculate total orders, unique customers, total revenue, average order value, and average basket size in a single summary query.

**SQL Logic Explanation:** Uses `COUNT(orderid)` for total orders, `COUNT(DISTINCT customerid)` for unique customers, and `SUM()`/`AVG()` on `totalprice` and `itemsincart`, all wrapped in `ROUND()` for clean reporting figures.

**Output Interpretation:** The business generated 1,200 total orders from 1,189 unique customers, totaling ₹1,264,761.96 in revenue. The average order value is ₹1,053.97, and the average basket size is 5.49 items.

**Business Insight:** With more unique customers (1,189) than orders (1,200), repeat purchases are extremely rare — most customers are placing a single order.

**Recommendation:** Invest in customer retention strategies (loyalty programs, post-purchase email campaigns) to convert first-time buyers into repeat customers, since the current business is heavily reliant on new customer acquisition.

---

#### Query 3 — Revenue Lost due to Cancelled Orders

<img src="images/query03.png" width="900">
<img src="images/output03.png" width="900">

**Business Question:** How much revenue is being lost specifically due to cancelled orders?

**SQL Objective:** Filter orders where `orderstatus = 'Cancelled'` and calculate the count and total revenue associated with them.

**SQL Logic Explanation:** A simple `WHERE` filter isolates cancelled orders, then `COUNT(*)` and `SUM(totalprice)` quantify the impact.

**Output Interpretation:** 250 orders were cancelled, accounting for ₹276,396.21 in lost revenue.

**Business Insight:** Cancellations alone account for a meaningful share of total potential revenue — larger than any single product category's contribution.

**Recommendation:** Analyze the cancellation reasons (if captured elsewhere) and review checkout/payment flow friction points that may be driving cancellations before order fulfillment.

---

#### Query 4 — Revenue Lost due to Returned Orders

<img src="images/query04.png" width="900">
<img src="images/output04.png" width="900">

**Business Question:** How much revenue is being lost due to returned orders?

**SQL Objective:** Filter orders where `orderstatus = 'Returned'` and calculate the count and total revenue.

**SQL Logic Explanation:** Identical structure to Query 3, filtered on `'Returned'` status instead.

**Output Interpretation:** 247 orders were returned, representing ₹243,277.70 in revenue.

**Business Insight:** Combined with cancellations, returns represent a second major source of revenue leakage, closely matching cancellations in both order count and value.

**Recommendation:** Investigate product quality, sizing/description accuracy, and delivery condition as potential return drivers, since the return rate is nearly as high as the cancellation rate.

---

#### Query 5 — What is the total Revenue Loss Percentage?

<img src="images/query05.png" width="900">
<img src="images/output05.png" width="900">

**Business Question:** What percentage of total revenue is lost to cancellations and returns combined?

**SQL Objective:** Calculate the combined share of revenue from cancelled and returned orders as a percentage of total revenue.

**SQL Logic Explanation:** A `CASE WHEN` statement inside `SUM()` adds `totalprice` only for orders with `orderstatus IN ('Cancelled','Returned')`, then divides by total `SUM(totalprice)` and multiplies by 100.

**Output Interpretation:** 41.09% of total revenue is associated with cancelled or returned orders.

**Business Insight:** Over two-fifths of the business's gross order value never converts into completed, retained revenue — this is the single most critical business health metric uncovered in this analysis.

**Recommendation:** Treat reducing the combined cancellation and return rate as the top business priority, since even a modest improvement here would have a larger revenue impact than any marketing or pricing change.

---

### 📦 Product Analysis

#### Query 2 — What percentage of total revenue is generated by each product?

<img src="images/query02.png" width="900">
<img src="images/output02.png" width="900">

**Business Question:** Which products contribute the most to total revenue, proportionally?

**SQL Objective:** Calculate each product's share of total revenue as a percentage.

**SQL Logic Explanation:** A correlated subquery (`SELECT SUM(totalprice) FROM data_order`) provides the grand total, against which each product's `SUM(totalprice)` is divided and multiplied by 100.

**Output Interpretation:** Chair and Printer are tied as the top contributors at 15.47% each, followed by Laptop (15.19%), Tablet (14.75%), Monitor (13.89%), Desk (13.24%), and Phone (12.00%) as the lowest contributor.

**Business Insight:** Revenue is fairly evenly spread across the seven product categories, with only a 3.47-percentage-point gap between the highest and lowest performer — there is no single dominant product.

**Recommendation:** Since no product dramatically outperforms the others, marketing budget can be distributed relatively evenly, with a slightly reduced allocation to Phone given its lower contribution.

---

#### Query 6 — Which product drives maximum business / generates highest revenue?

<img src="images/query06.png" width="900">
<img src="images/output06.png" width="900">

**Business Question:** Which single product generates the most revenue overall?

**SQL Objective:** Rank all products by their percentage contribution to total revenue.

**SQL Logic Explanation:** Functionally identical to Query 2 — calculates each product's percentage share and sorts in descending order to surface the top contributor.

**Output Interpretation:** Chair and Printer are jointly the top revenue drivers at 15.47% each.

**Business Insight:** Chair and Printer should be treated as the flagship product categories for this business based on revenue contribution.

**Recommendation:** Ensure consistent inventory availability for Chair and Printer, and consider featuring them prominently on the homepage or in email campaigns.

---

#### Query 7 — Which products are cancelled most frequently?

<img src="images/query07.png" width="900">
<img src="images/output07.png" width="900">

**Business Question:** Are cancellations concentrated in specific product categories?

**SQL Objective:** Count cancelled orders grouped by product.

**SQL Logic Explanation:** Filters `orderstatus = 'Cancelled'`, groups by `product`, and counts orders per group, sorted highest first.

**Output Interpretation:** Chair has the highest number of cancellations (45), notably higher than Printer, Monitor, Laptop, and Desk, which are clustered around 35 cancellations each. Tablet (34) and Phone (31) have the fewest cancellations.

**Business Insight:** Chair's high cancellation count (45) is concerning given it is also the joint-highest revenue product — this suggests a meaningful share of Chair's apparent revenue is never actually realized once cancellations are accounted for.

**Recommendation:** Investigate Chair-specific issues such as sizing/fit concerns, delivery damage, or price sensitivity, since it has both the highest revenue contribution and the highest cancellation count.

---

#### Query 8 — High Revenue + Low Quantity = Premium Products

<img src="images/query08.png" width="900">
<img src="images/output08.png" width="900">

**Business Question:** Which products generate the highest revenue per unit sold, indicating premium positioning?

**SQL Objective:** Calculate total quantity sold, total revenue, and revenue-per-unit for each product.

**SQL Logic Explanation:** `SUM(quantity)` and `SUM(totalprice)` are calculated per product, then divided to get `Revenue_Per_Unit`, sorted in descending order.

**Output Interpretation:** Tablet has the highest revenue per unit (₹375.39), followed by Phone (₹369.15), Monitor (₹365.94), Printer (₹360.91), Laptop (₹359.12), Chair (₹348.08), and Desk (₹329.65) as the lowest.

**Business Insight:** Tablet behaves like the most "premium" product in the catalog — it generates the highest value per unit sold despite not being the top product by total revenue or order count.

**Recommendation:** Position Tablet as a premium offering in marketing materials, and explore whether similar premium pricing strategies could be applied to Desk, which currently has the lowest revenue per unit.

---

### 📣 Marketing & Channel Analysis

#### Query 9 — Which channel deserves more budget?

<img src="images/query09.png" width="900">
<img src="images/output09.png" width="900">

**Business Question:** Which customer acquisition channel performs best and deserves increased marketing investment?

**SQL Objective:** Compare order volume, total revenue, average order value, and revenue per order across all referral sources.

**SQL Logic Explanation:** Groups by `referralsource` and aggregates `COUNT`, `SUM`, and `AVG` on `totalprice`, sorted by total revenue descending.

**Output Interpretation:** Instagram leads in both order volume (259) and total revenue (₹275,285.45). However, Facebook has the highest average order value (₹1,098.29) despite bringing in the fewest orders besides Referral. Referral is the weakest channel on every metric (222 orders, ₹226,815.58 revenue, ₹1,021.69 average order).

**Business Insight:** Instagram drives the most volume, but Facebook customers tend to spend more per order — these are two different kinds of channel value (volume vs. quality), while Referral underperforms on both fronts.

**Recommendation:** Increase budget for Instagram to capitalize on its volume advantage, while also testing increased investment in Facebook given its higher average order value. Redesign the referral program, since it currently delivers the lowest performance across every metric measured.

---

### 🎟️ Coupon & Discount Analysis

#### Query 11 — Did discounts (coupons) actually improve revenue?

<img src="images/query11.png" width="900">
<img src="images/output11.png" width="900">

**Business Question:** Do orders placed with a coupon generate more revenue than orders placed without one?

**SQL Objective:** Compare order count, total revenue, and average order value between orders with and without a coupon.

**SQL Logic Explanation:** A `CASE WHEN couponcode IS NULL THEN 'Without Coupon' ELSE 'With Coupon' END` expression creates the two groups, which are then aggregated.

**Output Interpretation:** Orders with a coupon (891 orders) generated ₹942,360.55 in revenue with an average order value of ₹1,057.64. Orders without a coupon (309 orders) generated ₹322,401.41 with an average order value of ₹1,043.37.

**Business Insight:** Average order value is nearly identical with or without a coupon (a difference of only ₹14.27), meaning coupons are not meaningfully increasing how much customers spend per order — their main effect is on order volume, since 74% of orders (891 of 1,200) involve a coupon.

**Recommendation:** Since coupons do not significantly lift average order value, evaluate whether coupon discounts are being offset by increased order volume, or whether they primarily protect existing demand rather than generate incremental spend.

---

#### Most Successful Coupon

<img src="images/query11b.png" width="900">
<img src="images/output11b.png" width="900">

**Business Question:** Which specific coupon code performs best in terms of revenue?

**SQL Objective:** Rank each coupon code by total order count and revenue generated.

**SQL Logic Explanation:** Groups by `couponcode` and aggregates `COUNT(*)` and `SUM(totalprice)`, sorted by revenue descending.

**Output Interpretation:** FREESHIP is the top-performing coupon, generating ₹335,036.99 from 313 orders. Orders without any coupon generated ₹322,401.41 (309 orders), followed by SAVE10 (₹304,840.02, 286 orders) and WINTER15 (₹302,483.54, 292 orders).

**Business Insight:** Free shipping is the single most effective incentive among all coupon types, slightly outperforming even the "no coupon" segment in total revenue.

**Recommendation:** Prioritize free shipping as the primary promotional lever going forward, and consider testing it more broadly given it already leads in both usage and revenue among coupon types.

---

### 💳 Payment Method Analysis

#### Query 12 — Which payment method is the most profitable?

<img src="images/query12.png" width="900">
<img src="images/output12.png" width="900">

**Business Question:** Which payment method contributes most to revenue, average order value, and peak transaction size?

**SQL Objective:** Compare order count, total revenue, average order value, and highest single order value across payment methods.

**SQL Logic Explanation:** Groups by `paymentmethod` and aggregates `COUNT`, `SUM`, `AVG`, and `MAX` on `totalprice`, sorted by revenue descending.

**Output Interpretation:** Credit Card leads in total revenue (₹263,847.63) and average order value (₹1,127.55), despite Online having the highest single transaction (₹3,456.40) and the most orders (258). Debit Card has both the lowest total revenue (₹232,361.18) and the lowest average order value (₹1,001.56).

**Business Insight:** Credit Card customers tend to place higher-value orders on average, making this payment method the most valuable per transaction, even though it isn't the most frequently used.

**Recommendation:** Ensure a frictionless checkout experience for Credit Card payments given their outsized contribution to average order value, and investigate why Debit Card orders trend lower in value — this may reflect a different, more price-sensitive customer segment.

---

#### Which Payment Method Has the Highest Cancellation Rate?

<img src="images/query12b.png" width="900">
<img src="images/output12b.png" width="900">

**Business Question:** Is order cancellation linked to a specific payment method?

**SQL Objective:** Count cancelled orders grouped by payment method.

**SQL Logic Explanation:** Filters `orderstatus = 'Cancelled'`, groups by `paymentmethod`, and counts orders, sorted highest first.

**Output Interpretation:** Credit Card has the highest number of cancellations (54), followed by Online (53), Gift Card (50), Cash (49), and Debit Card (44) with the fewest.

**Business Insight:** Credit Card, despite being the most profitable payment method by average order value (Query 12), also has the highest absolute number of cancellations — these two findings together suggest Credit Card orders carry both the highest reward and the highest risk.

**Recommendation:** Investigate whether Credit Card cancellations are linked to fraud checks, payment authorization failures, or buyer's remorse on higher-value purchases, since this method combines the highest order value with the highest cancellation count.

---

### 👥 Customer Analysis & Segmentation

#### Query 10 — How can customers be ranked based on total spending?

<img src="images/query10.png" width="900">
<img src="images/output10.png" width="900">

**Business Question:** Who are the highest-spending customers, and how do they rank against one another?

**SQL Objective:** Rank every customer by their total spending using a window function.

**SQL Logic Explanation:** `SUM(totalprice)` per `customerid` is calculated, and `RANK() OVER (ORDER BY SUM(totalprice) DESC)` assigns a rank to each customer based on total spend.

**Output Interpretation:** Customer C38840 ranks #1 with ₹5,723.23 in total spending — substantially higher than the #2-ranked customer, C57276, at ₹3,456.40.

**Business Insight:** There is a clear top customer (C38840) whose spending is over 65% higher than the next-highest customer, making them a uniquely high-value account worth individual attention.

**Recommendation:** Set up a dedicated VIP or account-management touchpoint for top-ranked customers like C38840, since losing a customer at this spending level would have an outsized impact on revenue.

---

#### Query 15 — Top 20 Customers by Revenue

<img src="images/query15a.png" width="900">
<img src="images/output15a.png" width="900">

**Business Question:** Who are the top 20 highest-spending customers, and what do they contribute?

**SQL Objective:** Use a CTE to rank all customers by total spending, then return the top 20.

**SQL Logic Explanation:** A CTE (`customer_sales`) calculates `SUM(totalprice)` and `RANK()` per customer; the outer query then filters this to the top 20 ranks using `LIMIT 20`.

**Output Interpretation:** The top 20 customers range from ₹5,723.23 (C38840, rank 1) down to ₹3,215.15 (C88205, rank 20).

**Business Insight:** All of the top 20 customers have spent at least ₹3,000, qualifying as "Gold" or "VIP" tier under the business's own spending thresholds — this group represents the most valuable, identifiable customer segment in the dataset.

**Recommendation:** Build a retention program specifically targeting these top 20 customers (e.g., early access to new products, dedicated support), since they are easily identifiable by customer ID and represent outsized account value.

---

#### Query 16 — Which customers purchase most frequently?

<img src="images/query16.png" width="900">
<img src="images/output16.png" width="900">

**Business Question:** Which customers place the most orders, regardless of total spend?

**SQL Objective:** Count the number of orders placed by each customer.

**SQL Logic Explanation:** Groups by `customerid` and counts `orderid`, sorted by order count descending.

**Output Interpretation:** The most frequent customers (such as C98474, C38840, C14847, and others) each placed only 2 orders — there is no customer in the dataset with 3 or more orders.

**Business Insight:** Repeat purchasing is essentially nonexistent in this dataset — the maximum order frequency for any customer is 2, confirming the single-purchase pattern already observed in Query 1.

**Recommendation:** Since almost no customers return for a second or third purchase, prioritize building a structured retention or loyalty mechanism (such as post-purchase email sequences or reorder discounts), as this is currently the business's biggest untapped growth lever.

---

#### Customer Segmentation — VIP, Gold, Silver, Regular

<img src="images/query17b.png" width="900">
<img src="images/output17b.png" width="900">

**Business Question:** How can customers be segmented into tiers for targeted marketing based on total spending?

**SQL Objective:** Classify every customer into a spending tier: VIP, Gold, Silver, or Regular.

**SQL Logic Explanation:** A `CASE WHEN` expression assigns `'VIP'` for total spend ≥ ₹5,000, `'Gold'` for ≥ ₹3,000, `'Silver'` for ≥ ₹1,000, and `'Regular'` for everything below that, applied to each customer's `SUM(totalprice)`.

**Output Interpretation:** Out of 1,189 unique customers, 1 customer qualifies as VIP, 33 as Gold, 478 as Silver, and 677 as Regular.

**Business Insight:** The customer base is heavily weighted toward Regular and Silver tiers (over 94% combined), while VIP-tier customers are extremely rare — only a single customer crosses the ₹5,000 threshold.

**Recommendation:** Focus retention marketing on the Silver tier (478 customers), since this group is large enough to move meaningfully and likely closest to crossing into Gold with the right incentives, while reserving white-glove treatment for the small VIP/Gold segment.

---

### 📅 Time-Based & Growth Analysis

#### Query 13 — Monthly Business Growth

<img src="images/query13.png" width="900">
<img src="images/output13.png" width="900">

**Business Question:** How has monthly revenue trended across the dataset's full time period?

**SQL Objective:** Calculate total revenue for each year-month combination.

**SQL Logic Explanation:** `EXTRACT(YEAR FROM date)` and `EXTRACT(MONTH FROM date)` break the date into year and month components, and `SUM(totalprice)` is grouped by both, ordered chronologically.

**Output Interpretation:** Monthly revenue fluctuates considerably across the dataset — for example, January 2023 generated ₹56,685.75, dropping to ₹27,751.71 in April 2023, before rising again to ₹63,836.84 in May 2023.

**Business Insight:** Revenue does not follow a smooth, predictable pattern month-to-month — there is meaningful volatility even within the same year.

**Recommendation:** Investigate the drivers behind the sharpest month-to-month swings (such as April 2023's dip) to determine whether they are tied to seasonality, marketing activity, or external factors, so future planning can account for this volatility.

---

#### Best Sales Month

<img src="images/query13b.png" width="900">
<img src="images/output13b.png" width="900">

**Business Question:** Which calendar month historically generates the most revenue?

**SQL Objective:** Identify the single month (aggregated across all years) with the highest total revenue.

**SQL Logic Explanation:** Groups by `EXTRACT(MONTH FROM date)` only (not year), sums `totalprice`, and returns the top result using `ORDER BY Revenue DESC LIMIT 1`.

**Output Interpretation:** June is the strongest month overall, with ₹170,616.13 in combined revenue across all years in the dataset.

**Business Insight:** June stands out as a recurring high-revenue month, making it a candidate for seasonal demand or a planned promotional period.

**Recommendation:** Plan inventory build-up and marketing campaigns ahead of June each year to capitalize on this consistently strong sales period.

---

#### Query 17 — Month-over-Month Growth

<img src="images/query17.png" width="900">
<img src="images/output17.png" width="900">

**Business Question:** How does revenue change from one month to the next, in percentage terms?

**SQL Objective:** Calculate each month's revenue alongside the previous month's revenue and the percentage growth between them.

**SQL Logic Explanation:** A CTE (`MonthlySales`) aggregates revenue by `DATE_TRUNC('month', date)`. The outer query then uses `LAG(revenue) OVER (ORDER BY month)` to pull the prior month's revenue, and calculates percentage growth as `((revenue - previous) / previous) * 100`.

**Output Interpretation:** Growth swings are large and frequent — for example, February 2023 declined -29.23% from January, March 2023 grew +21.17%, April 2023 dropped sharply by -42.91%, and May 2023 rebounded by +130.03%.

**Business Insight:** Month-over-month growth is highly volatile rather than steadily trending in one direction, with swings exceeding 100% in either direction in some months.

**Recommendation:** Given this volatility, avoid relying on any single month's growth figure for forecasting — instead use a rolling 3-month or quarterly average to get a more stable read on the underlying business trend.

---

### 🧮 Order Status Analysis

#### Query 14 — Order Status Distribution

<img src="images/query14.png" width="900">
<img src="images/output14.png" width="900">

**Business Question:** How are orders distributed across each status, and what share does each represent?

**SQL Objective:** Calculate the count and percentage of orders for each `orderstatus` value.

**SQL Logic Explanation:** Groups by `orderstatus`, counts orders per group, and divides by the total order count (via a subquery) multiplied by 100 to get the percentage share.

**Output Interpretation:** Cancelled orders represent the largest share at 20.83% (250 orders), followed closely by Returned at 20.58% (247 orders), Pending at 19.75% (237 orders), Shipped at 19.58% (235 orders), and Delivered at the lowest share, 19.25% (231 orders).

**Business Insight:** Delivered — the only status representing a fully successful transaction — is actually the smallest category, while Cancelled and Returned combined make up over 41% of all orders, matching the revenue loss percentage found in Query 5.

**Recommendation:** Treat order fulfillment reliability as a core business priority, since "successfully completed" orders are currently a minority outcome rather than the norm.

---

## 📈 Database Schema

<img src="images/database_schema.png" width="900">

The schema consists of a single table, `data_order`, containing 14 columns that capture every attribute of an e-commerce transaction — from order and customer identifiers to product, payment, fulfillment, and revenue details. `orderid` serves as the natural primary key, validated as 100% unique across all 1,200 records.

---

## 💡 Business Insights

1. **Revenue loss from cancellations and returns is the single biggest issue**, accounting for 41.09% of total order value (Query 5).
2. **Chair and Printer are the top revenue-generating products**, each contributing 15.47% of total revenue (Query 2/6).
3. **Chair also has the highest cancellation count (45 orders)**, meaning a portion of its apparent revenue leadership is offset by lost orders (Query 7).
4. **Tablet is the most "premium" product** by revenue per unit (₹375.39), despite not leading in total revenue (Query 8).
5. **Instagram is the strongest acquisition channel by volume and revenue**, while Referral is the weakest on every metric measured (Query 9).
6. **Coupons do not meaningfully increase average order value** — the difference between "With Coupon" and "Without Coupon" average order value is only ₹14.27 (Query 11).
7. **FREESHIP is the best-performing coupon**, generating the highest revenue (₹335,036.99) among all coupon types.
8. **Credit Card has both the highest average order value and the highest cancellation count**, making it the highest-reward, highest-risk payment method (Query 12).
9. **Customer C38840 is a uniquely high-value customer**, spending 65% more than the next-highest customer (Query 10).
10. **Repeat purchasing is nearly nonexistent** — no customer in the dataset placed more than 2 orders (Query 16).
11. **The customer base is dominated by Regular and Silver tiers** (over 94% combined), with only one VIP-tier customer out of 1,189.
12. **June is the strongest historical sales month**, generating ₹170,616.13 across all years combined.
13. **Month-over-month revenue growth is highly volatile**, with swings exceeding 100% in either direction in some months.

---

## ✅ Business Recommendations

- **Prioritize reducing cancellations and returns** above all other initiatives, since they represent over 41% of total order value — larger than any single product, channel, or coupon effect found in this analysis.
- **Investigate Chair's high cancellation rate specifically**, since it is simultaneously the top revenue product and the most-cancelled product.
- **Build a structured customer retention program**, since the data shows almost no repeat purchasing — this is the single largest untapped growth opportunity.
- **Increase marketing investment in Instagram** while testing higher Facebook spend, given its strong average order value performance.
- **Redesign the referral program**, since it underperforms every other acquisition channel across volume, revenue, and average order value.
- **Continue and expand free shipping promotions**, as FREESHIP is the top-performing coupon by both usage and revenue.
- **Create a dedicated high-value customer program** for top-ranked customers (especially C38840 and the top-20 cohort), given how concentrated high spending is among a small group.
- **Plan inventory and campaigns around June**, the historically strongest sales month.
- **Use rolling averages rather than single-month comparisons** when evaluating growth, due to high month-over-month volatility.

---

## 🔑 Key Findings

- The dataset required no structural cleanup at the database level — 0 duplicate orders, 0 null values in key fields, and 0 `TotalPrice` calculation mismatches.
- Revenue is fairly evenly distributed across the seven product categories, with no single dominant product.
- Cancelled and Returned orders combined account for 41.09% of total revenue — the most significant finding in the entire analysis.
- Customer loyalty is essentially absent: maximum order frequency per customer is 2.
- Coupons drive order volume more than they drive order value.

---

## ⚠️ Challenges Faced

- The dataset contains 309 orders with a `NULL` `couponcode`, which required explicit handling using `CASE WHEN couponcode IS NULL` logic throughout the coupon-related queries (Query 11 and the most-successful-coupon query) to avoid these orders being silently excluded from `GROUP BY couponcode` results.
- Two queries in the original script were both labeled "Query-15" (the top-20-customers CTE and the payment-method-cancellation query), and two queries were both labeled "Query-17" (Month-over-Month Growth and Customer Segmentation) — these were organized under distinct topic-based sections in this README to keep the analysis clear.
- The "top 20% customers" business question (Query 15) was implemented as a flat `LIMIT 20`, which returns the top 20 customers by rank rather than the top 20% of the full 1,189-customer base — this distinction is noted for transparency.

---

## 🔮 Future Improvements

- Convert the most frequently used queries (such as the business health summary and customer segmentation) into PostgreSQL **Views** for easier reuse.
- Build a Power BI or Tableau dashboard layered on top of these SQL queries for stakeholder-facing reporting.
- Extend the analysis with a proper RFM (Recency, Frequency, Monetary) model once more historical order data becomes available.
- Add a `JOIN`-based extension if a separate customer or product dimension table becomes available in the future.
- Automate the cancellation/return investigation with a dedicated query that cross-tabs status against product, payment method, and channel simultaneously.

---

## 📁 Project Folder Structure

```
Project
│
├── Dataset
│   └── Task-3_Dataset.csv
│
├── SQL Script
│   └── TASK-3_SQL_Queries.sql
│
├── Images
│   ├── database_schema.png
│   ├── query01.png / output01.png
│   ├── query02.png / output02.png
│   └── ... (one pair per query)
│
└── README.md
```

---

## ▶️ How to Run

1. **Install PostgreSQL** on your system if not already installed.
2. **Open pgAdmin** and connect to your local PostgreSQL server.
3. **Create the database** by running:
   ```sql
   CREATE DATABASE ecommerce_analysis;
   ```
4. **Create the table** using the `CREATE TABLE data_order (...)` script provided in `TASK-3_SQL_Queries.sql`.
5. **Import the dataset** (`Task-3_Dataset.csv`) into the `data_order` table using pgAdmin's Import/Export tool.
6. **Execute the queries** in `TASK-3_SQL_Queries.sql` sequentially — starting with the data cleaning checks, followed by the 17 business question queries.

---

## 🏁 Results

This project successfully transformed a single raw e-commerce orders table into a complete set of business answers using only SQL. The analysis surfaced the business's most pressing issue (a 41.09% revenue loss rate from cancellations and returns), identified its top products, customers, and channels, and quantified the real impact of coupons and payment methods on revenue — all without leaving the database.

---

## 📝 Conclusion

This project demonstrates the ability to translate raw transactional data into structured business answers using SQL alone — from basic filtering and aggregation to window functions and CTEs. By systematically working through revenue health, product performance, customer behavior, and channel effectiveness, the analysis produced findings that are directly actionable for a real e-commerce business, particularly around order fulfillment reliability and customer retention.

---

## 👩‍💻 Author

**Princy Khasa**
Aspiring Data Analyst
SQL | Python | Excel | Power BI
