---
title: "SQL Chapter 13 Time 1"
author: "db"
---

# Introduction

- In this chapter, we will develop datasets to answer different analytical questions.
  - Combine multiple concepts from previous chapters into more complex queries.
  - This is more advanced.

- Note:
  - The example database doesn't have enough data with correlations for actual analyses.
  - We won't look for trends in the output screenshots.
  - Focus is on designing and building a dataset using `SQL`.

- Analytical questions:
  - What factors correlate with fresh produce sales?
  - How do sales vary by customer zip code, market distance, and demographic data?
  - How does product price distribution affect market sales?

# What Factors Correlate with Fresh Produce Sales?

- Analytical question: "What factors are correlated with sales of fresh produce at the farmer's market?"
  - Determine relationships between different variables and fresh produce sales.
  - Summarize different variables over time and explore how sales change.

- Example question: "As the number of different available products at the market increases, do sales of fresh produce go up or down?"
  - Explore the relationship between product variety and sales.
  - Positive correlation: sales go up when product variety goes up.
  - Negative correlation: sales go down when product variety goes up.

- Summarize each value per week and create a scatterplot of weekly pairs of numbers.
  - Write a query to generate a dataset with one row per market week.
  - Include weekly summaries of each value to be explored.

- Determine what products are considered "fresh produce."
  - Calculate sales of those products per week.
  - Pull in other variables summarized per week to explore in relation to those sales.

- Ideas for values to compare to sales:
  - Product availability (number of vendors, volume of inventory, seasonal availability).
  - Product cost.
  - Time of year/season.
  - Sales trends over time.
  - Factors affecting all sales (weather, number of customers).

- First, look at all product categories to determine which to use.
  - The result of the following query is shown in Figure 13.1:

```sql
SELECT * FROM farmers_market.product_category
```

![Figure 13.1](Fotos/Chapter13/Fig_13.1.png)
<figcaption></figcaption>

- From the list in Figure 13.1:
  - Product category 1 is "Fresh Fruits & Vegetables," which sounds like "fresh produce."
  - "Plants & Flowers" and "Eggs & Meat" categories may also contain fresh produce.

- Generate a list of all products in categories 1, 5, and 6.
  - Take this list to the requester to confirm the products for analysis.
  - If the requester wants a list of products instead of a category, mention the need to check for product additions and changes over time.

- The output from this query is shown in Figure 13.2:

```sql
SELECT * FROM farmers_market.product 
WHERE product_category_id IN (1, 5, 6) 
ORDER BY product_category_id
```

![Figure 13.2](Fotos/Chapter13/Fig_13.2.png)
<figcaption></figcaption>

- Assume product category 1 is correct.
- Design the query structure:
  - Summary information about sales from `customer_purchases` table.
  - Availability data from `vendor_inventory` table.
  - Time-related information from `market_date_info` table.
  - Join tables by `market_date`.

- Start with product sales part of the question.
  - Join other information to the results of that query.

- Select details to summarize sales per week for products in "Fresh Fruits & Vegetables" category 1.
  - Inner join `customer_purchases` and `product` by `product_id`.
  - If looking at multiple categories, join `product_category` for category names.
  - Simplify and leave out `product_category` for now.

- The details are shown in Figure 13.3:

```sql
SELECT * 
FROM customer_purchases cp
    INNER JOIN product p
        ON cp.product_id = p.product_id
WHERE p.product_category_id = 1
```

![Figure 13.3](Fotos/Chapter13/Fig_13.3.png)
<figcaption></figcaption>

- I used an `INNER JOIN` instead of a `LEFT JOIN`.
  - Not interested in products without purchases for this sales calculation.
- Check the count of rows returned to ensure it makes sense.
- Look at the details to ensure correct data, joins, and filters.

- Since summarizing by week:
  - Don't need the `transaction_time` field.
  - Don't need the size of each product.
  - Might need `product_quantity_type` and vendor information later.

- Total sales is the dependent variable.
  - For vendor count, don't use `customer_purchases` table.
  - Use `vendor_inventory` table for products available for sale.

- Join `market_date_info` table to get the week number.
  - Makes summarization easier.
  - Get other date-related information like season and weather.
  - Use `RIGHT JOIN` to include market dates with no fresh produce sales.

- The result is shown in Figure 13.4:

```sql
SELECT 
    cp.market_date, 
    cp.customer_id, 
    cp.quantity, 
    cp.cost_to_customer_per_qty, 
    p.product_category_id, 
    mdi.market_date, 
    mdi.market_week, 
    mdi.market_year, 
    mdi.market_rain_flag, 
    mdi.market_snow_flag
FROM customer_purchases cp
    INNER JOIN product p
        ON cp.product_id = p.product_id
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = cp.market_date
WHERE p.product_category_id = 1
```

