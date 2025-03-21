---
title: "SQL Chapter 3 Time 3"
author: "db"
---

# The WHERE Clause: Filtering Results

- The WHERE clause follows the FROM statement.

- It precedes any GROUP BY, ORDER BY, or LIMIT statements in a SELECT query.

```sql
SELECT
    market_date, customer_id,
    vendor_id, product_id,
    quantity,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
WHERE customer_id = 4
ORDER BY market_date, vendor_id, product_id
LIMIT 5
```

<table>
    <caption>Table 3.1</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>product_id</th>
            <th>quantity</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>1.00</td>
            <td>4.0000</td>
        </tr>
        <tr>
            <td>2019-04-06</td>
            <td>4</td>
            <td>8</td>
            <td>5</td>
            <td>1.00</td>
            <td>6.5000</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>3.00</td>
            <td>12.0000</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>5.00</td>
            <td>20.0000</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
            <td>8</td>
            <td>7</td>
            <td>2.00</td>
            <td>36.0000</td>
        </tr>
    </tbody>
</table>

- The customer_id values in the table are integers, not strings.

- If customer_id values were strings, the comparison value need to be a string, meaning '4'.

# Filtering on Multiple Conditions

- Combine multiple conditions with "AND," "OR," or "AND NOT".

- The query for filtering customer id 4 or 3 would be:

```sql
SELECT
    market_date, customer_id,
    vendor_id, product_id,
    quantity,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
WHERE customer_id = 3 OR customer_id = 4
ORDER BY market_date, customer_id, vendor_id, product_id
LIMIT 5
```

<table>
    <caption>Table 3.2</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>product_id</th>
            <th>quantity</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>7</td>
            <td>4</td>
            <td>1.00</td>
            <td>4.0000</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>1.00</td>
            <td>4.0000</td>
        </tr>
        <tr>
            <td>2019-04-06</td>
            <td>4</td>
            <td>8</td>
            <td>5</td>
            <td>1.00</td>
            <td>6.5000</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>5.00</td>
            <td>20.0000</td>
        </tr>
        <tr>
            <td>2019-04-10</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>3.00</td>
            <td>12.0000</td>
        </tr>
    </tbody>
</table>

- What happens if the WHERE clause condition is "customer_id = 3 AND customer_id = 4"?
  
  - This means "Return each row where the customer ID is 3 and 4."
  - Since a single customer_id cannot be both 3 and 4, no rows are returned.

- When using the AND operator:
  
  - All conditions with AND must be TRUE for a row to be returned.
  - Example: "WHERE customer_id > 3 AND customer_id <= 5."

```sql
SELECT
    market_date, customer_id,
    vendor_id, product_id,
    quantity,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
WHERE customer_id > 3 AND customer_id <= 5
ORDER BY market_date, customer_id, vendor_id, product_id
LIMIT 5
```

<table>
    <caption>Table 3.3</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>product_id</th>
            <th>quantity</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>7</td>
            <td>4</td>
            <td>1.00</td>
            <td>4.0000</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>7</td>
            <td>4</td>
            <td>3.00</td>
            <td>12.0000</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>5</td>
            <td>8</td>
            <td>8</td>
            <td>1.00</td>
            <td>18.0000</td>
        </tr>
        <tr>
            <td>2019-04-06</td>
            <td>4</td>
            <td>8</td>
            <td>5</td>
            <td>1.00</td>
            <td>6.5000</td>
        </tr>
        <tr>
            <td>2019-04-06</td>
            <td>5</td>
            <td>7</td>
            <td>4</td>
            <td>1.00</td>
            <td>4.0000</td>
        </tr>
    </tbody>
</table>

- You can combine multiple AND, OR, and NOT conditions.

- Use parentheses to control their evaluation order.

- Conditions inside parentheses are evaluated first.

```sql
SELECT product_id, product_name
FROM farmers_market.product
WHERE
    product_id = 10
    OR (product_id > 3 
    AND product_id < 8)
```

