# `Case Study #5 - Data Mart`

 <img width="600" height="600" alt="image" src="https://github.com/user-attachments/assets/593b4ab8-6cb9-4882-9f92-ea6335d01f01" />

## Introduction
Danny, the founder of the international online supermarket Data Mart, implemented a total supply chain shift in June 2020. Every product moved to 100% sustainable packaging, from the farm level to final delivery. You are tasked with analyzing how this green initiative affected the bottom line.

## 🎯 Goals
- Quantifying the Change: Calculating the exact dollar and percentage difference in sales before and after the June 2020 rollout.

- Segment Deep-Dive: Identifying which specific platforms (Retail vs. Shopify), regions, demographics, and customer types showed the highest sensitivity or negative impact.

- Strategic Forecasting: Developing a data-driven "playbook" to minimize sales disruption for future sustainability updates.

## Entity Relationship Diagram
<img width="336" height="400" alt="image" src="https://github.com/user-attachments/assets/83a08bef-a239-4350-81c7-524ff0a31b28" />

## Tables
### Table : weekly_sales

**Schema (PostgreSQL v13)**

    CREATE SCHEMA data_mart;
    SET search_path = data_mart;
    
    
    DROP TABLE IF EXISTS data_mart.weekly_sales;
    CREATE TABLE data_mart.weekly_sales (
      "week_date" VARCHAR(7),
      "region" VARCHAR(13),
      "platform" VARCHAR(7),
      "segment" VARCHAR(4),
      "customer_type" VARCHAR(8),
      "transactions" INTEGER,
      "sales" INTEGER
    );


 
 [View more on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/558)

---

## 1. Data Cleansing Steps

- Convert the `week_date` to a DATE format

- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column

- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called `age_band` after the original segment column using the following mapping on the number inside the segment value
  
    <img width="244" height="183" alt="Screenshot 2026-01-20 at 3 13 28 PM" src="https://github.com/user-attachments/assets/279b1a5f-7140-4c8c-80f8-cdb3b514e979" />

- Add a new `demographic` column using the following mapping for the first letter in the segment values:
  <img width="243" height="139" alt="Screenshot 2026-01-20 at 3 14 18 PM" src="https://github.com/user-attachments/assets/f67ed401-7c65-4371-99e9-bc4fd0a0be0e" />

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the sales value divided by transactions rounded to 2 decimal places for each record

```sql
    CREATE TABLE data_mart.clean_weekly_sales AS
    WITH base_data AS (
      SELECT *, 
        -- Use TO_DATE for PostgreSQL
        TO_DATE(week_date, 'DD/MM/YY') AS formatted_date
      FROM data_mart.weekly_sales
    )
    SELECT
      formatted_date AS week_date,
      -- Postgres uses EXTRACT or DATE_PART for these:
      EXTRACT(WEEK FROM formatted_date) AS week_number,
      EXTRACT(MONTH FROM formatted_date) AS month_number,
      EXTRACT(YEAR FROM formatted_date) AS calendar_year,
      region,
      platform,
      CASE WHEN segment = 'null' THEN 'unknown' ELSE segment END AS segment,
      CASE 
        WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
        WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
        WHEN RIGHT(segment, 1) = '3' THEN 'Retirees'
        WHEN RIGHT(segment, 1) = '4' THEN 'Retirees'
        ELSE 'unknown' 
      END AS age_band,
      CASE
        WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
        WHEN LEFT(segment, 1) = 'F' THEN 'Families'
        ELSE 'unknown'
      END AS demographic,
      customer_type,
      transactions,
      sales,
      ROUND(sales::NUMERIC / transactions, 2) AS avg_transaction
    FROM base_data;
```

---

    
## 2. Data Exploration
    
**What day of the week is used for each week_date value?**

```sql
    SELECT DISTINCT TO_CHAR(week_date, 'FMDay') AS day_name
    FROM clean_weekly_sales
    GROUP BY week_date;
```

| day_name |
| -------- |
| Monday   |

---

**What range of week numbers are missing from the dataset?**

