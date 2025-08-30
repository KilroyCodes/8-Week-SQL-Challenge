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

To do this, we'll create new tables using the **CREATE TEMP TABLE** command as below. <br/>
We're also adding in a new row to **customer_orders_clean** called **row_id** so that each pizza in each order has a unique identifier.

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
ROW_NUMBER () OVER (ORDER BY order_id) AS row_id,
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
```sql
SELECT
coc.customer_id,
ROUND(AVG(CAST(roc.distance AS FLOAT))) AS avg_distance_km

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE roc.cancellation = ''

GROUP BY 1
ORDER BY 1 ASC;
```
| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20              |
| 102         | 17              |
| 103         | 23              |
| 104         | 10              |
| 105         | 25              |

---

**5. What was the difference between the longest and shortest delivery times for all orders?**
```sql
SELECT
MAX(roc.duration) AS max_delivery_time_min,
MIN(roc.duration) AS min_delivery_time_min,
CAST(MAX(roc.duration) AS INT) - CAST(MIN(roc.duration) AS INT) AS max_min_duration_diff

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE roc.cancellation = '';
```
| max_delivery_time_min | min_delivery_time_min | max_min_duration_diff |
| --------------------- | --------------------- | --------------------- |
| 40                    | 10                    | 30                    |

---

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
```sql
SELECT
roc.runner_id,
coc.customer_id,
roc.order_id,
COUNT(coc.pizza_id) AS pizza_count,
ROUND(CAST(
  CAST(roc.distance AS FLOAT)/
  (CAST(roc.duration AS FLOAT)/60) AS NUMERIC), 2) AS delivery_speed_kmph

FROM customer_orders_clean AS coc
LEFT JOIN runner_orders_clean AS roc ON
coc.order_id = roc.order_id

WHERE roc.cancellation = ''

GROUP BY 1,2,3,5
ORDER BY 1 ASC;
```
| runner_id | customer_id | order_id | pizza_count | delivery_speed_kmph |
| --------- | ----------- | -------- | ----------- | ------------------- |
| 1         | 101         | 1        | 1           | 37.50               |
| 1         | 101         | 2        | 1           | 44.44               |
| 1         | 102         | 3        | 2           | 40.20               |
| 1         | 104         | 10       | 2           | 60.00               |
| 2         | 102         | 8        | 1           | 93.60               |
| 2         | 103         | 4        | 3           | 35.10               |
| 2         | 105         | 7        | 1           | 60.00               |
| 3         | 104         | 5        | 1           | 40.00               |

<br/>

**Runner 1**
* Mostly maintains around 40 km/h, save for customer 104. It is possible the route to customer 104 involves a highway with a higher speed limit

**Runner 2**
* Travelled over 90 km/h for customer 102 where Runner 1 travelled at 40 km/h. This significant difference should be investigated
* Has the most variable speed among the runners

**Runner 3**
* Has completed only one delivery at 40 km/h

---

**7. What is the successful delivery percentage for each runner?**
```sql
SELECT
runner_id,
ROUND((SUM(CASE WHEN cancellation='' THEN 1.0 ELSE 0 END)) / COUNT(*), 2) AS delivery_success_ratio

FROM runner_orders_clean

GROUP BY 1
ORDER BY 1;
```
| runner_id | delivery_success_ratio |
| --------- | ---------------------- |
| 1         | 1.00                   |
| 2         | 0.75                   |
| 3         | 0.50                   |

---

### C. Ingredient Optimisation ###
**1. What are the standard ingredients for each pizza?**
```sql
WITH toppings AS (
SELECT
pizza_id,
CAST(UNNEST(STRING_TO_ARRAY(toppings,',')) AS INT) AS toppings
FROM pizza_recipes)

SELECT
DISTINCT(t.toppings),
pt.topping_name

FROM toppings AS t
  
JOIN toppings AS t2 ON
t.toppings = t2.toppings AND
t.pizza_id <> t2.pizza_id

JOIN pizza_toppings AS pt ON
pt.topping_id = t.toppings

WHERE t.toppings = t2.toppings;
```
<br/>

The toppings both pizzas share in common are:

