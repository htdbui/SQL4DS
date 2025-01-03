---
title: "SQL Chapter 6 Time 2"
author: "db"
---

# GROUP BY Syntax

```sql
SELECT [columns to return]
FROM [table]
WHERE [conditional filter statements]
GROUP BY [columns to group on]
HAVING [conditional filter statements that are run after grouping]
ORDER BY [columns to sort on]
```

- `GROUP BY` is followed by column names to summarize query results.

- Without grouping, a query listing customer IDs by market date shows duplicates per purchase.

```sql
SELECT 
    market_date, customer_id
FROM farmers_market.customer_purchases 
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.1</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
        </tr>
    </tbody>
</table>

- Use `GROUP BY` to get one row per customer per market date.
  - Group by `customer_id` and `market_date`.

```sql
SELECT 
    market_date, customer_id
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.3</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
        </tr>
    </tbody>
</table>

- You could also use `SELECT DISTINCT` to remove duplicates. This query provides the same result:

```sql
SELECT DISTINCT 
    market_date, customer_id
FROM farmers_market.customer_purchases 
ORDER BY market_date, customer_id
LIMIT 5
```

# Displaying Group Summaries

- After grouping, add aggregate functions like `SUM` and `COUNT` to summarize data.
- Use `COUNT()` to count rows in `customer_purchases` per market date per customer.

```sql
SELECT 
    market_date, customer_id,
    COUNT(*) AS items_purchased
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.4</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>items_purchased</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

- The granularity of `customer_purchases` affects `items_purchased`.
  - Three identical items bought at once show as 1 row with quantity 3.

- Separate purchases generate new rows.

- To count all items, sum the quantity column.

```sql
SELECT 
    market_date, customer_id,
    SUM(quantity) AS items_purchased
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.5</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>items_purchased</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>1.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>4.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
            <td>5.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>5.00</td>
        </tr>
    </tbody>
</table>

- To find how many different items each customer bought, count unique `product_id` values.
  - Use `DISTINCT` on `product_id` instead of `COUNT(*)` or `SUM(quantity)`.
  - This shows the variety of products each customer bought per market date.

```sql
SELECT 
    market_date, customer_id, 
    COUNT(DISTINCT product_id) AS different_products_purchased
FROM farmers_market.customer_purchases
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.6</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>different_products_purchased</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

- We can combine these summaries into a single query.

```sql
SELECT 
    market_date, customer_id, 
    SUM(quantity) AS items_purchased,
    COUNT(DISTINCT product_id) AS different_products_purchased
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.7</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>items_purchased</th>
            <th>different_products_purchased</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1.00</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>1.00</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>4.00</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
            <td>5.00</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>5.00</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

# Performing Calculations Inside Aggregate Functions

- Aggregate functions can include mathematical operations, calculated per row before summarizing.

```sql
SELECT 
    market_date, 
    customer_id, vendor_id, 
    quantity * cost_to_customer_per_qty AS price 
FROM farmers_market.customer_purchases
WHERE 
    customer_id = 3
ORDER BY market_date, vendor_id
LIMIT 5
```

<table>
  <caption>Table 6.8</caption>
  <tr>
    <th>market_date</th>
    <th>customer_id</th>
    <th>vendor_id</th>
    <th>price</th>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>3</td>
    <td>4.0000</td>
  </tr>
  <tr>
    <td>2019-04-13</td>
    <td>3</td>
    <td>2</td>
    <td>18.0000</td>
  </tr>
  <tr>
    <td>2019-04-13</td>
    <td>3</td>
    <td>4</td>
    <td>18.0000</td>
  </tr>
  <tr>
    <td>2019-04-13</td>
    <td>3</td>
    <td>4</td>
    <td>16.0000</td>
  </tr>
  <tr>
    <td>2019-04-13</td>
    <td>3</td>
    <td>4</td>
    <td>4.0000</td>
  </tr>
</table>

- To find total spending per customer on each `market_date`, group by `market_date` and use `SUM` to add item prices.

```sql
SELECT 
    customer_id, market_date,
    SUM(quantity * cost_to_customer_per_qty) AS total_spent
FROM farmers_market.customer_purchases 
WHERE 
    customer_id = 3 
GROUP BY market_date
ORDER BY market_date
LIMIT 5
```

