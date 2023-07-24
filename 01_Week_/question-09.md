# Question

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

## Solution

```sql
WITH sales_with_price AS (
    SELECT customer_id, order_date, m.product_id, product_name, price
    FROM sales
    INNER JOIN menu m ON sales.product_id = m.product_id
)
SELECT customer_id,
       SUM(
           CASE product_name
            WHEN 'sushi' THEN price * 10 * 2
            ELSE price * 10
            END
        ) AS total_points
FROM sales_with_price
GROUP BY customer_id;
```

| customer\_id | total\_points |
| :--- | :--- |
| B | 940 |
| C | 360 |
| A | 860 |


## Walkthrough

Let's start with pulling in the amounts from the `menu` table next to our sales records.

```sql
SELECT customer_id, order_date, m.product_id, product_name, price
FROM sales
INNER JOIN menu m ON sales.product_id = m.product_id
```

| customer\_id | order\_date | product\_id | product\_name | price |
| :--- | :--- | :--- | :--- | :--- |
| B | 2021-01-04 | 1 | sushi | 10 |
| A | 2021-01-01 | 1 | sushi | 10 |
| B | 2021-01-11 | 1 | sushi | 10 |
| B | 2021-01-01 | 2 | curry | 15 |
| B | 2021-01-02 | 2 | curry | 15 |
| A | 2021-01-01 | 2 | curry | 15 |
| A | 2021-01-07 | 2 | curry | 15 |
| A | 2021-01-11 | 3 | ramen | 12 |
| A | 2021-01-11 | 3 | ramen | 12 |
| A | 2021-01-10 | 3 | ramen | 12 |
| B | 2021-01-16 | 3 | ramen | 12 |
| B | 2021-02-01 | 3 | ramen | 12 |
| C | 2021-01-01 | 3 | ramen | 12 |
| C | 2021-01-01 | 3 | ramen | 12 |
| C | 2021-01-07 | 3 | ramen | 12 |


Then we'll `SUM` the transformed `price`s based on the values that `product_name` assumes:

```sql
WITH sales_with_price AS (
    SELECT customer_id, order_date, m.product_id, product_name, price
    FROM sales
    INNER JOIN menu m ON sales.product_id = m.product_id
)
SELECT customer_id,
       SUM(
           CASE product_name
            WHEN 'sushi' THEN price * 10 * 2
            ELSE price * 10
            END
        ) AS total_points
FROM sales_with_price
GROUP BY customer_id;
```

| customer\_id | total\_points |
| :--- | :--- |
| B | 940 |
| C | 360 |
| A | 860 |