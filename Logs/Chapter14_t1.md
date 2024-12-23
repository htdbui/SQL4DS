---
title: "SQL Chapter 14"
author: "db"
---

# Introduction

- We have covered many aspects of developing datasets for `machine learning`:
  - Selecting data from a database.
  - Preparing it for `machine learning` models.

- What do you do once you have designed your query and are ready to start analyzing the results?
  - Your `SQL` editor often allows you to write the results to a `CSV` file.
  - This can be imported into `Business Intelligence (BI)` software like `Tableau` or `machine learning` scripts in a language like `Python`.

- Sometimes it's preferable to store the dataset within the database:
  - For data governance.
  - For data security.
  - For teamwork.
  - For file size and processing speed purposes.

- In this chapter:
  - We'll cover some types of `SQL` queries beyond `SELECT` statements.
  - Such as `INSERT` statements.
  - These allow you to store the results of your query in a new table in the database.

# Storing SQL Datasets as Tables and Views

- In most databases, you can store query results as a `table` or a `view`.
  - Storing results as a `table`:
    - Takes a snapshot of the results at the time the query is run.
    - Saves the data as a new table or appends to an existing table.
  - Storing results as a `view`:
    - Stores the `SQL` itself.
    - Runs the `SQL` on-demand when referenced.
    - Generates a new dataset based on the current state of the database.

- If you have:
  - Database storage space.
  - Permissions to create tables or insert records.
  - No cost-prohibitive constraints.
  - It is good practice to store snapshots of datasets for `machine learning`.

- Check with your database administrator:
  - To determine if you can create and modify tables.
  - To know which schema(s) you have permission to write to.

- When testing `machine learning` algorithms:
  - Use the same static dataset for multiple tests.
  - Ensure input data doesn't change by writing results to a table.
  - Store a copy of the dataset for later reference.
  - Keep a record of exact values used in your model.

- To store query results, use a `CREATE TABLE` statement.

```sql
CREATE TABLE [schema_name].[new_table_name] AS
(
    [your query here]
)
```

- `SELECT` statements:
  - Indentation and line breaks don't matter.
  - Used for readability.

- `CREATE TABLE` statement:
  - Table name must be new and unique within the schema.
  - Running the same statement twice causes an error.

- After creating the table:
  - Query it like any other table or view.
  - Use the new name you gave it.

- To remove a table:
  - Use `DROP TABLE` statement.
  - Be careful to avoid accidental deletion.
  - Deleted data may not be recoverable.

- Best practices:
  - Have permissions to create and drop tables in a personal schema.
  - Keep personal schema separate from production schema.

- The syntax for dropping a table is simply:

```sql
DROP TABLE [schema_name].[table_name]
```

- To create, select from, and drop a table with a snapshot of data:
  - From the `Farmer's Market` database `product` table.
  - Filtered to products with a quantity type `unit`.

- Run these three queries in sequence:

```sql
CREATE TABLE farmers_market.product_units AS
(
    SELECT * 
    FROM farmers_market.product 
    WHERE product_qty_type = "unit"
)
;
 
SELECT * FROM farmers_market.product_units
;
 
DROP TABLE farmers_market.product_units
;
```

- Semicolons are used to separate multiple queries in the same file.

- Avoid accidentally running `DROP TABLE` statements.
  - Many SQL editors have a "run all" command.
  - Comment out `DROP TABLE` statements after running them.
  - Save the file with the `DROP TABLE` query commented out.

- In `MySQL Workbench`:
  - Comment out code with two dashes and a space (`-- `).
  - Or surround a block of code with `/*` and `*/`.

- `Database views` are similar to tables.
  - Created and dropped the same way.
  - Views store queries, not data.

- When you `create a view`:
  - You store a query to run later.

- When you `drop a view`:
  - You do not delete data.
  - You only remove the query reference.

```sql
CREATE VIEW farmers_market.product_units_vw AS
(
    SELECT * 
    FROM farmers_market.product 
    WHERE product_qty_type = "unit"
)
;
 
SELECT * FROM farmers_market.product_units_vw
;
 
DROP VIEW farmers_market.product_units_vw
;
```

- Some database systems support `SELECT INTO`.
  - Example: `SQL Server`.
  - Works like `CREATE TABLE`.

- `SELECT INTO` is often used to create backups of tables.

- Check your database's documentation for the correct syntax.

# Adding a Timestamp Column

- When you create or modify a `database table`:
  - You might want to keep a record of when each row was created or last modified.
  - Add a `timestamp column` to your `CREATE TABLE` or `UPDATE` statement.