<table>
    <caption>Table 6.9</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>market_date</th>
            <th>total_spent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>3</td>
            <td>2019-04-03</td>
            <td>4.0000</td>
        </tr>
        <tr>
            <td>3</td>
            <td>2019-04-13</td>
            <td>56.0000</td>
        </tr>
        <tr>
            <td>3</td>
            <td>2019-04-24</td>
            <td>20.0000</td>
        </tr>
        <tr>
            <td>3</td>
            <td>2019-04-27</td>
            <td>72.0000</td>
        </tr>
        <tr>
            <td>3</td>
            <td>2019-05-01</td>
            <td>52.0000</td>
        </tr>
    </tbody>
</table>

- `vendor_id` is removed from display and `ORDER BY` to get one row per customer per date.
  - Including it prevents aggregation, as customers can buy from multiple vendors on the same date.

- Add `customer_id` to `GROUP BY` to ensure the query works without errors. This query will provide the same result:

```sql
...
GROUP BY market_date, customer_id
...
```

- To find total spending per vendor, group by `customer_id` and `vendor_id`.

```sql
SELECT 
    customer_id, vendor_id,
    SUM(quantity * cost_to_customer_per_qty) AS total_spent 
FROM farmers_market.customer_purchases
WHERE 
    customer_id = 3
GROUP BY customer_id, vendor_id
ORDER BY customer_id, vendor_id
```

<table>
  <caption>Table 6.10</caption>
  <tr>
    <th>customer_id</th>
    <th>vendor_id</th>
    <th>total_spent</th>
  </tr>
  <tr>
    <td>3</td>
    <td>1</td>
    <td>291.9536</td>
  </tr>
  <tr>
    <td>3</td>
    <td>2</td>
    <td>332.3101</td>
  </tr>
  <tr>
    <td>3</td>
    <td>3</td>
    <td>713.0967</td>
  </tr>
  <tr>
    <td>3</td>
    <td>4</td>
    <td>599.0258</td>
  </tr>
  <tr>
    <td>3</td>
    <td>5</td>
    <td>310.5352</td>
  </tr>
  <tr>
    <td>3</td>
    <td>6</td>
    <td>438.3000</td>
  </tr>
  <tr>
    <td>3</td>
    <td>7</td>
    <td>242.4963</td>
  </tr>
  <tr>
    <td>3</td>
    <td>8</td>
    <td>558.4940</td>
  </tr>
  <tr>
    <td>3</td>
    <td>9</td>
    <td>345.9458</td>
  </tr>
</table>

- Again, this query provides the same result:

```sql
...
GROUP BY vendor_id
ORDER BY vendor_id
```

- Remove the `WHERE` clause to remove the `customer_id` filter.

- `GROUP BY` `customer_id` to list each customer and their total spending.

```sql
SELECT 
    customer_id, vendor_id,
    SUM(quantity * cost_to_customer_per_qty) AS total_spent
FROM farmers_market.customer_purchases
GROUP BY customer_id, vendor_id
ORDER BY customer_id, vendor_id
LIMIT 5
```

<table>
  <caption>Table 6.11</caption>
  <tr>
    <th>customer_id</th>
    <th>vendor_id</th>
    <th>total_spent</th>
  </tr>
  <tr>
    <td>1</td>
    <td>1</td>
    <td>303.7412</td>
  </tr>
  <tr>
    <td>1</td>
    <td>2</td>
    <td>371.8064</td>
  </tr>
  <tr>
    <td>1</td>
    <td>3</td>
    <td>374.9069</td>
  </tr>
  <tr>
    <td>1</td>
    <td>4</td>
    <td>425.1200</td>
  </tr>
  <tr>
    <td>1</td>
    <td>5</td>
    <td>419.9807</td>
  </tr>
</table>

- Aggregation can be done on joined tables too.
  - Join tables first, ensuring no duplicated fields.
  - Then, add the `GROUP BY`.

- To add customer and vendor details:
  - Join the three tables.
  - Inspect the output before grouping.

```sql
SELECT
    c.customer_first_name,
    c.customer_last_name,
    cp.customer_id,
    v.vendor_name,
    cp.vendor_id,
    cp.quantity * cp.cost_to_customer_per_qty AS price
FROM farmers_market.customer c
    LEFT JOIN farmers_market.customer_purchases cp
        ON c.customer_id = cp.customer_id 
    LEFT JOIN farmers_market.vendor v
        ON cp.vendor_id = v.vendor_id 
WHERE 
    cp.customer_id = 3
ORDER BY cp.customer_id, cp.vendor_id
LIMIT 5
```

