---
title: "SQL Chapter 7"
author: "db"
---

# Introduction

- All the functions we have learned so far, like ROUND(), return one value for each row in the results. When we use GROUP BY, the functions work on multiple values in a group of records, summarizing across many rows in the dataset, like AVG(). But each value returned is still linked to a single row in the results.
- Window functions also work on multiple records, but these records don't need to be grouped in the output. This allows us to compare the values from one row to a group of rows, or partition. This helps us answer questions like: If the dataset was sorted, where would this row be? How does a value in this row compare to the value in the previous row? How does a value in the current row compare to the average value for its group?
- So, window functions return group aggregate calculations along with individual row-level information for items in that group or partition. They can also be used to rank or sort values within each partition.
- One use for window functions in data science is to include information from a past record with the most recent record related to an entity. For example, we could use window functions to get the date of the first purchase a person made at the farmer’s market and show it with their detailed purchase records. This could help us see how long they had been a customer at the time of each purchase.

# ROW NUMBER

- Based on what you’ve learned in previous chapters, if you wanted to find out the cost of the most expensive product sold by each vendor, you could group the records in the `vendor_inventory` table by `vendor_id` and return the highest `original_price` value using the following query:

```sql
SELECT 
    vendor_id, 
    MAX(original_price) AS highest_price 
FROM farmers_market.vendor_inventory 
GROUP BY vendor_id
ORDER BY vendor_id
```

- But this just gives you the price of the most expensive item per vendor. If you wanted to know which item was the most expensive, how would you find out which `product_id` was linked to that `MAX(original_price)` per vendor?
- There is a window function that lets you rank rows by a value — in this case, ranking products per vendor by price — called `ROW_NUMBER()`. This approach will allow you to keep the detailed information that you would otherwise lose by aggregating like we did in the previous query:

```sql
SELECT 
    vendor_id, 
    market_date, 
    product_id, 
    original_price,
    ROW_NUMBER() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS 
price_rank
FROM farmers_market.vendor_inventoryORDER BY vendor_id, original_price DESC
```

- Let’s break that syntax down a bit. The `ROW_NUMBER()` line means “number the rows of inventory per vendor, sorted by original price, in descending order.” The part inside the parentheses shows how to apply the `ROW_NUMBER()` function. We’re going to `PARTITION BY vendor_id` (this is like a `GROUP BY` without combining the rows, so we’re telling it how to split the rows into groups, without aggregating). Then within the partition, the `ORDER BY` shows how to sort the rows. So, we’ll sort the rows by price, high to low, within each `vendor_id` partition, and number each row. That means the highest-priced item per vendor will be first, and assigned row number 1.
- You can see in Figure 7.1 that for each vendor, the products are sorted by `original_price`, high to low, and the row numbering column is called `price_rank`. The row numbering starts over when you get to the next `vendor_id`, so the most expensive item per vendor has a `price_rank` of 1.

![Figure 7.1](Fotos/Chapter7/Fig_7.1.png)
<figcaption></figcaption>

- To return only the record of the highest-priced item per vendor, you can query the results of the previous query (this is called a subquery) and limit the output to the #1 ranked item per `vendor_id`. With this approach, you’re not using a `GROUP BY` to aggregate the records. You’re sorting the records within each partition (a set of records that share a value or combination of values — `vendor_id` in this case), then filtering to a value (the row number called `price_rank` here) that was evaluated over that partition. Figure 7.2 shows the highest-priced product per vendor using the following query:

```sql
SELECT * FROM
(
    SELECT 
        vendor_id, 
        market_date,
        product_id, 
        original_price,
          ROW_NUMBER() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS 
price_rank
    FROM farmers_market.vendor_inventory 
    ORDER BY vendor_id) x
WHERE x.price_rank = 1
```

![Figure 7.2](Fotos/Chapter7/Fig_7.2.png)
<figcaption></figcaption>

- This will only return one row per vendor, even if there are multiple products with the same price. To return all products with the highest price per vendor when there is more than one with the same price, use the `RANK` function found in the next section. 
- If you want to determine which one of the multiple items gets returned by this `ROW_NUMBER` function, you can add additional sorting columns in the `ORDER BY` section of the `ROW_NUMBER` function. For example, you can sort by both `original_price` (descending) and `market_date` (ascending) to get the product brought to market by each vendor the earliest that had this top price.
- You’ll notice that the preceding query has a different structure than the queries we have written so far. There is one query embedded inside the other! Sometimes this is called “querying from a derived table,” but is more commonly called a “subquery.” 
- What we’re doing is treating the results of the “inner” `SELECT` statement like a table, here given the table alias x, selecting all columns from it, and filtering to only the rows with a particular `ROW_NUMBER`. Our `ROW_NUMBER` column is aliased `price_rank`, and we’re filtering to `price_rank = 1`, because we numbered the rows by `original_price` in descending order, so the most expensive item will have the lowest row number.
- The reason we have to structure this as a subquery is that the entire dataset has to be processed in order for the window function to find the highest price per vendor. So we can’t filter the results using a `WHERE` clause (which you’ll remember evaluates the conditional statements row by row) because when that filtering is applied, the `ROW_NUMBER` has not yet been calculated for every row.
- Figure 7.3 illustrates which parts of the SQL statement are considered the “inner” and “outer” queries. The “outer” part of a subquery is processed after the “inner” query is complete, so the row numbers have been determined, and we can then filter by the values in the price_rank column.

