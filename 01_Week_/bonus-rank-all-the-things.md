# Rank All The Things

Danny also requires further information about the **ranking** of customer products, but he purposely does not need the ranking for non-member purchases so he expects null **ranking** values for the records when customers are not yet part of the loyalty program.

<table>
    <thead>
      <tr>
        <th>customer_id</th>
        <th>order_date</th>
        <th>product_name</th>
        <th>price</th>
        <th>member</th>
        <th>ranking</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>A</td>
        <td>2021-01-01</td>
        <td>curry</td>
        <td>15</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>A</td>
        <td>2021-01-01</td>
        <td>sushi</td>
        <td>10</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>A</td>
        <td>2021-01-07</td>
        <td>curry</td>
        <td>15</td>
        <td>Y</td>
        <td>1</td>
      </tr>
      <tr>
        <td>A</td>
        <td>2021-01-10</td>
        <td>ramen</td>
        <td>12</td>
        <td>Y</td>
        <td>2</td>
      </tr>
      <tr>
        <td>A</td>
        <td>2021-01-11</td>
        <td>ramen</td>
        <td>12</td>
        <td>Y</td>
        <td>3</td>
      </tr>
      <tr>
        <td>A</td>
        <td>2021-01-11</td>
        <td>ramen</td>
        <td>12</td>
        <td>Y</td>
        <td>3</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-01-01</td>
        <td>curry</td>
        <td>15</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-01-02</td>
        <td>curry</td>
        <td>15</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-01-04</td>
        <td>sushi</td>
        <td>10</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-01-11</td>
        <td>sushi</td>
        <td>10</td>
        <td>Y</td>
        <td>1</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-01-16</td>
        <td>ramen</td>
        <td>12</td>
        <td>Y</td>
        <td>2</td>
      </tr>
      <tr>
        <td>B</td>
        <td>2021-02-01</td>
        <td>ramen</td>
        <td>12</td>
        <td>Y</td>
        <td>3</td>
      </tr>
      <tr>
        <td>C</td>
        <td>2021-01-01</td>
        <td>ramen</td>
        <td>12</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>C</td>
        <td>2021-01-01</td>
        <td>ramen</td>
        <td>12</td>
        <td>N</td>
        <td>null</td>
      </tr>
      <tr>
        <td>C</td>
        <td>2021-01-07</td>
        <td>ramen</td>
        <td>12</td>
        <td>N</td>
        <td>null</td>
      </tr>
    </tbody>
  </table>

## Solution

```sql
WITH sales_with_membership AS (
    SELECT sales.customer_id,
           order_date,
           product_name,
           price,
           (
               CASE order_date >= join_date
                   WHEN TRUE THEN 'Y'
                   ELSE 'N'
                   END
               ) member
    FROM sales
         LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
         INNER JOIN menu ON sales.product_id = menu.product_id
)
SELECT *,
       (
           CASE member
           WHEN 'N' THEN NULL
           ELSE
               DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
           END
        ) ranking
FROM sales_with_membership
ORDER BY customer_id, order_date;
```

| customer\_id | order\_date | product\_name | price | member | ranking |
| :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-01 | sushi | 10 | N | null |
| A | 2021-01-01 | curry | 15 | N | null |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | null |
| B | 2021-01-02 | curry | 15 | N | null |
| B | 2021-01-04 | sushi | 10 | N | null |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-07 | ramen | 12 | N | null |

## Walkthrough

Most of the data remains the same from the [previous bonus question](./bonus-join-all-the-things.md), except for the addition of a new column
that stores the ranking of the items per customer.

```sql
SELECT sales.customer_id,
        order_date,
        product_name,
        price,
        (
            CASE order_date >= join_date
                WHEN TRUE THEN 'Y'
                ELSE 'N'
            END
        ) member
FROM sales
        LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
        INNER JOIN menu ON sales.product_id = menu.product_id;
```

Now, we rank the rows by partitioning over `(customer_id, member)`.

```sql
WITH sales_with_membership AS (
    SELECT sales.customer_id,
           order_date,
           product_name,
           price,
           (
               CASE order_date >= join_date
                   WHEN TRUE THEN 'Y'
                   ELSE 'N'
                   END
               ) member
    FROM sales
         LEFT OUTER JOIN members ON sales.customer_id = members.customer_id
         INNER JOIN menu ON sales.product_id = menu.product_id
)
SELECT *,
       (
           CASE member
           WHEN 'N' THEN NULL
           ELSE
               DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
           END
        ) ranking
FROM sales_with_membership
ORDER BY customer_id, order_date;
```

| customer\_id | order\_date | product\_name | price | member | ranking |
| :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-01 | sushi | 10 | N | null |
| A | 2021-01-01 | curry | 15 | N | null |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | null |
| B | 2021-01-02 | curry | 15 | N | null |
| B | 2021-01-04 | sushi | 10 | N | null |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-07 | ramen | 12 | N | null |
