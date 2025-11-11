# Case Study 3 - Foodie_Fi #
Challenge link: [Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)

## Background ##
Danny and his friends have started a new food video subscription service called Foodie-Fi, giving customers access to food vides from all over the world.

All customers are given access to a **7-day free trial initially**. After the trial, there are two available subscription tiers: **Basic and Pro**. Basic subscriptions are only available on a monthly basis, while Pro is available either monthly or annually (at a discounted rate). 

After the trial period, a customer is **automatically subscribed to the monthly Pro plan** (the most expensive) unless they actively go into their account to cancel or choose a different plan.

Any plan **downgrades** (e.g., Pro annual to Basic monthly) must first finish the remainder of their current plan, while any **upgrades** (e.g., Basic monthly to Pro monthly) are immediately applied.

When a customer **cancels their plan**, they must finish the remainder of their current plan, but they are now classified as a **'churn' plan** in the data with the churn start date referring to the day they decided to cancel.

## Relational Datasets ##
There are two datasets to work with: plans, and subscriptions.

![Foodie-Fi](https://github.com/KilroyCodes/8-Week-SQL-Challenge/blob/3acd6486ace8fe9a94d346346662befcd7ab58a7/Case%20Study%20%233%20-%20Foodie-Fi/Foodie-Fi%20relational%20datasets.png)

## Case Study Questions & Solutions ##
Questions provided by Data with Danny. Solutions provided by me :grin:

### A. Customer Journey ###
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.  
  
*Note: The sample customers are customer ids 1, 2, 11, 13, 15, 16, 18, and 19*

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```sql
SELECT
s.customer_id,
s.plan_id,
p.plan_name,
s.start_date

FROM foodie_fi.subscriptions s

LEFT JOIN foodie_fi.plans p ON
s.plan_id = p.plan_id

WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)

GROUP BY 1,2,3,4
ORDER BY 1,2;
```

| customer_id | plan_id | plan_name     | start_date |
| ----------- | ------- | ------------- | ---------- |
| 1           | 0       | trial         | 2020-08-01 |
| 1           | 1       | basic monthly | 2020-08-08 |
| 2           | 0       | trial         | 2020-09-20 |
| 2           | 3       | pro annual    | 2020-09-27 |
| 11          | 0       | trial         | 2020-11-19 |
| 11          | 4       | churn         | 2020-11-26 |
| 13          | 0       | trial         | 2020-12-15 |
| 13          | 1       | basic monthly | 2020-12-22 |
| 13          | 2       | pro monthly   | 2021-03-29 |
| 15          | 0       | trial         | 2020-03-17 |
| 15          | 2       | pro monthly   | 2020-03-24 |
| 15          | 4       | churn         | 2020-04-29 |
| 16          | 0       | trial         | 2020-05-31 |
| 16          | 1       | basic monthly | 2020-06-07 |
| 16          | 3       | pro annual    | 2020-10-21 |
| 18          | 0       | trial         | 2020-07-06 |
| 18          | 2       | pro monthly   | 2020-07-13 |
| 19          | 0       | trial         | 2020-06-22 |
| 19          | 2       | pro monthly   | 2020-06-29 |
| 19          | 3       | pro annual    | 2020-08-29 |

**Customer journey summaries**  
  
**1** - After the free trial period, actively chose to continue into a Basic monthly plan.  
**2** - After the free trial period, actively chose to continue into a Pro annual plan.  
**11** - Only availed of free trial.  
**13** - After the free trial period, actively chose to continue into a Basic monthly plan. Upgraded to Pro monthly after 3 months.  
**15** - After the free trial, assumed automatic conversion to Pro monthly plan. Cancelled during second month.  
**16** - After the free trial period, actively chose to continue into a Basic monthly plan. Upgraded to Pro annual after 4 months.  
**18** - After the free trial, assumed automatic conversion to Pro monthly plan.  
**19** - After the free trial, assumed automatic conversion to Pro monthly plan. Upgraded to Pro annual after 2 months.  

---  

### B. Data Analysis Questions ###
**1. How many customers has Foodie-Fi ever had?**  
```sql
SELECT
COUNT(DISTINCT s.customer_id)

FROM foodie_fi.subscriptions s;
```
  
1000 (including trial-only users)  

---  

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**  
```sql
SELECT
EXTRACT(YEAR FROM s.start_date) AS year,
EXTRACT(MONTH FROM s.start_date) AS month,
COUNT(DISTINCT s.customer_id) AS customer_count

FROM foodie_fi.subscriptions s

WHERE s.plan_id = 0 --trial plan id

GROUP BY 1,2
ORDER BY 1,2;
```
  
| year | month | customer_count |
| ---- | ----- | -------------- |
| 2020 | 1     | 88             |
| 2020 | 2     | 68             |
| 2020 | 3     | 94             |
| 2020 | 4     | 81             |
| 2020 | 5     | 88             |
| 2020 | 6     | 79             |
| 2020 | 7     | 89             |
| 2020 | 8     | 88             |
| 2020 | 9     | 87             |
| 2020 | 10    | 79             |
| 2020 | 11    | 75             |
| 2020 | 12    | 84             |

---  

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**  
```sql
SELECT
EXTRACT(YEAR FROM s.start_date) AS year,
s.plan_id,
COUNT(DISTINCT s.customer_id) AS customer_count

FROM foodie_fi.subscriptions s

WHERE EXTRACT(YEAR FROM s.start_date) > 2020

GROUP BY 1,2
ORDER BY 1,2;
```  
| year | plan_id | customer_count |
| ---- | ------- | -------------- |
| 2021 | 1       | 8              |
| 2021 | 2       | 60             |
| 2021 | 3       | 63             |
| 2021 | 4       | 71             |
  
---  

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**  
```sql
SELECT
SUM(CASE WHEN s.plan_id=4 THEN 1 ELSE 0 END) AS churn_count,

ROUND(
  100.0*SUM(CASE WHEN s.plan_id=4 THEN 1 ELSE 0 END)/COUNT(DISTINCT s.customer_id)
  ,1) || '%' AS churn_share

FROM foodie_fi.subscriptions s;
```
| churn_count | churn_share |
| ----------- | ----------- |
| 307         | 30.7%       |
  
---  

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
```sql
WITH plan_evolution AS (
SELECT
s.customer_id,
STRING_AGG(s.plan_id::TEXT, ',' ORDER BY s.plan_id ASC) AS plans

FROM foodie_fi.subscriptions s

GROUP BY 1)

SELECT
SUM(CASE WHEN pe.plans = '0,4' THEN 1 ELSE 0 END) AS count_churned_posttrial,

ROUND(
  100.0*SUM(CASE WHEN pe.plans = '0,4' THEN 1 ELSE 0 END)/
  COUNT(DISTINCT pe.customer_id)
  ,0) || '%' AS share_churned_posttrial

FROM plan_evolution pe;
```  
| count_churned_posttrial | share_churned_posttrial |
| ----------------------- | ----------------------- |
| 92                      | 9%                      |

---  

**6. What is the number and percentage of customer plans after their initial free trial?**  
```sql
WITH plan_ranking AS (
SELECT
s.customer_id,
s.plan_id,
s.start_date,
RANK() OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS plan_rank

FROM foodie_fi.subscriptions s

GROUP BY 1,2,3
ORDER BY 1)

SELECT
pr.plan_id,
p.plan_name,
COUNT(*) AS customer_count,
ROUND(
  100*COUNT(*)/SUM(COUNT(*)) OVER()
  ,1) || '%' AS customer_share

FROM plan_ranking AS pr

LEFT JOIN foodie_fi.plans p ON
p.plan_id = pr.plan_id

WHERE pr.plan_rank = 2 --plan selected after trial which is rank 1

GROUP BY 1,2
ORDER BY 1;
```  
| plan_id | plan_name     | customer_count | customer_share |
| ------- | ------------- | -------------- | -------------- |
| 1       | basic monthly | 546            | 54.6%          |
| 2       | pro monthly   | 325            | 32.5%          |
| 3       | pro annual    | 37             | 3.7%           |
| 4       | churn         | 92             | 9.2%           |
  
---  

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**  
```sql
WITH plan_ranking AS (
SELECT
s.customer_id,
s.plan_id,
s.start_date,
RANK() OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS plan_rank

FROM foodie_fi.subscriptions s
  
WHERE s.start_date <= '2020-12-31'

GROUP BY 1,2,3
ORDER BY 1)

SELECT
pr.plan_id,
p.plan_name,
COUNT(*) AS customer_count,
ROUND(
  100*COUNT(*)/SUM(COUNT(*)) OVER()
  ,1) || '%' AS customer_share

FROM plan_ranking pr

LEFT JOIN foodie_fi.plans p ON
p.plan_id = pr.plan_id

WHERE (pr.customer_id, pr.plan_rank) IN (SELECT pr.customer_id, MAX(pr.plan_rank) FROM plan_ranking pr GROUP BY 1)

GROUP BY 1,2
ORDER BY 1;
```
  
| plan_id | plan_name     | customer_count | customer_share |
| ------- | ------------- | -------------- | -------------- |
| 0       | trial         | 19             | 1.9%           |
| 1       | basic monthly | 224            | 22.4%          |
| 2       | pro monthly   | 326            | 32.6%          |
| 3       | pro annual    | 195            | 19.5%          |
| 4       | churn         | 236            | 23.6%          |
  
---  

**8. How many customers have upgraded to an annual plan in 2020?**  
```sql
SELECT
COUNT(DISTINCT s.customer_id)

FROM foodie_fi.subscriptions s

WHERE s.plan_id = 3 AND --pro annual
EXTRACT(YEAR FROM s.start_date) = 2020;
```
  
| count |
| ----- |
| 195   |

---  

**9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**  
```sql
WITH annual_upgrade AS (
SELECT
s.customer_id,
CASE WHEN s.plan_id=3 THEN s.start_date END AS upgrade_date,
CASE WHEN s.plan_id=0 THEN s.start_date END AS join_date

FROM foodie_fi.subscriptions s

GROUP BY 1,2,3
ORDER BY 1),

upgraded_customers AS (
SELECT
au.customer_id,
MAX(au.upgrade_date) - MAX(au.join_date) AS upgrade_days

FROM annual_upgrade au

GROUP BY 1

HAVING MAX(au.upgrade_date) IS NOT NULL)

SELECT
ROUND(AVG(uc.upgrade_days),0) AS avg_days_to_annual_upgrade

FROM upgraded_customers uc;
```

| avg_days_to_annual_upgrade |
| -------------------------- |
| 105                        |
  
---  

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**  
```sql
WITH annual_upgrade AS (
SELECT
s.customer_id,
CASE WHEN s.plan_id=3 THEN s.start_date END AS upgrade_date,
CASE WHEN s.plan_id=0 THEN s.start_date END AS join_date

FROM foodie_fi.subscriptions s

GROUP BY 1,2,3
ORDER BY 1),

upgraded_customers AS (
SELECT
au.customer_id,
MAX(au.upgrade_date) - MAX(au.join_date) AS upgrade_days

FROM annual_upgrade au
  
GROUP BY 1
HAVING MAX(au.upgrade_date) IS NOT NULL),

upgrade_buckets AS (
SELECT
WIDTH_BUCKET(uc.upgrade_days, 0, 365, 12),
uc.customer_id

FROM upgraded_customers uc

GROUP BY 1,2)

SELECT
((ub.width_bucket-1)*30 || ' - ' || (ub.width_bucket*30) || ' days') AS days_elapsed_from_join,
COUNT(DISTINCT ub.customer_id) AS customer_count

FROM upgrade_buckets ub

GROUP BY ub.width_bucket
ORDER BY ub.width_bucket;
```
  
| days_elapsed_from_join | customer_count |
| ---------------------- | -------------- |
| 0 - 30 days            | 49             |
| 30 - 60 days           | 24             |
| 60 - 90 days           | 35             |
| 90 - 120 days          | 35             |
| 120 - 150 days         | 43             |
| 150 - 180 days         | 37             |
| 180 - 210 days         | 24             |
| 210 - 240 days         | 4              |
| 240 - 270 days         | 4              |
| 270 - 300 days         | 1              |
| 300 - 330 days         | 1              |
| 330 - 360 days         | 1              |

---  
  
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**  
```sql
WITH plan_ranking AS (
SELECT
s.customer_id,
s.plan_id,
s.start_date,
LEAD(s.plan_id, 1) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS next_plan

FROM foodie_fi.subscriptions s

WHERE 
EXTRACT(YEAR FROM s.start_date) = 2020

GROUP BY 1,2,3
ORDER BY 1)

SELECT
COUNT(DISTINCT pr.customer_id)

FROM plan_ranking pr

WHERE 
pr.plan_id = 2 AND
pr.next_plan = 1;
```

| count |
| ----- |
| 0     |
  
---  

Thank you!
