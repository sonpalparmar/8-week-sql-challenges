# Danny's Diner Case Study

## Introduction

Danny seriously loves Japanese food, so in the beginning of 2021, he decided to open up a cute little restaurant named Danny's Diner that sells his three favorite foods: sushi, curry, and ramen.

Danny's Diner needs assistance to utilize the data they've collected to help run the business effectively. They are looking for insights into customer behavior, spending patterns, and menu preferences. This information will help Danny enhance customer experience and make informed decisions, including whether to expand the customer loyalty program.

## Problem Statement

Danny wants to use the data to answer several questions about his customers' visiting patterns, expenditure, and favorite menu items. By gaining insights into customer behavior, Danny aims to improve personalized experiences for loyal customers.

This case study provides three key datasets:

- `sales`: Captures customer purchases with corresponding order dates and product IDs.
- `menu`: Maps product IDs to product names and prices.
- `members`: Captures the join dates of customers who joined the loyalty program.

## Case Study Questions

The case study involves solving a series of questions using SQL. Each question can be answered with a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier, how many points would each customer have?
10. In the first week after a customer joins the program, how many points do customer A and B have at the end of January?

## Interactive SQL Session

You can explore and solve the case study questions interactively using the provided example datasets and an SQL editor. The datasets are available in the `dannys_diner` schema.

You can use the [DB Fiddle SQL Editor](https://www.db-fiddle.com/f/ix3Zv9sKSGXszpG5ZQF2DS/0) to write and test your SQL queries.

## Bonus Questions

The case study includes bonus questions related to creating data tables for quick insights without the need for SQL joins.

### Join All The Things
Recreate the following table output using the available data:

| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| ...         | ...        | ...          | ...   | ...    |


### Rank All The Things

Generate a ranking table for customer products:

| customer_id | order_date | product_name | price | member | ranking |
|-------------|------------|--------------|-------|--------|---------|
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| ...         | ...        | ...          | ...   | ...    | ...     |



