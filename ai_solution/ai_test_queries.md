# NL → SQL → NL Test Suite
**Role:** Senior QA Engineer  
**System under test:** Delivery Analytics AI (NL→SQL→NL pipeline on `delivery_cata`)  
**Coverage:** Low complexity → High complexity, all tables, edge cases, ambiguity traps

---

## LEVEL 1 — Simple Lookups (single table, no joins)
*Expected: model picks `gold_ai.master_delivery_fact` or a single dim table*

| # | Question | What it tests | Expected answer shape |
|---|----------|---------------|-----------------------|
| L1-01 | How many deliveries are in the system? | Basic COUNT on fact | Single number |
| L1-02 | How many drivers are currently active? | Filter on dim_driver status | Single number |
| L1-03 | What are the valid vehicle types available? | DISTINCT on dim_vehicle | List: BIKE, VAN, TRUCK, CAR, SCOOTER |
| L1-04 | What is the total number of orders placed? | COUNT on fact | Single number |
| L1-05 | How many payments were made using UPI? | Filter payment_method | Single number |
| L1-06 | What is the delivery charge in pincode 302001? | Filter dim_area by pincode | Numeric value |
| L1-07 | How many orders were cancelled? | Filter order_status = CANCELLED | Single number |
| L1-08 | How many deliveries failed? | Filter delivery_status = FAILED | Single number |
| L1-09 | What is the maximum order amount? | MAX(order_amount) | Single number |
| L1-10 | How many vehicles are of type BIKE? | Filter vehicle_type | Single number |

---

## LEVEL 2 — Filtered Aggregations (single table + WHERE)
*Expected: model uses correct date syntax, pincode quoting, LIKE for names*

| # | Question | What it tests | Expected answer shape |
|---|----------|---------------|-----------------------|
| L2-01 | How many orders were delivered in December 2023? | Date filter by month+year | Single number |
| L2-02 | What is the total revenue from CASH payments? | SUM with method filter | Currency value |
| L2-03 | How many orders were assigned to driver Rohit? | LIKE name match | Single number |
| L2-04 | How many orders were placed in pincode 110001? | Pincode as STRING filter | Single number |
| L2-05 | What is the average order amount? | AVG aggregation | Numeric value |
| L2-06 | How many deliveries happened in 2024? | Year filter | Single number |
| L2-07 | What is the total UPI revenue in pincode 302001? | Multi-filter SUM | Currency value |
| L2-08 | How many orders were placed this year? | YEAR(CURRENT_DATE()) | Single number |
| L2-09 | What is the average distance covered per delivery? | AVG(distance_km) | Numeric value |
| L2-10 | How many customers are registered in Mumbai? | Filter customer_address | Single number |

---

## LEVEL 3 — Multi-table Joins (fact + 1-2 dims)
*Expected: model uses correct surrogate key joins with is_current = true*

| # | Question | What it tests | Expected answer shape |
|---|----------|---------------|-----------------------|
| L3-01 | Which driver delivered the most orders? | JOIN fact + dim_driver, GROUP BY, ORDER BY | Driver name + count |
| L3-02 | Which city has the highest number of delivered orders? | JOIN fact + dim_area, filter DELIVERED | City name + count |
| L3-03 | What is the total revenue by payment method? | JOIN fact + dim_payment, GROUP BY method | Table: method, total |
| L3-04 | Which vehicle type covers the most distance on average? | JOIN fact + dim_vehicle, AVG(distance_km) | Vehicle type + avg |
| L3-05 | Who are the top 5 customers by total spending? | JOIN fact + dim_customer, SUM(order_amount) | List of 5 names + amounts |
| L3-06 | How many orders were delivered by BIKE in 2024? | JOIN fact + dim_vehicle + dim_date | Single number |
| L3-07 | What is the average delivery time in minutes for each vehicle type? | JOIN fact + dim_vehicle, AVG(delivery_time_mins) | Table: type, avg_mins |
| L3-08 | Which area has the highest delivery charge? | dim_area ORDER BY delivery_charge DESC | Area name + charge |
| L3-09 | How many failed deliveries were there in Gujarat? | JOIN fact + dim_area, filter city + status | Single number |
| L3-10 | What is the total revenue from completed CARD payments in 2023? | JOIN fact + dim_payment + dim_date | Currency value |

---

## LEVEL 4 — Complex Aggregations & Rankings
*Expected: model uses window functions, subqueries, or multi-level GROUP BY correctly*

| # | Question | What it tests | Expected answer shape |
|---|----------|---------------|-----------------------|
| L4-01 | What is the month-wise count of delivered orders in 2024? | GROUP BY year+month, ORDER BY month | 12-row table |
| L4-02 | Which driver had the highest average delivery distance? | JOIN + AVG + ORDER BY | Driver name + avg km |
| L4-03 | What percentage of total payments were made via UPI? | Subquery or CTE for total, then ratio | Percentage value |
| L4-04 | Which pincode has the most failed deliveries? | JOIN fact + dim_area, filter FAILED, GROUP BY pincode | Pincode + count |
| L4-05 | What is the total revenue per city, sorted highest to lowest? | JOIN fact + dim_area, GROUP BY city, ORDER BY | Table: city, revenue |
| L4-06 | How many orders were placed on weekends in 2024? | JOIN dim_date, filter is_weekend = true | Single number |
| L4-07 | Which driver completed the most deliveries in December 2023? | JOIN fact + dim_driver + dim_date, filter month+year | Driver name + count |
| L4-08 | What is the average order amount per order status? | GROUP BY order_status | Table: status, avg_amount |
| L4-09 | How many unique customers placed orders in each city? | JOIN fact + dim_customer + dim_area, COUNT DISTINCT | Table: city, unique_customers |
| L4-10 | What is the total delivery charge collected in Maharashtra? | JOIN fact + dim_area, SUM(delivery_charge) | Currency value |