<table>
  <caption>Table 6.12</caption>
  <tr>
    <th>customer_first_name</th>
    <th>customer_last_name</th>
    <th>customer_id</th>
    <th>vendor_name</th>
    <th>vendor_id</th>
    <th>price</th>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>18.4536</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>1.0000</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>4.0000</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>8.0000</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>20.0000</td>
  </tr>
</table>

- To summarize at the level of one row per vendor for customer id 3:

```sql
SELECT 
    c.customer_first_name,
    c.customer_last_name,
    cp.customer_id,
    v.vendor_name,
    cp.vendor_id,
    SUM(quantity * cost_to_customer_per_qty) AS total_spent
FROM farmers_market.customer c
    LEFT JOIN farmers_market.customer_purchases cp
        ON c.customer_id = cp.customer_id 
    LEFT JOIN farmers_market.vendor v
        ON cp.vendor_id = v.vendor_id 
WHERE 
    cp.customer_id = 3 
GROUP BY 
    c.customer_first_name, 
    c.customer_last_name, 
    cp.customer_id, 
    v.vendor_name, 
    cp.vendor_id
ORDER BY cp.customer_id, cp.vendor_id
```

<table>
  <caption>Table 6.13</caption>
  <tr>
    <th>customer_first_name</th>
    <th>customer_last_name</th>
    <th>customer_id</th>
    <th>vendor_name</th>
    <th>vendor_id</th>
    <th>total_spent</th>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Chris's Sustainable Eggs & Meats</td>
    <td>1</td>
    <td>291.9536</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Hernández Salsa & Veggies</td>
    <td>2</td>
    <td>332.3101</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Mountain View Vegetables</td>
    <td>3</td>
    <td>713.0967</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Fields of Corn</td>
    <td>4</td>
    <td>599.0258</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Seashell Clay Shop</td>
    <td>5</td>
    <td>310.5352</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Mother's Garlic & Greens</td>
    <td>6</td>
    <td>438.3000</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Marco's Peppers</td>
    <td>7</td>
    <td>242.4963</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>558.4940</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Mediterranean Bakery</td>
    <td>9</td>
    <td>345.9458</td>
  </tr>
</table>

- Filter by vendor instead of customer to get a list of customers for a vendor.
  - Change the `WHERE` clause.
  - Grouping and output fields stay the same.
  - `customer_id` now varies, and `vendor_id` is limited to vendor 8.

```sql
SELECT 
    c.customer_first_name,
    c.customer_last_name,
    cp.customer_id,
    v.vendor_name,
    cp.vendor_id,
    SUM(quantity * cost_to_customer_per_qty) AS total_spent
FROM farmers_market.customer c
    LEFT JOIN farmers_market.customer_purchases cp
        ON c.customer_id = cp.customer_id 
    LEFT JOIN farmers_market.vendor v
        ON cp.vendor_id = v.vendor_id 
WHERE 
    cp.vendor_id = 8
GROUP BY
    c.customer_first_name, 
    c.customer_last_name, 
    cp.customer_id, 
    v.vendor_name, 
    cp.vendor_id
ORDER BY cp.customer_id, cp.vendor_id
LIMIT 5
```

<table>
  <caption>Table 6.14</caption>
  <tr>
    <th>customer_first_name</th>
    <th>customer_last_name</th>
    <th>customer_id</th>
    <th>vendor_name</th>
    <th>vendor_id</th>
    <th>total_spent</th>
  </tr>
  <tr>
    <td>Jane</td>
    <td>Connor</td>
    <td>1</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>506.1768</td>
  </tr>
  <tr>
    <td>Manuel</td>
    <td>Diaz</td>
    <td>2</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>614.0794</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>Wilson</td>
    <td>3</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>558.4940</td>
  </tr>
  <tr>
    <td>Deanna</td>
    <td>Washington</td>
    <td>4</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>456.9701</td>
  </tr>
  <tr>
    <td>Abigail</td>
    <td>Harris</td>
    <td>5</td>
    <td>Annie's Pies</td>
    <td>8</td>
    <td>598.3710</td>
  </tr>