<table>
    <caption>Table 3.4</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Whole Wheat Bread</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Cut Zinnias Bouquet</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Apple Pie</td>
        </tr>
        <tr>
            <td>10</td>
            <td>Eggs</td>
        </tr>
    </tbody>
</table>

```sql
SELECT product_id, product_name
FROM farmers_market.product
WHERE
    (product_id = 10
    OR product_id > 3) 
    AND product_id < 8
```

<table>
    <caption>Table 3.5</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Whole Wheat Bread</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Cut Zinnias Bouquet</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Apple Pie</td>
        </tr>
    </tbody>
</table>

- The row with product_id 10 is only returned by the first query.

# Multi-Column Conditional Filtering

- So far, only one field is refered for conditions.

- WHERE clauses can use values from multiple columns.

```sql
SELECT
    market_date,
    customer_id, vendor_id,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
WHERE customer_id = 4 AND vendor_id = 7
LIMIT 5
```

<table>
    <caption>Table 3.6</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-07-06</td>
            <td>4</td>
            <td>7</td>
            <td>1.8873</td>
        </tr>
        <tr>
            <td>2019-07-10</td>
            <td>4</td>
            <td>7</td>
            <td>14.8887</td>
        </tr>
        <tr>
            <td>2019-07-17</td>
            <td>4</td>
            <td>7</td>
            <td>21.1797</td>
        </tr>
        <tr>
            <td>2019-08-03</td>
            <td>4</td>
            <td>7</td>
            <td>0.5592</td>
        </tr>
        <tr>
            <td>2019-09-04</td>
            <td>4</td>
            <td>7</td>
            <td>15.9372</td>
        </tr>
    </tbody>
</table>

```sql
SELECT *
FROM farmers_market.vendor_booth_assignments 
WHERE
    vendor_id = 9
    AND market_date <= '2019-04-20'
ORDER BY market_date
```

<table>
    <caption>Table 3.7</caption>
    <thead>
        <tr>
            <th>vendor_id</th>
            <th>booth_number</th>
            <th>market_date</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-03</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-06</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-10</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-13</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-17</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2019-04-20</td>
        </tr>
    </tbody>
</table>

# More Ways to Filter

## BETWEEN

```sql
SELECT * 
FROM farmers_market.vendor_booth_assignments 
WHERE
    vendor_id = 9
    AND market_date BETWEEN '2020-09-09' and '2020-09-23' 
ORDER BY market_date
```

<table>
    <caption>Table 3.8</caption>
    <thead>
        <tr>
            <th>vendor_id</th>
            <th>booth_number</th>
            <th>market_date</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2020-09-09</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2020-09-12</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2020-09-16</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2020-09-19</td>
        </tr>
        <tr>
            <td>9</td>
            <td>8</td>
            <td>2020-09-23</td>
        </tr>
    </tbody>
</table>

## IN

- The first query:

```sql
SELECT 
    customer_id, 
    customer_first_name, 
    customer_last_name
FROM farmers_market.customer 
WHERE
    customer_last_name = 'Diaz'
    OR customer_last_name = 'Edwards' 
    OR customer_last_name = 'Wilson'
ORDER BY customer_last_name, customer_first_name
```

- The second query:

```sql
SELECT 
    customer_id, 
    customer_first_name, 
    customer_last_name
FROM farmers_market.customer 
WHERE
    customer_last_name IN ('Diaz' , 'Edwards', 'Wilson') 
ORDER BY customer_last_name, customer_first_name
```

<table>
    <caption>Table 3.9</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_first_name</th>
            <th>customer_last_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>17</td>
            <td>Carlos</td>
            <td>Diaz</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Manuel</td>
            <td>Diaz</td>
        </tr>
        <tr>
            <td>10</td>
            <td>Russell</td>
            <td>Edwards</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Bob</td>
            <td>Wilson</td>
        </tr>
    </tbody>
</table>

- Use the IN list comparison to search for a person when unsure of the spelling.

```sql
SELECT 
    customer_id, 
    customer_first_name, 
    customer_last_name
FROM farmers_market.customer 
WHERE
    customer_first_name IN ('Renee', 'Rene', 'Renée', 'René', 'Renne')
```

