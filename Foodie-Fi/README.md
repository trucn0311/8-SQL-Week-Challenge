# `Case Study #3 - Foodie-Fi`

<img width="600" height="600" alt="image" src="https://github.com/user-attachments/assets/fb09e048-7e12-4dc9-8fdf-80344cbdb9bd" />

## Introduction
Danny launched Foodie-Fi, a niche "Netflix-style" streaming service for cooking content, with the goal of making every business decision backed by data-driven insights. To ensure the startup's success, he needs to analyze customer subscription patterns—such as how they move from free trials to paid plans or when they cancel—to optimize investment, improve retention, and drive future growth.

## 🎯 Problem Statement
**The Concept** : A niche streaming service (like "Netflix for food") launched in 2020, offering exclusive global cooking content.

**The Model**: A subscription-based startup selling both monthly and annual access.

**The Mission**: Danny, the founder, built the company to be data-driven, using customer subscription patterns to guide all investments and features.

**The Goal**: Analyze digital subscription data to answer key business questions about growth, retention, and customer behavior.

## Entity Relationship Diagram
<img width="767" height="398" alt="image" src="https://github.com/user-attachments/assets/a5904117-7a7c-46c8-a725-0b0a5c683fde" />

## Tables

### Table 1: plans
<img width="326" height="274" alt="Screenshot 2026-01-13 at 3 52 41 PM" src="https://github.com/user-attachments/assets/cbbed88a-18e4-45d0-b71a-057508cedace" />

### Table 2: subscriptions
<img width="365" height="415" alt="Screenshot 2026-01-13 at 3 53 30 PM" src="https://github.com/user-attachments/assets/fb4d1f34-2168-4c94-9206-f71e0c387da4" />

## Schema (PostgreSQL v13)
[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)

 ## B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**

   ```sql
SELECT COUNT(DISTINCT customer_id) AS total_num_customer
    FROM subscriptions;
```
    

| total_num_customer |
| ------------------ |
| 1000               |

---

**2.What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

 ```sql
    SELECT EXTRACT(MONTH FROM start_date) AS monthly, count(*) AS monthly_distribution
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE plan_name = 'trial'
    GROUP BY monthly
    ORDER BY monthly;
 ```

| monthly | monthly_distribution |
| ------- | -------------------- |
| 1       | 88                   |
| 2       | 68                   |
| 3       | 94                   |
| 4       | 81                   |
| 5       | 88                   |
| 6       | 79                   |
| 7       | 89                   |
| 8       | 88                   |
| 9       | 87                   |
| 10      | 79                   |
| 11      | 75                   |
| 12      | 84                   |

---

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**
```sql
    SELECT plan_name,COUNT(*) AS events
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE EXTRACT(YEAR FROM start_date) > '2020'
    GROUP BY plan_name
    ORDER BY events;
```

| plan_name     | events |
| ------------- | ------ |
| basic monthly | 8      |
| pro monthly   | 60     |
| pro annual    | 63     |
| churn         | 71     |

---

**4. What is the customer count and percentage of customers who have churned,rounded to 1 decimal place?**

 ```sql
    SELECT COUNT(DISTINCT customer_id) AS churned_customers, 
           ROUND((COUNT(customer_id)* 100.0)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions ),1) AS churn_rate
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE plan_name = 'churn';
 ```

| churned_customers | churn_rate |
| ----------------- | ---------- |
| 307               | 30.7       |


---


**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

 ```sql
    --Ver1
    WITH user_journey AS (
        SELECT 
            customer_id, 
            plan_name,
            RANK() OVER(PARTITION BY customer_id ORDER BY start_date) as plan_rank,
            LEAD(plan_name) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_plan
        FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
    ),
    churn_counts AS (
        SELECT 
            -- Total unique customers in the dataset
            COUNT(DISTINCT customer_id) AS total_customers,
            -- Customers whose 1st plan was trial and 2nd was churn
            COUNT(CASE WHEN plan_rank = 1 AND plan_name = 'trial' AND next_plan = 'churn' THEN 1 END) AS direct_churns
        FROM user_journey
    )
    SELECT 
        direct_churns,
        ROUND(100.0 * direct_churns / total_customers) AS churn_percentage
    FROM churn_counts;
 ```

