All information regarding this case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/)

## Questions and Answers
Setting up a joint table to more easily analyze data:
```WITH joined AS(
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
