---
title: "SQL Chapter 2 Time 2"
author: "db"
---

# The Fundamental Syntax Structure of a SELECT Query

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

# Selecting Columns and Limiting the Number of Rows Returned

- Basic Syntax:
  - This query retrieves all columns.
  - `FROM schema.table` specifies the schema and table name.
- Example: 
```sql
SELECT * FROM farmers_market.product
```

- Example of using LIMIT for the first five rows:

```sql
SELECT *
FROM farmers_market.product
LIMIT 5
```

![Figure 2.1](Fotos/Chapter2/Fig_2.1.png)
<figcaption></figcaption>

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

![Figure 2.2](Fotos/Chapter2/Fig_2.2.png)
<figcaption></figcaption>

- List column names explicitly instead of using `*`:
  - `Schema Changes`: `SELECT *` can cause issues if the table changes.
  - `Consistent Output`: Listing columns ensures consistent output.

- Preventing Breakages:
  - `Automated Processes`: Unexpected changes can cause failures.
  - `Error Detection`: Listing columns alerts you if a column is removed or renamed.

```sql
SELECT market_date, vendor_id, booth_number 
FROM farmers_market.vendor_booth_assignments 
LIMIT 5
```

![Figure 2.3](Fotos/Chapter2/Fig_2.3.png)
<figcaption></figcaption>

- We sort this output by market date.
- 
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

![Figure 2.4](Fotos/Chapter2/Fig_2.4.png)
<figcaption></figcaption>

- DESC:

```sql
SELECT product_id, product_name 
FROM farmers_market.product 
ORDER BY product_id DESC
LIMIT 5
```

![Figure 2.5](Fotos/Chapter2/Fig_2.5.png)
<figcaption></figcaption>

- Rows in Figure 2.5 differ because `ORDER BY` runs before `LIMIT`.
  - `LIMIT` must be after `ORDER BY`.+
- First, sort by `market_date`, then by `vendor_id`.

```sql
SELECT market_date, vendor_id, booth_number
FROM farmers_market.vendor_booth_assignments
ORDER BY market_date, vendor_id
LIMIT 5
```

![Figure 2.6](Fotos/Chapter2/Fig_2.6.png)
<figcaption></figcaption>

# Introduction to Simple Inline Calculations

- `customer_purchases` table:

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    quantity,
    cost_to_customer_per_qty
FROM farmers_market.customer_purchases
LIMIT 10
```

![Figure 2.7](Fotos/Chapter2/Fig_2.7.png)
<figcaption></figcaption>

- Multiply `quantity` and `cost_per_quantity`:

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    quantity,
    cost_to_customer_per_qty,
    quantity * cost_to_customer_per_qty
FROM farmers_market.customer_purchases
LIMIT 10
```

![Figure 2.8](Fotos/Chapter2/Fig_2.8.png)
<figcaption></figcaption>

- Use `AS` to give a calculated column a meaningful name.
  - Example: Assign alias `price` to the result.

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    quantity * cost_to_customer_per_qty AS price
FROM farmers_market.customer_purchases
LIMIT 10
```

![Figure 2.9](Fotos/Chapter2/Fig_2.9.png)
<figcaption></figcaption>

- `AS` is optional in MySQL.
- The query will return the same result without it.

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    quantity * cost_to_customer_per_qty price
FROM farmers_market.customer_purchases
LIMIT 10
```

- Aggregating Data:
  - Summarize data across multiple rows.
  - Examples: `SUM`, `AVG`, `COUNT`.
  - Covered in Chapter 6.

- Current calculations apply to each row individually.
  - Example: Calculate price by multiplying quantity by cost per unit.
  - These do not summarize or combine data across rows.

# More Inline Calculation Examples: Rounding

- A `SQL` function takes inputs, performs an operation, and returns a value.
- Use functions to modify raw values before displaying them.
- Syntax for calling a `SQL` function:
  - Input parameters can be a column name or a constant value.
  - Refer to documentation for correct parameters: [MySQL Documentation](https://dev.mysql.com/doc).

- Example: `ROUND` function rounds values in the `price` column to two decimal places.

```sql
SELECT
    market_date,
    customer_id,
    vendor_id,
    ROUND(quantity * cost_to_customer_per_qty, 2) AS price
FROM farmers_market.customer_purchases
LIMIT 10
```

![Figure 2.10](Fotos/Chapter2/Fig_2.10.png)
<figcaption></figcaption>

- `ROUND` can accept negative values for the second parameter to round left of the decimal point.
    - Example: `ROUND(1245, -2)` returns 1200.

# More Inline Calculation Examples: Concatenating Strings

- `SQL` functions can manipulate text data.
- `CONCAT` function joins strings together.
- In the `customer` table, `first_name` and `last_name` are in separate columns.

```sql
SELECT *
FROM farmers_market.customer
LIMIT 5
```

![Figure 2.11](Fotos/Chapter2/Fig_2.11.png)
<figcaption></figcaption>

- Use `CONCAT` to combine `first_name`, a space, and `last_name` into `customer_name`.

```sql
SELECT
    customer_id,
    CONCAT(customer_first_name, " ", customer_last_name) AS customer_name
FROM farmers_market.customer
LIMIT 5
```

![Figure 2.12](Fotos/Chapter2/Fig_2.12.png)
<figcaption></figcaption>

- Sort by `last_name` using `ORDER BY`.
- Then concatenate `first_name` and `last_name`.

```sql
SELECT
    customer_id,
    CONCAT(customer_first_name, " ", customer_last_name) AS customer_name
FROM farmers_market.customer
ORDER BY customer_last_name, customer_first_name
LIMIT 5
```

![Figure 2.13](Fotos/Chapter2/Fig_2.13.png)
<figcaption></figcaption>

- You can nest functions.
  - Example: Use `UPPER` to convert the concatenated name to uppercase.
  - Next: `last_name`, a comma, a space, and `first_name`, all in uppercase.

```sql
SELECT
    customer_id,
    UPPER(CONCAT(customer_last_name, ", ", customer_first_name)) AS customer_name
FROM farmers_market.customer
ORDER BY customer_last_name, customer_first_name
LIMIT 5
```

![Figure 2.14](Fotos/Chapter2/Fig_2.14.png)
<figcaption></figcaption>

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