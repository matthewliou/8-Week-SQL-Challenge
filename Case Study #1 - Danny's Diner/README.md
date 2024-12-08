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
 db as(
  SELECT 
  	*,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as rank
  FROM joined
)

SELECT 
customer_id,
product_name
FROM db
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
db as(
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
FROM db
WHERE rank = 1
```
### Question #6
Which item was purchased first by the customer after they became a member?
```
db as(
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
	db
WHERE 
	rank = 1
GROUP BY 
	customer_id,
    product_name
```
### Question #7
Which item was purchased just before the customer became a member?
```
db as(
 SELECT
 	customer_id,
 	product_name,
  	order_date,
  DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date DESC)as rank
 FROM 
  	joined
 WHERE 
  	join_date > order_date
  )


SELECT 
	customer_id,
    product_name
FROM 
	db
WHERE
	rank = 1
GROUP BY 
	customer_id,
    product_name
```
### Question #8
What is the total items and amount spent for each member before they became a member?
```
db as(
 SELECT
 	customer_id,
 	COUNT(product_name) as total_products,
  	SUM(price) as total_spend
 FROM 
  	joined
 WHERE 
  	join_date > order_date
 GROUP BY 
    customer_id
  )
SELECT 
*
from db
ORDER BY customer_id ASC
```
### Question #9
If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
```
db as(
 SELECT
 	customer_id,
    product_id,
CASE WHEN product_id = 1 THEN price*20 ELSE price*10 END as points
 FROM 
  	joined
  )
SELECT 
customer_id,
SUM(points)	
from db
GROUP BY customer_id
ORDER BY customer_id ASC
```
### Question #10
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
```
db as(
 SELECT
 	customer_id,
    product_id,
  	join_date,
  order_date,
CASE 
  WHEN 
  	order_date BETWEEN join_date AND join_date +6 THEN price *20 
  WHEN
  	product_id = 1 THEN price*20 ELSE price*10 END as points
 FROM 
  	joined
  WHERE
  	order_date < '2021-02-01'
  )

SELECT 
customer_id,
sum(points) as total_points
from db
WHERE customer_id != 'C'
GROUP BY customer_id
ORDER BY customer_id
```
### Bonus Question
Join All The Things
```
WITH joined AS(
	(SELECT s.customer_id, s.order_date, s.product_id, m.product_name, m.price, me.join_date,
     CASE WHEN
     	s.order_date >= me.join_date THEN 'Y'ELSE 'N' END as member
     FROM dannys_diner.sales as s
     LEFT JOIN dannys_diner.menu as m
     ON s.product_id = m.product_id
     LEFT JOIN dannys_diner.members as me
     ON s.customer_id = me.customer_id
    )
  )
  
 SELECT 
 	customer_id,
    order_date,
    product_name,
    price,
    member
 from joined
 ORDER BY
 customer_id,
  order_date
```