```sql
    WITH master_weeks AS (
      SELECT * FROM GENERATE_SERIES(1, 52) AS week_no
    )
    
    SELECT 
      m.week_no AS missing_weeks
    FROM master_weeks m
    LEFT JOIN clean_weekly_sales s 
      ON m.week_no = s.week_number
    WHERE s.week_number IS NULL
    ORDER BY m.week_no;
```

| missing_weeks |
| ------------- |
| 1             |
| 2             |
| 3             |
| 4             |
| 5             |
| 6             |
| 7             |
| 8             |
| 9             |
| 10            |
| 11            |
| 12            |
| 37            |
| 38            |
| 39            |
| 40            |
| 41            |
| 42            |
| 43            |
| 44            |
| 45            |
| 46            |
| 47            |
| 48            |
| 49            |
| 50            |
| 51            |
| 52            |

---

**How many total transactions were there for each year in the dataset?**

```sql
    SELECT calendar_year, SUM(transactions)
    FROM clean_weekly_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;
```

| calendar_year | sum       |
| ------------- | --------- |
| 2018          | 346406460 |
| 2019          | 365639285 |
| 2020          | 375813651 |

---

**What is the total sales for each region for each month?**

```sql
    SELECT region, month_number, SUM(sales)
    FROM clean_weekly_sales
    GROUP BY region, month_number
    ORDER BY region,month_number;
```

| region        | month_number | sum        |
| ------------- | ------------ | ---------- |
| AFRICA        | 3            | 567767480  |
| AFRICA        | 4            | 1911783504 |
| AFRICA        | 5            | 1647244738 |
| AFRICA        | 6            | 1767559760 |
| AFRICA        | 7            | 1960219710 |
| AFRICA        | 8            | 1809596890 |
| AFRICA        | 9            | 276320987  |
| ASIA          | 3            | 529770793  |
| ASIA          | 4            | 1804628707 |
| ASIA          | 5            | 1526285399 |
| ASIA          | 6            | 1619482889 |
| ASIA          | 7            | 1768844756 |
| ASIA          | 8            | 1663320609 |
| ASIA          | 9            | 252836807  |
| CANADA        | 3            | 144634329  |
| CANADA        | 4            | 484552594  |
| CANADA        | 5            | 412378365  |
| CANADA        | 6            | 443846698  |
| CANADA        | 7            | 477134947  |
| CANADA        | 8            | 447073019  |
| CANADA        | 9            | 69067959   |
| EUROPE        | 3            | 35337093   |
| EUROPE        | 4            | 127334255  |
| EUROPE        | 5            | 109338389  |
| EUROPE        | 6            | 122813826  |
| EUROPE        | 7            | 136757466  |
| EUROPE        | 8            | 122102995  |
| EUROPE        | 9            | 18877433   |
| OCEANIA       | 3            | 783282888  |
| OCEANIA       | 4            | 2599767620 |
| OCEANIA       | 5            | 2215657304 |
| OCEANIA       | 6            | 2371884744 |
| OCEANIA       | 7            | 2563459400 |
| OCEANIA       | 8            | 2432313652 |
| OCEANIA       | 9            | 372465518  |
| SOUTH AMERICA | 3            | 71023109   |
| SOUTH AMERICA | 4            | 238451531  |
| SOUTH AMERICA | 5            | 201391809  |
| SOUTH AMERICA | 6            | 218247455  |
| SOUTH AMERICA | 7            | 235582776  |
| SOUTH AMERICA | 8            | 221166052  |
| SOUTH AMERICA | 9            | 34175583   |
| USA           | 3            | 225353043  |
| USA           | 4            | 759786323  |
| USA           | 5            | 655967121  |
| USA           | 6            | 703878990  |
| USA           | 7            | 760331754  |
| USA           | 8            | 712002790  |
| USA           | 9            | 110532368  |

---

**What is the total count of transactions for each platform**

```sql
    SELECT platform, sum(transactions) as total_transaction
    FROM clean_weekly_sales
    GROUP BY platform;
```

| platform | total_transaction |
| -------- | ----------------- |
| Shopify  | 5925169           |
| Retail   | 1081934227        |