</table>

- Removing the `WHERE` clause gives a row for every customer-vendor pair.
  - Useful for front-end filtering in tools like `Tableau`.
  - This query lists each customer and vendor, showing the total spent.
  - Users can dynamically filter by customer or vendor in the reporting tool.

# MIN and MAX

- To find the most and least expensive items per product category in the `vendor_inventory` table.
  - Vendors set and adjust prices per customer.
  - The `customer_purchases` table has a `cost_to_customer_per_qty` field for price overrides (selling price).
  - The `vendor_inventory` table has original prices per item on each market date.
  
- We can find the least and most expensive item prices in the entire table by `MIN()` and `MAX()` functions.

```sql
SELECT 
    MIN(original_price) AS minimum_price,
    MAX(original_price) AS maximum_price 
FROM farmers_market.vendor_inventory 
ORDER BY original_price
```

<table>
    <caption>Table 6.15</caption>
    <thead>
        <tr>
            <th>minimum_price</th>
            <th>maximum_price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0.50</td>
            <td>18.00</td>
        </tr>
    </tbody>
</table>

- To get the lowest and highest prices within each product category:
  - Group by `product_category_id`, which appears in the table `Product`.
  - Display `product_category_name`, which appears in the table `Product Category`
  - Table aliases are used to distinguish fields from multiple tables.

```sql
SELECT 
    pc.product_category_name,
    p.product_category_id, 
    MIN(vi.original_price) AS minimum_price, 
    MAX(vi.original_price) AS maximum_price
FROM farmers_market.vendor_inventory AS vi 
    INNER JOIN farmers_market.product AS p
        ON vi.product_id = p.product_id
    INNER JOIN farmers_market.product_category AS pc
        ON p.product_category_id = pc.product_category_id 
GROUP BY pc.product_category_name, p.product_category_id
```

<table>
    <caption>Table 6.16</caption>
    <thead>
        <tr>
            <th>product_category_name</th>
            <th>product_category_id</th>
            <th>minimum_price</th>
            <th>maximum_price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Fresh Fruits & Vegetables</td>
            <td>1</td>
            <td>0.50</td>
            <td>18.00</td>
        </tr>
        <tr>
            <td>Packaged Prepared Food</td>
            <td>3</td>
            <td>0.50</td>
            <td>18.00</td>
        </tr>
    </tbody>
</table>

- Adding `MIN(product_name)` and `MAX(product_name)` columns will not give the product names with the lowest and highest prices, but the alphabetically first and last product names.

- To find the products with the min and max prices per category, use window functions, which will be explained in the next chapter.

# COUNT and COUNT DISTINCT

- To count products for sale on each market date or how many different products each vendor offered, use `COUNT` and `COUNT DISTINCT`.

- `COUNT` counts rows within a group when used with `GROUP BY`.

- `COUNT DISTINCT` counts unique values in the specified field within the group.

- To find the number of products offered each market date:
  - Count rows in the `vendor_inventory` table, grouped by date.
  - This gives the number of products available, as each row represents a product for each vendor on each market date.

```sql
SELECT 
    market_date, 
    COUNT(product_id) AS product_count 
FROM farmers_market.vendor_inventory 
GROUP BY market_date
ORDER BY market_date
LIMIT 5
```

<table>
    <caption>Table 6.17</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>product_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-06</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-13</td>
            <td>4</td>
        </tr>
        <tr>
            <td>2019-04-17</td>
            <td>4</td>
        </tr>
    </tbody>
</table>

- If we wanted to know how many different products with unique `product_id`s each vendor brought to market during a date range:
  - Use `COUNT DISTINCT` on the `product_id` field.

```sql
SELECT 
    vendor_id, 
    COUNT(DISTINCT product_id) AS different_products_offered 
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-20'
GROUP BY vendor_id
ORDER BY vendor_id
```

- Note that the `DISTINCT` goes inside the parentheses for the `COUNT()` aggregate function.

