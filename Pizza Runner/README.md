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


## Schema (PostgreSQL v13)

    CREATE SCHEMA pizza_runner;
    SET search_path = pizza_runner;
    
    DROP TABLE IF EXISTS runners;
    CREATE TABLE runners (
      "runner_id" INTEGER,
      "registration_date" DATE
    );
    INSERT INTO runners
      ("runner_id", "registration_date")
    VALUES
      (1, '2021-01-01'),
      (2, '2021-01-03'),
      (3, '2021-01-08'),
      (4, '2021-01-15');
    
    
    DROP TABLE IF EXISTS customer_orders;
    CREATE TABLE customer_orders (
      "order_id" INTEGER,
      "customer_id" INTEGER,
      "pizza_id" INTEGER,
      "exclusions" VARCHAR(4),
      "extras" VARCHAR(4),
      "order_time" TIMESTAMP
    );
    
    INSERT INTO customer_orders
      ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
    VALUES
      ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
      ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
      ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
      ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
      ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
      ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
      ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
      ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
      ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
      ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
      ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');
    
    
    DROP TABLE IF EXISTS runner_orders;
    CREATE TABLE runner_orders (
      "order_id" INTEGER,
      "runner_id" INTEGER,
      "pickup_time" VARCHAR(19),
      "distance" VARCHAR(7),
      "duration" VARCHAR(10),
      "cancellation" VARCHAR(23)
    );
    
    INSERT INTO runner_orders
      ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
    VALUES
      ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
      ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
      ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
      ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
      ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
      ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
      ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
      ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
      ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
      ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');
    
    
    DROP TABLE IF EXISTS pizza_names;
    CREATE TABLE pizza_names (
      "pizza_id" INTEGER,
      "pizza_name" TEXT
    );
    INSERT INTO pizza_names
      ("pizza_id", "pizza_name")
    VALUES
      (1, 'Meatlovers'),
      (2, 'Vegetarian');
    
    
    DROP TABLE IF EXISTS pizza_recipes;
    CREATE TABLE pizza_recipes (
      "pizza_id" INTEGER,
      "toppings" TEXT
    );
    INSERT INTO pizza_recipes
      ("pizza_id", "toppings")
    VALUES
      (1, '1, 2, 3, 4, 5, 6, 8, 10'),
      (2, '4, 6, 7, 9, 11, 12');
    
    
    DROP TABLE IF EXISTS pizza_toppings;
    CREATE TABLE pizza_toppings (
      "topping_id" INTEGER,
      "topping_name" TEXT
    );
    INSERT INTO pizza_toppings
      ("topping_id", "topping_name")
    VALUES
      (1, 'Bacon'),
      (2, 'BBQ Sauce'),
      (3, 'Beef'),
      (4, 'Cheese'),
      (5, 'Chicken'),
      (6, 'Mushrooms'),
      (7, 'Onions'),
      (8, 'Pepperoni'),
      (9, 'Peppers'),
      (10, 'Salami'),
      (11, 'Tomatoes'),
      (12, 'Tomato Sauce');


## Case Study Questions + Solution
**Approach**:
* There are many ways to solve this challenge, but the most efficient method is to clean the data first, and then write the queries.

* The majority of the data cleaning involves converting the null values to a blank space (' ') and removing 'km' or 'minutes' from the distance and duration fields so we can deal with numbers only. It is very important that you convert the time into manageable time units; this will save you a lot of time writing the queries.

* I completed this challenge while still feeling unclear about the concept of cleaning SQL tables, so I didn't clean any of the data upfront. I just converted the data as I went for each query, which resulted in my queries looking very long and complicated.

___ 
**🍕A.PIZZA METRICS🍕**

**1.How many pizzas were ordered?**
```sql
SELECT COUNT(*) as num_of_pizza
FROM customer_orders ;
```

| num_of_pizza |
| ------------ |
| 14           |


**2.How many unique customer orders were made?**
```sql
SELECT COUNT(DISTINCT(order_id)) as unique_order
FROM customer_orders ;
```

| unique_order |
| ------------ |
| 10           |

**3.How many successful orders were delivered by each runner?**
```sql
SELECT runner_id, COUNT(order_id)
FROM runner_orders
WHERE distance != 'null'
GROUP BY runner_id;
```
| runner_id | count |
| --------- | ----- |
| 3         | 1     |
| 2         | 3     |
| 1         | 4     |

**4.How many of each type of pizza was delivered?**
```sql
SELECT pizza_name, COUNT(ro.order_id)
FROM customer_orders co JOIN pizza_names pn ON (co.pizza_id = pn.pizza_id) JOIN runner_orders ro ON (co.order_id = ro.order_id)
WHERE distance != 'null'
GROUP BY pizza_name;
```
| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |

**5.How many Vegetarian and Meatlovers were ordered by each customer?**
```sql
SELECT customer_id, pizza_name,  COUNT(co.pizza_id)
FROM customer_orders co JOIN pizza_names pn ON (co.pizza_id = pn.pizza_id)
GROUP BY pizza_name, customer_id
ORDER BY customer_id;
```

| customer_id | pizza_name | count |
| ----------- | ---------- | ----- |
| 101         | Meatlovers | 2     |
| 101         | Vegetarian | 1     |
| 102         | Meatlovers | 2     |
| 102         | Vegetarian | 1     |
| 103         | Meatlovers | 3     |
| 103         | Vegetarian | 1     |
| 104         | Meatlovers | 3     |
| 105         | Vegetarian | 1     |


