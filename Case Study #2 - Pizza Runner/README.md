# Case Study 2 - Pizza Runner #
Challenge link: [Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

## Background ##
lorem ipsum

## Relational Datasets ##
There are six datasets to work with: runner_orders, runners, customer_orders, pizza_names, pizza_recipes, and pizza_toppings.

![Pizza Runner](https://github.com/KilroyCodes/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/Danny's%20Diner%20relational%20databases.png)

## Case Study Questions & Solutions ##
Questions provided by Data with Danny. Solutions provided by me :grin:

### Data Cleaning ###
Before we begin answering questions, we must clean our datasets.
We encounter null values in both the **runner_orders** and **customer_orders** tables, and additionally inconsistent data entry for distance and duration in **runner_orders**.

**customer_orders**
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
<br/>
**runner_orders**
| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

<br/>
To do this, we'll create new tables using the **CREATE TEMP TABLE** command as below:
<br/>
```sql
CREATE TEMP TABLE runner_orders_clean AS (
SELECT
order_id,
runner_id,

CASE
WHEN (pickup_time IS NULL OR pickup_time LIKE 'null') THEN ''
ELSE pickup_time
END AS pickup_time,

CASE
WHEN (duration IS NULL OR CAST(duration AS TEXT) LIKE 'null') THEN ''
ELSE TRIM('minutes' FROM duration)
END AS duration,

CASE
WHEN (distance IS NULL OR CAST(distance AS TEXT) LIKE 'null') THEN ''
ELSE TRIM('kms' FROM distance)
END AS distance,

CASE
WHEN (cancellation IS NULL OR cancellation LIKE 'null') THEN ''
ELSE cancellation
END AS cancellation

FROM runner_orders)
```
```sql
CREATE TEMP TABLE customer_orders_clean AS (
SELECT
order_id,
customer_id,
pizza_id,

CASE
WHEN (exclusions IS NULL OR exclusions LIKE 'null') THEN ''
ELSE exclusions
END AS exclusions,

CASE
WHEN (extras IS NULL OR extras LIKE 'null') THEN ''
ELSE extras
END AS extras,

order_time
FROM customer_orders)
```
---
### A. Pizza Metrics ###
**1. How many pizzas were ordered?**
```sql
SELECT
COUNT(*) AS count_pizza_orders
FROM customer_orders_clean;
```
| count_pizza_orders |
| ------------------ |
| 14                 |

---

**2. How many unique customer orders were made?**
```sql
SELECT
COUNT(DISTINCT order_id) AS count_unique_orders
FROM customer_orders_clean;
```

| count_unique_orders |
| ------------------- |
| 10                  |

---

**3. How many successful orders were delivered by each runner?**
```sql
SELECT
runner_id,
COUNT(DISTINCT order_id) AS completed_orders
  
FROM runner_orders_clean
WHERE cancellation = ''

GROUP BY 1
ORDER BY 1;
```
| runner_id | completed_orders |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

---

**4. How many of each type of pizza was delivered?**
```sql
SELECT
pn.pizza_name,
count(coc.order_id) AS count_orders

FROM customer_orders_clean AS coc

LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

LEFT JOIN pizza_names AS pn ON
pn.pizza_id = coc.pizza_id

WHERE roc.cancellation = ''

GROUP BY 1
ORDER BY 1;
```
| pizza_name | count_orders |
| ---------- | ------------ |
| Meatlovers | 9            |
| Vegetarian | 3            |

---

**5. How many Vegetarian and Meatlovers were ordered by each customer?**
```sql
SELECT
coc.customer_id,
SUM(CASE WHEN p.pizza_name = 'Meatlovers' THEN 1 ELSE 0 END) AS Meatlovers,
SUM(CASE WHEN p.pizza_name = 'Vegetarian' THEN 1 ELSE 0 END) AS Vegetarian

FROM customer_orders_clean AS coc

LEFT JOIN pizza_names AS p ON
coc.pizza_id = p.pizza_id

GROUP BY 1
ORDER BY 1;
```
| customer_id | meatlovers | vegetarian |
| ----------- | ---------- | ---------- |
| 101         | 2          | 1          |
| 102         | 2          | 1          |
| 103         | 3          | 1          |
| 104         | 3          | 0          |
| 105         | 0          | 1          |

---

**6. What was the maximum number of pizzas delivered in a single order?**
```sql
WITH pizzas_per_order AS
(SELECT
coc.order_id,
COUNT(coc.pizza_id) AS count_pizzas_delivered

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE cancellation = ''

GROUP BY 1)

SELECT
MAX(count_pizzas_delivered) AS max_pizzas_delivered
FROM pizzas_per_order;
```
| max_pizzas_delivered |
| -------------------- |
| 3                    |

---

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
```sql
SELECT
coc.customer_id,
SUM(CASE WHEN (coc.exclusions !='' OR coc.extras !='') THEN 1 ELSE 0 END) AS count_orders_with_change,
SUM(CASE WHEN (coc.exclusions ='' AND coc.extras ='') THEN 1 ELSE 0 END) AS count_orders_without_change

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE cancellation = ''

GROUP BY 1;
```
| customer_id | count_orders_with_change | count_orders_without_change |
| ----------- | ------------------------ | --------------------------- |
| 101         | 0                        | 2                           |
| 102         | 0                        | 3                           |
| 103         | 3                        | 0                           |
| 104         | 2                        | 1                           |
| 105         | 1                        | 0                           |

---

**8. How many pizzas were delivered that had both exclusions and extras?**
```sql
SELECT
SUM(CASE WHEN (coc.exclusions !='' AND coc.extras !='') THEN 1 ELSE 0 END) AS count_orders_with_both_changes

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE cancellation = '';
```
| count_orders_with_both_changes |
| ------------------------------ |
| 1                              |

---

**9. What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT
EXTRACT(HOUR FROM order_time) AS hour,
COUNT(*) AS count_pizzas_ordered

FROM customer_orders_clean

GROUP BY 1
ORDER BY 1 ASC;
```
| hour | count_pizzas_ordered |
| ---- | -------------------- |
| 11   | 1                    |
| 13   | 3                    |
| 18   | 3                    |
| 19   | 1                    |
| 21   | 3                    |
| 23   | 3                    |

---

**10. What was the volume of orders for each day of the week?**
```sql
SELECT
TO_CHAR(order_time,'Day') AS day,
COUNT(*) AS count_pizzas_ordered

FROM customer_orders_clean

GROUP BY 1
ORDER BY 1 ASC;
```
| day       | count_pizzas_ordered |
| --------- | -------------------- |
| Friday    | 1                    |
| Saturday  | 5                    |
| Thursday  | 3                    |
| Wednesday | 5                    |

---

### B. Runner and Customer Experience ###
**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
```sql
SELECT
EXTRACT(WEEK FROM registration_date) AS week,
COUNT(runner_id) as count_runners

FROM runners

GROUP BY 1;
```
| week | count_runners |
| ---- | ------------- |
| 1    | 1             |
| 53   | 2             |
| 2    | 1             |

---

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
```sql
SELECT
roc.runner_id,
DATE_TRUNC('minute', AVG(TO_TIMESTAMP(pickup_time, 'yyyy-mm-dd HH24:MI:SS') - order_time)) AS average_pickup_time

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE roc.cancellation = ''

GROUP BY 1
ORDER BY 1;
```
| runner_id | average_pickup_time |
| --------- | ------------------- |
| 1         | 00:15:00            |
| 2         | 00:23:00            |
| 3         | 00:10:00            |

---

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
```sql
SELECT
coc.order_id,
COUNT(coc.pizza_id),
DATE_TRUNC('minute', AVG(TO_TIMESTAMP(pickup_time, 'yyyy-mm-dd HH24:MI:SS') - order_time)) AS average_pickup_time

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE roc.cancellation = ''

GROUP BY 1
ORDER BY 2 DESC;
```
| order_id | count | average_pickup_time |
| -------- | ----- | ------------------- |
| 4        | 3     | 00:29:00            |
| 3        | 2     | 00:21:00            |
| 10       | 2     | 00:15:00            |
| 7        | 1     | 00:10:00            |
| 8        | 1     | 00:20:00            |
| 5        | 1     | 00:10:00            |
| 2        | 1     | 00:10:00            |
| 1        | 1     | 00:10:00            |

<br/>
The higher the number of pizzas, the longer the order takes to prepare, with each pizza requiring around 10 minutes each.

---

**4. What was the average distance travelled for each customer?**

**5. What was the difference between the longest and shortest delivery times for all orders?**

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

**7. What is the successful delivery percentage for each runner?**


### C. Ingredient Optimisation ###
1. What are the standard ingredients for each pizza?
2. What was the most commonly added extra?
3. What was the most common exclusion?
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
  * Meat Lovers
  * Meat Lovers - Exclude Beef
  * Meat Lovers - Extra Bacon
  * Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?


### D. Pricing and Ratings ###
1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
2. What if there was an additional $1 charge for any pizza extras?
  * Add cheese is $1 extra
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
