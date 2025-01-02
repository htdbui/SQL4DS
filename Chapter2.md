---
title: "SQL Chapter 2 Time 3"
author: "db"
---

# The Syntax of a SELECT Query

```sql
SELECT [columns to return]
FROM [schema.table]
WHERE [conditional filter statements]
GROUP BY [columns to group on]
HAVING [conditional filter statements that are run after grouping]
ORDER BY [columns to sort on]
```

- `SELECT`: Choose columns to show. **Required**.
- `FROM`: Choose the table (and schema). **Required**.
- `WHERE`: Filter rows before grouping or selecting.
- `GROUP BY`: Group rows with the same values.
- `HAVING`: Filter groups after grouping.
- `ORDER BY`: Sort the data by columns.

# Selecting Columns and Limiting the Number of Rows

- `FROM schema.table` specifies the schema and table name.

```sql
SELECT * FROM farmers_market.product
```

<table>
    <caption>Table 2.1</caption>
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
            <td>1</td>
            <td>Habanero Peppers - Organic</td>
            <td>medium</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Jalapeno Peppers - Organic</td>
            <td>small</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Poblano Peppers - Organic</td>
            <td>large</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
            <td>8 oz</td>
            <td>3</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Whole Wheat Bread</td>
            <td>1.5 lbs</td>
            <td>3</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Cut Zinnias Bouquet</td>
            <td>medium</td>
            <td>5</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Apple Pie</td>
            <td>10"</td>
            <td>3</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>8</td>
            <td>Cherry Pie</td>
            <td>10"</td>
            <td>3</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>9</td>
            <td>Sweet Potatoes</td>
            <td>medium</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>10</td>
            <td>Eggs</td>
            <td>1 dozen</td>
            <td>6</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>11</td>
            <td>Pork Chops</td>
            <td>1 lb</td>
            <td>6</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>12</td>
            <td>Baby Salad Lettuce Mix - Bag</td>
            <td>1/2 lb</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>13</td>
            <td>Baby Salad Lettuce Mix</td>
            <td>1 lb</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
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
            <td></td>
            <td>1</td>
            <td>NULL</td>
        </tr>
        <tr>
            <td>16</td>
            <td>Sweet Corn</td>
            <td>Ear</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>17</td>
            <td>Carrots</td>
            <td>sold by weight</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>18</td>
            <td>Carrots - Organic</td>
            <td>bunch</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>19</td>
            <td>Farmer's Market Resuable Shopping Bag</td>
            <td>medium</td>
            <td>7</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>20</td>
            <td>Homemade Beeswax Candles</td>
            <td>6"</td>
            <td>7</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>21</td>
            <td>Organic Cherry Tomatoes</td>
            <td>pint</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>22</td>
            <td>Roma Tomatoes</td>
            <td>medium</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>23</td>
            <td>Maple Syrup - Jar</td>
            <td>8 oz</td>
            <td>2</td>
            <td>unit</td>
        </tr>
    </tbody>
</table>

- Limit for the first five rows:

```sql
SELECT *
FROM farmers_market.product
LIMIT 5
```

<table>
    <caption>Table 2.2</caption>
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
            <td>1</td>
            <td>Habanero Peppers - Organic</td>
            <td>medium</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Jalapeno Peppers - Organic</td>
            <td>small</td>
            <td>1</td>
            <td>lbs</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Poblano Peppers - Organic</td>
            <td>large</td>
            <td>1</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
            <td>8 oz</td>
            <td>3</td>
            <td>unit</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Whole Wheat Bread</td>
            <td>1.5 lbs</td>
            <td>3</td>
            <td>unit</td>
        </tr>
    </tbody>
</table>

- Different database systems use different syntax to limit results:
  - MS SQL Server: Use `TOP` before `SELECT`.
  - Oracle: Use `WHERE ROWNUM <= number`.
  - MySQL: Use `LIMIT` at the end of the query.

