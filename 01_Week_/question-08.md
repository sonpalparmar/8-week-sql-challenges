# Question

What is the total items and amount spent for each member before they became a member?

## Solution

```sql
WITH orders_by_customer_before_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date < m.join_date
)
SELECT orders_by_customer_before_membership.customer_id,
       SUM(price),
       COUNT(*)
FROM orders_by_customer_before_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_before_membership.product_id
GROUP BY customer_id;
```

| customer\_id | sum | count |
| :--- | :--- | :--- |
| B | 40 | 3 |
| A | 25 | 2 |

**Note**: Since Customer `C` never became a member, we won't include them in this calculation. But if we were asked to, we'd have to change the `orders_by_customer_before_membership` table to the following:

```diff
SELECT sales.customer_id, order_date, product_id
FROM sales
-    INNER JOIN members m ON sales.customer_id = m.customer_id
+    LEFT OUTER JOIN members m ON sales.customer_id = m.customer_id
- WHERE order_date < m.join_date
+ WHERE order_date < m.join_date OR m.join_date IS NULL
```
This would produce the following as the final result:

| customer\_id | sum | count |
| :--- | :--- | :--- |
| B | 40 | 3 |
| C | 36 | 3 |
| A | 25 | 2 |


## Walkthrough

Let's start by building an intermediate table of all the sales records for customer before they became a member.

```sql
SELECT sales.customer_id, order_date, product_id, m.join_date
FROM sales
    INNER JOIN members m ON sales.customer_id = m.customer_id
WHERE order_date < m.join_date;
```

| customer\_id | order\_date | product\_id | join\_date |
| :--- | :--- | :--- | :--- |
| A | 2021-01-01 | 1 | 2021-01-07 |
| A | 2021-01-01 | 2 | 2021-01-07 |
| B | 2021-01-01 | 2 | 2021-01-09 |
| B | 2021-01-02 | 2 | 2021-01-09 |
| B | 2021-01-04 | 1 | 2021-01-09 |

Now, the total number of items they purchased is the number of sales (i.e. `COUNT` of the rows), and the total amount they spent is the `SUM` of the `price`s of the items in the sales records.

```sql
WITH orders_by_customer_before_membership AS (
    SELECT sales.customer_id, order_date, product_id
    FROM sales
        INNER JOIN members m ON sales.customer_id = m.customer_id
    WHERE order_date < m.join_date
)
SELECT orders_by_customer_before_membership.customer_id,
       SUM(price),
       COUNT(*)
FROM orders_by_customer_before_membership
INNER JOIN menu ON menu.product_id = orders_by_customer_before_membership.product_id
GROUP BY customer_id;
```

| customer\_id | sum | count |
| :--- | :--- | :--- |
| B | 40 | 3 |
| A | 25 | 2 |