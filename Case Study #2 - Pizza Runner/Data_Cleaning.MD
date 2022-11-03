# ```🔄 Data Cleaning```

In Customer order table the null values defined as string we will modifty them.

```sql
SELECT ORDER_ID,CUSTOMER_ID,PIZZA_ID,
CASE
	WHEN EXCLUSIONS = 'null' OR EXCLUSIONS = '' THEN null
	else EXCLUSIONS
END AS EXCLUSIONS,
CASE 
	WHEN EXTRAS = 'null' OR EXTRAS = '' THEN null
	ELSE EXTRAS
END AS EXTRAS, 
ORDER_TIME
INTO #cleaned_customer_orders
FROM CUSTOMER_ORDERS;
```

Similarly, in runner_orders table we will modify null which are defined as string also distance and duration having unit attached to it we will remove it.

```sql
SELECT 
    order_id,
    runner_id,
    cast(CASE 
        WHEN pickup_time = 'null' THEN null
        ELSE pickup_time
    END as datetime) as pickup_time,
    cast(CASE 
        WHEN distance = 'null' THEN null
        ELSE REPLACE(distance ,'km','')
    END as float) as distance,
    cast(CASE
        WHEN duration = 'null' THEN null
        ELSE SUBSTRING(duration, 1, 2)
    END as int)as duration,
    CASE
        WHEN cancellation in ('null', '') THEN null
        ELSE cancellation
END as cancellation
INTO #cleaned_runner_orders
FROM runner_orders;
```