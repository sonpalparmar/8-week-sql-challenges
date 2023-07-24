# Question

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Solution

```sql
WITH sales_annotated_with_first_week_check AS (
    SELECT sales.customer_id,
           order_date,
           menu.product_id,
           product_name,
           price,
           join_date,
           ((sales.order_date < members.join_date + INTERVAL '7 DAYS') AND (sales.order_date >= members.join_date)) is_within_first_week
    FROM sales
         LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
         INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date <= '2021-01-31' AND sales.customer_id IN ('A', 'B')
),
sales_with_points_multiplier AS (
    SELECT customer_id,
           order_date,
           product_name,
           price,
           join_date,
           (
               CASE is_within_first_week
                   WHEN TRUE THEN 2
                   ELSE (
                       CASE product_name
                           WHEN 'sushi' THEN 2
                           ELSE 1
                           END
                       )
                   END
               ) points_multiplier
    FROM sales_annotated_with_first_week_check
)
SELECT customer_id, SUM(price * points_multiplier * 10) AS total_points
FROM sales_with_points_multiplier
GROUP BY customer_id;
```

| customer\_id | total\_points |
| :--- | :--- |
| A | 1370 |
| B | 820 |

## Walkthrough

Let's break this down into multiple sub-problems. 

First, we'll build an intermediate table that just stores whether the sales orders that the customer made are within the first week of them becoming a member (i.e. whether the new membership `2x` point bonus applies). 

While we're there, we'll also retain sales records only up to January 2021 and for customers `A`, and `B`.

```sql
SELECT sales.customer_id,
       order_date,
       menu.product_id,
       product_name,
       price,
       members.join_date,
       ((sales.order_date < members.join_date + INTERVAL '7 DAYS') AND (sales.order_date >= members.join_date)) is_within_first_week
FROM sales
    LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
    INNER JOIN menu ON sales.product_id = menu.product_id
WHERE order_date <= '2021-01-31' AND sales.customer_id IN ('A', 'B');
```

| customer\_id | order\_date | product\_id | product\_name | price | join\_date | is\_within\_first\_week |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-10 | 3 | ramen | 12 | 2021-01-07 | true |
| A | 2021-01-11 | 3 | ramen | 12 | 2021-01-07 | true |
| A | 2021-01-11 | 3 | ramen | 12 | 2021-01-07 | true |
| A | 2021-01-01 | 2 | curry | 15 | 2021-01-07 | false |
| A | 2021-01-07 | 2 | curry | 15 | 2021-01-07 | true |
| A | 2021-01-01 | 1 | sushi | 10 | 2021-01-07 | false |
| B | 2021-01-16 | 3 | ramen | 12 | 2021-01-09 | false |
| B | 2021-01-01 | 2 | curry | 15 | 2021-01-09 | false |
| B | 2021-01-02 | 2 | curry | 15 | 2021-01-09 | false |
| B | 2021-01-04 | 1 | sushi | 10 | 2021-01-09 | false |
| B | 2021-01-11 | 1 | sushi | 10 | 2021-01-09 | true |


Next, we'll determine a points multiplier for these sales based on whether the customer bought
Sushi or the order was made within the first week of gaining membership.

```sql
WITH sales_annotated_with_first_week_check AS (
    SELECT sales.customer_id,
           order_date,
           menu.product_id,
           product_name,
           price,
           members.join_date,
           ((sales.order_date < members.join_date + INTERVAL '7 DAYS') AND (sales.order_date >= members.join_date)) is_within_first_week
    FROM sales
        LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
        INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date <= '2021-01-31' AND sales.customer_id IN ('A', 'B')
)
SELECT customer_id,
       order_date,
       product_name,
       price,
       join_date,
       (
        CASE is_within_first_week
            WHEN TRUE THEN 2
            ELSE (
                CASE product_name
                    WHEN 'sushi' THEN 2
                    ELSE 1
                END
            )
        END
        ) points_multiplier

FROM sales_annotated_with_first_week_check;
```

| customer\_id | order\_date | product\_name | price | join\_date | points\_multiplier |
| :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-10 | ramen | 12 | 2021-01-07 | 2 |
| A | 2021-01-11 | ramen | 12 | 2021-01-07 | 2 |
| A | 2021-01-11 | ramen | 12 | 2021-01-07 | 2 |
| A | 2021-01-01 | curry | 15 | 2021-01-07 | 1 |
| A | 2021-01-07 | curry | 15 | 2021-01-07 | 2 |
| A | 2021-01-01 | sushi | 10 | 2021-01-07 | 2 |
| B | 2021-01-16 | ramen | 12 | 2021-01-09 | 1 |
| B | 2021-01-01 | curry | 15 | 2021-01-09 | 1 |
| B | 2021-01-02 | curry | 15 | 2021-01-09 | 1 |
| B | 2021-01-04 | sushi | 10 | 2021-01-09 | 2 |
| B | 2021-01-11 | sushi | 10 | 2021-01-09 | 2 |


Finally, we'll `SUM` up `price * points_multiplier * 10` for these sales records and report it as the total points.

```sql
WITH sales_annotated_with_first_week_check AS (
    SELECT sales.customer_id,
           order_date,
           menu.product_id,
           product_name,
           price,
           join_date,
           ((sales.order_date < members.join_date + INTERVAL '7 DAYS') AND (sales.order_date >= members.join_date)) is_within_first_week
    FROM sales
         LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
         INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date <= '2021-01-31' AND sales.customer_id IN ('A', 'B')
),
sales_with_points_multiplier AS (
    SELECT customer_id,
           order_date,
           product_name,
           price,
           join_date,
           (
               CASE is_within_first_week
                   WHEN TRUE THEN 2
                   ELSE (
                       CASE product_name
                           WHEN 'sushi' THEN 2
                           ELSE 1
                           END
                       )
                   END
               ) points_multiplier
    FROM sales_annotated_with_first_week_check
)
SELECT customer_id, SUM(price * points_multiplier * 10) AS total_points
FROM sales_with_points_multiplier
GROUP BY customer_id;
```

| customer\_id | total\_points |
| :--- | :--- |
| A | 1370 |
| B | 820 |