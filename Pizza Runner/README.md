# `Case Study #2 - Pizza Runner`
 <img width="600" height="600" alt="2" src="https://github.com/user-attachments/assets/42f79335-1db4-45eb-9679-b66a9bd7a0ac" />

## Introduction
A fictional pizza delivery startup that combines classic pizza service with an Uber-style runner network. The company collects detailed data on customer orders, pizza ingredients, and delivery runner performance. This challenge gives you the opportunity to explore a realistic business dataset and apply SQL to uncover insights about operational efficiency, customer behavior, and ingredient trends

## 🎯 Problem Statement
The goal of this challenge is to analyze the Pizza Runner database using SQL to help improve business decisions. You’ll work with multiple tables including customer orders, runner delivery records, pizza recipes, and toppings. The tasks require cleaning data, handling irregular values, and performing analytical queries to answer questions such as:
* `How many pizzas were ordered and delivered?`
* `What patterns exist in runner performance and delivery times?`
* `Which ingredients are most frequently excluded or added?`
* `How much revenue has the business generated under different pricing scenarios?`

## Entity Relationship Diagram
<img width="767" height="398" alt="Screenshot 2025-12-15 at 11 09 29 PM" src="https://github.com/user-attachments/assets/5e835254-087c-419b-ab50-e70964a5479a" />

## Tables
### Table 1 : runners
<img width="312" height="240" alt="Screenshot 2025-12-15 at 11 13 26 PM" src="https://github.com/user-attachments/assets/23a2b92c-f598-4045-af6e-2907eae7b03e" />

### Table 2 : customer_orders
<img width="549" height="488" alt="Screenshot 2025-12-15 at 11 14 34 PM" src="https://github.com/user-attachments/assets/9fb2fb0e-3d76-49f5-8538-880f79b9d418" />

### Table 3 : runner_orders
<img width="742" height="484" alt="Screenshot 2025-12-15 at 11 15 05 PM" src="https://github.com/user-attachments/assets/25dc03fe-1e7b-46d8-a5af-81e137bd716c" />

### Table 4 : pizza_names
<img width="253" height="149" alt="Screenshot 2025-12-15 at 11 15 20 PM" src="https://github.com/user-attachments/assets/49e4b9d7-0f53-4e6c-9139-c59c5c912bf8" />

### Table 5 : pizza_names
<img width="299" height="154" alt="Screenshot 2025-12-15 at 11 16 27 PM" src="https://github.com/user-attachments/assets/af24ef74-350f-493b-8256-7fee8a2a885d" />

### Table 6 : pizza_names
<img width="291" height="573" alt="Screenshot 2025-12-15 at 11 16 50 PM" src="https://github.com/user-attachments/assets/0d23ff78-537b-4e16-b9d9-5db4ba00af02" />

## Case Study Questions + Solution
**🍕A. Pizza Metrics🍕**

**1.How many pizzas were ordered?**
```sql
SELECT COUNT(*) as num_of_pizza
FROM customer_orders ;
```

**2.How many unique customer orders were made?**
```sql
SELECT COUNT(DISTINCT(order_id)) as unique_order
FROM customer_orders ;
```
**3.How many successful orders were delivered by each runner?**
```sql
SELECT runner_id, COUNT(order_id)
FROM runner_orders
WHERE distance != 'null'
GROUP BY runner_id;
```
**4.How many of each type of pizza was delivered?**
```sql
SELECT pizza_name, COUNT(ro.order_id)
FROM customer_orders co JOIN pizza_names pn ON (co.pizza_id = pn.pizza_id) JOIN runner_orders ro ON (co.order_id = ro.order_id)
WHERE distance != 'null'
GROUP BY pizza_name;
```
**5.How many Vegetarian and Meatlovers were ordered by each customer?**
```sql
SELECT customer_id, pizza_name,  COUNT(co.pizza_id)
FROM customer_orders co JOIN pizza_names pn ON (co.pizza_id = pn.pizza_id)
GROUP BY pizza_name, customer_id
ORDER BY customer_id;
```
**6.What was the maximum number of pizzas delivered in a single order?**
```sql
SELECT co.order_id, COUNT(*) as num_of_delivered_pizz
    FROM customer_orders co JOIN  runner_orders ro ON (co.order_id = ro.order_id)
   WHERE distance != 'null'
    GROUP BY co.order_id
   ORDER BY num_of_delivered_pizz desc limit 1;
```
**7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
```sql
SELECT co.customer_id,
SUM(CASE
	WHEN (exclusions is not NULL AND exclusions !='null') OR (extras is not NULL AND extras!= 'null') THEN 1 ELSE 0 
END) AS changed_pizza,
SUM(CASE
	WHEN (exclusions is NULL OR exclusions ='null') AND	(extras is NULL OR extras ='null') THEN 1 ELSE 0 
END) AS no_change
FROM customer_orders co JOIN  runner_orders ro ON (co.order_id = ro.order_id)
WHERE distance != 'null'
GROUP BY co.customer_id
ORDER BY co.customer_id ;
```
**8.How many pizzas were delivered that had both exclusions and extras?**
```sql
SELECT SUM(CASE
	WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1 ELSE 0 
END) AS exclude_and_extra_pizzas
FROM customer_orders co JOIN  runner_orders ro ON (co.order_id = ro.order_id)
WHERE cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation');
```
**9.What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT  EXTRACT(HOUR FROM order_time) as hour_of_day, COUNT(order_id)
FROM customer_orders
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
**10.What was the volume of orders for each day of the week?**
```sql
SELECT  CASE EXTRACT(DOW FROM order_time)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
        ELSE 'Invalid Day'
    END as day_of_week, COUNT(order_id) as TotalPizzaOrdered
FROM customer_orders
GROUP BY day_of_week
ORDER BY TotalPizzaOrdered DESC;
```
**B.🏃🏽‍♂️Runner and Customer Experience🏃🏽‍♂️**

