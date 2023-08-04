# Question

Which item was the most popular for each customer ?

## Solution

```sql
WITH number_of_purchases_per_customer_and_product AS (
    SELECT customer_id, product_id, COUNT(*) num_of_purchases
    FROM sales
    GROUP BY customer_id, product_id
),
purchases_ranked_by_customer AS (
    SELECT 
        customer_id, 
        product_id, 
        num_of_purchases,
        DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY num_of_purchases DESC) AS rank
    FROM number_of_purchases_per_customer_and_product
)
SELECT customer_id, menu.product_id, menu.product_name, num_of_purchases
FROM purchases_ranked_by_customer
INNER JOIN menu ON menu.product_id = purchases_ranked_by_customer.product_id
WHERE rank = 1;
```

| customer\_id | product\_id | product\_name | num\_of\_purchases |
| :--- | :--- | :--- | :--- |
| C | 3 | ramen | 3 |
| B | 3 | ramen | 2 |
| A | 3 | ramen | 3 |


## Walkthrough

Similar to [Question 3](./question-03.md), this looks like another "Top N per group" instance with `N=1` where rows in a group share the `customer_id`, and we wish to rank the rows by the total number of purchases per product.

Let's start by building an intermediate table that stores the number of purchases for every pair of `(customer_id, product_id)`.

```sql
SELECT customer_id, product_id, COUNT(*) num_of_purchases
FROM sales
GROUP BY customer_id, product_id;
```

| customer\_id | product\_id | num\_of\_purchases |
| :--- | :--- | :--- |
| B | 3 | 2 |
| A | 3 | 3 |
| A | 1 | 1 |
| C | 3 | 3 |
| B | 1 | 2 |
| B | 2 | 2 |
| A | 2 | 2 |

Next, we'd like to form groups of these rows based on the shared value of `customer_id` and rank the rows in the groups by the descending order of `num_of_purchases`.

```sql
WITH number_of_purchases_per_customer_and_product AS (
    SELECT customer_id, product_id, COUNT(*) num_of_purchases
    FROM sales
    GROUP BY customer_id, product_id
)
SELECT 
    customer_id, 
    product_id, 
    num_of_purchases,
    DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY num_of_purchases DESC) AS rank
FROM number_of_purchases_per_customer_and_product;
```

| customer\_id | product\_id | num\_of\_purchases | rank |
| :--- | :--- | :--- | :--- |
| A | 3 | 3 | 1 |
| A | 2 | 2 | 2 |
| A | 1 | 1 | 3 |
| B | 3 | 2 | 1 |
| B | 1 | 2 | 2 |
| B | 2 | 2 | 3 |
| C | 3 | 3 | 1 |

Finally, the rows of interest are those where `rank = 1`, so let's filter it out. Seems like we'll have to chain the <abbr title="Common Table Expression">CTE</abbr>.

```sql
WITH number_of_purchases_per_customer_and_product AS (
    SELECT customer_id, product_id, COUNT(*) num_of_purchases
    FROM sales
    GROUP BY customer_id, product_id
),
purchases_ranked_by_customer AS (
    SELECT 
        customer_id, 
        product_id, 
        num_of_purchases,
        DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY num_of_purchases DESC) AS rank
    FROM number_of_purchases_per_customer_and_product
)
SELECT customer_id, menu.product_id, menu.product_name, num_of_purchases
FROM purchases_ranked_by_customer
INNER JOIN menu ON menu.product_id = purchases_ranked_by_customer.product_id
WHERE rank = 1;
```

| customer\_id | product\_id | product\_name | num\_of\_purchases |
| :--- | :--- | :--- | :--- |
| C | 3 | ramen | 3 |
| B | 3 | ramen | 2 |
| A | 3 | ramen | 3 |
