
**Schema (PostgreSQL v13)**

    CREATE SCHEMA foodie_fi;
    SET search_path = foodie_fi;
    
    CREATE TABLE plans (
      plan_id INTEGER,
      plan_name VARCHAR(13),
      price DECIMAL(5,2)
    );

    
    CREATE TABLE subscriptions (
      customer_id INTEGER,
      plan_id INTEGER,
      start_date DATE
    );

 -- B. Data Analysis Questions


 -- 1. How many customers has Foodie-Fi ever had?
 
**Query #1**

   
    SELECT COUNT(DISTINCT customer_id) AS total_num_customer
    FROM subscriptions;

| total_num_customer |
| ------------------ |
| 1000               |

---
**Query #2**

    
    
    /*2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value*/
    SELECT EXTRACT(MONTH FROM start_date) AS monthly, count(*) AS monthly_distribution
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE plan_name = 'trial'
    GROUP BY monthly
    ORDER BY monthly;

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
**Query #3**

    
    
    
    /*3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name*/
    SELECT plan_name,COUNT(*) AS events
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE EXTRACT(YEAR FROM start_date) > '2020'
    GROUP BY plan_name
    ORDER BY events;

| plan_name     | events |
| ------------- | ------ |
| basic monthly | 8      |
| pro monthly   | 60     |
| pro annual    | 63     |
| churn         | 71     |

---
**Query #4**

    
    
    
    /*4. What is the customer count and percentage of customers who have churned,rounded to 1 decimal place?*/
    SELECT COUNT(DISTINCT customer_id) AS churned_customers, 
           ROUND((COUNT(customer_id)* 100.0)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions ),1) AS churn_rate
    FROM subscriptions s JOIN plans p ON (s.plan_id = p.plan_id)
    WHERE plan_name = 'churn';

| churned_customers | churn_rate |
| ----------------- | ---------- |
| 307               | 30.7       |

---
**Query #5**

    
    
    
    /*5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?*/
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

| direct_churns | churn_percentage |
| ------------- | ---------------- |
| 92            | 9                |

---
**Query #6**

    
    
    --ver2
    WITH ranking AS (
        SELECT 
            customer_id, 
            plan_name, 
            RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS ranks
        FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
    
    )
    SELECT COUNT(*) AS trial_to_churn_count, ROUND(COUNT(*) * 100.0 / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 0) AS churn_percentage
    FROM ranking r1
    JOIN ranking r2 ON r1.customer_id = r2.customer_id
    WHERE r1.ranks = 1 AND r1.plan_name = 'trial'
      AND r2.ranks = 2 AND r2.plan_name = 'churn';

| trial_to_churn_count | churn_percentage |
| -------------------- | ---------------- |
| 92                   | 9                |

---
**Query #7**

    
    
    
    
    /*6. What is the number and percentage of customer plans after their initial free trial?*/
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

| next_plan     | next_plan_counts | next_plan_rates |
| ------------- | ---------------- | --------------- |
| basic monthly | 546              | 54.6            |
| churn         | 92               | 9.2             |
| pro annual    | 37               | 3.7             |
| pro monthly   | 325              | 32.5            |

---
**Query #8**

    
    
    
    /*7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?*/
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

| plan_id | customers | percentage |
| ------- | --------- | ---------- |
| 0       | 19        | 1.9        |
| 1       | 224       | 22.4       |
| 2       | 326       | 32.6       |
| 3       | 195       | 19.5       |
| 4       | 236       | 23.6       |

---
**Query #9**

    
    
    --8. How many customers have upgraded to an annual plan in 2020?
    SELECT 
        COUNT(DISTINCT customer_id) AS annual_upgrade_2020
    FROM subscriptions s 
        JOIN plans p ON s.plan_id = p.plan_id
    WHERE plan_name = 'pro annual' 
      AND EXTRACT(YEAR FROM start_date) = '2020';

| annual_upgrade_2020 |
| ------------------- |
| 195                 |

---
**Query #10**

    
    
    /*9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?*/
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

| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---
**Query #11**

    
    
    /*10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)*/
    
    /*11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?*/
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

| count |
| ----- |
| 0     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)