- The syntax for creating a `timestamp` varies by `database system`.
  - In `MySQL`, use `CURRENT_TIMESTAMP` to get the current date and time.
  - You can give the `timestamp column` an alias like any calculated column.

- Keep in mind:
  - The `timestamp` is generated by the `database server`.
  - If the database is in another time zone, the `timestamp` may differ from your local time.
  - Many databases use `Coordinated Universal Time (UTC)` as their default `timestamp` time.
    - `UTC` is a global time standard synchronized using atomic clocks.
    - `UTC` aligns with `Greenwich Mean Time (GMT)` and doesn't change for `Daylight Savings Time`.

- Example of time zones:
  - `Eastern Standard Time (EST)` is `UTC-05:00`.
  - `Eastern Daylight Time (EDT)` is `UTC-04:00`.

- Time zone math can be complicated.
  - Many databases use a standard `UTC` clock time instead of a developer's local time.

- Modify the `CREATE TABLE` example to include a `timestamp` column:

```sql
CREATE TABLE farmers_market.product_units AS
(
    SELECT p.*,
        CURRENT_TIMESTAMP AS snapshot_timestamp 
    FROM farmers_market.product AS p
    WHERE product_qty_type = "unit"
)
```

![Figure 14.1](Fotos/Chapter14/Fig_14.1.png)
<figcaption></figcaption>

# Inserting Rows and Updating Values in Database Tables

- To modify data in an existing `database table`:
  - Use an `INSERT` statement to add a new row.
  - Use an `UPDATE` statement to modify an existing row.

- In this chapter:
  - We insert query results into another table.
  - This is a specific kind of `INSERT` statement called `INSERT INTO SELECT`.

```sql
INSERT INTO [schema_name].[table_name] ([comma- separated list of column 
names])
[your SELECT query here]
```

- If we wanted to add rows to our product_units table created earlier, we would write:

```sql
INSERT INTO farmers_market.product_units (product_id, product_name, 
product_size, product_category_id, product_qty_type, snapshot_timestamp)
    SELECT 
        product_id, 
        product_name,
        product_size, 
        product_category_id, 
        product_qty_type,
        CURRENT_TIMESTAMP
    FROM farmers_market.product AS p 
    WHERE product_id = 23
```

- It is important that the columns in both `queries` are in the same order.
  - The corresponding fields may not have identical names.
  - The system will insert the returned values from the `SELECT` statement in the column order listed in parentheses.

- When we query the `product_units` table:
  - We'll have a snapshot of the same product row at two different times.
  - This is shown in Figure 14.2.

![Figure 14.2](Fotos/Chapter14/Fig_14.2.png)
<figcaption></figcaption>

- The syntax for deleting a row is simply:

```sql
DELETE FROM [schema_name].[table_name]
WHERE [set of conditions that uniquely identifies the row]
```

- Start with `SELECT *` instead of `DELETE`.
  - This lets you see what rows will be deleted before running the `DELETE` statement.

- The `product_id` and `snapshot_timestamp` uniquely identify rows in the `product_units` table.
  - Use the following statement to delete the row added by the previous `INSERT INTO`:

```sql
DELETE FROM farmers_market.product_units 
WHERE product_id = 23
    AND snapshot_timestamp = '2021-04-18 00:49:24'
```

- Sometimes you want to update a value in an existing row instead of inserting a new row.

```sql
UPDATE [schema_name].[table_name]
SET [column_name] = [new value]
WHERE [set of conditions that uniquely identifies the rows you want to 
change]
```

- You have entered all vendor booth assignments for the next several months.
- Vendor 4 informs you they can't make it on October 10.
- You decide to upgrade vendor 8 to vendor 4's booth for the day.
  - Vendor 4's booth is larger and closer to the entrance.

- Before making any changes, snapshot the existing vendor booth assignments.
  - Include the vendor name and booth type.
  - Use the following `SQL`:

```sql
CREATE TABLE farmers_market.vendor_booth_log AS
(
    SELECT vba.*, 
        b.booth_type, 
        v.vendor_name,
        CURRENT_TIMESTAMP AS snapshot_timestamp 
    FROM farmers_market.vendor_booth_assignments vba
        INNER JOIN farmers_market.vendor v 
            ON vba.vendor_id = v.vendor_id
        INNER JOIN farmers_market.booth b
            ON vba.booth_number = b.booth_number
    WHERE market_date >= '2020-10-01'
)
```

- Selecting all records from this new log table produces the results shown in Figure 14.3.

![Figure 14.3](Fotos/Chapter14/Fig_14.3.png)
<figcaption></figcaption>

- To update vendor 8’s booth assignment:

```sql
UPDATE farmers_market.vendor_booth_assignments
SET booth_number = 7 
WHERE vendor_id = 8 and market_date = '2020-10-10
```

- To delete vendor 4’s booth assignment:

```sql
DELETE FROM farmers_market.vendor_booth_assignments 
WHERE vendor_id = 4 and market_date = '2020-10-10'
```

- Now, when we query the `vendor_booth_assignments` table:
  - There is no record that vendor 4 had a booth assignment on that date.
  - There is no record that vendor 8’s booth assignment used to be different.

- We have a record of the previous assignments in the `vendor_booth_log` we created.
- We can insert new records into the log table to record the latest changes.

```sql
INSERT INTO farmers_market.vendor_booth_log (vendor_id, booth_number, 
market_date, booth_type, vendor_name, snapshot_timestamp)
    SELECT 
        vba.vendor_id, 
        vba.booth_number, 
        vba.market_date,
        b.booth_type, 
        v.vendor_name,
        CURRENT_TIMESTAMP AS snapshot_timestamp 
    FROM farmers_market.vendor_booth_assignments vba
        INNER JOIN farmers_market.vendor v 
            ON vba.vendor_id = v.vendor_id
        INNER JOIN farmers_market.booth b
            ON vba.booth_number = b.booth_number
    WHERE market_date >= '2020-10-01'
```

- The original `vendor_booth_assignments` table doesn't contain the original booth assignments for these two vendors on October 10.
- If we had run an analysis at an earlier date, we could see the database values at that time.
- We can query the `vendor_booth_log` table to look at the values at different points in time.
- This is shown in Figure 14.4.

![Figure 14.4](Fotos/Chapter14/Fig_14.4.png)
<figcaption></figcaption>

# Using SQL Inside Scripts

- Importing datasets into your machine learning script is beyond this book's scope.
- Search the internet for tutorials on combining `SQL` with your scripting language.
  - Example search: "import SQL python pandas dataframe".
  - Tutorials will show how to connect to a database, run a `SQL` query, and import results into a `pandas` dataframe.

- You can paste the `SQL` query into your script and store it in a string variable.
- Alternatively, reference the `SQL` stored in a text document.

- Some special characters in your query need to be "escaped".
  - If you use double quotes to store the query in Python, double quotes within the query will cause an error.
  - Use a backslash to escape quotes inside quoted strings in Python.

- Representing this `SQL` query as a string in Python:

```python
SELECT * FROM farmers_market.product WHERE product_qty_type = "unit"
```

- requires escaping the internal double quotes, like so:

```python
my_query = "SELECT * FROM farmers_market.product WHERE product_qty_type = \"unit\""
```

- This is common in programming languages where strings are surrounded by quotes.
  - Many strings contain quotes in the enclosed text.
  - Search for "string escape characters" with your chosen script tool or language to learn how to modify your `SQL` for use within your script.

- You can also write data from your script back to the database using `SQL`.
  - The approach varies based on the scripting language and database type.
  - In Python, packages help connect and write to databases without writing dynamic `SQL INSERT` statements.
  - These packages generate the `SQL` to insert values from a `dataframe` into a database table.

- Another approach is to create a file to temporarily store your data.
  - Transfer the file to a location accessible from your script and database.
  - Load the results from the file into your table.
  - Example: Write data from a `pandas dataframe` to a CSV file, transfer it to an AWS S3 bucket, then load it into a Redshift database.

- One use case is storing your transformed dataset after feature engineering and data preprocessing.
- Another use case is storing results generated by your predictive model.
  - Create a table to store unique identifiers, timestamps, model names, and scores.
  - Insert new rows each time you refresh your model scores.
  - Use a `SQL` query to filter the model score log table and join it with the input dataset.

- There are many ways to connect to and interact with data from a database in your scripts.
  - Some methods may not require `SQL`.

# In Closing

- Now that you know `SQL` basics, you have the foundation to create datasets for your machine learning models.
  - You may need to search the internet for functions and syntax not covered in this book.

- I have been a data scientist for five years.
  - All of the queries I have written for machine learning datasets are variations of the `SQL` I learned in school 20 years ago.

- I hope this book has given you the `SQL` skills to achieve your analysis goals faster and more independently.
- I hope you find pulling and modifying your own datasets as empowering as I do!

# Exercises

1. If you include a `CURRENT_TIMESTAMP` column when you create a view, what would you expect the values of that column to be when you query the view?

2. Write a query to determine what the data from the `vendor_booth_assignment` table looked like on October 3, 2020 by querying the `vendor_booth_log` table created in this chapter.
  - Assume records have been inserted into the log table any time changes were made to the `vendor_booth_assignment` table.