| direct_churns | churn_percentage |
| ------------- | ---------------- |
| 92            | 9                |

 ```sql
    --ver2
    WITH ranking AS (
        SELECT 
            customer_id, 
            plan_name, 
            RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS ranks
        FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
    
    )
    SELECT COUNT(*) AS trial_to_churn_count, ROUND(COUNT(*) * 100.0 /
        (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 0) AS churn_percentage
    FROM ranking r1
    JOIN ranking r2 ON r1.customer_id = r2.customer_id
    WHERE r1.ranks = 1 AND r1.plan_name = 'trial'
      AND r2.ranks = 2 AND r2.plan_name = 'churn';
 ```

| trial_to_churn_count | churn_percentage |
| -------------------- | ---------------- |
| 92                   | 9                |


---


**6. What is the number and percentage of customer plans after their initial free trial?**

 ```sql
    WITH user_journey AS (
        SELECT
            customer_id, 
            plan_name,
            RANK() OVER(PARTITION BY customer_id ORDER BY start_date) as plan_rank,
            LEAD(plan_name) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_plan
        FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
        
    )
    SELECT next_plan,
            COUNT(next_plan) AS next_plan_counts,
            ROUND(100.0*COUNT(next_plan)/ (SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) as next_plan_rates
        FROM user_journey
        where next_plan is not null AND plan_name = 'trial'
        GROUP BY next_plan;
 ```

| next_plan     | next_plan_counts | next_plan_rates |
| ------------- | ---------------- | --------------- |
| basic monthly | 546              | 54.6            |
| churn         | 92               | 9.2             |
| pro annual    | 37               | 3.7             |
| pro monthly   | 325              | 32.5            |


---


**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

 ```sql
    WITH CTE AS (
      SELECT
        customer_id,
        plan_id,
      	start_date,
        LEAD(start_date) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS next_date
      FROM subscriptions
      WHERE start_date <= '2020-12-31'
    )
    
    SELECT
    	plan_id, 
    	COUNT(DISTINCT customer_id) AS customers,
      ROUND(100.0 * 
        COUNT(DISTINCT customer_id)
        / (SELECT COUNT(DISTINCT customer_id) 
          FROM subscriptions)
      ,1) AS percentage
    FROM CTE
    WHERE next_date IS NULL
    GROUP BY plan_id;
 ```

| plan_id | customers | percentage |
| ------- | --------- | ---------- |
| 0       | 19        | 1.9        |
| 1       | 224       | 22.4       |
| 2       | 326       | 32.6       |
| 3       | 195       | 19.5       |
| 4       | 236       | 23.6       |

---
**8. How many customers have upgraded to an annual plan in 2020?**

 ```sql
    SELECT 
        COUNT(DISTINCT customer_id) AS annual_upgrade_2020
    FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
    WHERE plan_name = 'pro annual' 
      AND EXTRACT(YEAR FROM start_date) = '2020';
 ```

| annual_upgrade_2020 |
| ------------------- |
| 195                 |

---


**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

 ```sql
    WITH trial_dates AS (
        -- Get the first date for every customer
        SELECT 
            customer_id, 
            MIN(start_date) AS join_date
        FROM subscriptions
      GROUP BY customer_id
       
    ),
    annual_dates AS (
        -- Get the start date for the Pro Annual plan
        SELECT 
            customer_id, 
            start_date AS annual_date
        FROM subscriptions
        WHERE plan_id = 3
    )
    SELECT 
        ROUND(AVG(annual_date - join_date), 0) AS avg_days_to_upgrade
    FROM trial_dates t
    JOIN annual_dates a ON t.customer_id = a.customer_id;
 ```

| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---


**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

 ```sql
    WITH pro_monthly AS
    (
      SELECT customer_id, start_date AS old_date
      FROM subscriptions s JOIN plans p ON s.plan_id = p.plan_id
      WHERE plan_name = 'pro monthly'
      
      ),
    basic_monthly AS (
      SELECT customer_id, start_date AS new_date
      FROM subscriptions s JOIN plans p ON s.plan_id = p.plan_id
      WHERE plan_name = 'basic monthly'
      )
    
    SELECT COUNT(*)
    FROM pro_monthly p JOIN basic_monthly b ON p.customer_id = b.customer_id AND old_date < new_date
    ;
 ```

| count |
| ----- |
| 0     |


