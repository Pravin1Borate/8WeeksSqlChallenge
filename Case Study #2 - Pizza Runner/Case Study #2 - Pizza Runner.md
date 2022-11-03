##                  ``` A. Pizza Metrics```

❓ ```How many pizzas were ordered?```
```sql
SELECT COUNT(pizza_id) as total_pizza_ordered FROM customer_orders;
```

❓ ```How many unique customer orders were made?```
```sql
SELECT COUNT(DISTINCT(customer_id)) as DISTINCT_CUSTOMER_ORDERS FROM customer_orders;
```
❓ ```How many successful orders were delivered by each runner?```
```sql
SELECT runner_id,
COUNT(ORDER_ID)
FROM #cleaned_runner_orders
WHERE pickup_time IS NOT NULL and distance IS NOT NULL and duration  IS NOT NULL
group by runner_id;
```
How many of each type of pizza was delivered?
How many Vegetarian and Meatlovers were ordered by each customer?
What was the maximum number of pizzas delivered in a single order?
For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
How many pizzas were delivered that had both exclusions and extras?
What was the total volume of pizzas ordered for each hour of the day?
What was the volume of orders for each day of the week?