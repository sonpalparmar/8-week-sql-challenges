# Question

What is the total amount each customer spent at the restaurant?

## Solution

```sql
SELECT customer_id, '$' || SUM(price) as total_amount_spent
FROM sales
INNER JOIN menu on sales.product_id = menu.product_id
GROUP BY customer_id;
```

| customer\_id | total\_amount\_spent |
| :--- | :--- |
| B | $74 |
| C | $36 |
| A | $76 |


## Walkthrough

Let's start by asking who has visited our restaurant so far?

```sql
SELECT DISTINCT(customer_id) 
FROM sales;
```

| customer\_id |
| :--- |
| B |
| C |
| A |

Okay, what kinda purchases were made by `A` ?

```sql
SELECT *
FROM sales 
WHERE customer_id = 'A';
```

| customer\_id | order\_date | product\_id |
| :--- | :--- | :--- |
| A | 2021-01-01 | 1 |
| A | 2021-01-01 | 2 |
| A | 2021-01-07 | 2 |
| A | 2021-01-10 | 3 |
| A | 2021-01-11 | 3 |
| A | 2021-01-11 | 3 |

Nice. Can we get the prices for these products next to their sale records?

```sql
SELECT *
FROM sales
INNER JOIN menu ON sales.product_id = menu.product_id
WHERE customer_id = 'A';
```

| customer\_id | order\_date | product\_id | product\_id | product\_name | price |
| :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-01 | 1 | 1 | sushi | 10 |
| A | 2021-01-07 | 2 | 2 | curry | 15 |
| A | 2021-01-01 | 2 | 2 | curry | 15 |
| A | 2021-01-11 | 3 | 3 | ramen | 12 |
| A | 2021-01-11 | 3 | 3 | ramen | 12 |
| A | 2021-01-10 | 3 | 3 | ramen | 12 |


Okay, so the total amount that customer `A` spent must be the sum of all these prices. There's a `SUM` aggregator we could use here.

```sql
SELECT SUM(price)
FROM sales
INNER JOIN menu ON sales.product_id = menu.product_id
WHERE customer_id = 'A';
```

| sum |
| :-- |
| 76 |

Can we perform this summation for every customer and store it as one row per customer. Yup, `GROUP BY` comes to the rescue.

```sql
SELECT customer_id, SUM(price)
FROM sales
INNER JOIN menu ON sales.product_id = menu.product_id
GROUP BY customer_id;
```

| customer\_id | sum |
| :--- | :--- |
| B | 74 |
| C | 36 |
| A | 76 |

Excellent, we got the data we need. 
Maybe polish it slightly by prefixing the numbers with a `$` and naming the column to something better?

```sql
SELECT customer_id, SUM(price) as total_amount_spent
FROM sales
INNER JOIN menu on sales.product_id = menu.product_id
GROUP BY customer_id;
```

| customer\_id | total\_amount\_spent |
| :--- | :--- |
| B | 74 |
| C | 36 |
| A | 76 |
