# Week 2: Pizza Runner 🍕

## 1. Business Context
Pizza Runner is a pizza delivery business that combines a pizza restaurant with an on-demand delivery service. This project uses SQL to analyze sales, delivery performance, ingredient usage, and profitability to support business decisions.

## 2. Objectives
- Analyze customer ordering patterns and pizza sales.
- Evaluate delivery performance and runner efficiency.
- Explore ingredient usage, exclusions, and extras.
- Estimate revenue, costs, and profitability.
- Apply SQL techniques to solve business problems.

## 3. Dataset
The dataset is provided by the [8 Week SQL Challenge – Case Study #2](https://8weeksqlchallenge.com/case-study-2/).

| Table | Description |
|---|---|
| `runners` | Runner registration information |
| `customer_orders` | Customer orders, pizza types, exclusions, and extras |
| `runner_orders` | Delivery details, distances, durations, and cancellations |
| `pizza_names` | Pizza IDs and names |
| `pizza_recipes` | Standard ingredients for each pizza |
| `pizza_toppings` | Topping IDs and ingredient names |

## 4. Tools
PostgreSQL, VS Code, and pgAdmin.

## 5. Data Exploration & Cleaning
- **Customer Orders:** Standardized `exclusions` and `extras`, handled missing values, and created `customer_orders_clean`.
- **Runner Orders:** Converted `pickup_time` to `TIMESTAMP`, removed units from `distance` and `duration`, converted them into numeric values, handled missing values, and created `runner_orders_clean` while preserving the original data.

## 6. Case Study Sections
- **A. Pizza Metrics:** Order volume, pizza preferences, customer ordering patterns, and ordering frequency.
- **B. Runner and Customer Experience:** Preparation time, delivery distance and speed, runner performance, and successful delivery rates.
- **C. Ingredient Optimisation:** Standard recipes, popular extras and exclusions, and ingredient requirements.
- **D. Pricing and Ratings:** Revenue, delivery costs, pricing scenarios, and customer ratings.

## 7. Repository Structure
```text
case-02-pizza-runner/
├── README.md
├── schema.sql
├── data_cleaning.sql
├── solution_A.sql
├── solution_B.sql
├── solution_C.sql
└── solution_D.sql
```

- `schema.sql`: Creates and populates the source tables.
- `data_cleaning.sql`: Profiles, cleans, and prepares data for analysis.
- `solution_A.sql`: Solves business questions in Pizza Metrics.
- `solution_B.sql`: Analyzes runner performance and customer experience.
- `solution_C.sql`: Explores ingredient optimization.
- `solution_D.sql`: Analyzes pricing and ratings.
- `README.md`: Documents the business context, dataset, SQL techniques, and analytical approach.
## 8. Source
Case Study #2 – Pizza Runner, created by Danny Ma as part of the [8 Week SQL Challenge](https://8weeksqlchallenge.com/case-study-2/).

This repository contains my SQL practice and analysis based on the original case study.

## 9. Business questions & Queries

### **A. Pizza Metrics:** Order volume, pizza preferences, customer ordering patterns, and ordering frequency.

```sql

-- Q1. How many pizzas were ordered?
SELECT COUNT(*) AS total_pizzas_ordered
FROM customer_orders_clean;
```
<img width="246" height="97" alt="image" src="https://github.com/user-attachments/assets/04b77906-44ff-4d82-8f20-407eb73dc518" />

```sql
-- Q2. How many unique customer orders were made?
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders_clean;
```
<img width="277" height="120" alt="image" src="https://github.com/user-attachments/assets/908672e4-fd16-4eae-be62-b7768e018488" />

```sql
-- Q3. How many successful orders were delivered by each runner?
SELECT
    runner_id,
    COUNT(DISTINCT order_id) AS successful_orders
FROM runner_orders_clean
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
```
<img width="317" height="146" alt="image" src="https://github.com/user-attachments/assets/a04e16ff-2dec-459c-a3b8-98c4900d537c" />

```sql
-- Q4. How many of each type of pizza was delivered?
SELECT
    pizza_name,
    COUNT(*) AS pizzas_delivered
FROM customer_orders_clean AS co
JOIN runner_orders_clean AS ro
    ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL
GROUP BY pizza_name
ORDER BY pizzas_delivered DESC;
```
<img width="316" height="175" alt="image" src="https://github.com/user-attachments/assets/098875a7-df78-47ae-9901-84d7cf568555" />


```sql
-- Q5. How many Vegetarian and Meatlovers were ordered by each customer?
SELECT
    co.customer_id,
    SUM(CASE WHEN pn.pizza_name = 'Meatlovers' THEN 1 ELSE 0 END)
        AS meatlovers_count,
    SUM(CASE WHEN pn.pizza_name = 'Vegetarian' THEN 1 ELSE 0 END)
        AS vegetarian_count
FROM customer_orders_clean AS co
JOIN pizza_names AS pn
    ON co.pizza_id = pn.pizza_id
GROUP BY co.customer_id
ORDER BY co.customer_id;


-- Q6. What was the maximum number of pizzas delivered in a single order?
SELECT MAX(pizza_count) AS max_pizzas_in_single_order
FROM (
    SELECT
        co.order_id,
        COUNT(*) AS pizza_count
    FROM customer_orders_clean AS co
    JOIN runner_orders_clean AS ro
        ON co.order_id = ro.order_id
    WHERE ro.cancellation IS NULL
    GROUP BY co.order_id
) AS order_summary;


-- Q7. For each customer, how many delivered pizzas had 
-- at least 1 change and how many had no changes?
SELECT
    co.customer_id,
    SUM(CASE WHEN co.exclusions IS NOT NULL
              OR co.extras IS NOT NULL
            THEN 1 ELSE 0 END) AS pizzas_with_changes,
    SUM(CASE WHEN co.exclusions IS NULL
             AND co.extras IS NULL
            THEN 1 ELSE 0 END) AS pizzas_without_changes
FROM customer_orders_clean AS co
JOIN runner_orders_clean AS ro
    ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL
GROUP BY co.customer_id
ORDER BY co.customer_id;


-- Q8. How many pizzas were delivered that had both exclusions and extras?
SELECT COUNT(*) AS pizzas_with_exclusions_and_extras
FROM customer_orders_clean
JOIN runner_orders_clean
  USING (order_id)
WHERE cancellation IS NULL
  AND exclusions IS NOT NULL
  AND extras IS NOT NULL;


-- Q9. What was the total volume of pizzas ordered for each hour of the day?
-- Requires an order_time column in customer_orders_clean.
SELECT
    EXTRACT(HOUR FROM order_time)::INTEGER AS order_hour,
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean
GROUP BY order_hour
ORDER BY order_hour;


-- Q10. What was the volume of orders for each day of the week?
-- Requires an order_time column in customer_orders_clean.
SELECT
    TO_CHAR(order_time, 'FMDay') AS day_of_week,
    COUNT(DISTINCT order_id) AS total_orders
FROM customer_orders_clean
GROUP BY EXTRACT(ISODOW FROM order_time),
         TO_CHAR(order_time, 'FMDay')
ORDER BY EXTRACT(ISODOW FROM order_time);
    
```
<img width="399" height="157" alt="image" src="https://github.com/user-attachments/assets/38149830-7da4-4de8-b451-f2219a993359" />

### **B. Runner and Customer Experience:** Preparation time, delivery distance and speed, runner performance, and successful delivery rates.
### **C. Ingredient Optimisation:** Standard recipes, popular extras and exclusions, and ingredient requirements.
### **D. Pricing and Ratings:** Revenue, delivery costs, pricing scenarios, and customer ratings.