---

## LEVEL 5 — Edge Cases & Ambiguity Traps
*Expected: model handles ambiguity gracefully, doesn't hallucinate columns, returns "no data" correctly*

| # | Question | What it tests | Expected behavior |
|---|----------|---------------|-------------------|
| L5-01 | How many orders were placed by Rohit? | Ambiguous — driver Rohit or customer Rohit? | Should ask for clarification OR query both |
| L5-02 | Show me all orders from last month | Relative date — ADD_MONTHS(CURRENT_DATE(), -1) | Correct dynamic date, not hardcoded |
| L5-03 | How many deliveries were done in pincode 999999? | Non-existent pincode | "No records found" — not an error |
| L5-04 | What is the revenue from BITCOIN payments? | Invalid payment method not in schema | "No records found" or graceful answer |
| L5-05 | How many orders were placed by driver Amit in pincode 380001 in January 2024? | Multi-filter: name LIKE + pincode + date | Single number, uses master_delivery_fact |
| L5-06 | Which drivers are currently on a trip? | status = ON_TRIP — tests if silver fix is reflected | List of driver names |
| L5-07 | What is the total revenue from orders where payment failed? | Cross-status join: payment_status = FAILED | Currency value |
| L5-08 | How many orders have no payment record? | LEFT JOIN or anti-join pattern | Single number |
| L5-09 | What is the busiest day of the week for deliveries? | GROUP BY day_name, ORDER BY count DESC | Day name + count |
| L5-10 | Compare total orders in Q1 2024 vs Q1 2023 | Two-period comparison, uses dim_date.quarter | Two numbers side by side |

---

## LEVEL 6 — Business Intelligence Questions (multi-hop reasoning)
*Expected: model correctly chains 3+ tables, uses correct aggregation logic*

| # | Question | What it tests | Expected answer shape |
|---|----------|---------------|-----------------------|
| L6-01 | Which driver has the best delivery success rate (delivered vs total)? | Ratio: COUNT(DELIVERED)/COUNT(*) per driver | Driver name + % |
| L6-02 | What is the average payment amount for orders delivered by TRUCK? | 3-table join: fact + dim_vehicle + dim_payment | Currency value |
| L6-03 | Which city has the highest average order amount for delivered orders only? | fact + dim_area, filter DELIVERED, AVG | City + avg amount |
| L6-04 | How many drivers have delivered more than 5 orders? | GROUP BY driver, HAVING COUNT > 5 | Single number (count of such drivers) |
| L6-05 | What is the total revenue lost from failed or refunded payments? | SUM where payment_status IN (FAILED, REFUNDED) | Currency value |
| L6-06 | Which vehicle type has the lowest failure rate in deliveries? | fact + dim_vehicle, ratio of FAILED/total | Vehicle type + % |
| L6-07 | What is the trend of total orders month over month in 2024? | GROUP BY year, month — 12 rows | Table: month, count, growth |
| L6-08 | Which top 3 pincodes generate the most delivery revenue? | fact + dim_area, SUM(delivery_charge), TOP 3 | 3 pincodes + amounts |
| L6-09 | How many customers placed more than one order? | GROUP BY customer, HAVING COUNT > 1 | Single number |
| L6-10 | What is the average time between order placement and delivery completion, in hours? | TIMESTAMPDIFF or (delivery_time - order_date) | Numeric value in hours |

---

## Quick Smoke Test (run these first — 5 questions)

```
1. How many deliveries are in the system?
2. Which driver delivered the most orders?
3. What is the total revenue from UPI payments?
4. How many deliveries failed this year?
5. How many orders were placed in pincode 302001?
```

---

## Known Gotchas to Watch For

| Gotcha | What to check |
|--------|---------------|
| Pincode as INT | Model must quote it: `pincode = '302001'` not `pincode = 302001` |
| Driver name LIKE | Must use `LOWER(driver_name) LIKE LOWER('%Rohit%')` — not exact match |
| is_current filter | All dim joins must include `AND dr.is_current = true` |
| SUCCESS vs COMPLETED | After silver fix, raw SUCCESS is now COMPLETED in silver/gold |
| ON_TRIP drivers | After silver fix, these are now in silver — model should return them |
| date_sk format | `dim_date.date_sk` is INT in YYYYMMDD format — don't filter on it directly, use `full_date` or `year`/`month` |
| master_delivery_fact row count | Only ~2,007 rows — complex aggregations need `fact_delivery` + dims |
| Empty status rows | Were quarantined — model should not expect PENDING in deliveries |