- Use line breaks and indentation for readability. It does not affect execution.
- To specify columns, list column names after SELECT, separated by commas.

```sql
SELECT product_id, product_name
FROM farmers_market.product
LIMIT 5
```

<table>
    <caption>Table 2.3</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Habanero Peppers - Organic</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Jalapeno Peppers - Organic</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Poblano Peppers - Organic</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Whole Wheat Bread</td>
        </tr>
    </tbody>
</table>

- List column names explicitly instead of using `*`:
  - `Schema Changes`: `SELECT *` can cause issues if the table changes.
  - `Consistent Output`: Listing columns ensures consistent output.

- Preventing Breakages:
  - `Automated Processes`: Unexpected changes can cause failures.
  - `Error Detection`: Listing columns alerts you if a column is removed or renamed.

# The ORDER BY Clause: Sorting Results

- `ORDER BY` clause sorts rows by columns.
  - Sort order: `ASC` (ascending) or `DESC` (descending).
  - `ASC`: Text alphabetically, numbers low to high.
  - `DESC`: Reverse order.
  - MySQL: `NULL` values first in ascending order.
  - Default: Ascending.

- ASC:

```sql
SELECT product_id, product_name 
FROM farmers_market.product 
ORDER BY product_name
LIMIT 5
```

<table>
    <caption>Table 2.4</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7</td>
            <td>Apple Pie</td>
        </tr>
        <tr>
            <td>13</td>
            <td>Baby Salad Lettuce Mix</td>
        </tr>
        <tr>
            <td>12</td>
            <td>Baby Salad Lettuce Mix - Bag</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Banana Peppers - Jar</td>
        </tr>
        <tr>
            <td>17</td>
            <td>Carrots</td>
        </tr>
    </tbody>
</table>

- DESC:

<table>
    <caption>Table 2.5</caption>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>23</td>
            <td>Maple Syrup - Jar</td>
        </tr>
        <tr>
            <td>22</td>
            <td>Roma Tomatoes</td>
        </tr>
        <tr>
            <td>21</td>
            <td>Organic Cherry Tomatoes</td>
        </tr>
        <tr>
            <td>20</td>
            <td>Homemade Beeswax Candles</td>
        </tr>
        <tr>
            <td>19</td>
            <td>Farmer's Market Resuable Shopping Bag</td>
        </tr>
    </tbody>
</table>

- Rows in Table 2.5 differ because `ORDER BY` runs before `LIMIT`.
  - `LIMIT` must be after `ORDER BY`.
- First, sort by `market_date`, then by `vendor_id`.

```sql
SELECT market_date, vendor_id, booth_number
FROM farmers_market.vendor_booth_assignments
ORDER BY market_date, vendor_id
LIMIT 5
```

<table>
    <caption>Table 2.6</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>vendor_id</th>
            <th>booth_number</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-04-03</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>3</td>
            <td>1</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>4</td>
            <td>7</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>7</td>
            <td>11</td>
        </tr>
        <tr>
            <td>2019-04-03</td>
            <td>8</td>
            <td>6</td>
        </tr>
    </tbody>
</table>

# Simple Inline Calculations

```sql
SELECT
    market_date, customer_id,
    vendor_id, quantity,
    cost_to_customer_per_qty
FROM farmers_market.customer_purchases
LIMIT 5
```

<table>
    <caption>Table 2.7</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>quantity</th>
            <th>cost_to_customer_per_qty</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>0.99</td>
            <td>6.99</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>2.18</td>
            <td>6.99</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>15</td>
            <td>7</td>
            <td>1.53</td>
            <td>6.99</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>16</td>
            <td>7</td>
            <td>2.02</td>
            <td>6.99</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>22</td>
            <td>7</td>
            <td>0.66</td>
            <td>6.99</td>
        </tr>
    </tbody>
</table>

- Multiply `quantity` and `cost_per_quantity`:

