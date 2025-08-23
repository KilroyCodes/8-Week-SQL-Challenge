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

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