| toppings | topping_name |
| -------- | ------------ |
| 6        | Mushrooms    |
| 4        | Cheese       |

---

**2. What was the most commonly added extra?**
```sql
WITH extras_count AS (
SELECT
CAST(UNNEST(STRING_TO_ARRAY(coc.extras, ',')) AS INT) AS topping_id,
COUNT(*) AS order_count
  
FROM customer_orders_clean AS coc

WHERE coc.extras <> ''

GROUP BY 1)

SELECT
ec.topping_id,
pt.topping_name,
ec.order_count

FROM extras_count AS ec
JOIN pizza_toppings AS pt ON
pt.topping_id = ec.topping_id

ORDER BY 3 DESC;
```
| topping_id | topping_name | order_count |
| ---------- | ------------ | ----------- |
| 1          | Bacon        | 4           |
| 4          | Cheese       | 1           |
| 5          | Chicken      | 1           |

<br/>

Bacon is the most commonly added extra

---

**3. What was the most common exclusion?**
```sql
WITH exclusion_count AS (
SELECT
CAST(UNNEST(STRING_TO_ARRAY(coc.exclusions, ',')) AS INT) AS topping_id,
COUNT(*) AS order_count

FROM customer_orders_clean AS coc

WHERE coc.exclusions <> ''

GROUP BY 1)

SELECT
ec.topping_id,
pt.topping_name,
ec.order_count

FROM exclusion_count AS ec
JOIN pizza_toppings AS pt ON
pt.topping_id = ec.topping_id

ORDER BY 3 DESC;
```
| topping_id | topping_name | order_count |
| ---------- | ------------ | ----------- |
| 4          | Cheese       | 4           |
| 2          | BBQ Sauce    | 1           |
| 6          | Mushrooms    | 1           |

The most common exclusion is cheese.

---

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:**
  * Meat Lovers
  * Meat Lovers - Exclude Beef
  * Meat Lovers - Extra Bacon
  * Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