**1.How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
```sql
SELECT EXTRACT(WEEK FROM registration_date + 3) AS week_of_year,
COUNT(runner_id)
FROM runners
GROUP BY week_of_year
ORDER BY week_of_year;
```
**2.What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
```sql

```
**3.Is there any relationship between the number of pizzas and how long the order takes to prepare?**
```sql
with cte as(
SELECT COUNT(*) as num_of_order, EXTRACT(EPOCH FROM (pickup_time::timestamp - order_time::timestamp)) / 60 as time_prepare
FROM customer_orders co JOIN runner_orders ro ON (co.order_id = ro.order_id)
WHERE ro.distance !='null' OR 
    (pickup_time IS NOT NULL
    AND pickup_time != 'null')   
GROUP BY co.order_id,time_prepare
ORDER BY num_of_order)
SELECT num_of_order,ROUND(AVG(time_prepare)::numeric,0) as Avgtime
FROM cte
GROUP BY num_of_order;
```
**4.What was the average distance travelled for each customer?**
```sql
SELECT customer_id, ROUND(AVG(CAST( REPLACE(distance,'km','') as numeric)),2) as average_distance
FROM customer_orders co JOIN runner_orders ro ON co.order_id = ro.order_id
WHERE distance IS NOT NULL AND distance != 'null'
GROUP BY customer_id;
```
**5.What was the difference between the longest and shortest delivery times for all orders?**
```sql
SELECT MAX(CAST( REPLACE(distance,'km','') as numeric))- MIN(CAST( REPLACE(distance,'km','') as numeric)) as longest_vs_shortest_deliverytime
FROM customer_orders co JOIN runner_orders ro ON co.order_id = ro.order_id
WHERE   distance != 'null';
```
**6.What was the average speed for each runner for each delivery and do you notice any trend for these values?**
```sql
SELECT runner_id,
    order_id,
    ROUND(AVG(
        CAST(REGEXP_REPLACE(NULLIF(distance, 'null'), '[^0-9.]', '', 'g') AS NUMERIC )* 60 / 
        CAST(REGEXP_REPLACE(NULLIF(duration, 'null'), '[^0-9.]', '', 'g')AS NUMERIC) 
    ),2) AS km_per_hr 
FROM
    runner_orders
WHERE
    NULLIF(distance, 'null') IS NOT NULL 
    AND NULLIF(duration, 'null') IS NOT NULL
GROUP BY
    runner_id, order_id
ORDER BY order_id;
```
**7.What is the successful delivery percentage for each runner?**
```sql
SELECT
    runner_id,
    -- Formula: (Successful Orders / Total Orders) * 100
    ROUND( ( CAST
       (SUM
         (CASE
           WHEN cancellation = 'Restaurant Cancellation' OR cancellation = 'Customer Cancellation' THEN 0
           ELSE 1 
          END) AS DECIMAL)
        / COUNT(*)
    ) * 100, 0) AS successful_delivery_percentage
FROM
    runner_orders
GROUP BY
    runner_id
ORDER BY
    runner_id;
```

