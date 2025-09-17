---
title: "SQL Chapter 7 Time 2"
author: "db"
---

# ROW NUMBER

To find the cost of the most expensive product sold by each vendor:

```sql
SELECT 
    vendor_id,
    MAX(original_price) AS highest_price 
FROM farmers_market.vendor_inventory 
GROUP BY vendor_id
ORDER BY vendor_id
```

<table>
  <thead>
    <tr>
      <th>vendor_id</th>
      <th>highest_price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>2</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>3</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>4</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>5</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>6</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>7</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>8</td>
      <td>18.00</td>
    </tr>
    <tr>
      <td>9</td>
      <td>18.00</td>
    </tr>
  </tbody>
</table>

- To find the `product_id` linked to `MAX(original_price)` per vendor, use a window function.
  - Use `ROW_NUMBER()` to rank products by price per vendor.

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price,
    ROW_NUMBER() OVER (
        PARTITION BY vendor_id 
        ORDER BY original_price DESC
    ) AS price_rank
FROM farmers_market.vendor_inventory
ORDER BY 
    vendor_id, 
    original_price DESC
LIMIT 5;
```

<table>
  <thead>
    <tr>
      <th>vendor_id</th>
      <th>market_date</th>
      <th>product_id</th>
      <th>original_price</th>
      <th>price_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>2020-03-07</td>
      <td>5</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-11-13</td>
      <td>4</td>
      <td>17.50</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-09-26</td>
      <td>5</td>
      <td>17.50</td>
      <td>3</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-07-06</td>
      <td>3</td>
      <td>17.00</td>
      <td>4</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-08-19</td>
      <td>16</td>
      <td>17.00</td>
      <td>5</td>
    </tr>
  </tbody>
</table>

- To get the highest-priced item per vendor:
  - Use a subquery to rank items by `vendor_id`.
  - Limit the output to the top-ranked item per vendor.
- This method avoids `GROUP BY`:
  - It sorts records within partitions (`vendor_id` groups).
  - Filters by the `price_rank` row number for each partition.

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price
FROM (
    SELECT 
        vendor_id, 
        market_date, 
        product_id, 
        original_price,
        ROW_NUMBER() OVER (
            PARTITION BY vendor_id 
            ORDER BY original_price DESC
        ) AS price_rank
    FROM farmers_market.vendor_inventory
) x
WHERE x.price_rank = 1;
```

<table>
  <thead>
    <tr>
      <th>vendor_id</th>
      <th>market_date</th>
      <th>product_id</th>
      <th>original_price</th>
      <th>price_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>2020-03-07</td>
      <td>5</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2020-08-22</td>
      <td>8</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2020-03-18</td>
      <td>7</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2019-05-15</td>
      <td>4</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>5</td>
      <td>2019-12-21</td>
      <td>4</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>6</td>
      <td>2019-08-28</td>
      <td>8</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>7</td>
      <td>2019-06-15</td>
      <td>8</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>8</td>
      <td>2020-08-19</td>
      <td>8</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>9</td>
      <td>2020-03-11</td>
      <td>4</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
  </tbody>
</table>

- This returns one row per vendor, even if several products share the top price.
  - To list all highest-priced products per vendor, use the `RANK` function.
- Unlike earlier queries, this one uses a subquery—an inner query inside another.
- A subquery is needed because window functions like `ROW_NUMBER` work on the whole dataset, so you can’t filter with `WHERE` based on their results.
- Subqueries are common with window functions since filtering usually happens after calculation.
- Most SQL editors let you run just the inner query to preview its output.
- If you remove `PARTITION BY` from `ROW_NUMBER`, it ranks all rows across the dataset, so only the single highest-priced item gets `price_rank = 1`.

# RANK and DENSE_RANK

- Two other window functions work like `ROW_NUMBER` but give slightly different results.
- `RANK` numbers rows like `ROW_NUMBER`, but assigns the same rank to rows with equal values.

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price,
    RANK() OVER (
        PARTITION BY vendor_id 
        ORDER BY original_price DESC
    ) AS price_rank