<table>
  <caption>Table 6.18</caption>
  <tr>
    <th>vendor_id</th>
    <th>different_products_offered</th>
  </tr>
  <tr>
    <td>2</td>
    <td>3</td>
  </tr>
  <tr>
    <td>3</td>
    <td>2</td>
  </tr>
  <tr>
    <td>4</td>
    <td>3</td>
  </tr>
  <tr>
    <td>5</td>
    <td>2</td>
  </tr>
  <tr>
    <td>6</td>
    <td>3</td>
  </tr>
  <tr>
    <td>7</td>
    <td>1</td>
  </tr>
  <tr>
    <td>8</td>
    <td>2</td>
  </tr>
  <tr>
    <td>9</td>
    <td>2</td>
  </tr>
</table>

# Average

- What if we also want the average original price of a product per vendor, in addition to the count of different products per vendor?
  - We can add a line to the previous query and use the `AVG()` function.

```sql
SELECT 
    vendor_id, 
    COUNT(DISTINCT product_id) AS different_products_offered, 
    AVG(original_price) AS average_product_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-20' 
GROUP BY vendor_id
ORDER BY vendor_id
```

<table>
  <caption>Table 6.19</caption>
  <tr>
    <th>vendor_id</th>
    <th>different_products_offered</th>
    <th>average_product_price</th>
  </tr>
  <tr>
    <td>2</td>
    <td>3</td>
    <td>9.100000</td>
  </tr>
  <tr>
    <td>3</td>
    <td>2</td>
    <td>8.625000</td>
  </tr>
  <tr>
    <td>4</td>
    <td>3</td>
    <td>11.500000</td>
  </tr>
  <tr>
    <td>5</td>
    <td>2</td>
    <td>9.750000</td>
  </tr>
  <tr>
    <td>6</td>
    <td>3</td>
    <td>5.100000</td>
  </tr>
  <tr>
    <td>7</td>
    <td>1</td>
    <td>16.500000</td>
  </tr>
  <tr>
    <td>8</td>
    <td>2</td>
    <td>9.500000</td>
  </tr>
  <tr>
    <td>9</td>
    <td>2</td>
    <td>1.500000</td>
  </tr>
</table>

- We need to consider the meaning of "average product price."
  - The table has one row per product type, so each price is included only once.
  - For example, if a vendor sells 100 tomatoes and bouquets at $20, both prices count only once.
  - This calculation gives the average of one tomato and one bouquet.

- To get a true average price:
  - Multiply each item's quantity by its price.
  - Sum these values and divide by the total quantity of items.
  - Let's perform a calculation using these summary values.

```sql
SELECT 
    vendor_id, 
    COUNT(DISTINCT product_id) AS different_products_offered, 
    SUM(quantity * original_price) AS value_of_inventory, 
    SUM(quantity) AS inventory_item_count,
    SUM(quantity * original_price) / SUM(quantity) AS average_item_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-20'
GROUP BY vendor_id
ORDER BY vendor_id
```

- Multiply `quantity * original_price` per row, then calculate the aggregate `SUM`s.

- Divide one `SUM` by the other to get the “average item price.”

- This involves operations both before and after `GROUP BY` summarization.

<table>
  <caption>Table 6.20</caption>
  <tr>
    <th>vendor_id</th>
    <th>different_products_offered</th>
    <th>value_of_inventory</th>
    <th>inventory_item_count</th>
    <th>average_item_price</th>
  </tr>
  <tr>
    <td>2</td>
    <td>3</td>
    <td>700.0000</td>
    <td>70.00</td>
    <td>10.00000000</td>
  </tr>
  <tr>
    <td>3</td>
    <td>2</td>
    <td>921.0000</td>
    <td>110.00</td>
    <td>8.37272727</td>
  </tr>
  <tr>
    <td>4</td>
    <td>3</td>
    <td>419.0000</td>
    <td>46.00</td>
    <td>9.10869565</td>
  </tr>
  <tr>
    <td>5</td>
    <td>2</td>
    <td>474.0000</td>
    <td>63.00</td>
    <td>7.52380952</td>
  </tr>
  <tr>
    <td>6</td>
    <td>3</td>
    <td>253.0000</td>
    <td>86.00</td>
    <td>2.94186047</td>
  </tr>
  <tr>
    <td>7</td>
    <td>1</td>
    <td>115.5000</td>
    <td>7.00</td>
    <td>16.50000000</td>
  </tr>
  <tr>
    <td>8</td>
    <td>2</td>
    <td>488.0000</td>
    <td>46.00</td>
    <td>10.60869565</td>
  </tr>
  <tr>
    <td>9</td>
    <td>2</td>
    <td>24.0000</td>
    <td>16.00</td>
    <td>1.50000000</td>
  </tr>