![Figure 13.4](Fotos/Chapter13/Fig_13.4.png)
<figcaption></figcaption>

- The `RIGHT JOIN` didn't show market dates without sales in this category.
  - The `WHERE` clause filtered out rows without sales.
  - No product categories are associated with dates without sales.
  - This defeats the purpose of the `RIGHT JOIN`.

- Solution: Move the product category filter to the `JOIN ON` clause.
  - Join `product` table on `product_id` and `product_category_id`.
  - Filter `product_category_id` in the `ON` clause.
  - This filter only applies to the `product` table and `customer_purchases` table.

- Now all market dates will be returned.
  - Moved `market_date_info` fields to appear first.
  - Modified the join to include the filter.

- No `WHERE` clause, but still filtering results of one table being joined.
- The output of this query is displayed in Figure 13.5:

```sql
SELECT 
    mdi.market_date, 
    mdi.market_week, 
    mdi.market_year, 
    mdi.market_rain_flag, 
    mdi.market_snow_flag, 
    cp.market_date, 
    cp.customer_id, 
    cp.quantity, 
    cp.cost_to_customer_per_qty, 
    p.product_category_id
FROM customer_purchases cp
    INNER JOIN product p
        ON cp.product_id = p.product_id
            AND p.product_category_id = 1
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = cp.market_date
```

![Figure 13.5](Fotos/Chapter13/Fig_13.5.png)
<figcaption></figcaption>

- Now we can see rows for market dates with no fresh produce purchases.

- Summarize customer purchases to one row per week.
  - Group by `market_year` and `market_week`.
  - Remove unnecessary columns with additional details about purchases.

- Use `COALESCE` to handle NULL values in sales calculation.
  - `COALESCE([value 1], 0)` returns 0 if "value 1" is NULL.
  - Wrap `SUM` function in `COALESCE` to return 0 if no sales.
  - `ROUND` the result to two digits after the decimal point.

- Return `MAX` of snow and rain flags.
  - If there was precipitation at either market during the week, return 1.

- Return the minimum `market_season` value.
  - Only one value is returned if a week is split across two seasons.

- The updated result using the following query is displayed in Figure 13.6:

```sql
SELECT 
    mdi.market_year,
    mdi.market_week,
    MAX(mdi.market_rain_flag) AS market_week_rain_flag, 
    MAX(mdi.market_snow_flag) AS market_week_snow_flag, 
    MIN(mdi.market_min_temp) AS minimum_temperature, 
    MAX(mdi.market_max_temp) AS maximum_temperature, 
    MIN(mdi.market_season) AS market_season,
    ROUND(COALESCE(SUM(cp.quantity * cp.cost_to_customer_per_qty), 0), 2) AS 
weekly_category1_sales
FROM customer_purchases cp
    INNER JOIN product p
        ON cp.product_id = p.product_id
            AND p.product_category_id = 1
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = cp.market_date
GROUP BY 
    mdi.market_year,
    mdi.market_week    
```

![Figure 13.6](Fotos/Chapter13/Fig_13.6.png)
<figcaption></figcaption>

- Now we have total sales by week.

- Other aggregate values to add:
  - Number of vendors carrying products in the category.
  - Volume of inventory available for purchase.
  - Special high-demand product seasonal availability.

- These values come from the `vendor_inventory` table.
  - We want to know what vendors brought to market, regardless of purchases.

- Set up the query for `vendor_inventory` like we did for `customer_purchases`.
  - Join to `product` and `market_date_info` tables the same way.
  - Filter to `product_category_id = 1` in the `JOIN` statement.

- The results of this query are shown in Figure 13.7:

```sql
SELECT 
    mdi.market_date,
    mdi.market_year, 
    mdi.market_week, 
    vi.*,
    p.*
FROM vendor_inventory vi 
    INNER JOIN product p
        ON vi.product_id = p.product_id 
            AND p.product_category_id = 1
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = vi.market_date
```

![Figure 13.7](Fotos/Chapter13/Fig_13.7.png)
<figcaption></figcaption>

- Remove unnecessary fields for the weekly summary.
- Narrow down the `vendor_inventory` table to keep needed features:
  - `vendor_id` for the number of vendors.
  - `product_id` for the number of products.
  - `quantity` for the volume of products.

- Use `product_id` to flag the existence of certain products.
  - Example: Flag availability of sweet corn (product 16).
  - Create a product availability flag `corn_available_flag`.