---

**What is the percentage of sales for Retail vs Shopify for each month?**

```sql
    SELECT calendar_year, month_number, 
    	round((100.0 *SUM(CASE WHEN platform = 'Shopify' THEN sales END)/SUM(sales)),2) as shopify_sale, 
    	round((100.0 *SUM(CASE WHEN platform = 'Retail' THEN sales END)/ SUM(sales)),2) as retail_sale
    FROM clean_weekly_sales
    GROUP BY month_number, calendar_year
    ORDER BY calendar_year,month_number;
```

| calendar_year | month_number | shopify_sale | retail_sale |
| ------------- | ------------ | ------------ | ----------- |
| 2018          | 3            | 2.08         | 97.92       |
| 2018          | 4            | 2.07         | 97.93       |
| 2018          | 5            | 2.27         | 97.73       |
| 2018          | 6            | 2.24         | 97.76       |
| 2018          | 7            | 2.25         | 97.75       |
| 2018          | 8            | 2.29         | 97.71       |
| 2018          | 9            | 2.32         | 97.68       |
| 2019          | 3            | 2.29         | 97.71       |
| 2019          | 4            | 2.20         | 97.80       |
| 2019          | 5            | 2.48         | 97.52       |
| 2019          | 6            | 2.58         | 97.42       |
| 2019          | 7            | 2.65         | 97.35       |
| 2019          | 8            | 2.79         | 97.21       |
| 2019          | 9            | 2.91         | 97.09       |
| 2020          | 3            | 2.70         | 97.30       |
| 2020          | 4            | 3.04         | 96.96       |
| 2020          | 5            | 3.29         | 96.71       |
| 2020          | 6            | 3.20         | 96.80       |
| 2020          | 7            | 3.33         | 96.67       |
| 2020          | 8            | 3.49         | 96.51       |

---

**What is the percentage of sales by demographic for each year in the dataset?**

```sql
    SELECT calendar_year,
    	round((100.0 *SUM(CASE WHEN demographic = 'Couples' THEN sales END)/SUM(sales)),2) AS couples_percentage, 
    	round((100.0 *SUM(CASE WHEN demographic = 'Families' THEN sales END)/ SUM(sales)),2) AS families_percentage,
        round((100.0 *SUM(CASE WHEN demographic = 'unknown' THEN sales END)/ SUM(sales)),2) AS unknown_percentage
    FROM clean_weekly_sales
    GROUP BY  calendar_year
    ORDER BY calendar_year;
```

| calendar_year | couples_percentage | families_percentage | unknown_percentage |
| ------------- | ------------------ | ------------------- | ------------------ |
| 2018          | 26.38              | 31.99               | 41.63              |
| 2019          | 27.28              | 32.47               | 40.25              |
| 2020          | 28.72              | 32.73               | 38.55              |

---

**Which age_band and demographic values contribute the most to Retail sales?**

```sql
    SELECT 
        demographic, 
        age_band, 
        SUM(sales) AS total_sales,
        ROUND(100.0 * SUM(sales) / SUM(SUM(sales)) OVER (), 2) AS demographic_percentage
    FROM clean_weekly_sales
    WHERE platform = 'Retail'
    GROUP BY demographic, age_band
    ORDER BY total_sales DESC;
```

| demographic | age_band     | total_sales | demographic_percentage |
| ----------- | ------------ | ----------- | ---------------------- |
| unknown     | unknown      | 16067285533 | 40.52                  |
| Families    | Retirees     | 6634686916  | 16.73                  |
| Couples     | Retirees     | 6370580014  | 16.07                  |
| Families    | Middle Aged  | 4354091554  | 10.98                  |
| Couples     | Young Adults | 2602922797  | 6.56                   |
| Couples     | Middle Aged  | 1854160330  | 4.68                   |
| Families    | Young Adults | 1770889293  | 4.47                   |

- The **unknown age_band** and **demographic** have highest retail sales accounting for 42% . Followed by **retired families** at 16.73% and retired couples at 16.07%.
---

**Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**
    