</table>

# Filtering with HAVING

- Filtering can occur after summarization with the `HAVING` clause.
  - `WHERE` filters rows before grouping, as shown previously.

- To filter after aggregation:
  - Use the `HAVING` clause to filter groups based on summary values.

- For example, to filter vendors with more 220 items over a period:
  - Add a `HAVING` clause to the query.

```sql
SELECT 
    vendor_id, 
    COUNT(DISTINCT product_id) AS different_products_offered, 
    SUM(quantity * original_price) AS value_of_inventory,
    SUM(quantity) AS inventory_item_count,
    SUM(quantity * original_price) / SUM(quantity) AS average_item_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-20' 
GROUP BY vendor_id
HAVING inventory_item_count > 50
ORDER BY vendor_id
```

<table>
  <caption>Table 6.21</caption>
  <tr>
    <th>vendor_id</th>
    <th>different_products_offered</th>
    <th>value_of_inventory</th>
    <th>inventory_item_count</th>
    <th>average_item_price</th>
  </tr>
  <tr>
    <td>2</td>
    <td>3</td>
    <td>700.0000</td>
    <td>70.00</td>
    <td>10.00000000</td>
  </tr>
  <tr>
    <td>3</td>
    <td>2</td>
    <td>921.0000</td>
    <td>110.00</td>
    <td>8.37272727</td>
  </tr>
  <tr>
    <td>5</td>
    <td>2</td>
    <td>474.0000</td>
    <td>63.00</td>
    <td>7.52380952</td>
  </tr>
  <tr>
    <td>6</td>
    <td>3</td>
    <td>253.0000</td>
    <td>86.00</td>
    <td>2.94186047</td>
  </tr>
</table>

- Use `GROUP BY` on all distinct fields, then add a `HAVING COUNT(*) > 1` clause.

- Any returned results indicate unwanted duplicates in your dataset.

# CASE Statements Inside Aggregate Functions

- Earlier in this chapter, in the query that generated the output in Table 6.7, we added up the quantity value in the `customer_purchases` table.
  - This included discrete items sold individually as well as bulk items sold by ounce or pound.
  - It was awkward to add those quantities together.

- In Chapter 4, “Conditionals / CASE Statements,” you learned about conditional `CASE` statements.
  - Here, we’ll use a `CASE` statement to specify which type of item quantities to add together using each `SUM` aggregate function.
- First, we’ll need to `JOIN` the `customer_purchases` table to the `product` table to pull in the `product_qty_type` column.
  - This column currently only contains the values “unit” and “lbs.”

- In Table 6.7, we added quantity values from `customer_purchases`, mixing individual and bulk items, which was awkward.

- Using `CASE` statements, as learned in Chapter 4, we can specify which item quantities to sum.

- First, `JOIN` the `customer_purchases` table with the `product` table to get the `product_qty_type` column, which contains "unit" and "lbs."

```sql
SELECT 
    cp.market_date,
    cp.vendor_id, 
    cp.customer_id, 
    cp.product_id, 
    cp.quantity, 
    p.product_name, 
    p.product_size, 
    p.product_qty_type
FROM farmers_market.customer_purchases AS cp
    INNER JOIN farmers_market.product AS p
        ON cp.product_id = p.product_id
ORDER BY cp.market_date
LIMIT 5
```

<table>
  <caption>Table 6.22</caption>
  <tr>
    <th>market_date</th>
    <th>vendor_id</th>
    <th>customer_id</th>
    <th>product_id</th>
    <th>quantity</th>
    <th>product_name</th>
    <th>product_size</th>
    <th>product_qty_type</th>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>3</td>
    <td>4</td>
    <td>1.00</td>
    <td>Banana Peppers - Jar</td>
    <td>8 oz</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>4</td>
    <td>4</td>
    <td>1.00</td>
    <td>Banana Peppers - Jar</td>
    <td>8 oz</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>5</td>
    <td>4</td>
    <td>3.00</td>
    <td>Banana Peppers - Jar</td>
    <td>8 oz</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>6</td>
    <td>4</td>
    <td>4.00</td>
    <td>Banana Peppers - Jar</td>
    <td>8 oz</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>7</td>
    <td>4</td>
    <td>5.00</td>
    <td>Banana Peppers - Jar</td>
    <td>8 oz</td>
    <td>unit</td>
  </tr>