```sql
SELECT
    market_date, customer_id,
    vendor_id, quantity,
    cost_to_customer_per_qty,
    quantity * cost_to_customer_per_qty
FROM farmers_market.customer_purchases
LIMIT 5
```

<table>
    <caption>Table 2.8</caption>
    <thead>
        <tr>
            <th>market_date</th>
            <th>customer_id</th>
            <th>vendor_id</th>
            <th>quantity</th>
            <th>cost_to_customer_per_qty</th>
            <th>quantity * cost_to_customer_per_qty</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>0.99</td>
            <td>6.99</td>
            <td>6.9201</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>2.18</td>
            <td>6.99</td>
            <td>15.2382</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>15</td>
            <td>7</td>
            <td>1.53</td>
            <td>6.99</td>
            <td>10.6947</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>16</td>
            <td>7</td>
            <td>2.02</td>
            <td>6.99</td>
            <td>14.1198</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>22</td>
            <td>7</td>
            <td>0.66</td>
            <td>6.99</td>
            <td>4.6134</td>
        </tr>
    </tbody>
</table>

- Use `AS` to give a calculated column a meaningful name.
  - Example: Assign alias `price` to the result.

```sql
SELECT
    market_date, customer_id,
    vendor_id,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
LIMIT 5
```

<table>
    <caption>Table 2.9</caption>
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
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>6.9201</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>15.2382</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>15</td>
            <td>7</td>
            <td>10.6947</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>16</td>
            <td>7</td>
            <td>14.1198</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>22</td>
            <td>7</td>
            <td>4.6134</td>
        </tr>
    </tbody>
</table>

- `AS` is optional in MySQL.
- The query will return the same result without it.

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    quantity * cost_to_customer_per_qty price
FROM farmers_market.customer_purchases
LIMIT 5
```

- Aggregating Data:
  - Summarize data across multiple rows.
  - Examples: `SUM`, `AVG`, `COUNT`.
  - Covered in Chapter 6.

- Current calculations apply to each row individually.
  - Example: Calculate price by multiplying quantity by cost per unit.
  - These do not summarize or combine data across rows.

# More Inline Calculation

- A `SQL` function takes inputs, performs an operation, and returns a value.

- Use functions to modify raw values before displaying them.

- Syntax for calling a `SQL` function:
  - Input parameters can be a column name or a constant value.
  - Refer to documentation for correct parameters: [MySQL Documentation](https://dev.mysql.com/doc).

- Example: `ROUND` function rounds values in the `price` column to two decimal places.

```sql
SELECT
    market_date, customer_id,
    vendor_id,
    ROUND(quantity * cost_to_customer_per_qty, 2) AS price
FROM farmers_market.customer_purchases
LIMIT 5
```
<table>
    <caption>Table 2.10</caption>
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
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>6.92</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>14</td>
            <td>7</td>
            <td>15.24</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>15</td>
            <td>7</td>
            <td>10.69</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>16</td>
            <td>7</td>
            <td>14.12</td>
        </tr>
        <tr>
            <td>2019-07-03</td>
            <td>22</td>
            <td>7</td>
            <td>4.61</td>
        </tr>
    </tbody>
</table>

- `ROUND` can accept negative values for the second parameter to round left of the decimal point.
    - Example: `ROUND(1245, -2)` returns 1200.

- `SQL` functions can manipulate text data.

- `CONCAT` function joins strings together.

- In the `customer` table, `first_name` and `last_name` are in separate columns.

```sql
SELECT *
FROM farmers_market.customer
LIMIT 5
```
<table>
    <caption>Table 2.11</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_first_name</th>
            <th>customer_last_name</th>
            <th>customer_zip</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Jane</td>
            <td>Connor</td>
            <td>22801</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Manuel</td>
            <td>Diaz</td>
            <td>22821</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Bob</td>
            <td>Wilson</td>
            <td>22821</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Deanna</td>
            <td>Washington</td>
            <td>22801</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Abigail</td>
            <td>Harris</td>
            <td>22801</td>
        </tr>
    </tbody>
</table>

- Use `CONCAT` to combine `first_name`, a space, and `last_name` into `customer_name`.

```sql
SELECT
    customer_id,
    CONCAT(customer_first_name, " ", customer_last_name) AS customer_name