![Figure 7.3](Fotos/Chapter7/Fig_7.3.png)
<figcaption></figcaption>

- Many SQL editors allow you to run the inner query by itself. You can do this by highlighting it and executing the selected SQL only. This allows you to preview the results of the inner query that will then be used by the outer query.
- If we didn’t use a subquery and tried to filter based on the values in the `price_rank` field by adding a `WHERE` clause to the first query with the `ROW_NUMBER` function, we would get an error. The `price_rank` value is unknown when the `WHERE` clause conditions are evaluated per row because the window functions have not yet checked the entire dataset to determine the ranking. If we tried to put the `ROW_NUMBER` function in the `WHERE` clause instead of referencing the `price_rank` alias, we would get a different error for the same reason.
- You will see the subquery format throughout this chapter. This is because if you want to do anything with the results of most window functions, you have to allow them to calculate across the entire dataset first. Then, by treating the results like a table, you can query from and filter by the results returned by the window functions.
- Note that you can also use `ROW_NUMBER` without a `PARTITION BY` clause to number every record across the whole result (instead of numbering per partition). If you use the same ORDER BY clause we did earlier and eliminate the `PARTITION BY` clause, then only one item with the highest price in the entire results set would get the `price_rank` of 1, instead of one item per vendor.

# RANK and DENSE RANK

- Two other window functions are very similar to `ROW_NUMBER`. They have the same syntax but provide slightly different results.
- The `RANK` function numbers the results like `ROW_NUMBER`. However, it gives rows with the same value the same ranking. If we run the same query as before but replace `ROW_NUMBER` with `RANK`, we get the output shown in Figure 7.4.

```sql
SELECT 
    vendor_id, 
    market_date,
    product_id, 
    original_price,
    RANK() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS 
price_rank
    FROM farmers_market.vendor_inventory
ORDER BY vendor_id, original_price DESC
```

![Figure 7.4](Fotos/Chapter7/Fig_7.4.png)
<figcaption></figcaption>

- If we use a subquery structure and embed this query inside another `SELECT` statement like we did previously, and filter to `price_rank = 1`, multiple rows per vendor would be returned.
- Notice in Figure 7.4 that the ranking for `vendor_id 1` goes from 1 to 2 to 4, skipping 3. This happens because there is a tie for second place, so there is no third place.
- If you do not want to skip numbers like this in your ranking when there is a tie, use the `DENSE_RANK` function. In this case, the items for `vendor_id` in the example would be numbered 1 and 2 instead of 1 and 4.
- If you do not want any ties in your numbering at all and want each row to have its own number, use the `ROW_NUMBER` function. Compare the output in Figure 7.4 to the output in Figure 7.1.

# NTILE

- The `ROW_NUMBER()` and `RANK()` functions can help answer questions like “What are the top 10 items sold at the farmer’s market, by price?” You can filter the results to rows numbered less than or equal to 10. But what if you were asked to return the “top tenth” of the inventory, when sorted by price?
- You could start by running a query that uses the `COUNT()` function. Divide the number returned by 10. Then write another query that numbers the rows and filter to those with a row number less than or equal to the number you just determined. But that is not a dynamic solution. You would have to modify it as the number of rows in the database changes.
- The dynamic solution is to use the `NTILE` function. With `NTILE`, you specify a number inside the parentheses, `NTILE(n)`, to indicate that you want the results broken up into `n` blocks. So, to get the top tenth, you could put 10 in the parentheses, with no partition (segmenting the entire results set), then filter to the rows in `NTILE 1`, like so.

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

