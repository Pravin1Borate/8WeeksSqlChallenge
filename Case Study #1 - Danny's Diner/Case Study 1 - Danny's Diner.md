# Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

1. sales
2. menu
3. members

You can inspect the entity relationship diagram and example data below.

# DDL For Problem Statement
```sql 
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

# ER Diagram :
![image](https://user-images.githubusercontent.com/36075516/162094702-20266a74-afb8-483b-ba5a-37a676df2948.png)

❓ ```What is the total amount each customer spent at the restaurant?```
```sql
SELECT S.CUSTOMER_ID,SUM(M.PRICE) FROM MENU AS M
INNER JOIN 
SALES AS S ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY S.CUSTOMER_ID
```


❓ ```How many days has each customer visited the restaurant?```
```sql
SELECT CUSTOMER_ID,COUNT(DISTINCT(order_date)) 
FROM SALES
GROUP BY CUSTOMER_ID;
```

❓ ```What was the first item from the menu purchased by each customer?```
```sql
SELECT customer_id,product_name  FROM 
(Select S.customer_id, 
       M.product_name, 
       S.order_date,
       DENSE_RANK() OVER (PARTITION BY S.Customer_ID Order by S.order_date) as rank
From Menu m
join Sales s
On m.product_id = s.product_id
group by S.customer_id, M.product_name,S.order_date) D WHERE RANK = 1
```
❓ ```What is the most purchased item on the menu and how many times was it purchased by all customers?```
```sql
SELECT TOP 1
M.PRODUCT_NAME,COUNT(S.PRODUCT_ID) AS PURCHASE_COUNT
FROM MENU M
INNER JOIN SALES S
ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY M.PRODUCT_NAME
ORDER BY PURCHASE_COUNT DESC
```
❓ ```Which item was the most popular for each customer?```
```sql
SELECT CUSTOMER_ID,PRODUCT_NAME,PURCHASE_COUNT FROM 
(SELECT
S.CUSTOMER_ID,M.PRODUCT_NAME,COUNT(S.PRODUCT_ID) AS PURCHASE_COUNT,
DENSE_RANK() OVER (PARTITION BY S.CUSTOMER_ID ORDER BY COUNT(S.PRODUCT_ID) DESC) AS RANK
FROM MENU M
INNER JOIN SALES S
ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY S.CUSTOMER_ID,M.PRODUCT_NAME) D WHERE RANK = 1
```

❓ ```Which item was purchased first by the customer after they became a member?```
```sql
SELECT CUSTOMER_ID,PRODUCT_ID,PRODUCT_NAME FROM 
(SELECT 
ME.CUSTOMER_ID,ME.JOIN_DATE,
S.ORDER_DATE,S.PRODUCT_ID,M.PRODUCT_NAME,
DENSE_RANK() OVER (PARTITION BY ME.CUSTOMER_ID ORDER BY S.ORDER_DATE) AS RANK
FROM SALES S
INNER JOIN MENU M
ON S.PRODUCT_ID = M.PRODUCT_ID
INNER JOIN MEMBERS ME
ON S.CUSTOMER_ID = ME.CUSTOMER_ID
WHERE S.ORDER_DATE >= ME.JOIN_DATE) D WHERE RANK = 1
```
❓ ```Which item was purchased just before the customer became a member?```
```sql
SELECT CUSTOMER_ID,PRODUCT_ID,PRODUCT_NAME FROM 
(SELECT 
ME.CUSTOMER_ID,ME.JOIN_DATE,
S.ORDER_DATE,S.PRODUCT_ID,M.PRODUCT_NAME,
DENSE_RANK() OVER (PARTITION BY ME.CUSTOMER_ID ORDER BY S.ORDER_DATE) AS RANK
FROM SALES S
INNER JOIN MENU M
ON S.PRODUCT_ID = M.PRODUCT_ID
INNER JOIN MEMBERS ME
ON S.CUSTOMER_ID = ME.CUSTOMER_ID
WHERE S.ORDER_DATE <ME.JOIN_DATE) D WHERE RANK = 1;
```

❓ ```What is the total items and amount spent for each member before they became a member?```
```sql
SELECT ME.CUSTOMER_ID,COUNT(S.PRODUCT_ID) AS TOTAL_ITEMS,SUM(M.PRICE) AS AMOUNT_SPENT
FROM SALES S
INNER JOIN MENU M
ON S.PRODUCT_ID = M.PRODUCT_ID
INNER JOIN MEMBERS ME
ON S.CUSTOMER_ID = ME.CUSTOMER_ID
WHERE S.ORDER_DATE < ME.JOIN_DATE
GROUP BY ME.CUSTOMER_ID
```
❓ ```If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?```
```sql
WITH POINTS AS
(SELECT *,
CASE 
	WHEN M.PRODUCT_NAME != 'sushi' THEN M.PRICE * 10
	ELSE M.PRICE * 20
END AS POINTS_EARNED FROM MENU M)
SELECT S.CUSTOMER_ID,SUM(P.POINTS_EARNED) AS POINTS_EARNED
FROM SALES S
INNER JOIN POINTS P
ON S.PRODUCT_ID = P.PRODUCT_ID
GROUP BY S.CUSTOMER_ID;
```
❓ ```In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?```
```sql
WITH dates_cte AS(
	SELECT *, 
		DATEADD(DAY, 6, join_date) AS valid_date, 
		EOMONTH('2021-01-1') AS last_date
	FROM members
)SELECT
	s.customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		WHEN s.order_date between d.join_date and d.valid_date THEN price*20
		ELSE price*10 
	END) as total_points
FROM
	dates_cte d,
	sales s,
	menu m
WHERE
	d.customer_id = s.customer_id
	AND
	m.product_id = s.product_id
	AND
	s.order_date <= d.last_date
GROUP BY s.customer_id;
```