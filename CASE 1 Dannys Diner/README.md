# DannyMa--SQL-challenge
## 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

This repository contains SQL queries and solutions for a set of case study questions related to Danny's Diner restaurant. The questions cover various aspects of customer transactions and preferences. All the information from the case study can be found here: [here](https://8weeksqlchallenge.com/case-study-1/).

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. This is what you learn!!

Common Table Expressions
Group By Aggregates
Window Functions for ranking
Table Joins

***
## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***
## Question and Solution

Executing the queries using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).
## SQL Queries

### 1. Total Amount Spent by Each Customer
```sql
SELECT
  s.customer_id AS Customer_Name, 
  CONCAT(SUM(e.price), ' USD') AS Total_Amount_Spent_By_Customer 
FROM dannys_diner.sales s 
JOIN dannys_diner.menu e ON s.product_id = e.product_id 
GROUP BY s.customer_id 
ORDER BY s.customer_id
````
***
### 2. Number of Days Each Customer Visited the Restaurant
```sql
SELECT
  s.customer_id AS Customer_Name, 
  COUNT(DISTINCT s.order_date) AS Days_Visited 
FROM dannys_diner.sales s 
GROUP BY s.customer_id 
ORDER BY s.customer_id
````
***
### 3. First Item Purchased by Each Customer
```sql
WITH First_order_for_customer_cte AS 
(
  SELECT 
    customer_id, 
    product_name, 
    order_date,
    ROW_number() OVER (PARTITION BY customer_id ORDER BY customer_id, order_date ASC) AS RowNumberAfterOrderByDate
  FROM dannys_diner.sales s 
  JOIN dannys_diner.menu e ON s.product_id = e.product_id 
)
SELECT 
  customer_id, 
  product_name, 
  RowNumberAfterOrderByDate
FROM First_order_for_customer_cte
WHERE RowNumberAfterOrderByDate = 1
````
***
### 4. Most Purchased Item on the Menu
```sql
SELECT 
  COUNT(customer_id) AS Num_of_times_purchased,
  s.product_id , 
  product_name 
FROM dannys_diner.sales s 
JOIN dannys_diner.menu e ON s.product_id = e.product_id 
GROUP BY s.product_id, product_name 
ORDER BY s.product_id DESC LIMIT 1
````
***
### 5. Most Popular Item for Each Customer
```sql
WITH popular_item_cte AS 
(
  SELECT 
    customer_id, 
    product_id,
    RANK() OVER (PARTITION by customer_id ORDER BY COUNT(order_date) DESC) AS popularitem
  FROM dannys_diner.sales
  GROUP BY customer_id, product_id
)
SELECT 
  customer_id,  
  popularitem,
  p.product_id,
  product_name 
FROM popular_item_cte p 
JOIN dannys_diner.menu e ON p.product_id = e.product_id 
WHERE popularitem = 1 
ORDER BY customer_id
````
***
### 6. First Item Purchased by Customers After Joining
```sql
WITH First_purchase_cte AS
(
  SELECT 
    m.customer_id, 
    product_name,
    order_date,
    join_date,
    RANK() OVER (PARTITION BY m.customer_id ORDER BY order_date ASC ) AS First_purchase_rank
  FROM dannys_diner.members m 
  JOIN dannys_diner.sales s ON m.customer_id = s.customer_id 
  JOIN dannys_diner.menu e ON s.product_id = e.product_id
  WHERE join_date <= order_date 
)
SELECT * FROM First_purchase_cte WHERE First_purchase_rank = 1
````
***
### 7. Item Purchased Just Before Customer Membership
```sql
WITH First_purchase_before_member_cte AS
(
  SELECT 
    m.customer_id, 
    product_name,
    order_date,
    join_date,
    RANK() OVER (PARTITION BY m.customer_id ORDER BY order_date DESC) AS First_purchase_rank
  FROM dannys_diner.members m 
  JOIN dannys_diner.sales s ON m.customer_id = s.customer_id 
  JOIN dannys_diner.menu e ON s.product_id = e.product_id
  WHERE join_date > order_date 
)
SELECT * FROM First_purchase_before_member_cte WHERE First_purchase_rank = 1
````
***
### 8. Total Items and Amount Spent for Each Member Before Joining
```sql
SELECT 
  s.customer_id, 
  CONCAT(SUM(price), ' USD') AS Total_amount, 
  COUNT (s.product_id) AS Total_items 
FROM dannys_diner.members b 
JOIN dannys_diner.sales s ON b.customer_id = s.customer_id 
JOIN dannys_diner.menu e ON s.product_id = e.product_id 
WHERE join_date > order_date 
GROUP BY s.customer_id 
ORDER BY s.customer_id
````
***
### 9. Points Calculation for Each Customer
```sql
WITH Points_cte AS
(
  SELECT 
    s.customer_id,
    CASE 
      WHEN s.product_id = 1 THEN SUM(e.price)*20
      ELSE SUM(e.price)*10
    END AS Points
  FROM dannys_diner.sales s 
  JOIN dannys_diner.menu e ON s.product_id = e.product_id 
  GROUP BY s.customer_id,  s.product_id
  ORDER BY s.customer_id
)
SELECT 
  customer_id, 
  SUM(POINTS) AS Customer_Points 
FROM Points_cte 
GROUP BY customer_id
````
***
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH Points_joindate_cte AS
(
SELECT s.customer_id,
       SUM(CASE
               WHEN  order_date BETWEEN join_date AND (join_date + INTERVAL '1 week' ) THEN price*20
               ELSE price*10
           END) AS customer_points
FROM dannys_diner.menu AS m
JOIN dannys_diner.sales AS s 
ON m.product_id = s.product_id
JOIN dannys_diner.members AS a
ON s.customer_id = a.customer_id
WHERE order_date <='2021-01-31'
AND order_date >=join_date
GROUP BY s.customer_id
ORDER BY s.customer_id
)

SELECT customer_id, 
       SUM(customer_points) AS Customer_Points 
FROM Points_joindate_cte 
GROUP BY customer_id
````
***

