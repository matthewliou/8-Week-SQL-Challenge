## Questions and Answers
### Cleaning and Transforming Data
```
WITH
db_customer_orders as (
  SELECT 
  	order_id,
  	customer_id,
  	pizza_id,
  	CASE WHEN exclusions = 'null' or exclusions is NULL then '' ELSE exclusions END as exclusions,
  	CASE WHEN extras = 'null' or extras is NULL then '' ELSE extras END as extras,
  	order_time
  FROM
  	customer_orders
)
,

db_runner_orders as (
  SELECT
  	order_id,
  	runner_id,
  	CASE WHEN pickup_time = 'null' or pickup_time is NULL then '' ELSE pickup_time END as pickup_time,
  	CASE WHEN distance = 'null' or distance is NULL then '' ELSE distance END as distance,
  	CASE WHEN duration = 'null' or duration is NULL then '' ELSE duration END as duration,
  	CASE WHEN cancellation = 'null' or cancellation is NULL then '' ELSE cancellation END as cancellation
  FROM
  	runner_orders
  )
```
### Joining tables for ease of querying
```
db as (
  SELECT 
  co.order_id,
  co.customer_id,
  co.pizza_id,
  co.exclusions,
  co.extras,
  co.order_time,
  ro.runner_id,
  ro.pickup_time,
  ro.distance,
  ro.duration,
  ro.cancellation,
  pn.pizza_name,
  r.registration_date
  FROM
  	db_customer_orders co
  LEFT JOIN
	db_runner_orders ro on co.order_id = ro.order_id
  LEFT JOIN
  pizza_names pn on co.pizza_id = pn.pizza_id
  LEFT JOIN
  runners r on ro.runner_id = r.runner_id
 )
```
### Question 1: How many pizzas were ordered?
```
 SELECT 
 	COUNT(pizza_id) as orderedpizzas
 FROM db
```
Answer:

![image](https://github.com/user-attachments/assets/461c8cba-50a2-44eb-823e-d7ec38236938)

### Question 2: How many unique customer orders were made?
```
SELECT 
    count(distinct order_id) as orders
 FROM db
```
Answer:
![image](https://github.com/user-attachments/assets/2585f3ac-2ad7-4d40-92c6-5465533139cf)

### Question 3: How many successful orders were delivered by each runner?
```
SELECT 
 	runner_id,
    count(distinct order_id) as successfulorders
 FROM db
WHERE cancellation NOT LIKE '%Cancellation%'
 GROUP BY runner_id
```
Answer:
![image](https://github.com/user-attachments/assets/56f2df83-70b3-4d6d-ab92-36387255019f)

### Question 4: How many of each type of pizza was delivered?
```
 SELECT 
 	pizza_name,
    count(pizza_name) as orders
 FROM 
 	db
 WHERE 
 cancellation NOT LIKE '%Cancellation%'
 GROUP BY 
 	pizza_name
```
Answer:
![image](https://github.com/user-attachments/assets/162bdff7-9a9d-4b86-9564-50c08ac8194f)

### Question 5: How many Vegetarian and Meatlovers were ordered by each customer?

```
SELECT
 	customer_id,
    pizza_name,
    count(pizza_name) as orders
 FROM
 	db
 GROUP BY
 	customer_id,
    pizza_name
ORDER BY
	customer_id ASC
```
Answer:
![image](https://github.com/user-attachments/assets/c7f418a2-de5e-4567-94cc-1fcd75997757)

### Question 6: What was the maximum number of pizzas delivered in a single order?
```
SELECT
	order_id,
  	count(order_id) as orders
FROM
db
GROUP BY order_id
ORDER BY orders DESC
  LIMIT 1
```
Answer:
![image](https://github.com/user-attachments/assets/75ec1e26-c2ae-4ec5-bc82-c82b587f594c)

### Question 7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```
SELECT
	customer_id,
    SUM(CASE WHEN exclusions <> '' OR extras <> '' THEN 1 ELSE 0 END) as changes,
	SUM(CASE WHEN exclusions = '' AND extras = '' THEN 1 ELSE 0 END) as nochanges
FROM 
db
WHERE
cancellation NOT LIKE '%Cancellation%'
GROUP BY customer_id
ORDER BY customer_id ASC
```
Answer:
![image](https://github.com/user-attachments/assets/3fe60d66-f7d5-425e-9a67-c32d62a75895)

