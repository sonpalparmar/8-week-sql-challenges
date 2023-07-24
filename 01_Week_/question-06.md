# Question

Which item was purchased first by the customer after they became a member?

## Solution

```sql
WITH orders_by_customer_after_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date >= m.join_date
),
orders_by_customer_ranked_by_date_after_membership AS (
    SELECT customer_id,
           order_date,
           product_id,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
    FROM orders_by_customer_after_membership
)
SELECT customer_id, order_date, menu.product_id, menu.product_name
FROM orders_by_customer_ranked_by_date_after_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_ranked_by_date_after_membership.product_id
WHERE rank = 1;
```

| customer\_id | order\_date | product\_id | product\_name |
| :--- | :--- | :--- | :--- |
| B | 2021-01-11 | 1 | sushi |
| A | 2021-01-07 | 2 | curry |


## Walkthrough


A variant of [Question 3](./question-05.md), this is a "Top N per group" instance with `N=1` where rows in a group share the `customer_id`, and we wish to rank the rows by the ascending order of the `order_date` but we only include those rows in the groups that have `order_date >= join_date` for that member.

Let's start by building an intermediate table representing a set of rows that will participate in the `customer_id` grouping, filtering out those sales records where the user wasn't yet a member.

```sql
SELECT sales.customer_id, order_date, product_id
FROM sales
    INNER JOIN members m ON sales.customer_id = m.customer_id
WHERE order_date >= m.join_date;
```

| customer\_id | order\_date | product\_id |
| :--- | :--- | :--- |
| A | 2021-01-07 | 2 |
| A | 2021-01-10 | 3 |
| A | 2021-01-11 | 3 |
| A | 2021-01-11 | 3 |
| B | 2021-01-11 | 1 |
| B | 2021-01-16 | 3 |
| B | 2021-02-01 | 3 |

Now, we'll rank the rows in the groups by the ascending order of `order_date`.

```sql
WITH orders_by_customer_after_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date >= m.join_date
)
SELECT customer_id,
       order_date,
       product_id,
       DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
FROM orders_by_customer_after_membership;
```

| customer\_id | order\_date | product\_id | rank |
| :--- | :--- | :--- | :--- |
| A | 2021-01-07 | 2 | 1 |
| A | 2021-01-10 | 3 | 2 |
| A | 2021-01-11 | 3 | 3 |
| A | 2021-01-11 | 3 | 4 |
| B | 2021-01-11 | 1 | 1 |
| B | 2021-01-16 | 3 | 2 |
| B | 2021-02-01 | 3 | 3 |

Finally, the rows of interest are those with `rank = 1`. So, we'll retain those, and bring in `menu` to get the name of the products alongside.

```sql
WITH orders_by_customer_after_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date >= m.join_date
),
orders_by_customer_ranked_by_date_after_membership AS (
    SELECT customer_id,
           order_date,
           product_id,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
    FROM orders_by_customer_after_membership
)
SELECT customer_id, order_date, menu.product_id, menu.product_name
FROM orders_by_customer_ranked_by_date_after_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_ranked_by_date_after_membership.product_id
WHERE rank = 1;
```

| customer\_id | order\_date | product\_id | product\_name |
| :--- | :--- | :--- | :--- |
| B | 2021-01-11 | 1 | sushi |
| A | 2021-01-07 | 2 | curry |