</table>

- Create columns to add quantities sold by unit, by pound, and a third for other units (like bulk ounces).
- Review results with `CASE` statements before grouping:
  - The `CASE` statements separate quantities into three columns by `product_qty_type`.
  - These values will be summed per group next.

```sql
SELECT 
    cp.market_date,
    cp.vendor_id,
    cp.customer_id,
    cp.product_id,
    CASE WHEN product_qty_type = "unit" THEN quantity ELSE 0 END AS 
quantity_units,
    CASE WHEN product_qty_type = "lbs" THEN quantity ELSE 0 END AS 
quantity_lbs,
    CASE WHEN product_qty_type NOT IN ("unit","lbs") THEN quantity ELSE 0 END AS quantity_other,
    p.product_qty_type
FROM farmers_market.customer_purchases cp
    INNER JOIN farmers_market.product p 
        ON cp.product_id = p.product_id
ORDER BY cp.market_date
LIMIT 5
```

<table>
  <caption>Table 6.23</caption>
  <tr>
    <th>market_date</th>
    <th>vendor_id</th>
    <th>customer_id</th>
    <th>product_id</th>
    <th>quantity_units</th>
    <th>quantity_lbs</th>
    <th>quantity_other</th>
    <th>product_qty_type</th>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>3</td>
    <td>4</td>
    <td>1.00</td>
    <td>0</td>
    <td>0</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>4</td>
    <td>4</td>
    <td>1.00</td>
    <td>0</td>
    <td>0</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>5</td>
    <td>4</td>
    <td>3.00</td>
    <td>0</td>
    <td>0</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>6</td>
    <td>4</td>
    <td>4.00</td>
    <td>0</td>
    <td>0</td>
    <td>unit</td>
  </tr>
  <tr>
    <td>2019-04-03</td>
    <td>3</td>
    <td>7</td>
    <td>4</td>
    <td>5.00</td>
    <td>0</td>
    <td>0</td>
    <td>unit</td>
  </tr>
</table>

- Add `SUM` functions around each `CASE` statement to sum values per market date per customer, as defined in the `GROUP BY` clause.

```sql
SELECT 
    cp.market_date,
    cp.customer_id,
    SUM(CASE WHEN product_qty_type = "unit" THEN quantity ELSE 0 END) AS qty_units_purchased,
    SUM(CASE WHEN product_qty_type = "lbs" THEN quantity ELSE 0 END) AS qty_lbs_purchased,
    SUM(CASE WHEN product_qty_type NOT IN ("unit","lbs") THEN quantity 
ELSE 0 END) AS qty_other_purchased
FROM farmers_market.customer_purchases cp
    INNER JOIN farmers_market.product p 
        ON cp.product_id = p.product_id
GROUP BY market_date, customer_id 
ORDER BY market_date, customer_id
LIMIT 5
```

<table>
    <caption>Table 6.25</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>qty_units_purchased</th>
            <th>qty_lbs_purchased</th>
            <th>qty_other_purchased</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1.00</td>
            <td>0.00</td>
            <td>0.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>1.00</td>
            <td>0.00</td>
            <td>0.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>4.00</td>
            <td>0.00</td>
            <td>0.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>6</td>
            <td>5.00</td>
            <td>0.00</td>
            <td>0.00</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>5.00</td>
            <td>0.00</td>
            <td>0.00</td>
        </tr>
    </tbody>
</table>

# Exercises Using the Included Database

1. Write a query that determines how many times each vendor has rented a booth at the farmer’s market.
  - Count the vendor booth assignments per `vendor_id`.

2. In Chapter 5, “SQL Joins,” Exercise 3, we asked, “When is each type of fresh fruit or vegetable in season, locally?”
  - Write a query that displays the product category name, product name, earliest date available, and latest date available for every product in the “Fresh Fruits & Vegetables” product category.

3. The Farmer’s Market Customer Appreciation Committee wants to give a bumper sticker to everyone who has ever spent more than $50 at the market.
  - Write a query that generates a list of customers for them to give stickers to.
  - Sort by last name, then first name.
  - (Hint: This query requires you to join two tables, use an aggregate function, and use the `HAVING` keyword.)