*(I'm unsure what this question is asking)*

**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients** <br/>
*For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"*
<br/><br/>
We'll first create a new temp table to separate each default pizza's toppings into separate rows
```sql
CREATE TEMP TABLE toppings AS (
SELECT
pizza_id,
CAST(
UNNEST(STRING_TO_ARRAY(toppings,',')) AS INT) AS toppings
FROM pizza_recipes);
```
```sql
WITH toppings_expanded AS (
SELECT
coc.row_id,
coc.order_id,
coc.pizza_id,
t.toppings,
extras.extras,
exclusions.exclusions

FROM customer_orders_clean AS coc
LEFT JOIN LATERAL (SELECT CAST(UNNEST(STRING_TO_ARRAY(coc.extras, ',')) AS INT) AS extras) extras ON TRUE

LEFT JOIN LATERAL (SELECT CAST(UNNEST(STRING_TO_ARRAY(coc.exclusions, ',')) AS INT) AS exclusions) exclusions ON TRUE

JOIN toppings AS t ON
t.pizza_id = coc.pizza_id

JOIN pizza_toppings AS pt ON
pt.topping_id = t.toppings

GROUP BY 1,2,3,4,5,6
ORDER BY 1,2),

order_expanded AS (
SELECT
DISTINCT *,
CASE 
WHEN te.toppings = te.exclusions THEN 0
WHEN te.toppings = te.extras THEN 2
ELSE 1 END AS toppings_count

FROM toppings_expanded AS te
JOIN pizza_toppings AS pt ON
pt.topping_id = te.toppings

JOIN pizza_names AS pn ON
pn.pizza_id = te.pizza_id),

order_toppings AS (
SELECT
oe.order_id,
oe.row_id,
oe.pizza_name,
CONCAT(oe.topping_name, ' x', MAX(oe.toppings_count)) AS topping
         
FROM order_expanded AS oe

WHERE oe.toppings_count > 0

GROUP BY 1,2,3, oe.topping_name
ORDER BY 4 ASC)

SELECT
order_id,
pizza_name,
STRING_AGG(DISTINCT(topping), ', ' ORDER BY topping ASC) AS toppings

FROM order_toppings

GROUP BY 1,2, row_id
ORDER BY 1;
```
| order_id | pizza_name | toppings                                                                                      |
| -------- | ---------- | --------------------------------------------------------------------------------------------- |
| 1        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 2        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 3        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 3        | Vegetarian | Cheese x1, Mushrooms x1, Onions x1, Peppers x1, Tomato Sauce x1, Tomatoes x1                  |
| 4        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1            |
| 4        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1            |
| 4        | Vegetarian | Mushrooms x1, Onions x1, Peppers x1, Tomato Sauce x1, Tomatoes x1                             |
| 5        | Meatlovers | BBQ Sauce x1, Bacon x2, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 6        | Vegetarian | Cheese x1, Mushrooms x1, Onions x1, Peppers x1, Tomato Sauce x1, Tomatoes x1                  |
| 7        | Vegetarian | Cheese x1, Mushrooms x1, Onions x1, Peppers x1, Tomato Sauce x1, Tomatoes x1                  |
| 8        | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 9        | Meatlovers | BBQ Sauce x1, Bacon x2, Beef x1, Chicken x2, Mushrooms x1, Pepperoni x1, Salami x1            |
| 10       | Meatlovers | BBQ Sauce x1, Bacon x1, Beef x1, Cheese x1, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |
| 10       | Meatlovers | BBQ Sauce x1, Bacon x2, Beef x1, Cheese x2, Chicken x1, Mushrooms x1, Pepperoni x1, Salami x1 |

We can now see which toppings are required for each pizza order, taking into account extras (denoted as 2x) and exclusions (removed from toppings required list).

---

**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
```sql
WITH order_expanded AS (

WITH toppings_expanded AS (
  
WITH toppings AS (
SELECT
pizza_id,
CAST(
UNNEST(STRING_TO_ARRAY(toppings,',')) AS INT) AS toppings
FROM pizza_recipes)
  
SELECT
coc.order_id,
coc.pizza_id,
COUNT(*) AS count_order,
t.toppings,
CAST(UNNEST(STRING_TO_ARRAY(coc.extras, ',')) AS INT) AS extras,
CAST(UNNEST(STRING_TO_ARRAY(coc.exclusions, ',')) AS INT) AS exclusions

FROM customer_orders_clean AS coc
JOIN toppings AS t ON
t.pizza_id = coc.pizza_id

JOIN pizza_toppings AS pt ON
pt.topping_id = t.toppings

GROUP BY 1,2,4,5,6
ORDER BY 1,2)

SELECT
te.order_id,
te.pizza_id,
pn.pizza_name,
te.count_order,
te.toppings,
pt.topping_name,
te.extras,
te.exclusions,
CASE 
WHEN te.toppings = te.exclusions THEN 0
WHEN te.toppings = te.extras THEN 2
ELSE 1 END AS toppings_count

FROM toppings_expanded AS te
JOIN pizza_toppings AS pt ON
pt.topping_id = te.toppings

JOIN pizza_names AS pn ON
pn.pizza_id = te.pizza_id

GROUP BY 1,2,3,4,5,6,7,8)

SELECT
oe.topping_name,
SUM(oe.toppings_count) AS total_quantity

FROM order_expanded AS oe
JOIN runner_orders_clean AS roc ON
roc.order_id = oe.order_id

WHERE roc.cancellation = ''

GROUP BY 1
ORDER BY 2 DESC;
```
| topping_name | total_quantity |
| ------------ | -------------- |
| Bacon        | 6              |
| Cheese       | 5              |
| Mushrooms    | 5              |
| Pepperoni    | 4              |
| Salami       | 4              |
| Chicken      | 4              |
| Beef         | 4              |
| BBQ Sauce    | 3              |
| Tomatoes     | 2              |
| Onions       | 2              |
| Peppers      | 2              |
| Tomato Sauce | 2              |

Bacon is the most used topping throughout all delivered pizza orders, and tomato sauce the

---

### D. Pricing and Ratings ###
**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

**2. What if there was an additional $1 charge for any pizza extras?**
  * Add cheese is $1 extra

**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**
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

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**