**6.What was the maximum number of pizzas delivered in a single order?**
```sql
SELECT co.order_id, COUNT(*) as num_of_delivered_pizz
    FROM customer_orders co JOIN  runner_orders ro ON (co.order_id = ro.order_id)
   WHERE distance != 'null'
    GROUP BY co.order_id
   ORDER BY num_of_delivered_pizz desc limit 1;
```
| order_id | num_of_delivered_pizz |
| -------- | --------------------- |
| 4        | 3                     |

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

| customer_id | changed_pizza | no_change |
| ----------- | ------------- | --------- |
| 101         | 2             | 0         |
| 102         | 2             | 1         |
| 103         | 3             | 0         |
| 104         | 2             | 1         |
| 105         | 1             | 0         |


**8.How many pizzas were delivered that had both exclusions and extras?**
```sql
SELECT SUM(CASE
	WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1 ELSE 0 
END) AS exclude_and_extra_pizzas
FROM customer_orders co JOIN  runner_orders ro ON (co.order_id = ro.order_id)
WHERE cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation');
```
| exclude_and_extra_pizzas |
| ------------------------ |
| 6                        |

**9.What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT  EXTRACT(HOUR FROM order_time) as hour_of_day, COUNT(order_id)
FROM customer_orders
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
| hour_of_day | count |
| ----------- | ----- |
| 11          | 1     |
| 13          | 3     |
| 18          | 3     |
| 19          | 1     |
| 21          | 3     |
| 23          | 3     |

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
| day_of_week | totalpizzaordered |
| ----------- | ----------------- |
| Saturday    | 5                 |
| Wednesday   | 5                 |
| Thursday    | 3                 |
| Friday      | 1                 |

---
**B.🏃🏽‍♂️RUNNER AND CUSTOMER EXPERIENCE🏃🏽‍♂️**

**1.How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
```sql
SELECT EXTRACT(WEEK FROM registration_date + 3) AS week_of_year,
COUNT(runner_id)
FROM runners
GROUP BY week_of_year
ORDER BY week_of_year;
```

| week_of_year | count |
| ------------ | ----- |
| 1            | 2     |
| 2            | 1     |
| 3            | 1     |


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
| num_of_order | avgtime |
| ------------ | ------- |
| 1            | 12      |
| 2            | 18      |
| 3            | 29      |

**4.What was the average distance travelled for each customer?**
```sql
SELECT customer_id, ROUND(AVG(CAST( REPLACE(distance,'km','') as numeric)),2) as average_distance
FROM customer_orders co JOIN runner_orders ro ON co.order_id = ro.order_id
WHERE distance IS NOT NULL AND distance != 'null'
GROUP BY customer_id;
```

| customer_id | average_distance |
| ----------- | ---------------- |
| 101         | 20.00            |
| 103         | 23.40            |
| 104         | 10.00            |
| 105         | 25.00            |
| 102         | 16.73            |

**5.What was the difference between the longest and shortest delivery times for all orders?**
```sql
SELECT MAX(CAST( REPLACE(distance,'km','') as numeric))- MIN(CAST( REPLACE(distance,'km','') as numeric)) as longest_vs_shortest_deliverytime
FROM customer_orders co JOIN runner_orders ro ON co.order_id = ro.order_id
WHERE   distance != 'null';
```

| longest_vs_shortest_deliverytime |
| -------------------------------- |
| 15                               |


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
| runner_id | order_id | km_per_hr |
| --------- | -------- | --------- |
| 1         | 1        | 37.50     |
| 1         | 2        | 44.44     |
| 1         | 3        | 40.20     |
| 2         | 4        | 35.10     |
| 3         | 5        | 40.00     |
| 2         | 7        | 60.00     |
| 2         | 8        | 93.60     |
| 1         | 10       | 60.00     |

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
| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 1         | 100                            |
| 2         | 75                             |
| 3         | 50                             |

___ 

**🥗C.INGREDIENT OPTIMISATION🥗**

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
| most_common_extra | item_count |
| ----------------- | ---------- |
| 1                 | 4          |
| 5                 | 1          |
| 4                 | 1          |

| topping_id | topping_name |
| ---------- | ------------ |
| 1          | Bacon        |
| 4          | Cheese       |
| 5          | Chicken      |


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
| most_common_exclusion | item_count |
| --------------------- | ---------- |
| 4                     | 4          |
| 6                     | 1          |
| 2                     | 1          |

| topping_id | topping_name |
| ---------- | ------------ |
| 2          | BBQ Sauce    |
| 4          | Cheese       |
| 6          | Mushrooms    |

**💰D.PRICING AND RATINGS💰**
**1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**
```sql
SELECT SUM(CASE WHEN pizza_id = 1 THEN  12 ELSE 10 END) AS total_money
FROM customer_orders co JOIN runner_orders ro ON (co.order_id = ro.order_id)
WHERE  distance !='null' ;
```
| total_money |
| ----------- |
| 138         |

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
| extras_revenue | total_money_made |
| -------------- | ---------------- |
| 4              | 142              |

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
| money_left_over |
| --------------- |
| 94.440          |



