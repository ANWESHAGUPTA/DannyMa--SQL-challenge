# DannyMa--SQL-challenge
## :pizza: Case Study #2: Pizza Runner Part B
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

1. ### How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT 
 COUNT(runner_id), 
 EXTRACT(WEEK FROM registration_date) AS week_number 
FROM pizza_runner.runners
GROUP BY week_number
````
***
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT
  runner_id,
  AVG(EXTRACT(EPOCH FROM TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS') - order_time) / 60.0) AS average_time_minutes
FROM
pizza_runner.runner_orders r JOIN pizza_runner.customer_orders cu
ON  r.order_id = cu.order_id
WHERE pickup_time != 'null'
GROUP BY
runner_id
ORDER BY 
runner_id DESC
````
***

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare? 

```sql

 WITH Prep_time_cte AS
(
SELECT
COUNT(cu.order_id) AS pizza_count, 
EXTRACT(EPOCH FROM TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS') - order_time) / 60.0 AS prep_time
FROM pizza_runner.customer_orders cu
JOIN pizza_runner.runner_orders r
ON cu.order_id = r.order_id
WHERE pickup_time != 'null'
GROUP BY cu.order_id , pickup_time, order_time
)
SELECT pizza_count, ROUND (CAST (AVG (prep_time) AS numeric),2) AS prep_time_minutes FROM Prep_time_cte GROUP BY pizza_count ORDER BY pizza_count DESC

````
***

### 4. What was the average distance travelled for each customer?

```sql

SELECT 
CONCAT(ROUND (AVG (CAST(RTRIM(distance, 'km') AS numeric )), 1), ' KM' )AS Avg_dist,
customer_id
FROM pizza_runner.runner_orders
JOIN pizza_runner.customer_orders
ON pizza_runner.runner_orders.order_id = pizza_runner.customer_orders.order_id
WHERE distance != 'null'
GROUP BY customer_id

````
***

### 5. What was the difference between the longest and shortest delivery times for all orders? 

```sql
SELECT 
MAX(CAST((RTRIM(RTRIM(duration, 'mins'), 'minutes')) AS numeric)) AS Max_duration_mins,
MIN(CAST((RTRIM(RTRIM(duration, 'mins'), 'minutes')) AS numeric)) AS Min_Duration_mins,
MAX(CAST((RTRIM(RTRIM(duration, 'mins'), 'minutes')) AS numeric)) - MIN(CAST((RTRIM(RTRIM(duration, 'mins'), 'minutes')) AS numeric))  AS Diff_duration_mins
FROM pizza_runner.runner_orders
WHERE duration != 'null'
````
***

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql

WITH  Average_speed_cte AS
(
SELECT 
      (REGEXP_MATCHES(distance, '\d+\.?\d*'))[1] :: NUMERIC AS Distance_in_km,
      (REGEXP_MATCHES(duration, '\d+\.?\d*'))[1] :: NUMERIC AS Duration,
  Runner_id
  FROM pizza_runner.runner_orders
  WHERE distance != 'null' AND duration != 'null'
  ORDER BY runner_id
)
SELECT runner_id,
       Distance_in_km,
       CAST (Duration/60 AS DECIMAL(10,2)) AS Duration_in_hr,
       CAST (Distance_in_km/CAST (Duration/60 AS DECIMAL(10,2)) AS DECIMAL(10,2)) AS Average_Speed 
 FROM Average_speed_cte 
 GROUP BY runner_id, Distance_in_km, Duration_in_hr, Average_Speed
 ORDER BY runner_id

````
***
Insight: The difference in average speed for the same amount of distance traveled is much wider for Runner 2 than Runner 1. It could be attributed to the fact that one of runner 2 delivery routes had a traffic/obstacle issue.

### 7. What is the successful delivery percentage for each runner?

```sql
WITH OrderStatus AS (
    SELECT
        runner_id,
        CASE
            WHEN pickup_time ='null' THEN 'no'
            ELSE 'yes'
        END AS delivered_order
    FROM pizza_runner.runner_orders
)


SELECT
    runner_id,
    ROUND(100.0 * SUM(CASE WHEN delivered_order = 'yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS delivery_success_percentage
FROM OrderStatus
GROUP BY runner_id
ORDER BY runner_id;
````
***

