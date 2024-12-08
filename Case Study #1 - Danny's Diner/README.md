## Questions and Answers
Setting up a joint table to more easily analyze data:
```
WITH joined AS(
	(SELECT s.customer_id, s.order_date, s.product_id, m.product_name, m.price, me.join_date
     FROM dannys_diner.sales as s
     LEFT JOIN dannys_diner.menu as m
     ON s.product_id = m.product_id
     LEFT JOIN dannys_diner.members as me
     ON s.customer_id = me.customer_id
    )
  )
```
### Question #1
What is the total amount each customer spent at the restaurant?
```
SELECT 
  	customer_id,
    SUM(price) as total_sales
  FROM joined
  GROUP BY customer_id
  ORDER BY customer_id ASC
```
### Question #2
How many days has each customer visited the restaurant?
```
SELECT 
  	customer_id,
    COUNT (DISTINCT order_date)
  FROM joined
  GROUP BY customer_id
  ORDER BY customer_id ASC
```
### Question #3
What was the first item from the menu purchased by each customer?
```
 d as(
  SELECT 
  	*,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as rank
  FROM joined
)

SELECT 
customer_id,
product_name
FROM d
WHERE rank = 1
GROUP BY customer_id, product_name
```
### Question #4
What is the most purchased item on the menu and how many times was it purchased by all customers?
```
 SELECT
 	product_name,
    COUNT(customer_id) as count
 FROM
 	joined
 GROUP BY
 	product_name
 ORDER BY
 	count DESC
 LIMIT 1
```
### Question #5
Which item was the most popular for each customer?
```
a as(
	SELECT
	customer_id,
	product_name,
	COUNT(product_name) as count,
	DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) as rank
	FROM
 	joined
 GROUP BY
 	customer_id,
    product_name
)
SELECT
customer_id,
product_name,
count
FROM a
WHERE rank = 1
```
### Question #6
Which item was purchased first by the customer after they became a member?
```
a as(
 SELECT
 	customer_id,
 	product_name,
  	order_date,
  	DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date DESC) as rank
 FROM 
  	joined
 WHERE 
  	join_date < order_date
  )
  
SELECT 
	customer_id,
    product_name
FROM 
	a
WHERE 
	rank = 1
GROUP BY 
	customer_id,
    product_name
```
### Question #7
Which item was purchased first by the customer after they became a member?

