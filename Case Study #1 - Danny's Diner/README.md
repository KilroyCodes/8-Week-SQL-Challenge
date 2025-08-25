# Case Study 1 - Danny's Diner #
Challenge link: [Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

## Background ##
Danny has opened up a new diner selling three food items: sushi, curry, and ramen.
He has collected data on his menu, customer orders, and existing members of his loyalty program.

## Relational Datasets ##
There are three datasets to work with: sales, menu, and members.

![Danny's Diner](https://github.com/KilroyCodes/8-Week-SQL-Challenge/blob/main/Danny's%20Diner%20relational%20databases.png)

## Case Study Questions & Solutions ##
Questions provided by Data with Danny. Solutions provided by me :grin:

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT
s.customer_id,
sum(m.price) as total_spent

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

GROUP BY 1;
```

| customer_id | total_spent |
| ----------- | ----------- |
| B           | 74          |
| C           | 36          |
| A           | 76          |


**2. How many days has each customer visited the restaurant?**
```sql
SELECT
customer_id,
COUNT(DISTINCT order_date) as days_visited
FROM dannys_diner.sales

GROUP BY customer_id;
```

| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |


**3. What was the first item from the menu purchased by each customer?**
```sql
WITH first_orders AS
(
SELECT
s.customer_id,
min(s.order_date) AS first_date

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

GROUP BY 1)

SELECT
first_orders.customer_id,
m.product_name

FROM first_orders
LEFT JOIN dannys_diner.sales AS s ON
s.customer_id = first_orders.customer_id AND
s.order_date = first_orders.first_date

LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

GROUP BY 1,2
ORDER BY 1;

```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
**Most purchased item**
```sql
SELECT
m.product_name,
COUNT(s.product_id)

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

GROUP BY 1
ORDER BY 2 DESC

LIMIT 1;
```
| product_name | count |
| ------------ | ----- |
| ramen        | 8     |

**How many times purchased by all customers**
```sql
SELECT
s.customer_id,
m.product_name,
COUNT(s.product_id)

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

WHERE m.product_name = 'ramen'

GROUP BY 1,2
ORDER BY 3 DESC;
```
| customer_id | product_name | count |
| ----------- | ------------ | ----- |
| A           | ramen        | 3     |
| C           | ramen        | 3     |
| B           | ramen        | 2     |


**5. Which item was the most popular for each customer?**
```sql
WITH ranking AS(
SELECT
s.customer_id,
m.product_name,
COUNT(s.product_id) AS order_count,
RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS order_rank

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON
s.product_id = m.product_id

GROUP BY 1,2)

SELECT
customer_id,
product_name,
order_count

FROM ranking
WHERE order_rank = 1;
```

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH first_orders AS(
SELECT
m.customer_id,
m.join_date,
s.order_date,
s.product_id,
RANK() OVER (PARTITION BY m.customer_id ORDER BY s.order_date ASC) AS order_rank
FROM members AS m

LEFT JOIN sales AS s ON
s.customer_id = m.customer_id

WHERE s.order_date > m.join_date

ORDER BY 3 ASC)

SELECT
f.customer_id,
f.join_date,
f.order_date,
m.product_name

FROM first_orders AS f

LEFT JOIN menu as m ON
f.product_id = m.product_id

WHERE f.order_rank = 1;
```

| customer_id | join_date  | order_date | product_name |
| ----------- | ---------- | ---------- | ------------ |
| A           | 2021-01-07 | 2021-01-10 | ramen        |
| B           | 2021-01-09 | 2021-01-11 | sushi        |

Customer C is not an active member

**7. Which item was purchased just before the customer became a member?**
```sql
WITH last_orders AS(
SELECT
m.customer_id,
m.join_date,
s.order_date,
s.product_id,
RANK() OVER (PARTITION BY m.customer_id ORDER BY s.order_date ASC) AS order_rank
FROM members AS m

LEFT JOIN sales AS s ON
s.customer_id = m.customer_id

WHERE s.order_date < m.join_date

ORDER BY 3 ASC)

SELECT
f.customer_id,
f.join_date,
f.order_date,
m.product_name

FROM last_orders AS f

LEFT JOIN menu as m ON
f.product_id = m.product_id

WHERE f.order_rank = 1;
```
| customer_id | join_date  | order_date | product_name |
| ----------- | ---------- | ---------- | ------------ |
| A           | 2021-01-07 | 2021-01-01 | sushi        |
| A           | 2021-01-07 | 2021-01-01 | curry        |
| B           | 2021-01-09 | 2021-01-01 | curry        |

**8. What is the total items and amount spent for each member before they became a member?**
```sql
WITH last_orders AS(
SELECT
m.customer_id,
m.join_date,
s.order_date,
s.product_id

FROM members AS m

LEFT JOIN sales AS s ON
s.customer_id = m.customer_id

WHERE s.order_date < m.join_date

ORDER BY 3 ASC)

SELECT
f.customer_id,
COUNT(DISTINCT m.product_name) AS items_ordered,
SUM(m.price) AS total_spent

FROM last_orders AS f

LEFT JOIN menu as m ON
f.product_id = m.product_id

GROUP BY 1;
```
| customer_id | items_ordered | total_spent |
| ----------- | ------------- | ----------- |
| A           | 2             | 25          |
| B           | 2             | 40          |

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
\n (Assuming that this question means to determine potential points earned per customer regardless of membership status. Points will be calculated based on entire customer order history)
```sql
WITH points AS(
SELECT
s.customer_id,
m.product_name,
CASE
WHEN m.product_name = 'sushi' THEN m.price*20
ELSE m.price* 10
END AS points

FROM sales AS s

LEFT JOIN menu AS m ON
s.product_id = m.product_id)

SELECT
customer_id,
SUM(points)

FROM points

GROUP BY 1
ORDER BY 1;
```
| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