- The query and result are shown in Figure 13.8:

```sql
SELECT 
    mdi.market_year,
    mdi.market_week,
    COUNT(DISTINCT vi.vendor_id) AS vendor_count,
    COUNT(DISTINCT vi.product_id) AS unique_product_count,
    SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) 
AS unit_products_qty,
    SUM(CASE WHEN p.product_qty_type = 'lbs' THEN vi.quantity ELSE 0 END) AS 
bulk_products_lbs,
    ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS 
total_product_value,
    MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_flag 
FROM vendor_inventory vi
    INNER JOIN product p
        ON vi.product_id = p.product_id
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = vi.market_date
 GROUP BY 
    mdi.market_year,
    mdi.market_week
```

![Figure 13.8](Fotos/Chapter13/Fig_13.8.png)
<figcaption></figcaption>

- Now that I see these results, I realize:
  - I would like to have a count of `vendors` selling and `products` available at the entire `market`.
  - I also want the `product availability` for `product category 1`.

- To avoid developing another query:
  - I will remove the `product_category_id` filter.
  - I will use `CASE` statements to create a set of fields.
  - These fields will provide the same metrics but only for products in the category.

- Then, the existing fields will:
  - Turn into a count for all `vendors` and `products` at the `market`.

```sql
    mdi.market_year,
    mdi.market_week,
    COUNT(DISTINCT vi.vendor_id) AS vendor_count,
    COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.vendor_id 
ELSE NULL END) AS vendor_count_product_category1, 
    COUNT(DISTINCT vi.product_id) AS unique_product_count,
    COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.product_id 
ELSE NULL END) AS unique_product_count_product_category1,
    SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) 
AS unit_products_qty,
    SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type = 'unit' 
THEN vi.quantity ELSE 0 END) AS unit_products_qty_product_category1,
    SUM(CASE WHEN p.product_qty_type  = 'lbs' THEN vi.quantity ELSE 0 END) 
AS bulk_products_lbs,
    SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type  = 'lbs' 
THEN vi.quantity ELSE 0 END) AS bulk_products_lbs_product_category1,
    ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS total_ 
product_value,
    ROUND(COALESCE(SUM(CASE WHEN p.product_category_id = 1 THEN vi.quantity 
* vi.original_price ELSE 0 END), 0), 2) AS total_product_value_product_ 
category1,    
    MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_ 
flag
FROM vendor_inventory vi
    INNER JOIN product p
        ON vi.product_id = p.product_id
    RIGHT JOIN market_date_info mdi
        ON mdi.market_date = vi.market_date
GROUP BY 
    mdi.market_year,
    mdi.market_week
```

- Figures 13.9, 13.10, and 13.11 show the columns from this query.
  - Each figure compresses the width of the columns.
  - This allows examples of all columns in the output to be shown.

- One easy quality check:
  - Ensure every field with the `_category1` suffix has an equal or lower value than the corresponding field without the suffix.
  - The totals include `product category 1`.
  - The category value should never be higher than the overall total.

- Now I can combine the results of these two queries:
  - I'll alias each of them in the `WITH` clause (`CTE`).
  - Then join the views.

```sql
WITH 
my_customer_purchases AS
(
       SELECT 
            mdi.market_year,
            mdi.market_week,
            MAX(mdi.market_rain_flag) AS market_week_rain_flag, 
            MAX(mdi.market_snow_flag) AS market_week_snow_flag, 
            MIN(mdi.market_min_temp) AS minimum_temperature,
            MAX(mdi.market_max_temp) AS maximum_temperature, 
            MIN(mdi.market_season) AS market_season, 
            ROUND(COALESCE(SUM(cp.quantity * cp.cost_to_customer_per_qty), 0), 2) 
AS weekly_category1_sales
        FROM customer_purchases cp
            INNER JOIN product p
                ON cp.product_id = p.product_id
                    AND p.product_category_id = 1
            RIGHT JOIN market_date_info mdi
                ON mdi.market_date = cp.market_date
        GROUP BY 
            mdi.market_year,
            mdi.market_week 
),
my_vendor_inventory AS
(
    SELECT 
        mdi.market_year,
        mdi.market_week,
        COUNT(DISTINCT vi.vendor_id) AS vendor_count,
        COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.vendor_id ELSE NULL 
END) AS vendor_count_product_category1,
        COUNT(DISTINCT vi.product_id) unique_product_count,
        COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.product_id ELSE 
NULL END) AS unique_product_count_product_category1,
        SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS 
unit_products_qty,
        SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type = 'unit' THEN 
vi.quantity ELSE 0 END) AS unit_products_qty_product_category1,
        SUM(CASE WHEN p.product_qty_type <> 'unit' THEN vi.quantity ELSE 0 END) AS 
bulk_products_qty,
        SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type <> 'unit' THEN 
vi.quantity ELSE 0 END) AS bulk_products_qty_product_category1,
        ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS 
total_product_value,
        ROUND(COALESCE(SUM(CASE WHEN p.product_category_id = 1 THEN vi.quantity * 
vi.original_price ELSE 0 END), 0), 2) AS total_product_value_product_category1,    
        MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_flag 
    FROM vendor_inventory vi
        INNER JOIN product p
            ON vi.product_id = p.product_id
        RIGHT JOIN market_date_info mdi
            ON mdi.market_date = vi.market_date
    GROUP BY 
        mdi.market_year,
        mdi.market_week 
)
 
SELECT * 
FROM my_vendor_inventory
    LEFT JOIN my_customer_purchases
         ON my_vendor_inventory.market_year = my_customer_purchases.market_year
              AND my_vendor_inventory.market_week = my_customer_purchases.market_week 
ORDER BY my_vendor_inventory.market_year, my_vendor_inventory.market_week
```