FROM farmers_market.vendor_inventory
ORDER BY 
    vendor_id, 
    original_price DESC
LIMIT 5;
```

<table>
  <thead>
    <tr>
      <th>vendor_id</th>
      <th>market_date</th>
      <th>product_id</th>
      <th>original_price</th>
      <th>price_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>2020-03-07</td>
      <td>5</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-11-13</td>
      <td>4</td>
      <td>17.50</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-09-26</td>
      <td>5</td>
      <td>17.50</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-07-06</td>
      <td>3</td>
      <td>17.00</td>
      <td>4</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-08-19</td>
      <td>16</td>
      <td>17.00</td>
      <td>4</td>
    </tr>
  </tbody>
</table>

- A subquery with `price_rank = 1` can return several rows per vendor.
- For `vendor_id 1`, the ranking skips 3 because of a tie at second place.
- Use `DENSE_RANK` to avoid skipped numbers in ties (e.g., 1, 2, 3 instead of 1, 4).
- Use `ROW_NUMBER` for unique, tie-free rankings.

<table>
  <thead>
    <tr>
      <th>vendor_id</th>
      <th>market_date</th>
      <th>product_id</th>
      <th>original_price</th>
      <th>price_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>2020-03-07</td>
      <td>5</td>
      <td>18.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-11-13</td>
      <td>4</td>
      <td>17.50</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-09-26</td>
      <td>5</td>
      <td>17.50</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2019-07-06</td>
      <td>3</td>
      <td>17.00</td>
      <td>3</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020-08-19</td>
      <td>16</td>
      <td>17.00</td>
      <td>3</td>
    </tr>
  </tbody>
</table>

# NTILE

- The `ROW_NUMBER()` and `RANK()` functions can help answer questions like “What are the top 10 items sold at the farmer’s market, by price?”
  - You can filter the results to rows numbered less than or equal to 10.
  - But what if you were asked to return the “top tenth” of the inventory, when sorted by price?
- You could start by running a query that uses the `COUNT()` function.
  - Divide the number returned by 10.
  - Then write another query that numbers the rows and filter to those with a row number less than or equal to the number you just determined.
  - But that is not a dynamic solution.
  - You would have to modify it as the number of rows in the database changes.
- The dynamic solution is to use the `NTILE` function.
  - With `NTILE`, you specify a number inside the parentheses, `NTILE(n)`, to indicate that you want the results broken up into `n` blocks.
  - So, to get the top tenth, you could put 10 in the parentheses, with no partition (segmenting the entire results set), then filter to the rows in `NTILE 1`, like so.

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price,
    NTILE(10) OVER (ORDER BY original_price DESC) AS price_ntile
FROM farmers_market.vendor_inventory
ORDER BY original_price DESC
```

- If the number of rows in the results set can be divided evenly:
  - The results will be broken up into `n` equally sized groups, labeled 1 to `n`.
  - If they cannot be divided up evenly, some groups will end up with one more row than others.
- Note that `NTILE` is only using the count of rows to split the groups (or to split the partition into groups, if you specify a partition).
  - It is not using a field value to determine where to make the splits.
  - Therefore, it is possible that two rows with the same value specified in the `ORDER BY` clause (two products with the same `original_price`, in this case) will end up in two different `NTILE` groups.
- You can sort on additional fields if you want more control over how the rows are split into `NTILE` groups.
  - But if you want to ensure that all items with the same price are grouped together, it would make more sense to use `RANK` than `NTILE`.
  - In that case, you are not looking for evenly sized groupings.

# Aggregate Window Functions

- You learned about aggregate SQL functions like `SUM()` in Chapter 6, “Aggregating Results for Analysis.”
  - In this chapter, you have learned about window functions that partition the results set.
  - Can you imagine how they might be used together?
- It turns out that you can use most aggregate functions across partitions like the window functions.
  - This returns an aggregate calculation for a partition on every row in that partition.
  - If you do not use the `PARTITION BY` clause, it applies to the whole results set.
  - One way this approach can be used is to compare each row’s value to the aggregate value for that grouped category.
