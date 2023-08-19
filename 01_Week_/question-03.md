# Question

What was the first item from the menu purchased by each customer?

## Solution

```sql
WITH orders_ranked_by_date AS (
    SELECT customer_id,
           product_id,
           order_date,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
    FROM sales
)
SELECT orders_ranked_by_date.customer_id, menu.product_id, menu.product_name, orders_ranked_by_date.order_date
FROM orders_ranked_by_date
INNER JOIN menu ON menu.product_id = orders_ranked_by_date.product_id
WHERE rank = 1;
```

| customer\_id | product\_id | product\_name | order\_date |
| :--- | :--- | :--- | :--- |
| A | 1 | sushi | 2021-01-01 |
| B | 2 | curry | 2021-01-01 |
| C | 3 | ramen | 2021-01-01 |

## Walkthrough

Let's start by asking a simpler question..

What was the first item from the menu purchased by customer `B`?

```sql
SELECT customer_id, product_name, order_date
FROM sales
INNER JOIN menu m on sales.product_id = m.product_id
WHERE customer_id = 'B'
ORDER BY order_date
LIMIT 1;
```

Now, if we only had a specific customer, then this would be good enough, but the resulting dataset we want varies over `customer_id` and lists one such row per customer. 

Essentially, we wish to select the first product purchased (by date) per customer in the group of orders made by that customer. This is an instance of the "Top N per group" problem with `N=1`, and the usual way we approach it is via assigning distinct ranks to all rows of the same group.

In this case, we assign ranks `1, 2, ...` to the orders made by customer `A` with the lowest rank going to the earliest `order_date`. Then, we can pluck out the lowest ranked row out of all groups.

In practice, this is implemented using [window functions](https://www.databasejournal.com/ms-sql/introduction-to-the-partition-by-window-function/) (or `DENSE_RANK() OVER (PARTITION BY <colA> ...)`, to be precise), where `<colA>` is a value shared by all rows within a group (i.e. `customer_id` in this case, since each customer forms their own group of orders).

```sql
SELECT customer_id,
        product_id,
        order_date,
        DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
FROM sales
```

| customer\_id | product\_id | order\_date | rank |
| :--- | :--- | :--- | :--- |
| A | 1 | 2021-01-01 | 1 |
| A | 2 | 2021-01-01 | 2 |
| A | 2 | 2021-01-07 | 3 |
| A | 3 | 2021-01-10 | 4 |
| A | 3 | 2021-01-11 | 5 |
| A | 3 | 2021-01-11 | 6 |
| B | 2 | 2021-01-01 | 1 |
| B | 2 | 2021-01-02 | 2 |
| B | 1 | 2021-01-04 | 3 |
| B | 1 | 2021-01-11 | 4 |
| B | 3 | 2021-01-16 | 5 |
| B | 3 | 2021-02-01 | 6 |
| C | 3 | 2021-01-01 | 1 |
| C | 3 | 2021-01-01 | 2 |
| C | 3 | 2021-01-07 | 3 |


Notice that the first order for every customer corresponds to the row with the lowest rank (i.e. rank = `1`) in that group. So, let's filter that out, and with a [Common Table Expression](https://learnsql.com/blog/what-is-common-table-expression/), join this data with `menu` to get the product names.

```sql
WITH orders_ranked_by_date AS (
    SELECT customer_id,
           product_id,
           order_date,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
    FROM sales
)
SELECT orders_ranked_by_date.customer_id, menu.product_id, menu.product_name, orders_ranked_by_date.order_date
FROM orders_ranked_by_date
INNER JOIN menu ON menu.product_id = orders_ranked_by_date.product_id
WHERE rank = 1;
```

| customer\_id | product\_id | product\_name | order\_date |
| :--- | :--- | :--- | :--- |
| A | 1 | sushi | 2021-01-01 |
| B | 2 | curry | 2021-01-01 |
| C | 3 | ramen | 2021-01-01 |