![Figure 13.9](Fotos/Chapter13/Fig_13.9.png)
<figcaption></figcaption>

![Figure 13.10](Fotos/Chapter13/Fig_13.10.png)
<figcaption></figcaption>

![Figure 13.11](Fotos/Chapter13/Fig_13.11.png)
<figcaption></figcaption>

- I can alter the final `SELECT` statement:
  - Include the prior week's `product category 1` sales.
  - Prior week's sales might indicate what to expect this week.

- I can use the `LAG` window function:
  - Introduced in Chapter 7, "Window Functions and Subqueries."

- I'll list all the column names:
  - Avoid showing duplicate `market_year` and `market_week` columns.
  - These columns are in both `CTEs` and show twice when using `*`.

- Note:
  - This query should be preceded by the same `CTE`/`WITH` clause as the previous query.
  - Removed here to save space.

```sql
SELECT 
    mvi.market_year, 
    mvi.market_week,
    mcp.market_week_rain_flag,
    mcp.market_week_snow_flag,
    mcp.minimum_temperature,
    mcp.maximum_temperature,
    mcp.market_season,
    mvi.vendor_count,
    mvi.vendor_count_product_category1,
    mvi.unique_product_count, 
    mvi.unique_product_count_product_category1,
    mvi.unit_products_qty,
    mvi.unit_products_qty_product_category1,
    mvi.bulk_products_qty,
    mvi.bulk_products_qty_product_category1,
    mvi.total_product_value,
    mvi.total_product_value_product_category1, 
    LAG(mcp.weekly_category1_sales, 1) OVER (ORDER BY mvi.market_year, 
mvi.market_week) AS previous_week_category1_sales,
    mcp.weekly_category1_sales 
FROM my_vendor_inventory mvi
    LEFT JOIN my_customer_purchases mcp
        ON mvi.market_year = mcp.market_year
            AND mvi.market_week = mcp.market_week 
ORDER BY mvi.market_year, mvi.market_week
```

- Now we have a detailed dataset:
  - Summarized to one row per week.
  - Can be used to explore relationships between `market weather`, `product availability`, and `fresh produce sales`.

- Some columns in Figure 13.12:
  - Shown in previous figures.
  - Condensed so the newly added `LAG` column is visible.

- Before reading this book:
  - A query like this might have looked large and indecipherable.
  - Now you know it's made by using various combinations of straightforward `SQL` statements.
  - These statements are combined into datasets with a progressively larger number of columns for analysis.

# How Do Sales Vary by Customer Zip Code, Market Distance, and Demographic Data?

- We have a couple of core questions:
  - How do sales vary by customer `zip code` and distance from the `market`?
  - Can we integrate `demographic data` into this analysis?

- To answer these questions:
  - We need to pull together several pieces of information.
  - Group all sales by `zip code`.
  - Group sales by `customer` for more meaningful answers.
  - Look at summary statistics and distributions of per-customer sales totals by `zip code`.

- Customer data:
  - We have the 5-digit `zip code` of each customer.
  - We don't have their full address or 9-digit `zip code`.
  - Calculate distances between the `market` location and a central location in each `zip code`.