- For example, what if you are a farmer selling products at the market, and you want to know which of your products were above the average price per product on each market date?
  - Remember that because of the way our database is designed, this is not a true average for the full inventory.
  - We are not multiplying by a quantity, but you can think of it as the average display price in a product catalog.
  - We can use the `AVG()` function as a window function, partitioned by `market_date`, and compare each product’s price to that value.
- First, let’s try using `AVG()` as a window function.

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price,
    AVG(original_price) OVER (PARTITION BY market_date ORDER BY 
market_date) 
            AS average_cost_product_by_market_date 
FROM farmers_market.vendor_inventory
```

![Figure 7.5](Fotos/Chapter7/Fig_7.5.png)
<figcaption></figcaption>

- The `AVG()` function in this query is structured as a window function.
  - This means it has “OVER (PARTITION BY __ ORDER BY __)” syntax.
  - Instead of returning a single row per group with the average for that group, like you would get with `GROUP BY`, this function displays the average for the partition on every row within the partition.
  - You can see in Figure 7.5 that when you get to a new `market_date` value in the results dataset, the `average_cost_product_by_market_date` value changes.
- Now, let’s wrap that query inside another query (use it as a subquery) so we can compare the original price per item to the average cost of products on each market date that has been calculated by the window function.
  - In this example, we are comparing the values in the last two columns of Figure 7.5.
  - Remember that we cannot compare the two values in the original query, because the window function is calculated over multiple rows and will not have a value for the partition yet when the `WHERE` clause filters are being applied row by row.
- Using a subquery, we can filter the results to a single vendor, with `vendor_id 1`, and only display products that have prices above the market date’s average product cost.
  - Here we will also format the `average_cost_product_by_market_date` to two digits after the decimal point using the `ROUND()` function.

```sql
SELECT * FROM
(
    SELECT 
        vendor_id, 
        market_date,
        product_id, 
        original_price,
        ROUND(AVG(original_price) OVER (PARTITION BY market_date ORDER BY 
market_date), 2)
            AS average_cost_product_by_market_date 
    FROM farmers_market.vendor_inventory
) x
WHERE x.vendor_id = 1 
    AND x.original_price > x.average_cost_product_by_market_date 
ORDER BY x.market_date, x.original_price DESC
```

- Note that we will get different (and incorrect) results if we put the `WHERE` clause filtering by `vendor_id` inside the parentheses with the original query in this case.
  - That is because the results set of the inner `SELECT` statement would be filtered to `vendor_id 1` before the window function was calculated.
  - We would only be calculating the average price of vendor 1’s products!
- Since we want to compare vendor 1’s prices on each market date to the average price of all vendors’ products on each market date:
  - We do not want to filter to `vendor_id 1` until after the averages have been calculated.
  - So we put the `WHERE` clause on the “outer” query outside the parentheses.
- In the output, `vendor_id 1` had a single product, with `product_id 11`, that was above the average product cost on each of the market dates listed.

![Figure 7.6](Fotos/Chapter7/Fig_7.6.png)
<figcaption></figcaption>

- Another use of an aggregate window function is to count how many items are in each partition.
  - The following is a query that counts how many different products each vendor brought to market on each date, and displays that count on each row.
  - This way, even if the results were not sorted in a way that let you quickly determine how many inventory rows there are for each vendor, you would know that the row you are looking at represents just one of the products in a counted set.
- You can see in the output that even if I am only looking at one row for `vendor 9` on March 9, 2019, I would know that it is one of three products that vendor had in their inventory on that market date.

```sql
SELECT 
    vendor_id, 
    market_date,
    product_id, 
    original_price,
    COUNT(product_id) OVER (PARTITION BY market_date, vendor_id)  
vendor_product_count_per_market_date
    FROM farmers_market.vendor_inventory 
