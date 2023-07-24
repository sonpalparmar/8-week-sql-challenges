# Question

How many days has each customer visited the restaurant?

## Solution

```sql
SELECT customer_id, COUNT(DISTINCT order_date)
FROM sales
GROUP BY customer_id;
```
| customer\_id | count |
| :--- | :--- |
| A | 4 |
| B | 6 |
| C | 2 |

## Walkthrough

Let's start by simplifying the question to an individual customer.

Which days did customer `A` visit the restaurant?

```sql
SELECT order_date
FROM sales
WHERE customer_id = 'A';
```

| order\_date |
| :--- |
| 2021-01-01 |
| 2021-01-01 |
| 2021-01-07 |
| 2021-01-10 |
| 2021-01-11 |
| 2021-01-11 |


Okay. There's repeats though. We can get rid of the duplicates if we apply `DISTINCT` to the `order_date`.

```sql
SELECT DISTINCT(order_date)
FROM sales
WHERE customer_id = 'A';
```

| order\_date |
| :--- |
| 2021-01-01 |
| 2021-01-07 |
| 2021-01-10 |
| 2021-01-11 |

Now, we could get the `COUNT` of these distinct dates to get the number of different days that customer `A` visited the restaurant.

```sql
SELECT COUNT(DISTINCT(order_date))
FROM sales
WHERE customer_id = 'A';
```

| count |
| :--- |
| 4 |

We'd be done if we could generalize this over all customers and get one row per `customer_id`. So, `GROUP BY` is our friend.

```sql
SELECT customer_id, COUNT(DISTINCT order_date)
FROM sales
GROUP BY customer_id;
```

| customer\_id | count |
| :--- | :--- |
| A | 4 |
| B | 6 |
| C | 2 |
