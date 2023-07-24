# Question

Which item was purchased just before the customer became a member?


## Solution

```sql
WITH orders_by_customer_before_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date < m.join_date
),
orders_by_customer_ranked_by_date_before_membership AS (
    SELECT customer_id,
           order_date,
           product_id,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rank
    FROM orders_by_customer_before_membership
)
SELECT customer_id, order_date, menu.product_id, menu.product_name
FROM orders_by_customer_ranked_by_date_before_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_ranked_by_date_before_membership.product_id
WHERE rank = 1;
```

| customer\_id | order\_date | product\_id | product\_name |
| :--- | :--- | :--- | :--- |
| B | 2021-01-04 | 1 | sushi |
| A | 2021-01-01 | 1 | sushi |

## Walkthrough

In the [previous question](./question-06.md), we built an intermediate table that filtered out all the sales records **before** the customer became a member. Then we ranked the grouped rows based on `customer_id` in the **ascending** order of `order_date`. Finally, we retained the highest ranked rows.

To answer this question, we'll take a similar but mirrored approach (i.e. reflection across the vertical line `x=0`). Our intermediate table will filter out all the sales records **after** the customer became a member, inclusive of the start date. Then we'll rank the grouped rows based on `customer_id` in the **descending** order of `order_date` and retain the highest ranked rows.

```sql
SELECT sales.customer_id, order_date, product_id
FROM sales
    INNER JOIN members m ON sales.customer_id = m.customer_id
WHERE order_date < m.join_date;
```

| customer\_id | order\_date | product\_id |
| :--- | :--- | :--- |
| A | 2021-01-01 | 1 |
| A | 2021-01-01 | 2 |
| B | 2021-01-01 | 2 |
| B | 2021-01-02 | 2 |
| B | 2021-01-04 | 1 |


```sql
WITH orders_by_customer_before_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date < m.join_date
)
SELECT customer_id,
       order_date,
       product_id,
       DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rank
FROM orders_by_customer_before_membership;
```

| customer\_id | order\_date | product\_id | rank |
| :--- | :--- | :--- | :--- |
| A | 2021-01-01 | 1 | 1 |
| A | 2021-01-01 | 2 | 2 |
| B | 2021-01-04 | 1 | 1 |
| B | 2021-01-02 | 2 | 2 |
| B | 2021-01-01 | 2 | 3 |


```sql
WITH orders_by_customer_before_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date < m.join_date
),
orders_by_customer_ranked_by_date_before_membership AS (
    SELECT customer_id,
           order_date,
           product_id,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rank
    FROM orders_by_customer_before_membership
)
SELECT customer_id, order_date, menu.product_id, menu.product_name
FROM orders_by_customer_ranked_by_date_before_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_ranked_by_date_before_membership.product_id
WHERE rank = 1;
```

| customer\_id | order\_date | product\_id | product\_name |
| :--- | :--- | :--- | :--- |
| B | 2021-01-04 | 1 | sushi |
| A | 2021-01-01 | 1 | sushi |