ORDER BY vendor_id, market_date, original_price DESC
```

![Figure 7.7](Fotos/Chapter7/Fig_7.7.png)
<figcaption></figcaption>

- You can also use aggregate window functions to calculate running totals.
  - In the first query shown next, we are not using a `PARTITION BY` clause.
  - So the running total of the price is calculated across the entire results set, in the sort order specified in the `ORDER BY` clause of the `SUM()` window function.

```sql
SELECT customer_id,  
    market_date, 
    vendor_id, 
    product_id, 
    quantity * cost_to_customer_per_qty AS price,
    SUM(quantity * cost_to_customer_per_qty) OVER (ORDER BY market_date, 
transaction_time, customer_id, product_id) AS running_total_purchases 
FROM farmers_market.customer_purchases
```

![Figure 7.8](Fotos/Chapter7/Fig_7.8.png)
<figcaption></figcaption>

- In this next query, we are calculating the same running total, but it is partitioned by `customer_id`.
  - That means that each time we get to a new `customer_id`, the running total resets.
  - So we are getting a running total of the cost of items purchased by each customer, sorted by the date and time, and the product ID (in case any two items have identical purchase times).

```sql
SELECT customer_id, 
    market_date, 
    vendor_id, 
    product_id, 
    quantity * cost_to_customer_per_qty AS price,
    SUM(quantity * cost_to_customer_per_qty) OVER (PARTITION BY 
customer_id ORDER BY market_date, transaction_time, product_id) AS 
customer_spend_running_total
FROM farmers_market.customer_purchases
```

![Figure 7.9](Fotos/Chapter7/Fig_7.9.png)
<figcaption></figcaption>

- This `SUM` functions as a running total because of the combination of the `PARTITION BY` and `ORDER BY` clauses in the window function.
  - We showed what happens when there is only an `ORDER BY` clause, and when both clauses are present.
  - What do you expect to happen when there is only a `PARTITION BY` clause (and no `ORDER BY` clause)?

```sql
SELECT customer_id, 
    market_date, 
    vendor_id, 
    product_id, 
    ROUND(quantity * cost_to_customer_per_qty, 2) AS price, 
    ROUND(SUM(quantity * cost_to_customer_per_qty) OVER (PARTITION BY 
customer_id), 2) AS customer_spend_total 
FROM farmers_market.customer_purchases
```

- As hinted at by the field name alias, this version with no in-partition sorting calculates the total spent by the customer and displays that summary total on every row.
  - So, without the `ORDER BY`, the `SUM` is calculated across the entire partition, instead of as a per-row running total, as shown in Figure 7.10.
  - We also added the `ROUND()` function so this final output displays the prices with two numbers after the decimal point.

# LAG and LEAD

- With the running total example in the previous section, you can start to see how SQL can be used to calculate changes in a value over time.
- Using the `vendor_booth_assignments` table in the Farmer’s Market database:
  - We can display each vendor’s booth assignment for each `market_date` alongside their previous booth assignments using the `LAG()` function.
- `LAG` retrieves data from a row that is a selected number of rows back in the dataset.
  - You can set the number of rows (offset) to any integer value `x` to count `x` rows backwards, following the sort order specified in the `ORDER BY` section of the window function.

```sql
SELECT 
    market_date, 
    vendor_id,
    booth_number,
    LAG(booth_number,1) OVER (PARTITION BY vendor_id ORDER BY market_date, 
vendor_id) AS previous_booth_number
FROM farmers_market.vendor_booth_assignments 
ORDER BY market_date, vendor_id, booth_number
```

- In this case, for each `vendor_id` for each `market_date`, we are pulling the `booth_number` the vendor had 1 market date in the past.
  - As you can see in Figure 7.11, the values are all `NULL` for the first market date, because there is no prior market date to pull values from.

![Figure 7.11](Fotos/Chapter7/Fig_7.11.png)
<figcaption></figcaption>

- The recipient of a report like this, such as the manager of the farmer’s market, may want to filter these query results to a specific market date.
  - This helps determine which vendors are new or changing booths that day, so we can contact them and ensure setup goes smoothly.
  - We will create this report by wrapping the query with the `LAG` function in another query.
  - This allows us to filter the results to a `market_date` and vendors whose current `booth_number` is different from their `previous_booth_number`.

```sql
SELECT * FROM
(
    SELECT 
        market_date, 
        vendor_id,
        booth_number,
          LAG(booth_number,1) OVER (PARTITION BY vendor_id ORDER BY market_ 
date, vendor_id) AS previous_booth_number
        FROM farmers_market.vendor_booth_assignments 
        ORDER BY market_date, vendor_id, booth_number
) x
WHERE x.market_date = '2019-04-10'
        AND (x.booth_number <> x.previous_booth_number OR x.previous_ 
booth_number IS NULL)
```

![Figure 7.12](Fotos/Chapter7/Fig_7.12.png)
<figcaption></figcaption>

- If you look closely at Figure 7.11, you can see that for the April 10, 2019 market:
  - `vendor 1` and `vendor 4` have swapped booths compared to the previous market date.
  - This would be hard to spot from a printout of this output.
- To show another example use case, let’s say we want to find out if the total sales on each market date are higher or lower than they were on the previous market date.
  - In this example, we are going to use the `customer_purchases` table from the Farmer’s Market database.
  - We will also add in a `GROUP BY` function, which the previous examples did not include.
  - The window functions are calculated after the grouping and aggregation occurs.
- First, we need to get the total sales per market date, using a `GROUP BY` and regular aggregate `SUM`.

![Figure 7.13](Fotos/Chapter7/Fig_7.13.png)
<figcaption></figcaption>

- Then, we can add the `LAG()` window function to output the previous `market_date`'s calculated sum on each row.
  - We `ORDER BY market_date` in the window function to ensure it is the previous market date we are comparing to and not another date.
  - You can see in Figure 7.14 that each row has a new total value (for that market date), as well as the previous market date’s total.

```sql
SELECT 
    market_date, 
    SUM(quantity * cost_to_customer_per_qty) AS market_date_total_sales, 
    LAG(SUM(quantity * cost_to_customer_per_qty), 1) OVER (ORDER BY 
market_date) AS previous_market_date_total_sales 
FROM farmers_market.customer_purchases
GROUP BY market_date
ORDER BY market_date
```

![Figure 7.14](Fotos/Chapter7/Fig_7.14.png)
<figcaption></figcaption>

- `LEAD` works the same way as `LAG`, but it gets the value from the next row instead of the previous row (assuming the offset integer is 1).
  - You can set the offset integer to any value `x` to count `x` rows forward, following the sort order specified in the `ORDER BY` section of the window function.
  - If the rows are sorted by a time value, `LAG` would be retrieving data from the past, and `LEAD` would be retrieving data from the future (relative to the current row).
  - These values can also now be used in calculations; for example, to determine the change in sales week to week.
- This chapter just covers the tip of the iceberg when it comes to window functions!
  - Look in the documentation for the type of database you are working with to see what other functions are available, and what caveats to be aware of for each.
  - Some database systems offer additional capabilities.
    - For example, PostgreSQL supports something called “window naming.”
    - Oracle has additional useful aggregate functions like `LISTAGG` (which operates on string values).
    - Some database systems allow for additional clauses like `RANGE`.
- Once you understand the concept of a window function and how to use it in your query, you have the knowledge you need to research and apply the many variations.

# Exercises Using the Included Database

1. Do the following two steps:
   - a. Write a query that selects from the `customer_purchases` table and numbers each customer’s visits to the farmer’s market.
     - Label each market date with a different number.
     - Each customer’s first visit is labeled 1, the second visit is labeled 2, and so on.
     - We are not counting visits where no purchases are made, because we have no record of those.
     - You can either display all rows in the `customer_purchases` table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits.
     - Hint: One of these approaches uses `ROW_NUMBER()` and one uses `DENSE_RANK()`.
   - b. Reverse the numbering of the query from part a so each customer’s most recent visit is labeled 1.
     - Then write another query that uses this one as a subquery and filters the results to only the customer’s most recent visit.

2. Using a `COUNT()` window function, include a value along with each row of the `customer_purchases` table that indicates how many different times that customer has purchased that `product_id`.

3. In the last query associated with Figure 7.14 from the chapter, we used `LAG` and sorted by `market_date`.
   - Can you think of a way to use `LEAD` in place of `LAG`, but get the exact same output?
