# Case Study 3 - Foodie_Fi #
Challenge link: [Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)

## Background ##
Danny and his friends have started a new food video subscription service called Foodie-Fi, giving customers access to food vides from all over the world.

All customers are given access to a 7-day free trial initially. After the trial, there are two available subscription tiers: Basic and Pro. Basic subscriptions are only available on a monthly basis, while Pro is available either monthly or annually (at a discounted rate). 

After the trial period, a customer is automatically subscribed to the monthly Pro plan (the most expensive) unless they actively go into their account to cancel or choose a different plan.

Any plan downgrades (e.g., Pro annual to Basic monthly) must first finish the remainder of their current plan, while any upgrades (e.g., Basic monthly to Pro monthly) are immediately applied.

When a customer cancels their plan, they must finish the remainder of their current plan, but they are now classified as a 'churn' plan in the data.

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

---

**Customer journey summaries**
**1** - After the free trial period, actively chose to continue into a monthly plan.
**2** - After the free trial period, automatically converted to pro annual.

### B. Data Analysis Questions ###
**1. How many customers has Foodie-Fi ever had?**

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

**6. What is the number and percentage of customer plans after their initial free trial?**

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

**8. How many customers have upgraded to an annual plan in 2020?**

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