- Because `avg_transaction` is already an average per row. Averaging those averages is wrong because it ignores transaction volume (a week with 10 sales counts the same as a week with 1,000).
  
- The correct method is to sum **total sales** and **total transactions**, then divide. This properly weights high-volume periods (like Christmas) and gives a fair yearly comparison.Using this method, Shopify will likely have a higher average transaction than Retail because online shoppers build larger carts.
- The `avg_transaction` column should only be used for row-level filtering(Ex: Show me all weeks where the average transaction was over $50).

```sql
    SELECT 
        calendar_year,
        platform,
        SUM(sales) AS total_sales,
        SUM(transactions) AS total_transactions,
        ROUND(SUM(sales) / SUM(transactions), 2) AS avg_transaction_size
    FROM clean_weekly_sales
    WHERE platform IN ('Retail', 'Shopify')
    GROUP BY calendar_year, platform
    ORDER BY calendar_year, platform;
```

| calendar_year | platform | total_sales | total_transactions | avg_transaction_size |
| ------------- | -------- | ----------- | ------------------ | -------------------- |
| 2018          | Retail   | 12611171318 | 344919513          | 36.00                |
| 2018          | Shopify  | 286209509   | 1486947            | 192.00               |
| 2019          | Retail   | 13397806727 | 363740159          | 36.00                |
| 2019          | Shopify  | 348225773   | 1899126            | 183.00               |
| 2020          | Retail   | 13645638392 | 373274555          | 36.00                |
| 2020          | Shopify  | 454582508   | 2539096            | 179.00               |

---

## 3. Before & After Analysis

**What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?**

```sql
    --Find out which week number is June 15th:
    SELECT DISTINCT week_number
    FROM clean_weekly_sales
    WHERE week_date = '2020-06-15';
```

| week_number |
| ----------- |
| 25          |


```sql    
    WITH sales_summary AS (
        SELECT 
            SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN sales ELSE 0 END) AS sales_before,
            SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE 0 END) AS sales_after
        FROM clean_weekly_sales
        WHERE calendar_year = 2020 
    
    )
    SELECT 
        sales_before,
        sales_after,
        (sales_after - sales_before) AS sale_difference,
        ROUND(100.0 * (sales_after - sales_before) / sales_before, 2) AS growth_rate
    FROM sales_summary;
```

| sales_before | sales_after | sale_difference | growth_rate |
| ------------ | ----------- | --------------- | ----------- |
| 2345878357   | 2318994169  | -26884188       | -1.15       |

 
- The transition to sustainable packaging coincided with a $26.88 million revenue contraction, representing a 1.15% decline in sales performance. This suggests that while the initiative meets sustainability goals, it may have inadvertently triggered a 'brand recognition gap.' The visual departure from our legacy packaging likely made the product less discoverable on-shelf, leading to reduced purchase frequency

--- 

**What about the entire 12 weeks before and after?**

 ```sql
    WITH sales_summary AS (
        SELECT 
            SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS sales_before,
            SUM(CASE WHEN week_number BETWEEN 25 AND 37 THEN sales ELSE 0 END) AS sales_after
        FROM clean_weekly_sales
        WHERE calendar_year = 2020 
    )
    SELECT 
        sales_before,
        sales_after,
        (sales_after - sales_before) AS sale_difference,
        ROUND(100.0 * (sales_after - sales_before) / sales_before, 2) AS growth_rate
    FROM sales_summary;
```

| sales_before | sales_after | sale_difference | growth_rate |
| ------------ | ----------- | --------------- | ----------- |
| 7126273147   | 6973947753  | -152325394      | -2.14       |

---

**How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?**
    
```sql    
    WITH yearly_comparison AS (
        SELECT 
            calendar_year,
            SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN sales ELSE 0 END) AS sales_before,
            SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE 0 END) AS sales_after
        FROM clean_weekly_sales
        WHERE calendar_year IN (2018, 2019, 2020)
        GROUP BY calendar_year
    )
    SELECT 
        calendar_year,
        sales_before,
        sales_after,
        (sales_after - sales_before) AS total_diff,
        ROUND(100.0 * (sales_after - sales_before) / sales_before, 2) AS growth_rate
    FROM yearly_comparison
    ORDER BY calendar_year;
```
| calendar_year | sales_before | sales_after | total_diff | growth_rate |
| ------------- | ------------ | ----------- | ---------- | ----------- |
| 2018          | 2125140809   | 2129242914  | 4102105    | 0.19        |
| 2019          | 2249989796   | 2252326390  | 2336594    | 0.10        |
| 2020          | 2345878357   | 2318994169  | -26884188  | -1.15       |


