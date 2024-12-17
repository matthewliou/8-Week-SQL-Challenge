## Questions and Answers
# Cleaning and Transforming Data
```
WITH
db_customer_orders as (
  SELECT 
  	order_id,
  	customer_id,
  	pizza_id,
  	CASE WHEN exclusions = 'null' or exclusions is NULL then ' ' ELSE exclusions END as exclusions,
  	CASE WHEN extras = 'null' or extras is NULL then ' ' ELSE extras END as extras,
  	order_time
  FROM
  	customer_orders
)
,

db_runner_orders as (
  SELECT
  	order_id,
  	runner_id,
  	CASE WHEN pickup_time = 'null' or pickup_time is NULL then ' ' ELSE pickup_time END as pickup_time,
  	CASE WHEN distance = 'null' or distance is NULL then ' ' ELSE distance END as distance,
  	CASE WHEN duration = 'null' or duration is NULL then ' ' ELSE duration END as duration,
  	CASE WHEN cancellation = 'null' or cancellation is NULL then ' ' ELSE cancellation END as cancellation
  FROM
  	runner_orders
  )
```