- Latitude and longitude data:
  - One source is [OpenDataSoft](https://public.opendatasoft.com/explore/dataset/us-zip-code-latitude-and-longitude/table/?q=22821).
  - Import this data into our database.
  - Join it to our queries to add a latitude and longitude per `zip code`.

- Demographic data:
  - Available online related to `zip codes`.
  - Includes census data summarized by `ZCTAs` (Zip Code Tabulation Areas).
  - Pull in age distributions, wealth statistics, and other demographics summarized by `zip`.

- Example approach:
  - Summarize sales per `customer`.
  - Join demographic data to each `customer`'s record.
  - Use `zip code` data as an input into a customer-level model or for reporting purposes.

- Steps:
  - Summarize sales per `customer`.
  - Include how long they've been a `customer`.
  - Count of market dates at which they've made a purchase.
  - Total each `customer` has spent to date.
  - Join the purchase summary to the `customer` table to include each `customer`'s `zip code`.

- The output of the following query is shown in Figure 13.13:

```sql
SELECT 
    c.customer_id,
    c.customer_zip,
    DATEDIFF(MAX(market_date), MIN(market_date)) customer_duration_days, 
    COUNT(DISTINCT market_date) number_of_markets,
    ROUND(SUM(quantity * cost_to_customer_per_qty), 2) total_spent,
    ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT 
market_date), 2) average_spent_per_market
FROM farmers_market.customer c
LEFT JOIN farmers_market.customer_purchases cp
    ON cp.customer_id = c.customer_id 
GROUP BY c.customer_id
```

![Figure 13.13](Fotos/Chapter13/Fig_13.13.png)
<figcaption></figcaption>

- Note:
  - The sample data shows customers with similar values.
  - This isn't typical for real-world data.

- Example demographic data:
  - Loaded into a new table called `zip_data`.
  - Shown in Figure 13.14.

```sql
SELECT * FROM farmers_market.zip_data
```

![Figure 13.14](Fotos/Chapter13/Fig_13.14.png)
<figcaption></figcaption>

- I will join this table to my existing query by the `zip code` fields.
  - Every customer in a `zip code` will have the same demographic data added to their record.
  - Shown in Figure 13.15.

- I condensed the columns already shown in Figure 13.13:
  - To fit the newly added columns in view.

```sql
SELECT 
    c.customer_id,
    DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days, 
    COUNT(DISTINCT market_date) AS number_of_markets,
    ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent, 
    ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT 
market_date), 2) AS average_spent_per_market,
    c.customer_zip,
    z.median_household_income AS zip_median_household_income, 
    z.percent_high_income AS zip_percent_high_income, 
    z.percent_under_18 AS zip_percent_under_18, 
    z.percent_over_65 AS zip_percent_over_65, 
    z.people_per_sq_mile AS zip_people_per_sq_mile, 
    z.latitude,
    z.longitude
FROM farmers_market.customer c
    LEFT JOIN farmers_market.customer_purchases cp
        ON cp.customer_id = c.customer_id 
    LEFT JOIN zip_data z
        ON c.customer_zip = z.zip_code_5 
GROUP BY c.customer_id
```

![Figure 13.15](Fotos/Chapter13/Fig_13.15.png)
<figcaption></figcaption>

- One part of the analysis question was about distance from the `market`.
  - The data we have doesn't exactly tell us this.

- There is a calculation for distance between two latitudes and longitudes:
  - Found on Dayne Batten's blog: [Latitude-Longitude Distance in SQL](https://daynebatten.com/2015/09/latitude-longitude-distance-sql/).

- If the Farmer's Market is at coordinates 38.4463, –78.8712:
  - The calculation for distance between the latitude and longitude fields of a record in the dataset and the Farmer's Market location is:

```sql
ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((latitude - 38.4463) / 2)),2) + COS(RADIANS(38.4463)) * COS(RADIANS(latitude)) * POWER((SIN(RADIANS((longitude - -78.8712) / 2))), 2))))
```

- Don't worry about understanding how that calculation works.
  - Just know it returns the distance between two pairs of latitude and longitude.
  - Rounded to the nearest mile.

- I condensed the columns already shown in Figure 13.13 and 13.15:
  - To display the new fields fully in Figure 13.16.

- Replacing the latitude and longitude fields in our query with this calculation:
  - We now have the query:

```sql
SELECT 
    c.customer_id,
    DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days, 
    COUNT(DISTINCT market_date) AS number_of_markets,
    ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent, 
    ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT 
market_date), 2) AS average_spent_per_market,
    c.customer_zip,
    z.median_household_income AS zip_median_household_income, 
    z.percent_high_income AS zip_percent_high_income,
    z.percent_under_18 AS zip_percent_under_18,
    z.percent_over_65 AS zip_percent_over_65,
    z.people_per_sq_mile AS zip_people_per_sq_mile,
    ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((z.latitude -  38.4463) / 2)),2) +  
COS(RADIANS(38.4463)) * COS(RADIANS(z.latitude)) * POWER((SIN(RADIANS((z.longitude -  -  
78.8712) / 2))), 2)))) AS zip_miles_from_market
FROM farmers_market.customer AS c
    LEFT JOIN farmers_market.customer_purchases AS cp
        ON cp.customer_id = c.customer_id 
    LEFT JOIN zip_data AS z
        ON c.customer_zip = z.zip_code_5 
GROUP BY c.customer_id
```

- This will allow me to analyze customer metrics:
  - Total amount spent.
  - Number of markets attended.
  - By customer `zip code` or calculated distance from the `market`.

- I could build a scatterplot of `zip_miles_from_market` versus `total_spent`:
  - Customer data points in each `zip code` would overlap on the distance axis.
  - Add some jitter or size the dots to indicate how many people are represented by that point.

- With the newly added information per `zip`:
  - Create a rural versus urban flag based on population density.
  - Look at customer behavior based on that value.
  - Assess customer longevity by `zip code`.
  - Look at distributions of amount spent or customer durations by `zip code`.
  - Correlate the percent of high-income residents per `zip code` with customers’ total amount spent at the `market`.

- Additional customer summary fields that could be added:
  - Total purchases by product category.
  - Top vendor purchased from.
  - Number of different vendors purchased from.
  - Number of different products purchased.
  - Most frequent time of day of purchase.

- Example usage of this dataset:
  - Put the preceding query into a `CTE`.
  - Select from it to get the count of customers and the average total spent per customer for each `zip code`.
  - `zip_miles_from_market` is the same per row, so min, max, or average would return the same value.
  - Add `zip_miles_from_market` to the `GROUP BY` statement.
  - Change the structure of the query to join the `zip code` data when summarizing customers by `zip`.
  - For summary purposes, either approach is fine.

- The output of the following approach is shown in Figure 13.17:

```sql
WITH customer_and_zip_data AS
(
    SELECT 
         c.customer_id,
        DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days, 
        COUNT(DISTINCT market_date) AS number_of_markets, 
        ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent, 
        ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT market_date), 
2) AS average_spent_per_market,
        c.customer_zip,
        z.median_household_income AS zip_median_household_income, 
        z.percent_high_income AS zip_percent_high_income, 
        z.percent_under_18 AS zip_percent_under_18, 
        z.percent_over_65 AS zip_percent_over_65, 
        z.people_per_sq_mile AS zip_people_per_sq_mile,
           ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((z.latitude -  38.4463) / 2)),2) +  
COS(RADIANS(38.4463)) * COS(RADIANS(z.latitude)) * POWER((SIN(RADIANS((z.longitude -  - 
78.8712) / 2))), 2)))) AS zip_miles_from_market
    FROM farmers_market.customer AS c
        LEFT JOIN farmers_market.customer_purchases AS cp
            ON cp.customer_id = c.customer_id
        LEFT JOIN zip_data AS z
            ON c.customer_zip = z.zip_code_5
    GROUP BY c.customer_id
)
 
SELECT 
    cz.customer_zip,
    COUNT(cz.customer_id) AS customer_count, 
    ROUND(AVG(cz.total_spent)) AS average_total_spent, 
    MIN(cz.zip_miles_from_market) AS zip_miles_from_market
FROM customer_and_zip_data AS cz 
GROUP BY cz.customer_zip
```

![Figure 13.17](Fotos/Chapter13/Fig_13.17.png)
<figcaption></figcaption>

- One caveat to this analysis:
  - We only store the customer's current `zip code` in this database.
  - If a customer used to live in a different `zip code`, that past connection is lost.
  - All of their purchase history is now associated with the current `zip code` on their customer record.

- To maintain changes in `zip code` over time:
  - Add another table to the database to store the customer `zip code` history.

# How Does Product Price Distribution Affect Market Sales?

- What if a manager asks:
  - "What does our distribution of product prices look like at the `market`?"
  - "Are our low-priced items or high-priced items generating more sales for the `market`?"

- To answer these questions:
  - Use `window functions`.

- Clarifying question for the requester:
  - Related to time.
  - Should we answer for the most recent market season?
  - Compare year over year?
  - Look at all sales for the entire history?

- Requester wants to look at product price distributions for each market season over time:
  - Types of products sold can be different in summer versus winter holidays.
  - Changes over the years.

- First step:
  - Get raw data on the price per product per market date.

- Question about "product":
  - Is each `product_id` in the database a product?
  - Should each `product_id` sold by each vendor be considered a separate product?

- Look at the average price per product per vendor per season:
  - Use `original_price` per product specified by the vendor in the `vendor_inventory` table.
  - Ignore special discounts in the `customer_purchases` table.

- This query's results are shown in Figure 13.18:

```sql
SELECT 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type, 
    vi.vendor_id, 
    vi.market_date, 
    SUM(vi.quantity), 
    AVG(vi.original_price)
FROM product AS p
    LEFT JOIN vendor_inventory AS vi 
        ON vi.product_id = p.product_id 
GROUP BY 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type, 
    vi.vendor_id, 
    vi.market_date
```

![Figure 13.18](Fotos/Chapter13/Fig_13.18.png)
<figcaption></figcaption>

- Next steps:
  - Pull in the `market_season` from the `market_date_info` table.
  - Use the year to talk about seasons over time.

- Anticipated challenge:
  - Seasons are strings.
  - Sorting by season and year can be tricky.
  - Keep the minimum month value in the dataset.
  - Use it to sort the seasons in the correct order.

- See Figure 13.19 for the output of this query:

```sql
SELECT 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type,
    vi.vendor_id, 
    MIN(MONTH(vi.market_date)) AS month_market_season_sort, 
    mdi.market_season, 
    mdi.market_year,
    SUM(vi.quantity) AS quantity_available, 
    AVG(vi.original_price) AS avg_original_price
FROM product AS p
    LEFT JOIN vendor_inventory AS vi 
        ON vi.product_id = p.product_id 
    LEFT JOIN market_date_info AS mdi 
        ON vi.market_date = mdi.market_date 
GROUP BY 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type, 
    vi.vendor_id, 
    mdi.market_year, 
    mdi.market_season
```

![Figure 13.19](Fotos/Chapter13/Fig_13.19.png)
<figcaption></figcaption>

- One thing you might have noticed:
  - My attempt to pull in a sortable value to order the `market seasons` has resulted in multiple `month_market_season_sort` values per season.
  - We are getting the minimum month per season per product.
  - Some products aren't offered in the earliest month of each season.
  - These differing values could affect our results later on.

- Solution:
  - Use a `window function` to get the minimum month per `market_season` across all rows for that season.
  - Ignore the `product_id` values.
  - Switch to this in the next version of the query.

- Realization:
  - Need to pull in the `customer_purchases` data.
  - The second question is about sales.
  - The `vendor_inventory` table only contains available items, not items sold or the total amount generated by sales.

- Join the `customer_purchases` table:
  - Use it to calculate the aggregate sales data for the second question.
  - Join on all three matching keys: `product_id`, `vendor_id`, and `market_date`.

- Results are shown in Figure 13.20:

```sql
SELECT 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type,
    vi.vendor_id, 
    MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS 
month_market_season_sort, 
    mdi.market_season, 
    mdi.market_year,
    AVG(vi.original_price) AS avg_original_price,
    SUM(cp.quantity) AS quantity_sold,
    SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales
FROM product AS p
    LEFT JOIN vendor_inventory AS vi 
        ON vi.product_id = p.product_id 
    LEFT JOIN market_date_info AS mdi 
        ON vi.market_date = mdi.market_date 
    LEFT JOIN customer_purchases AS cp 
        ON vi.product_id = cp.product_id 
        AND vi.vendor_id = cp.vendor_id 
        AND vi.market_date = cp.market_date
GROUP BY 
    p.product_id, 
    p.product_name, 
    p.product_category_id, 
    p.product_qty_type, 
    vi.vendor_id, 
    mdi.market_year, 
    mdi.market_season
```

![Figure 13.20](Fotos/Chapter13/Fig_13.20.png)
<figcaption></figcaption>

- It would be good to:
  - Pick a few products.
  - Look through the detailed availability and purchase data.
  - Quality check these summarized results before continuing.

- Now that I have a sale price of each item:
  - I could start exploring the distribution of prices.

- When developing these queries:
  - I tried several different approaches.
  - The small number of products in the database can expose an issue with using `NTILE`s.
  - Two different items of the same price can end up in different `NTILE`s if the number of `NTILE`s splits the set at that point.

- After attempting multiple `NTILE` number options:
  - I realized I shouldn't be ranking the products.
  - I should be ranking the prices.

- Modified approach:
  - Group the records by `market_year`, `market_season`, and `original_price`.
  - Use an `NTILE` value of 3.
  - This gives groupings of the top, middle, and bottom 1/3 of prices.
  - Summarize the sales of products that fall into each of these price points.

- Here's the query that creates three price groupings per season:

```sql
SELECT  
    mdi.market_season, 
    mdi.market_year,
    MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS 
month_market_season_sort,
    vi.original_price, 
    NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price) AS 
price_ntile,
    NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price 
DESC) AS price_ntile_desc,
    COUNT(DISTINCT CONCAT(vi.product_id, vi.vendor_id)) product_count,
    SUM(cp.quantity) AS quantity_sold,
    SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales
FROM product AS p
    LEFT JOIN vendor_inventory AS vi 
        ON vi.product_id = p.product_id 
    LEFT JOIN market_date_info AS mdi 
        ON vi.market_date = mdi.market_date 
    LEFT JOIN customer_purchases AS cp 
        ON vi.product_id = cp.product_id 
        AND vi.vendor_id = cp.vendor_id 
        AND vi.market_date = cp.market_date
WHERE market_year IS NOT NULL 
GROUP BY 
    mdi.market_year, 
    mdi.market_season, 
    vi.original_price
```

![Figure 13.21](Fotos/Chapter13/Fig_13.21.png)
<figcaption></figcaption>

- Note:
  - The `price:ntile` break points vary by season.
  - In Summer 2019 and Summer 2020, a $4.00 item is in the middle price grouping.
  - In Spring, a $4.00 item is in the low price grouping.

- Created a column using the `NTILE` window function:
  - Same number of groups as `price:ntile`.
  - Sorting the price descending.
  - Filter to `price:ntile = 1` for the lowest price group.
  - Filter to `price:ntile_desc = 1` for the highest price group.

- In the previous query:
  - Didn't `COUNT(DISTINCT product_id)` values.
  - Concatenated `product_id` and `vendor_id`.
  - Did a distinct count of the combined values.
  - Different `product_id` sold by different vendors is considered a different product.

- Caveat:
  - Summing up different types of quantities.
  - Counting an ounce, pound, or unit product as “an item sold.”
  - Quantity isn't exactly apples-to-apples across seasons.
  - Gives a quick sales volume measure for rough comparison.

- Output in Figure 13.21:
  - Illustrates why I created a `month_market_season_sort`.
  - Alphabetically, the seasons sort out of order.
  - Use the sort values in the next query.

- Now we'll use the previous query as a `CTE` and summarize it:

```sql
WITH product_prices AS
(
    SELECT  
        mdi.market_season, 
        mdi.market_year,
        MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS 
month_market_season_sort,
        vi.original_price, 
        NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY 
original_price) AS price_ntile,
        NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY 
original_price DESC) AS price_ntile_desc,
        COUNT(DISTINCT CONCAT(vi.product_id, vi.vendor_id)) AS product_count,
        SUM(cp.quantity) AS quantity_sold,
        SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales
    FROM product AS p
        LEFT JOIN vendor_inventory AS vi 
            ON vi.product_id = p.product_id
        LEFT JOIN market_date_info AS mdi 
            ON vi.market_date = mdi.market_date
        LEFT JOIN customer_purchases AS cp 
            ON vi.product_id = cp.product_id 
            AND vi.vendor_id = cp.vendor_id 
            AND vi.market_date = cp.market_date
    WHERE market_year IS NOT NULL 
    GROUP BY 
        mdi.market_year, 
        mdi.market_season, 
        vi.original_price
)
 
SELECT 
    market_year, 
    market_season, 
    price_ntile, 
    SUM(product_count) AS product_count, 
    SUM(quantity_sold) AS quantity_sold, 
    MIN(original_price) AS min_price, 
    MAX(original_price) AS max_price, 
    SUM(total_sales) AS total_sales
FROM product_prices 
GROUP BY 
    market_year, 
    market_season, 
    price_ntile
ORDER BY 
    market_year, 
    month_market_season_sort, 
    price_ntile
```

![Figure 13.22](Fotos/Chapter13/Fig_13.22.png)
<figcaption></figcaption>

- In Figure 13.22:
  - We're now sorting the seasons in the correct order.
  - Displaying the minimum and maximum price per grouping.
  - Showing the `total_sales`.

- This gives us the data to answer the second question:
  - With this sample data, `total_sales` is highest for `price:ntile 3` in every season.
  - Higher-priced items are generating the most sales for the `market`.

- Hopefully, these dataset creation walkthroughs:
  - Show the variety of ways to combine and summarize data.
  - Help create a dataset to answer analytical questions.
  - Highlight the clarifying questions an experienced analyst might ask.

- Keep in mind:
  - Each dataset can be refreshed by rerunning the queries.
  - The results can be reused to answer many questions, not just the ones initially asked!