FROM farmers_market.customer
LIMIT 5
```

<table>
    <caption>Table 2.12</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Jane Connor</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Manuel Diaz</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Bob Wilson</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Deanna Washington</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Abigail Harris</td>
        </tr>
    </tbody>
</table>

- First, sort by `last_name` and `first name`.

- Then concatenate `first_name` and `last_name`.

```sql
SELECT
    customer_id,
    CONCAT(customer_first_name, " ", customer_last_name) AS customer_name
FROM farmers_market.customer
ORDER BY customer_last_name, customer_first_name
LIMIT 5
```


<table>
    <caption>Table 2.13</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7</td>
            <td>Jessica Armenta</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Betty Bullard</td>
        </tr>
        <tr>
            <td>1</td>
            <td>Jane Connor</td>
        </tr>
        <tr>
            <td>17</td>
            <td>Carlos Diaz</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Manuel Diaz</td>
        </tr>
    </tbody>
</table>

- You can nest functions.
  - Example: Use `UPPER` to convert the concatenated name to uppercase.

```sql
SELECT
    customer_id,
    UPPER(CONCAT(customer_last_name, ", ", customer_first_name)) AS customer_name
FROM farmers_market.customer
ORDER BY customer_last_name, customer_first_name
LIMIT 5
```

<table>
    <caption>Table 2.14</caption>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>customer_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7</td>
            <td>ARMENTA, JESSICA</td>
        </tr>
        <tr>
            <td>6</td>
            <td>BULLARD, BETTY</td>
        </tr>
        <tr>
            <td>1</td>
            <td>CONNOR, JANE</td>
        </tr>
        <tr>
            <td>17</td>
            <td>DIAZ, CARLOS</td>
        </tr>
        <tr>
            <td>2</td>
            <td>DIAZ, MANUEL</td>
        </tr>
    </tbody>
</table>

- Notes:
  - Sorting is done on `customer_last_name` and `customer_first_name`, not on `customer_name`.
  - Aliases might not be reusable in other parts of the query.
  - You can use functions in `ORDER BY`. Example: `ORDER BY UPPER(customer_last_name)` to sort uppercased last names.

# Evaluating Query Output

- Steps to ensure SQL query results are as expected:
  - Run the query with `LIMIT` to preview the first few rows.
    - Verify changes in the output.
    - Check column names and values.
  - Verify Total Rows:
    - Run without `LIMIT` or use `COUNT` to confirm total rows.

- Use the Query Editor to review results:
  - Quick sanity check, not a substitute for full quality control.
  - Remove `LIMIT` to review the full dataset.
  - In MySQL Workbench, use "Don't Limit" under the Query menu.

- Run the query to generate full output:
  - Check total row count to match expectations.
  - Example: 21 rows in `customer_purchases`.
  - In MySQL Workbench, check the Message in the Output section.

- Review the resulting dataset ("Result Grid" in MySQL Workbench):
  - Check column headers.
  - Spot-check values.
  - Verify sorting if using `ORDER BY`.

- Manually sort each column:
  - Example: Sort by `market_date` in ascending order.
  - Sort by `vendor_id` column.

- Check minimum and maximum values in each column:
  - Find errors like unexpected negative values or NULLs.
  - Look for strings starting with numbers or spaces, or other anomalies.

# Exercises Using the Included Database

- Exercises for the `customer` table:
  - Columns and example rows are shown in Figure 2.11.

1. Retrieve all columns from the `customer` table.
2. Display all columns and 10 rows, sorted by `last_name`, then `first_name`.
3. List all `customer_id` and `first_name`, sorted by `first_name`.