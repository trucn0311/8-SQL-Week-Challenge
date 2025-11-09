# `Case Study #1 - Danny's Diner`
<img width="600" height="600" alt="image" src="https://github.com/user-attachments/assets/ff37b0ee-63d1-4c20-bbb5-a09756bdb9b3" />

## Introduction
Danny's Diner, a small Japanese restaurant selling sushi, curry, and ramen, needs help analyzing its initial months of sales and membership data to survive and potentially expand its loyalty program. 🍜

## 🎯 Problem Statement
The core goal is to use the provided customer data to generate business insights regarding:
* `Customer Visiting Patterns`: How often and when customers visit.
* `Spending Habits`: How much money customers have spent.
* `Favorite Menu Items`: Which products are most popular.

These insights will help Danny personalize the customer experience and inform his decision on expanding the loyalty program. Additionally, you need to provide basic processed datasets for his team's non-SQL inspection.
## Entity Relationship Diagram
<img width="672" height="339" alt="Screenshot 2025-11-09 at 12 51 46 PM" src="https://github.com/user-attachments/assets/92f7ef26-d1db-4902-97b6-95f1f65784e5" />

## Case Study Questions + Solution
**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT customer_id, SUM(price)
FROM menu LEFT JOIN sales ON (sales.product_id = menu.product_id)
GROUP BY customer_id
ORDER BY customer_id ;
```
* A spent total of 76
* B spent total of 74
* C spent total of 36

**2. How many days has each customer visited the restaurant?**

```sql
SELECT customer_id, count(distinct(sales.order_date))as no_of_visits
FROM sales 
GROUP BY customer_id;
```
* A has visited for 4 days
* B has visited for 6 days
* C has visited for 2 days


**3. What was the first item from the menu purchased by each customer?**

```sql
SELECT customer_id, product_name, order_date
FROM menu JOIN sales ON (sales.product_id=menu.product_id)
WHERE order_date = '2021-01-01'
GROUP BY customer_id,product_name, order_date
ORDER BY customer_id;
```
* A - SUSHI
* B - CURRY
* C - RAMEN

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT product_name, COUNT(*) AS amount_of_item
FROM menu JOIN sales ON (sales.product_id=menu.product_id)
GROUP BY product_name
ORDER BY amount_of_item DESC LIMIT 1;
```
* Ramen, 8 times

**5. Which item was the most popular for each customer?**

```sql
SELECT customer_id, product_name, COUNT(*) AS amount_of_item
FROM menu JOIN sales ON (sales.product_id=menu.product_id)
GROUP BY customer_id, product_name
ORDER BY amount_of_item DESC ;
```
* A enjoys ramen
* B equally enjoyed all items on the menu
* C enjoys ramen


**6. Which item was purchased first by the customer after they became a member?**

```sql
-- Customer A

SELECT customer_id, order_date, product_name 
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'A' AND order_date > '2021-01-07' 
ORDER BY order_date
LIMIT 1;
```

```sql
-- Customer B

SELECT customer_id, order_date, product_name 
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'B' AND order_date > '2021-01-09' -- date after membership
ORDER BY order_date
LIMIT 1;
```
* A - Ramen
* B - Sushi

**7. Which item was purchased just before the customer became a member?**

```sql
-- Customer A

SELECT customer_id, order_date, product_name 
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'A' AND order_date < '2021-01-07'
ORDER BY order_date DESC;
```

```sql
-- Customer B?

SELECT customer_id, order_date, product_name 
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'B' AND order_date < '2021-01-09'
ORDER BY order_date DESC
LIMIT 1;
```
* A - Sushi, Curry 
* B - Sushi

**8. What is the total items and amount spent for each member before they became a member?**

```sql
-- Customer A

SELECT customer_id, order_date, COUNT(product_name) total_items, SUM(price) amount_spent
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'A' AND order_date < '2021-01-07'
GROUP BY customer_id, order_date
ORDER BY order_date;
```

```sql
-- Customer B

SELECT customer_id, order_date, COUNT(product_name) total_items, SUM(price) amount_spent
FROM sales
LEFT JOIN menu 
  ON sales.product_id = menu.product_id
WHERE customer_id = 'B' AND order_date < '2021-01-09' -- dates before membership
GROUP BY customer_id,order_date
ORDER BY order_date;
```
* A : 3 items , $ 25 
* B : 5 items , $ 40

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
SELECT sales.customer_id, 
    SUM(
    CASE 
        WHEN menu.product_name  = 'sushi' THEN menu.price*20 
        ELSE menu.price* 10
    END) as total_point
    FROM menu JOIN sales ON (sales.product_id=menu.product_id)
    GROUP BY sales.customer_id
    ORDER BY sales.customer_id, total_point DESC;
```
* A - 860 points
* B - 940 points
* C - 360 points

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
SELECT
    s.customer_id,
    SUM(
        CASE
            WHEN s.order_date >= mb.join_date AND s.order_date <= mb.join_date + interval '6 day' THEN m.price * 20
            WHEN m.product_name = 'sushi' THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS total_point
FROM
    members AS mb
JOIN
    sales AS s ON mb.customer_id = s.customer_id
JOIN
    menu AS m ON s.product_id = m.product_id
WHERE
    s.order_date <= '2021-01-31'
    AND s.order_date >= mb.join_date
GROUP BY
    s.customer_id
ORDER BY
    s.customer_id;

```
* A - 1020 points
* B - 320 points