**C. Ingredient Optimisation**

**1.What are the standard ingredients for each pizza?**
**2.What was the most commonly added extra?**
```sql
SELECT
    T.extra_item AS most_common_extra,
    COUNT(T.extra_item) AS item_count
FROM
    customer_orders co,
    LATERAL UNNEST(string_to_array(
        -- Clean the extras column (replace 'null', remove spaces, cast to array)
        REPLACE(NULLIF(co.extras, 'null'), ' ', ''), 
    ',')) AS T(extra_item)
WHERE
    co.extras IS NOT NULL 
    AND co.extras != 'null'
    AND co.extras != ''
GROUP BY
   T.extra_item
ORDER BY
    item_count DESC;

SELECT topping_id, topping_name
FROM pizza_toppings
WHERE topping_id IN ( 1,4,5);
```
**3.What was the most common exclusion?**
```sql
SELECT
T.exclusion_item AS most_common_exclusion,
COUNT(T.exclusion_item) AS item_count
FROM
customer_orders co,
LATERAL UNNEST(string_to_array(
-- Clean the extras column (replace 'null', remove spaces, cast to array)
REPLACE(NULLIF(co.exclusions, 'null'), ' ', ''), 
',')) AS T(exclusion_item)
WHERE
co.exclusions IS NOT NULL 
AND co.exclusions != 'null'
AND co.exclusions != ''
GROUP BY
T.exclusion_item
ORDER BY
 item_count DESC;
    
SELECT topping_id, topping_name
FROM pizza_toppings
WHERE topping_id IN (4,6,2);
```


**D. Pricing and Ratings**
**1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**
```sql
SELECT SUM(CASE WHEN pizza_id = 1 THEN  12 ELSE 10 END) AS total_money
FROM customer_orders co JOIN runner_orders ro ON (co.order_id = ro.order_id)
WHERE  distance !='null' ;
```
**2.What if there was an additional $1 charge for any pizza extras?**
***Add cheese is $1 extra**
```sql
WITH OrderExtrasCount AS (
    SELECT
        co.order_id,
        co.pizza_id,
        CASE
            WHEN co.extras IS NULL OR co.extras IN ('null', 'NaN', '') THEN 0
            ELSE CARDINALITY(string_to_array(REPLACE(REPLACE(co.extras, ' ', ''), 'null', ''), ','))
        END AS extras_count
    FROM
        customer_orders co
    JOIN
        runner_orders ro ON co.order_id = ro.order_id
    WHERE
        ro.distance !='null' 
)

SELECT
    -- A. Calculate Extras Revenue ($1 per extra)
    SUM(oec.extras_count * 1) AS extras_revenue, 
    -- B. Total Money Made
    SUM(CASE WHEN oec.pizza_id = 1 THEN 12 ELSE 10 END) + SUM(oec.extras_count * 1) AS total_money_made
```

**3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**
```sql
CREATE TABLE runner_ratings (
    rating_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL UNIQUE, -- UNIQUE constraint ensures one rating per order
    runner_id INTEGER NOT NULL,
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    rating_time TIMESTAMP NOT NULL
);
```
**4.**
**5.If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

```sql
WITH RevenueCTE AS (
    -- Calculate total revenue from pizza sales
    SELECT
        SUM(CASE WHEN co.pizza_id = 1 THEN 12 ELSE 10 END) AS total_revenue_usd
    FROM
        customer_orders co
    JOIN
        runner_orders ro ON co.order_id = ro.order_id
    WHERE
        NULLIF(ro.distance, 'null') IS NOT NULL
        AND LENGTH(REGEXP_REPLACE(NULLIF(ro.distance, 'null'), '[^0-9.]', '', 'g')) > 0
),
CostsCTE AS (
    -- Calculate total runner payment costs
    SELECT
        SUM(
            CAST(REGEXP_REPLACE(NULLIF(ro.distance, 'null'), '[^0-9.]', '', 'g') AS NUMERIC)
            * 0.30
        ) AS total_cost_usd
    FROM
        runner_orders ro
    WHERE
        NULLIF(ro.distance, 'null') IS NOT NULL
        AND LENGTH(REGEXP_REPLACE(NULLIF(ro.distance, 'null'), '[^0-9.]', '', 'g')) > 0
)
SELECT
    -- Final calculation: Revenue - Cost
    (SELECT total_revenue_usd FROM RevenueCTE) - (SELECT total_cost_usd FROM CostsCTE) AS money_left_over

```