## LIKE

- If you know a customer's name starts with "Jer" but aren't sure if it's "Jerry," "Jeremy," or "Jeremiah":
  - the % wildcard represents any number of characters (including none).
  - LIKE 'Jer%' will search for strings that start with "Jer" and have any (or no) characters after "r".

```sql
SELECT 
    customer_id, 
    customer_first_name, 
    customer_last_name
FROM farmers_market.customer 
WHERE
    customer_first_name LIKE 'Jer%'
```

<table>
    <caption>Table 3.10</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_first_name</th>
            <th>customer_last_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>13</td>
            <td>Jeremy</td>
            <td>Gruber</td>
        </tr>
        <tr>
            <td>18</td>
            <td>Jeri</td>
            <td>Mitchell</td>
        </tr>
    </tbody>
</table>

## IS NULL

- It's useful to find rows where a field is blank or NULL.

```sql
SELECT * 
FROM farmers_market.product 
WHERE product_size IS NULL
```

<table>
    <caption>Table 3.11</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
            <th>product_size</th>
            <th>product_category_id</th>
            <th>product_qty_type</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>14</td>
            <td>Red Potatoes</td>
            <td>NULL</td>
            <td>1</td>
            <td>NULL</td>
        </tr>
    </tbody>
</table>

- The TRIM() function removes spaces from the beginning or end of a string.

- Use TRIM() and a blank string comparison to find rows that are blank or contain only spaces.

```sql
SELECT * 
FROM farmers_market.product 
WHERE 
    product_size IS NULL 
    OR TRIM(product_size) = ''
```

<table>
    <caption>Table 3.12</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
            <th>product_size</th>
            <th>product_category_id</th>
            <th>product_qty_type</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>14</td>
            <td>Red Potatoes</td>
            <td>NULL</td>
            <td>1</td>
            <td>NULL</td>
        </tr>
        <tr>
            <td>15</td>
            <td>Red Potatoes - Small</td>
            <td>NULL</td>
            <td>1</td>
            <td>NULL</td>
        </tr>
    </tbody>
</table>

# A Warning About Null Comparisons

- Nothing "equals" NULL, not even NULL.

- This is important for other types of comparisons as well.

- Ideally, the database should prevent the quantity value from being NULL.

- To return all records that don't have NULL values in a field, use the condition "[field name] IS NOT NULL" in the WHERE clause.

# Filtering Using Subqueries

```sql
SELECT 
    market_date, 
    customer_id, vendor_id, 
    quantity * cost_to_customer_per_qty price 
FROM farmers_market.customer_purchases
WHERE 
    market_date IN
        (
        SELECT market_date
        FROM farmers_market.market_date_info 
        WHERE market_rain_flag = 1
        )
ORDER BY market_date
LIMIT 5
```

<table>
    <caption>Table 3.13</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-07-31</td>
            <td>3</td>
            <td>7</td>
            <td>18.4536</td>
        </tr>
        <tr>
            <td>2019-07-31</td>
            <td>8</td>
            <td>7</td>
            <td>26.7717</td>
        </tr>
        <tr>
            <td>2019-07-31</td>
            <td>19</td>
            <td>7</td>
            <td>25.7931</td>
        </tr>
        <tr>
            <td>2019-07-31</td>
            <td>22</td>
            <td>7</td>
            <td>7.4793</td>
        </tr>
        <tr>
            <td>2019-07-31</td>
            <td>3</td>
            <td>7</td>
            <td>8.1666</td>
        </tr>
    </tbody>
</table>

# Exercises

1. Use the data in Table 3.1. Write a query to return all customer purchases of product IDs 4 and 9.
2. Use the data in Table 3.1. Write two queries:
   - One using two conditions with an AND operator.
   - One using the BETWEEN operator.
   - Return all customer purchases from vendors with vendor IDs between 8 and 10 (inclusive).
3. Think of two ways to change the final query in the chapter to return purchases from days when it wasn’t raining.