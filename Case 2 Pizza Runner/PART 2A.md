# DannyMa--SQL-challenge
## :pizza: Case Study #2: Pizza Runner Part A
<img src="https://camo.githubusercontent.com/ddf14996bc51bccddc9d5bdb606c27af94db9ce9f4c19a774e29dc9235c683cf/68747470733a2f2f387765656b73716c6368616c6c656e67652e636f6d2f696d616765732f636173652d73747564792d64657369676e732f322e706e67">

This repository contains SQL queries and solutions for a set of case study questions related to Danny's Diner restaurant. The questions cover various aspects of customer transactions and preferences. All the information from the case study can be found here: [here](https://8weeksqlchallenge.com/case-study-1/).

## Business Task
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. This is what you learn!!

Common table expressions
Group by aggregates
Table joins
String transformations
Dealing with null values
Regular expressions
***
## Entity Relationship Diagram

![image]( https://github.com/manaswikamila05/8-Week-SQL-Challenge/blob/main/Case%20Study%20%23%202%20-%20Pizza%20Runner/ERD.jpg)

***
## Question and Solution

Executing the queries using PostgreSQL on [DB Fiddle]( https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65).
## SQL Queries

### 1. How many pizzas were ordered?

```sql
SELECT COUNT(*) AS total_pizza_order
FROM pizza_runner.customer_orders
````
***
### 2. How many unique customer orders were made?

```sql

SELECT COUNT(DISTINCT order_id) AS unique_order_count
FROM pizza_runner.customer_orders;
````
***

### 3. How many successful orders were delivered by each runner?

```sql
WITH pizza_cte AS
(
SELECT
CASE
WHEN cancellation IS NULL OR cancellation = '' OR cancellation = 'null' THEN 'Not Cancelled'
ELSE 'Cancelled'
END AS cancellation_status, runner_id
FROM pizza_runner.runner_orders 
)

SELECT COUNT(runner_id)AS Successful_deliveries, runner_id FROM pizza_cte WHERE cancellation_status = 'Not Cancelled'
GROUP BY runner_id

 Note: You can also do distance !=0, but I have opted for this to show data-cleaning/ manipulation purposes.
````
***

### 4. How many of each type of pizza was delivered?

```sql
WITH pizza_cte_type AS
(
SELECT
CASE
WHEN cancellation IS NULL OR cancellation = '' OR cancellation = 'null' THEN 'Delivered'
ELSE 'Cancelled'
END AS status, a.pizza_id, pizza_name
FROM pizza_runner.runner_orders r JOIN pizza_runner.customer_orders c
ON r.order_id = c.order_id
JOIN pizza_runner.pizza_names a
ON c.pizza_id = a.pizza_id
)
SELECT 
 COUNT(pizza_id)AS No_of_pizza_delivered, 
 status, 
 pizza_name 
FROM pizza_cte_type 
WHERE status = 'Delivered'
GROUP BY pizza_name, status
````
***

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT customer_id, 
       COUNT (cu.pizza_id), 
       pizza_name 
FROM pizza_runner.customer_orders cu 
JOIN pizza_runner.pizza_names pi 
ON cu.pizza_id = pi.pizza_id 
GROUP BY customer_id, pizza_name 
ORDER BY customer_id
````
***

### 6. What was the maximum number of pizzas delivered in a single order?

```sql
SELECT order_id, 
       COUNT(pizza_id) AS pizza_count
FROM pizza_runner.customer_orders
GROUP BY order_id
ORDER BY COUNT(pizza_id) DESC
LIMIT 1
````
***

### 7. For each customer, how many delivered pizzas had at least 1 change and how many -had no changes?

```sql
WITH Changed_cte AS (
  SELECT
    customer_id,
    CASE 
      WHEN exclusions IS NOT NULL AND exclusions <> '' THEN 1
      ELSE 0
    END AS exclusions_changed,
    CASE
      WHEN extras IS NOT NULL AND extras <> '' THEN 1
      ELSE 0
    END AS extras_changed
  FROM pizza_runner.customer_orders
)

SELECT 
  customer_id,
  COUNT(CASE WHEN exclusions_changed + extras_changed > 0 THEN 1 END) AS pizzas_with_changes,
  COUNT(CASE WHEN exclusions_changed + extras_changed = 0 THEN 1 END) AS pizzas_without_changes
FROM Changed_cte
GROUP BY customer_id;
````
***

### 8. How many pizzas were delivered that had both exclusions and extras?
Note: I have accounted for null as a system error during data entry.

```sql
WITH Both_Changed_cte AS (
  SELECT
    CASE 
      WHEN exclusions IS NOT NULL AND exclusions <> '' THEN 1
      ELSE 0
    END AS exclusions_changed,
    CASE
      WHEN extras IS NOT NULL AND extras <> '' THEN 1
      ELSE 0
    END AS extras_changed
  FROM pizza_runner.customer_orders
)

SELECT 
  COUNT(CASE WHEN exclusions_changed = 1 AND extras_changed = 1 THEN 1 END) AS pizzas_with_both
FROM Both_Changed_cte
````
***

### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT
  EXTRACT(HOUR FROM order_time) AS hour,
  COUNT(order_id) AS num_pizzas_ordered,
  ROUND(100.0 * COUNT(order_id) / SUM(COUNT(order_id)) OVER (), 2) AS pizza_volume_percentage
FROM pizza_runner.customer_orders
GROUP BY hour
ORDER BY hour
````
***

### 10. What was the volume of orders for each day of the week?

```sql
SELECT
  EXTRACT(DAY FROM order_time) AS days,
  COUNT(order_id) AS num_pizzas_ordered,
  ROUND(100.0 * COUNT(order_id) / SUM(COUNT(order_id)) OVER (), 2) AS pizza_volume_percentage
FROM pizza_runner.customer_orders
WHERE order_time IS NOT NULL
GROUP BY days
ORDER BY days;
````
***