- If the number of rows in the results set can be divided evenly, the results will be broken up into `n` equally sized groups, labeled 1 to `n`. If they cannot be divided up evenly, some groups will end up with one more row than others.
- Note that `NTILE` is only using the count of rows to split the groups (or to split the partition into groups, if you specify a partition). It is not using a field value to determine where to make the splits. Therefore, it is possible that two rows with the same value specified in the `ORDER BY` clause (two products with the same `original_price`, in this case) will end up in two different `NTILE` groups.
- You can sort on additional fields if you want more control over how the rows are split into `NTILE` groups. But if you want to ensure that all items with the same price are grouped together, it would make more sense to use `RANK` than `NTILE`. In that case, you are not looking for evenly sized groupings.

# Aggregate Window Functions

- You learned about aggregate SQL functions like `SUM()` in Chapter 6, “Aggregating Results for Analysis.” In this chapter, you have learned about window functions that partition the results set. Can you imagine how they might be used together? It turns out that you can use most aggregate functions across partitions like the window functions. This returns an aggregate calculation for a partition on every row in that partition. If you do not use the `PARTITION BY` clause, it applies to the whole results set. One way this approach can be used is to compare each row’s value to the aggregate value for that grouped category.
- For example, what if you are a farmer selling products at the market, and you want to know which of your products were above the average price per product on each market date? Remember that because of the way our database is designed, this is not a true average for the full inventory. We are not multiplying by a quantity, but you can think of it as the average display price in a product catalog. We can use the `AVG()` function as a window function, partitioned by `market_date`, and compare each product’s price to that value. First, let’s try using `AVG()` as a window function. The output of the following query is shown in Figure 7.5.

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

- The `AVG()` function in this query is structured as a window function. This means it has “OVER (PARTITION BY __ ORDER BY __)” syntax. Instead of returning a single row per group with the average for that group, like you would get with `GROUP BY`, this function displays the average for the partition on every row within the partition. You can see in Figure 7.5 that when you get to a new `market_date` value in the results dataset, the `average_cost_product_by_market_date` value changes.
- Now, let’s wrap that query inside another query (use it as a subquery) so we can compare the original price per item to the average cost of products on each market date that has been calculated by the window function. In this example, we are comparing the values in the last two columns of Figure 7.5. Remember that we cannot compare the two values in the original query, because the window function is calculated over multiple rows and will not have a value for the partition yet when the `WHERE` clause filters are being applied row by row.
- Using a subquery, we can filter the results to a single vendor, with `vendor_id 1`, and only display products that have prices above the market date’s average product cost. Here we will also format the `average_cost_product_by_market_date` to two digits after the decimal point using the `ROUND()` function.

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

- Note that we will get different (and incorrect) results if we put the `WHERE` clause filtering by `vendor_id` inside the parentheses with the original query in this case. That is because the results set of the inner `SELECT` statement would be filtered to `vendor_id 1` before the window function was calculated. We would only be calculating the average price of vendor 1’s products! Since we want to compare vendor 1’s prices on each market date to the average price of all vendors’ products on each market date, we do not want to filter to `vendor_id 1` until after the averages have been calculated. So we put the `WHERE` clause on the “outer” query outside the parentheses.
- The results of the preceding query are shown in Figure 7.6. So `vendor_id 1` had a single product, with `product_id 11`, that was above the average product cost on each of the market dates listed.

![Figure 7.6](Fotos/Chapter7/Fig_7.6.png)
<figcaption></figcaption>

- Another use of an aggregate window function is to count how many items are in each partition. The following is a query that counts how many different products each vendor brought to market on each date, and displays that count on each row. This way, even if the results were not sorted in a way that let you quickly determine how many inventory rows there are for each vendor, you would know that the row you are looking at represents just one of the products in a counted set.

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

- The output for this query is shown in Figure 7.7. You can see that even if I am only looking at one row for `vendor 9` on March 9, 2019, I would know that it is one of three products that vendor had in their inventory on that market date.

![Figure 7.7](Fotos/Chapter7/Fig_7.7.png)
<figcaption></figcaption>

- You can also use aggregate window functions to calculate running totals. In the first query shown next, we are not using a `PARTITION BY` clause, so the running total of the price is calculated across the entire results set, in the sort order specified in the `ORDER BY` clause of the `SUM()` window function. The results are displayed in Figure 7.8.

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

- In this next query, we are calculating the same running total, but it is partitioned by `customer_id`. That means that each time we get to a new `customer_id`, the running total resets. So we are getting a running total of the cost of items purchased by each customer, sorted by the date and time, and the product ID (in case any two items have identical purchase times). The result is shown in Figure 7.9.

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

- This `SUM` functions as a running total because of the combination of the `PARTITION BY` and `ORDER BY` clauses in the window function. We showed what happens when there is only an `ORDER BY` clause, and when both clauses are present. What do you expect to happen when there is only a `PARTITION BY` clause (and no `ORDER BY` clause)?

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

