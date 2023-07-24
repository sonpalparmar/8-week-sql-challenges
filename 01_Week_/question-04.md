# Question

What is the most purchased item on the menu and how many times was it purchased by all customers?

## Solution

```sql
SELECT sales.product_id, product_name, COUNT(*) AS total_purchases
FROM sales
INNER JOIN menu m on sales.product_id = m.product_id
GROUP BY sales.product_id, product_name
ORDER BY total_purchases DESC
LIMIT 1;
```

| product\_id | product\_name | total\_purchases |
| :--- | :--- | :--- |
| 3 | ramen | 8 |

## Walkthrough

We can simply `COUNT` the rows for every group of sales for a given `product_id`.

```sql
SELECT product_id, COUNT(*) AS total_purchases
FROM sales
GROUP BY product_id
ORDER BY total_purchases DESC
LIMIT 1;
```

| product\_id | total\_purchases |
| :--- | :--- |
| 3 | 8 |


Now, bring in `menu` to get the name of the product alongside it.

```sql
SELECT sales.product_id, product_name, COUNT(*) AS total_purchases
FROM sales
INNER JOIN menu m on sales.product_id = m.product_id
GROUP BY sales.product_id, product_name
ORDER BY total_purchases DESC
LIMIT 1;
```

| product\_id | product\_name | total\_purchases |
| :--- | :--- | :--- |
| 3 | ramen | 8 |