- The "Reluctant" Trend: In 2018 and 2019, the business saw a small "natural" lift of around 0.1% during this time of year.
    
- The 2020 Performance: Instead of seeing that natural lift, 2020 saw a drop of 1.15%. This suggests that the changes made around June 15, 2020, likely had a negative impact on sales, or failed to capture the seasonal growth seen in prior years.

- Scale of Impact: A $26.8 million loss in a 4-week period is a significant shift compared to the million-dollar gains in previous years.
 
    
---
    
## 4. Bonus Question
    
**Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?**
    
- region
- platform
- age_band
- demographic
- customer_type

```sql   
    WITH sales_summary AS (
        SELECT region, platform,age_band, demographic, customer_type,
           SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS sales_before,
            SUM(CASE WHEN week_number BETWEEN 25 AND 37 THEN sales ELSE 0 END) AS sales_after
        FROM clean_weekly_sales
        WHERE calendar_year = 2020
        GROUP BY region, platform,age_band, demographic, customer_type
    )
    SELECT region, platform, age_band, demographic, customer_type,
        sales_before,
        sales_after,
        (sales_after - sales_before) AS sale_difference,
        ROUND(100.0 * (sales_after - sales_before) / sales_before, 2) AS growth_rate
    FROM sales_summary
    ORDER BY growth_rate ASC LIMIT 10;
```

| region        | platform | age_band     | demographic | customer_type | sales_before | sales_after | sale_difference | growth_rate |
| ------------- | -------- | ------------ | ----------- | ------------- | ------------ | ----------- | --------------- | ----------- |
| SOUTH AMERICA | Shopify  | unknown      | unknown     | Existing      | 11785        | 6808        | -4977           | -42.23      |
| EUROPE        | Shopify  | Retirees     | Families    | New           | 7292         | 4834        | -2458           | -33.71      |
| EUROPE        | Shopify  | Young Adults | Families    | New           | 15863        | 11426       | -4437           | -27.97      |
| SOUTH AMERICA | Retail   | unknown      | unknown     | Existing      | 127781       | 98131       | -29650          | -23.20      |
| SOUTH AMERICA | Retail   | Retirees     | Families    | New           | 168495       | 132639      | -35856          | -21.28      |
| SOUTH AMERICA | Shopify  | Middle Aged  | Families    | New           | 13382        | 10742       | -2640           | -19.73      |
| SOUTH AMERICA | Shopify  | Retirees     | Families    | New           | 7672         | 6211        | -1461           | -19.04      |
| SOUTH AMERICA | Shopify  | Retirees     | Couples     | New           | 61268        | 50001       | -11267          | -18.39      |
| SOUTH AMERICA | Retail   | Retirees     | Couples     | Existing      | 1188794      | 989082      | -199712         | -16.80      |
| SOUTH AMERICA | Retail   | Retirees     | Couples     | New           | 627163       | 527702      | -99461          | -15.86      |




- **By Region - South America**: South America is the primary driver of the decline. Retail in South America lost the most actual volume (-29,650). Shopify in South America had the most severe percentage crash (-42.23%).

- **By Platform - Retail** (High Volume Loss): The single line for South America Retail shows a loss of 29,650, which is more than the combined losses of all the Shopify segments you listed.

- **By Age Band - "Unknown" & "Young Adults"**: Unknown group in South America is responsible for the largest total loss. Young Adults in Europe saw a significant -27.97% drop.

- **By Demographic - "Families"**: The Families demographic in Europe declines between -28% and -34%. This suggests that whatever change occurred on June 15th specifically alienated family shoppers in the European market.

  