- As hinted at by the field name alias, this version with no in-partition sorting calculates the total spent by the customer and displays that summary total on every row. So, without the `ORDER BY`, the `SUM` is calculated across the entire partition, instead of as a per-row running total, as shown in Figure 7.10. We also added the `ROUND()` function so this final output displays the prices with two numbers after the decimal point.

# LAG and LEAD

- With the running total example in the previous section, you can start to see how SQL can be used to calculate changes in a value over time.
- Using the `vendor_booth_assignments` table in the Farmer’s Market database, we can display each vendor’s booth assignment for each `market_date` alongside their previous booth assignments using the `LAG()` function.
- `LAG` retrieves data from a row that is a selected number of rows back in the dataset. You can set the number of rows (offset) to any integer value `x` to count `x` rows backwards, following the sort order specified in the `ORDER BY` section of the window function.

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

- In this case, for each `vendor_id` for each `market_date`, we are pulling the `booth_number` the vendor had 1 market date in the past. As you can see in Figure 7.11, the values are all `NULL` for the first market date, because there is no prior market date to pull values from.

![Figure 7.11](Fotos/Chapter7/Fig_7.11.png)
<figcaption></figcaption>

- The recipient of a report like this, such as the manager of the farmer’s market, may want to filter these query results to a specific market date. This helps determine which vendors are new or changing booths that day, so we can contact them and ensure setup goes smoothly. We will create this report by wrapping the query with the `LAG` function in another query. This allows us to filter the results to a `market_date` and vendors whose current `booth_number` is different from their `previous_booth_number`.

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

- If you look closely at Figure 7.11, you can see that for the April 10, 2019 market, vendor 1 and vendor 4 have swapped booths compared to the previous market date. This would be hard to spot from a printout of this output, but using the preceding query, we can return just the rows with booth changes on the specified date, as shown in Figure 7.12.

![Figure 7.12](Fotos/Chapter7/Fig_7.12.png)
<figcaption></figcaption>

- To show another example use case, let’s say we want to find out if the total sales on each market date are higher or lower than they were on the previous market date. In this example, we are going to use the `customer_purchases` table from the Farmer’s Market database, and also add in a `GROUP BY` function, which the previous examples did not include. The window functions are calculated after the grouping and aggregation occurs.
- First, we need to get the total sales per market date, using a `GROUP BY` and regular aggregate `SUM`. The results of the following query are shown in Figure 7.13.

![Figure 7.13](Fotos/Chapter7/Fig_7.13.png)
<figcaption></figcaption>

- Then, we can add the `LAG()` window function to output the previous `market_date`'s calculated sum on each row. We `ORDER BY market_date` in the window function to ensure it is the previous market date we are comparing to and not another date. You can see in Figure 7.14 that each row has a new total value (for that market date), as well as the previous market date’s total.

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

- `LEAD` works the same way as `LAG`, but it gets the value from the next row instead of the previous row (assuming the offset integer is 1). You can set the offset integer to any value `x` to count `x` rows forward, following the sort order specified in the `ORDER BY` section of the window function. If the rows are sorted by a time value, `LAG` would be retrieving data from the past, and `LEAD` would be retrieving data from the future (relative to the current row). These values can also now be used in calculations; for example, to determine the change in sales week to week.
- This chapter just covers the tip of the iceberg when it comes to window functions! Look in the documentation for the type of database you are working with to see what other functions are available, and what caveats to be aware of for each. Some database systems offer additional capabilities. For example, PostgreSQL supports something called “window naming,” Oracle has additional useful aggregate functions like `LISTAGG` (which operates on string values), and some database systems allow for additional clauses like `RANGE`.
- Once you understand the concept of a window function and how to use it in your query, you have the knowledge you need to research and apply the many variations.

# Exercises Using the Included Database

1. Do the following two steps:
   a. Write a query that selects from the `customer_purchases` table and numbers each customer’s visits to the farmer’s market. Label each market date with a different number. Each customer’s first visit is labeled 1, the second visit is labeled 2, and so on. We are not counting visits where no purchases are made, because we have no record of those. You can either display all rows in the `customer_purchases` table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. Hint: One of these approaches uses `ROW_NUMBER()` and one uses `DENSE_RANK()`.
   b. Reverse the numbering of the query from part a so each customer’s most recent visit is labeled 1. Then write another query that uses this one as a subquery and filters the results to only the customer’s most recent visit.

2. Using a `COUNT()` window function, include a value along with each row of the `customer_purchases` table that indicates how many different times that customer has purchased that `product_id`.

3. In the last query associated with Figure 7.14 from the chapter, we used `LAG` and sorted by `market_date`. Can you think of a way to use `LEAD` in place of `LAG`, but get the exact same output?