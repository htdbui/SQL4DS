---
title: "SQL Chapter 9"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">
<section aria-labelledby="c09_1" type="chapter" role="doc-chapter" class="calibre2">
<header class="calibre3">
<h1 id="calibre_link-39" class="calibre4"><span aria-label="127" type="pagebreak" id="calibre_link-40" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-41" class="calibre6"></a><span class="calibre">CHAPTER 9</span><br class="calibre7" /><span class="calibre">Exploratory Data Analysis with SQL</span></h1>
</header>
<section aria-label="chapter opening" class="calibre2"><span id="calibre_link-42" class="calibre"></span>
<p id="calibre_link-43" class="calibre8">Exploratory Data Analysis (EDA) is often discussed in a data science context as a first step in the predictive modeling process, when a data scientist explores what the data in a provided dataset looks like prior to using it to build a predictive model. The SQL we'll be using in this chapter could be used at that point in the process, to explore an already-prepared dataset. But what if you don't have a dataset to work with yet?</p>
<p id="calibre_link-44" class="calibre8">Here we'll show examples that could occur even earlier in the data pipeline, as we explore raw data straight from the database tables (as opposed to an already-aggregated dataset in which the raw data has been combined and transformed using SQL that is ready to be ingested into a model). If you are given access to a database for the first time, these are the types of queries you can run to familiarize yourself with the tables and data in it.</p>
<p id="calibre_link-45" class="calibre8">There are of course many ways to conduct EDA, including in a Jupyter notebook with Python code, in a Tableau workbook, or using SQL. (I regularly do all three in my job as a data scientist.) In the later EDA, once a dataset has been prepared, the focus is often on distributions of values, relationships between columns, and identifying correlations between input features and the target variable (column with values to be predicted by the model). Here, we will use the types of queries we've covered so far in this book to explore some tables in the Farmer's Market database, as a demonstration of a real EDA focusing on familiarizing ourselves with the data in the database for the first time.</p>
</section>
<section aria-labelledby="head-2-67" class="calibre2"><span id="calibre_link-46" class="calibre"></span>
<h2 id="calibre_link-47" class="calibre9">Demonstrating Exploratory Data Analysis with SQL</h2>
<p id="calibre_link-48" class="calibre8"><span aria-label="128" type="pagebreak" id="calibre_link-49" role="doc-pagebreak" class="calibre10"></span>Let's start with a real-world scenario for this example Exploratory Data Analysis: Let's say the Director of the Farmer's Market asks us to help them build some reports to use throughout the year, and gives us access to the database referenced in this book. They haven't yet given us any specific report requirements, but they have told us that they'll be asking questions related to general product availability and purchase trends, and have given us the E-R diagram found in <a href="c01.xhtml" class="calibre6">Chapter 1</a>, “Data Sources,” so we know the relationships between the tables.</p>
<p id="calibre_link-50" class="calibre8">Based on the little information we have, we might guess that we should familiarize ourselves with the 
<code class="calibre11">product</code>
, 
<code class="calibre11">vendor_inventory</code>
, and 
<code class="calibre11">customer_purchases</code>
 tables, because we've been told we'll be building reports on “product availability” and “purchase trends.”</p>
<p class="calibre8">Some sensible questions to ask via query are:</p>
<ul class="square" id="calibre_link-51">
<li id="calibre_link-52" class="calibre12">How large are the tables, and how far back in time does the data go?</li>
<li id="calibre_link-53" class="calibre12">What kind of information is available about each product and each purchase?</li>
<li id="calibre_link-54" class="calibre12">What is the granularity of each of these tables; what makes a row unique?</li>
<li id="calibre_link-55" class="calibre12">Since we'll be looking at trends over time, what kind of date and time dimensions are available, and how do the different values look when summarized over time?</li>
<li id="calibre_link-56" class="calibre12">How is the data in each table related to the other tables? How might we join them together to summarize the details for reporting?</li>
</ul>
</section>
<section aria-labelledby="head-2-68" class="calibre2"><span id="calibre_link-57" class="calibre"></span>
<h2 id="calibre_link-58" class="calibre9">Exploring the Products Table</h2>
<p id="calibre_link-59" class="calibre8">Some databases (like MySQL) offer a function called 
<code class="calibre11">DESCRIBE</code>
 [table name] or 
<code class="calibre11">DESC</code>
 [table name], or have a special schema to select from to list the columns, data types, and other settings for fields in tables, but this function isn't available in every database system and doesn't show a preview of the data, so we'll take a more universal approach here to preview data in a table.</p>
<p class="calibre8">Let's start with the 
<code class="calibre11">product</code>
 table first. We'll select everything in the table, to see what kind of data is in each column, but limit it to 10 rows in case it is a large table:</p>
<pre id="calibre_link-60" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.product</code>
<code class="calibre11">LIMIT 10</code>
</pre>
<p id="calibre_link-61" class="calibre8">The output from this query is shown in <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 9.1</a>. What do we notice in this output? We can see that there is a 
<code class="calibre11">product_id</code>
, 
<code class="calibre11">product_name</code>
, 
<code class="calibre11">product_size</code>
, <span aria-label="129" type="pagebreak" id="calibre_link-62" role="doc-pagebreak" class="calibre10"></span>
<code class="calibre11">product_category_id</code>
, and 
<code class="calibre11">product_qty_type</code>
 on each row, and at least in this small subset, it appears that most of the fields are populated (there aren't a lot of NULL values). This table looks like a catalog of products, with product metadata like name and category. It doesn't list individual items for sale by vendors or purchased by customers, like a transactional table would. The 
<code class="calibre11">product_category_id</code>
 is an integer, and we know there is a 
<code class="calibre11">product_category</code>
 table, so we might assume that is a foreign key and check out that relationship later in the EDA. There are many 
<code class="calibre11">product_name</code>
 and 
<code class="calibre11">product_size</code>
 values, but in this preview, only two 
<code class="calibre11">product_qty_type</code>
 values, “lbs” and “unit.”</p>
<figure class="calibre14">
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 9.1</a></span></p>
</figcaption>
</figure>
<p class="calibre8">The 
<code class="calibre11">product_id</code>
 appears to be the primary key. What if we didn't know whether it was a unique identifier? To check to see if any two rows have the same 
<code class="calibre11">product_id</code>
, we can write a query that groups by the 
<code class="calibre11">product_id</code>
 and returns any groups with more than one record. This isn't a guarantee that it is the primary key, but can tell you whether, at least currently, the 
<code class="calibre11">product_id</code>
 is unique per record:</p>
<pre id="calibre_link-63" class="calibre13">
<code class="calibre11">SELECT product_id, count(*)</code>
<code class="calibre11">FROM farmers_market.product</code>
<code class="calibre11">GROUP BY product_id</code>
<code class="calibre11">HAVING count(*)&gt; 1</code>
</pre>
<p id="calibre_link-64" class="calibre8">There are no results returned, so no 
<code class="calibre11">product_id</code>
 groups have more than one row, meaning each 
<code class="calibre11">product_id</code>
 is unique, and we can say that this table has a <i class="calibre16">granularity</i> of one row per product</p>
<p class="calibre8">What about the product categories we see IDs for in the 
<code class="calibre11">product</code>
 table? How many different categories are there, and what do those look like? Let's see what's in the 
<code class="calibre11">product_category</code>
 table:</p>
<pre id="calibre_link-65" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.product_category</code>
</pre>
<p id="calibre_link-66" class="calibre8">The results of this query are shown in <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 9.2</a>, and the listing of categories gives a sense of how the Farmer's Market groups its different types of products. This might be useful when we need to write reports, because we could report on inventory and trends by product category.<span aria-label="130" type="pagebreak" id="calibre_link-67" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records product category id and product category name." class="center" src="images/000000.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 9.2</a></span></p>
</figcaption>
</figure>
<p class="calibre8">How many different products are there in the catalog-like product metadata table?</p>
<pre id="calibre_link-68" class="calibre13">
<code class="calibre11">SELECT count(*) FROM farmers_market.product</code>
</pre>
<p id="calibre_link-69" class="calibre8">As you can see in <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 9.3</a>, only 23 products have been entered into this database so far. If this were a report, we would want to give this column a header besides 
<code class="calibre11">count(*)</code>
, but during an Exploratory Data Analysis, you are often moving quickly and writing queries for your own information and not for display purposes, so there is not a need to alias the columns in every query.</p>
<figure class="calibre14">
<img alt="A table records the calculation of count." class="center" src="images/000001.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 9.3</a></span></p>
</figcaption>
</figure>
<p class="calibre8">We might next ask, “How many products are there per product category?” We'll quickly join the 
<code class="calibre11">product</code>
 table and the 
<code class="calibre11">product_category</code>
 table to pull in the category names that we think go with the IDs here, and count up the products in each category:</p>
<pre id="calibre_link-70" class="calibre13">
<code class="calibre11">SELECT pc.product_category_id, pc.product_category_name, </code>
   
<code class="calibre11"> count(product_id) AS count_of_products</code>
<code class="calibre11">FROM farmers_market.product_category AS pc</code>
<code class="calibre11">LEFT JOIN farmers_market.product AS p </code>
   
<code class="calibre11"> ON pc.product_category_id = p.product_category_id</code>
<code class="calibre11">GROUP BY pc.product_category_id</code>
</pre>
<p id="calibre_link-71" class="calibre8">In <a href="#calibre_link-7" id="calibre_link-8" class="calibre6">Figure 9.4</a>, you can see that the IDs did all match up with categories, and the most common product category is “Fresh Fruits &amp; Vegetables.” The “Freshly Prepared Food” category does not yet have any products in it.<span aria-label="131" type="pagebreak" id="calibre_link-72" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records product category id and product category name." class="center" src="images/000002.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-8" id="calibre_link-7" role="doc-backlink" class="calibre6">Figure 9.4</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-69" class="calibre2"><span id="calibre_link-73" class="calibre"></span>
<h2 id="calibre_link-74" class="calibre9">Exploring Possible Column Values</h2>
<p class="calibre8">To further familiarize ourselves with the range of values in a column that appears to contain string values that represent categories, we might ask, “What is in the 
<code class="calibre11">product_qty_type</code>
 field we saw in our first preview of the 
<code class="calibre11">product</code>
 table? And how many different quantity types are there?” which could be answered with this query using the 
<code class="calibre11">DISTINCT</code>
 keyword:</p>
<pre id="calibre_link-75" class="calibre13">
<code class="calibre11">SELECT DISTINCT product_qty_type</code>
<code class="calibre11">FROM farmers_market.product</code>
</pre>
<p id="calibre_link-76" class="calibre8">As you can see in <a href="#calibre_link-9" id="calibre_link-10" class="calibre6">Figure 9.5</a>, the two quantity types we saw in the 10-row preview turned out to be the only two values in the column, though some products have a NULL 
<code class="calibre11">product_qty_type</code>
, so we'll have to remember that when doing any sort of filtering or calculation based on this column.</p>
<figure class="calibre14">
<img alt="A table records the product quantity type." class="center" src="images/000003.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-9" role="doc-backlink" class="calibre6">Figure 9.5</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Let's take a look at some of the data in the 
<code class="calibre11">vendor_inventory</code>
 table next:</p>
<pre id="calibre_link-77" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.vendor_inventory</code>
<code class="calibre11">LIMIT 10</code>
</pre>
<p id="calibre_link-78" class="calibre8">In the output shown in <a href="#calibre_link-11" id="calibre_link-12" class="calibre6">Figure 9.6</a>, we can see that it looks like there is one row per 
<code class="calibre11">market_date</code>
, 
<code class="calibre11">vendor_id</code>
, and 
<code class="calibre11">product_id</code>
, with the quantity of that product that the vendor brought to each market, and the 
<code class="calibre11">original_price</code>
. We might need to ask the Director what the “original price” of an item is, but with that name, we might guess that it is the item price before any sales or special deals. And it is tracked per market date, so changes over time would be recorded. We can also see that the quantity is a decimal value, at least for the product displayed.<span aria-label="132" type="pagebreak" id="calibre_link-79" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records market date, quantity, vendor id, product id, and original price." class="center" src="images/000004.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-12" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 9.6</a></span></p>
</figcaption>
</figure>
<p class="calibre8">We should confirm our assumption about the primary key. If your database system offers the 
<code class="calibre11">DESCRIBE</code>
 function or a schema to query from to see table properties, that could give definite confirmation that a field is set to be a unique primary key. (See your database's documentation.) Otherwise, we can group by the fields we expect are unique and use 
<code class="calibre11">HAVING</code>
 to check whether there is more than one record with each combination, like we did previously with the 
<code class="calibre11">product_id</code>
 in the 
<code class="calibre11">product</code>
 table:</p>
<pre id="calibre_link-80" class="calibre13">
<code class="calibre11">SELECT market_date, vendor_id, product_id, count(*)</code>
<code class="calibre11">FROM farmers_market.vendor_inventory</code>
<code class="calibre11">GROUP BY market_date, vendor_id, product_id</code>
<code class="calibre11">HAVING count(*)&gt; 1</code>
</pre>
<p id="calibre_link-81" class="calibre8">There are no combinations of these three values that occur on more than one row, so at least with the currently entered data, this combination of fields is indeed unique, and the 
<code class="calibre11">vendor_inventory</code>
 table has a granularity of one record per market date, vendor, and product. It is not visible in the version of the E-R diagram displayed in <a href="c01.xhtml" class="calibre6">Chapter 1</a>, but it's possible to highlight the primary key of a table in MySQL Workbench, as shown in <a href="#calibre_link-13" id="calibre_link-14" class="calibre6">Figure 9.7</a>. Here, we confirm that it is a composite primary key, made up of the three fields we guessed.<span aria-label="133" type="pagebreak" id="calibre_link-82" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records the options in vendor inventory." class="center" src="images/000005.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-14" id="calibre_link-13" role="doc-backlink" class="calibre6">Figure 9.7</a></span></p>
</figcaption>
</figure>
<p class="calibre8">We also saw in <a href="#calibre_link-11" class="calibre6">Figure 9.6</a> that there are dates in this table, in the 
<code class="calibre11">market_date</code>
 field. So we might ask, “How far back does the data go: When was the first market that was tracked in this database, and how recent is the latest data?” To answer this, we can get the minimum (earliest) and maximum (latest) values from that field:</p>
<pre id="calibre_link-83" class="calibre13">
<code class="calibre11">SELECT min(market_date), max(market_date)</code>
<code class="calibre11">FROM farmers_market.vendor_inventory</code>
</pre>
<p id="calibre_link-84" class="calibre8">As you can see in <a href="#calibre_link-15" id="calibre_link-16" class="calibre6">Figure 9.8</a>, it looks like we have about one and a half years’ worth of records. So, if we're asked to build any kind of forecast involving an annual seasonality, we will have to explain that we have limited training data for that, since we don't have multiple complete years of seasonal trends yet. It would be good to check to see if purchases were tracked during that entire period, as well.</p>
<figure class="calibre14">
<img alt="A table records minimum and maximum market date." class="center" src="images/000006.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-16" id="calibre_link-15" role="doc-backlink" class="calibre6">Figure 9.8</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Another question we might ask to better understand the Farmer's Market and the data it generates is: How many different vendors are there, and when did they each start selling at the market? And which are still selling at the most recent 
<code class="calibre11">market_date</code>
? We can do that by grouping the previous query by 
<code class="calibre11">vendor_id</code>
 to get the earliest and latest dates for which each vendor had inventory. We will also sort by these dates, to see which vendors were selling the longest ago, and the most recently:</p>
<pre id="calibre_link-85" class="calibre13">
<code class="calibre11">SELECT vendor_id, min(market_date), max(market_date)</code>
<code class="calibre11">FROM farmers_market.vendor_inventory</code>
<code class="calibre11">GROUP BY vendor_id</code>
<code class="calibre11">ORDER BY min(market_date), max(market_date)</code>
</pre>
<p id="calibre_link-86" class="calibre8">The output in <a href="#calibre_link-17" id="calibre_link-18" class="calibre6">Figure 9.9</a> shows that the inventories of only three vendors have been added to the database so far. Vendors 7 and 8 have been selling since April 3, 2019, and most recently brought inventory to the October 10, 2020 market. Vendor 4 started later and was last at the market on September 30, 2020.<span aria-label="134" type="pagebreak" id="calibre_link-87" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records vendor id, minimum and maximum market date." class="center" src="images/000007.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-18" id="calibre_link-17" role="doc-backlink" class="calibre6">Figure 9.9</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-70" class="calibre2"><span id="calibre_link-88" class="calibre"></span>
<h2 id="calibre_link-89" class="calibre9">Exploring Changes Over Time</h2>
<p id="calibre_link-90" class="calibre8">One thought that comes to mind looking at the dates during this exploration is that we should check to find out if there is an indicator in the database showing whether the market changed format, perhaps allowing online orders during the COVID-19 pandemic in 2020, since that might have impacted sales, and that indicator could be valuable for predictive modeling.</p>
<p class="calibre8">Another question we might ask the database after seeing the output in <a href="#calibre_link-17" class="calibre6">Figure 9.9</a> is: Do most vendors sell at the market year-round, or is there a certain time of year when there are different numbers of vendors at the farmer’s market? We'll extract the month and year from each market date and look at the counts of vendors that appear in the data each month:</p>
<pre id="calibre_link-91" class="calibre13">
<code class="calibre11">SELECT</code>
<code class="calibre11">EXTRACT(YEAR FROM market_date) AS market_year,</code>
<code class="calibre11">EXTRACT(MONTH FROM market_date) AS market_month,</code>
<code class="calibre11">COUNT(DISTINCT vendor_id) AS vendors_with_inventory</code>
<code class="calibre11">FROM farmers_market5.vendor_inventory</code>
<code class="calibre11">GROUP BY EXTRACT(YEAR FROM market_date), EXTRACT(MONTH FROM market_date)</code>
<code class="calibre11">ORDER BY EXTRACT(YEAR FROM market_date), EXTRACT(MONTH FROM market_date)</code>
</pre>
<p id="calibre_link-92" class="calibre8">Since only three vendors have inventory entered into this example database, there isn't much variation seen in this output, but you can see in <a href="#calibre_link-19" id="calibre_link-20" class="calibre6">Figure 9.10</a> that there are three vendors in June through September, and two vendors per month the rest of the year. So one of the vendors (likely vendor 4, from the date ranges we saw in <a href="#calibre_link-17" class="calibre6">Figure 9.9</a>) may be a seasonal vendor.<span aria-label="135" type="pagebreak" id="calibre_link-93" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records market year, month, and vendors with inventory." class="center" src="images/000008.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-20" id="calibre_link-19" role="doc-backlink" class="calibre6">Figure 9.10</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-94" class="calibre8">Perhaps surprisingly, we can also see in <a href="#calibre_link-19" class="calibre6">Figure 9.10</a> that there are no months 1 and 2 listed in this output, only months 3&ndash;12, so the farmer’s market must be closed in January and February. (We'll have to remember to check with the Director to see whether that is true, because if not, there might be an issue with the data.) These are the aspects of the data that we want to discover during EDA instead of later when we're doing analysis, so we know what to expect in the report output.</p>
<p class="calibre8">Next, let's look at the details of what a particular vendor's inventory looks like. We saw a vendor with an ID of 7 in a previous query, so we'll explore their inventory first:</p>
<pre id="calibre_link-95" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.vendor_inventory</code>
<code class="calibre11">WHERE vendor_id = 7</code>
<code class="calibre11">ORDER BY market_date, product_id</code>
</pre>
<p id="calibre_link-96" class="calibre8">Part of the output of this query is shown in <a href="#calibre_link-21" id="calibre_link-22" class="calibre6">Figure 9.11</a>. In the limited amount of data visible in the screenshot, we can see that this vendor was only selling one product through most of May and June, then there are some other products (with IDs 1&ndash;3) that appear in July. We can also see that product 4 has not changed price at all and is $4.00 throughout the period visible in <a href="#calibre_link-21" class="calibre6">Figure 9.11</a>.</p>
<figure class="calibre14">
<img alt="A table records market date, quantity, vendor id, product id, and original price." class="center" src="images/000009.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-22" id="calibre_link-21" role="doc-backlink" class="calibre6">Figure 9.11</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-71" class="calibre2"><span id="calibre_link-97" class="calibre"></span>
<h2 id="calibre_link-98" class="calibre9">Exploring Multiple Tables Simultaneously</h2>
<p id="calibre_link-99" class="calibre8">Some of the products have round quantities, and some appear to be continuous numbers, possibly products sold by weight. This vendor always brings either 30 or 40 of product 4 to each market. It would be interesting to see how many of those items are sold at each market, in comparison.</p>
<p class="calibre8"><span aria-label="136" type="pagebreak" id="calibre_link-100" role="doc-pagebreak" class="calibre10"></span>Let's jump over to the 
<code class="calibre11">customer_purchases</code>
 table to get a sense of what the purchases of that product look like, compared to the vendor's inventory. First, we need to see what data is available in that table, so we'll select all fields and 10 rows to get a preview:</p>
<pre id="calibre_link-101" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.customer_purchases</code>
<code class="calibre11">LIMIT 10</code>
</pre>
<p id="calibre_link-102" class="calibre8">As shown in <a href="#calibre_link-23" id="calibre_link-24" class="calibre6">Figure 9.12</a>, each row has a 
<code class="calibre11">product_id</code>
, 
<code class="calibre11">vendor_id</code>
, 
<code class="calibre11">market_date</code>
, 
<code class="calibre11">customer_id</code>
, 
<code class="calibre11">quantity</code>
, 
<code class="calibre11">cost_to_customer_per_qty</code>
, and 
<code class="calibre11">transaction_time</code>
. We can see that each product purchased by a customer is in its own row, but multiple units of the same product can be recorded in one row, as indicated by the quantity field. We might also guess from the 
<code class="calibre11">cost_to_customer_per_qty</code>
 field in this output that the reason the 
<code class="calibre11">original_price</code>
 is recorded in the 
<code class="calibre11">vendor_inventory</code>
 table is because different customers might pay different prices for the same items. We also notice that the quantity and 
<code class="calibre11">cost_to_customer_per_qty</code>
 values are numeric values with two digits after the decimal point.</p>
<figure class="calibre14">
<img alt="A table records product id, vendor id, market date, customer id, quantity, cost to customer per quantity, and transaction time." class="center" src="images/000010.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-24" id="calibre_link-23" role="doc-backlink" class="calibre6">Figure 9.12</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-103" class="calibre8">It's also interesting that each individual purchase's 
<code class="calibre11">transaction_time</code>
 is recorded, in addition to the date. This could allow us to build reports on the flow of customers to the market throughout the day, or to estimate how long each customer spends at the market, by looking for their earliest and latest purchase times.</p>
<p class="calibre8">Since we see 
<code class="calibre11">vendor_id</code>
 and 
<code class="calibre11">product_id</code>
 are both included here, we can look closer at purchases of vendor 7's product #4:</p>
<pre id="calibre_link-104" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.customer_purchases</code>
<code class="calibre11">WHERE vendor_id = 7 AND product_id = 4</code>
<code class="calibre11">ORDER BY market_date, transaction_time</code>
</pre>
<p id="calibre_link-105" class="calibre8">Glancing at this data in <a href="#calibre_link-25" id="calibre_link-26" class="calibre6">Figure 9.13</a>, we can see that for the two market dates that are visible, most customers are buying between 1 and 5 of these items at a time and spending $4 per item.<span aria-label="137" type="pagebreak" id="calibre_link-106" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records product id, vendor id, market date, customer id, quantity, cost to customer per quantity, and transaction time." class="center" src="images/000011.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-26" id="calibre_link-25" role="doc-backlink" class="calibre6">Figure 9.13</a></span></p>
</figcaption>
</figure>
<p class="calibre8">I see 
<code class="calibre11">customer_id</code>
 12 in <a href="#calibre_link-25" class="calibre6">Figure 9.13</a> several times, and out of curiosity, we might run the same query but filtered to, or sorted by, the 
<code class="calibre11">customer_id</code>
 to explore one customer's purchase history of a product in more detail:</p>
<pre id="calibre_link-107" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.customer_purchases</code>
<code class="calibre11">WHERE vendor_id = 7 AND product_id = 4 AND customer_id = 12</code>
<code class="calibre11">ORDER BY customer_id, market_date, transaction_time</code>
</pre>
<p id="calibre_link-108" class="calibre8">This is simulated (generated) data, and I know this result shown in <a href="#calibre_link-27" id="calibre_link-28" class="calibre6">Figure 9.14</a> occurred because there are a limited number of customer IDs in the database to associate with purchases so far, but it looks like customer 12 really likes this product. They first purchased it on April 3, then came back and bought five more three days later on April 6, only to come back 30 minutes later on the same day to buy five more! Sorting and filtering the data different ways&mdash;by market date and time, by 
<code class="calibre11">vendor_id</code>
, by 
<code class="calibre11">customer_id</code>
&mdash;you can get a sense of the processes that the data represents and see if anything stands out that might be worth following up on with the subject matter experts.</p>
<figure class="calibre14">
<img alt="A table records product id, vendor id, market date, customer id, quantity, cost to customer per quantity, and transaction time." class="center" src="images/000012.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-28" id="calibre_link-27" role="doc-backlink" class="calibre6">Figure 9.14</a></span></p>
</figcaption>
</figure>
<p class="calibre8">We can see by looking at this detailed view of the 
<code class="calibre11">customer_purchases</code>
 data that there are multiple sales recorded per vendor per product per day, but we can't get a good sense of what that looks like in summary. And since we wanted to compare the sales per day to the inventory that the vendor brought to each market, we'll want to aggregate these sales by market date. So, we can group by <span aria-label="138" type="pagebreak" id="calibre_link-109" role="doc-pagebreak" class="calibre10"></span>
<code class="calibre11">market_date</code>
, 
<code class="calibre11">vendor_id</code>
, and 
<code class="calibre11">product_id</code>
, and add up the quantities sold and each row's quantity (?): multiplied by the cost to get the total sales for each group:</p>
<pre id="calibre_link-110" class="calibre13">
<code class="calibre11">SELECT market_date, </code>
   
<code class="calibre11"> vendor_id, </code>
   
<code class="calibre11"> product_id, </code>
   
<code class="calibre11"> SUM(quantity) quantity_sold, </code>
   
<code class="calibre11"> SUM(quantity * cost_to_customer_per_qty) total_sales</code>
<code class="calibre11">FROM farmers_market.customer_purchases</code>
<code class="calibre11">WHERE vendor_id = 7 and product_id = 4</code>
<code class="calibre11">GROUP BY market_date, vendor_id, product_id</code>
<code class="calibre11">ORDER BY market_date, vendor_id, product_id</code>
</pre>
<p id="calibre_link-111" class="calibre8">You can see the results of this in <a href="#calibre_link-29" id="calibre_link-30" class="calibre6">Figure 9.15</a>.</p>
<figure class="calibre14">
<img alt="A table records market id, vendor id, product id, quantity sold, and total sales." class="center" src="images/000013.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-30" id="calibre_link-29" role="doc-backlink" class="calibre6">Figure 9.15</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-112" class="calibre8">Again, if we were building a report, we would probably want to round those dollars in the last column to two numbers after the decimal, but since this is an exploration for our own information, I won't spend the extra time on formatting.</p>
</section>
<section aria-labelledby="head-2-72" class="calibre2"><span id="calibre_link-113" class="calibre"></span>
<h2 id="calibre_link-114" class="calibre9">Exploring Inventory vs. Sales</h2>
<p id="calibre_link-115" class="calibre8">Now we have all of the information we need to answer our question about the sales of this product compared to the inventory except the inventory counts!</p>
<p id="calibre_link-116" class="calibre8">Throughout the EDA so far, we have gotten a sense of what the data in each of these related tables looks like, and now we can start joining them together to get a better sense of the relationship between entities. For example, now that we have aggregated the 
<code class="calibre11">customer_purchases</code>
 to the same granularity of the 
<code class="calibre11">vendor_inventory</code>
 table&mdash;one row per 
<code class="calibre11">market_date</code>
, 
<code class="calibre11">vendor_id</code>
, and 
<code class="calibre11">product_id</code>
&mdash;we can join the two tables together to view inventory side by side with sales.</p>
<p class="calibre8"><span aria-label="139" type="pagebreak" id="calibre_link-117" role="doc-pagebreak" class="calibre10"></span>First, we'll join the two tables (the details of one and the summary of the other) and display all columns to check to make sure it's combining the way we expect it to. We'll give the 
<code class="calibre11">customer_purchases</code>
 summary table the alias “sales,” and limit the output to 10 rows since we haven't filtered to a vendor or product yet, so this query will return a lot of rows:</p>
<pre id="calibre_link-118" class="calibre13">
<code class="calibre11">SELECT * FROM farmers_market.vendor_inventory AS vi</code>
   
<code class="calibre11"> LEFT JOIN</code>
       
<code class="calibre11"> (</code>
       
<code class="calibre11"> SELECT market_date, </code>
           
<code class="calibre11"> vendor_id, </code>
           
<code class="calibre11"> product_id, </code>
           
<code class="calibre11"> SUM(quantity) AS quantity_sold, </code>
           
<code class="calibre11"> SUM(quantity * cost_to_customer_per_qty) AS total_sales</code>
       
<code class="calibre11"> FROM farmers_market.customer_purchases</code>
       
<code class="calibre11"> GROUP BY market_date, vendor_id, product_id</code>
       
<code class="calibre11"> ) AS sales</code>
       
<code class="calibre11"> ON vi.market_date = sales.market_date </code>
           
<code class="calibre11"> AND vi.vendor_id = sales.vendor_id </code>
           
<code class="calibre11"> AND vi.product_id = sales.product_id</code>
<code class="calibre11">ORDER BY vi.market_date, vi.vendor_id, vi.product_id</code>
<code class="calibre11">LIMIT 10</code>
</pre>
<p id="calibre_link-119" class="calibre8">We can see in <a href="#calibre_link-31" id="calibre_link-32" class="calibre6">Figure 9.16</a> that the 
<code class="calibre11">vendor_id</code>
, 
<code class="calibre11">product_id</code>
, and 
<code class="calibre11">market_date</code>
 do match on every row, and the summary values for 
<code class="calibre11">vendor_id</code>
 8 and 
<code class="calibre11">product_id</code>
 4 match what we saw when looking at the 
<code class="calibre11">customer_purchases</code>
 table alone. This confirms that our join looks right, and we can remove these redundant columns by specifying which columns we want to display from each table. We'll make sure to pull these columns in from the 
<code class="calibre11">vendor_inventory</code>
 table, which we made the “left” side of the 
<code class="calibre11">JOIN</code>
 relationship because a customer can't buy inventory that doesn't exist (Theoretically! We should check the data to make sure that never happens, which might indicate a vendor made a mistake in entering their available inventory.) But there could be inventory that doesn't get purchased, which we would want to see in the output, but would be missing if we chose a 
<code class="calibre11">RIGHT JOIN</code>
 instead.<span aria-label="140" type="pagebreak" id="calibre_link-120" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records market id, vendor id, product id, quantity sold, and total sales." class="center" src="images/000014.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-32" id="calibre_link-31" role="doc-backlink" class="calibre6">Figure 9.16</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-121" class="calibre8">We can also join in additional “lookup” tables to convert the various IDs to human-readable values, pulling in the vendor name and product names.</p>
<p class="calibre8">Then, we can filter to vendor 7 and product 4 to get the information we were looking for earlier, comparing this vendor's inventory of this product to the sales made at each market:</p>
<pre id="calibre_link-122" class="calibre13">
<code class="calibre11">SELECT vi.market_date, </code>
   
<code class="calibre11"> vi.vendor_id, </code>
   
<code class="calibre11"> v.vendor_name, </code>
   
<code class="calibre11"> vi.product_id, </code>
   
<code class="calibre11"> p.product_name,</code>
   
<code class="calibre11"> vi.quantity AS quantity_available,</code>
   
<code class="calibre11"> sales.quantity_sold, </code>
   
<code class="calibre11"> vi.original_price, </code>
   
<code class="calibre11"> sales.total_sales</code>
<code class="calibre11">FROM farmers_market.vendor_inventory AS vi</code>
   
<code class="calibre11"> LEFT JOIN</code>
       
<code class="calibre11"> (</code>
       
<code class="calibre11"> SELECT market_date, </code>
           
<code class="calibre11"> vendor_id, </code>
           
<code class="calibre11"> product_id, </code>
           
<code class="calibre11"> SUM(quantity) AS quantity_sold, </code>
           
<code class="calibre11"> SUM(quantity * cost_to_customer_per_qty) AS total_sales</code>
       
<code class="calibre11"> FROM farmers_market.customer_purchases</code>
       
<code class="calibre11"> GROUP BY market_date, vendor_id, product_id</code>
       
<code class="calibre11"> ) AS sales</code>
   
<code class="calibre11"> ON vi.market_date = sales.market_date </code>
       
<code class="calibre11"> AND vi.vendor_id = sales.vendor_id</code>
       
<code class="calibre11"> AND vi.product_id = sales.product_id</code>
   
<code class="calibre11"> LEFT JOIN farmers_market.vendor v </code>
       
<code class="calibre11"> ON vi.vendor_id = v.vendor_id</code>
   
<code class="calibre11"> LEFT JOIN farmers_market.product p</code>
       
<code class="calibre11"> ON vi.product_id = p.product_id</code>
<code class="calibre11">WHERE vi.vendor_id = 7 </code>
       
<code class="calibre11"> AND vi.product_id = 4</code>
<code class="calibre11">ORDER BY vi.market_date, vi.vendor_id, vi.product_id</code>
</pre>
<p id="calibre_link-123" class="calibre8">Now we can see in a sample of this query's output in <a href="#calibre_link-33" id="calibre_link-34" class="calibre6">Figure 9.17</a> that this vendor is called Marco's Peppers, and the product we were looking at is jars of Banana Peppers. He brings 30&ndash;40 jars each time and sells between 1 and 40 jars per market (which we quickly determined by sorting the output in the SQL editor ascending and descending by the 
<code class="calibre11">quantity_sold</code>
, but we could've also done by adding 
<code class="calibre11">quantity_sold</code>
 to the 
<code class="calibre11">ORDER BY</code>
 clause of the query and scrolling to the top and bottom of the output or surrounding this query with another query that calculated the 
<code class="calibre11">MIN</code>
 and 
<code class="calibre11">MAX quantity_sold</code>
).<span aria-label="141" type="pagebreak" id="calibre_link-124" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records market id, vendor id, product id, quantity sold, and total sales." class="center" src="images/000015.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-34" id="calibre_link-33" role="doc-backlink" class="calibre6">Figure 9.17</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-125" class="calibre8">To take a closer look at the distribution of sales and continue our EDA visually, or to easily filter to different combinations of vendors and products, we can remove the 
<code class="calibre11">WHERE</code>
 clause filter and pull the dataset generated by this query into reporting software such as Tableau, building dashboards and reports. <a href="#calibre_link-35" id="calibre_link-37" class="calibre6">Figure 9.18</a> shows a dashboard with the inventory and sales of the selected products during the selected date range. <a href="#calibre_link-36" id="calibre_link-38" class="calibre6">Figure 9.19</a> shows a histogram with how many different market dates each count of jars of Banana Peppers were sold.</p>
<p id="calibre_link-126" class="calibre8">We could also do a lot more calculations in SQL using this dataset, such as calculating the percent of each vendor's inventory sold at each market, but the purpose of this demonstration was to show how much you can learn about the data in a database using the types of SQL queries you have learned in this book so far.</p>
<p id="calibre_link-127" class="calibre8">In the next chapter, we'll explore more analytical queries for building reports using SQL that go beyond simple data previews and basic summaries.</p>
<figure class="calibre14">
<img alt="A graph titled, Marco&apos;s peppers inventory versus solid." class="center" src="images/000016.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-37" id="calibre_link-35" role="doc-backlink" class="calibre6">Figure 9.18</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="Histogram titled, Banana peppers jar sales per market." class="center" src="images/000017.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-38" id="calibre_link-36" role="doc-backlink" class="calibre6">Figure 9.19</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-73" class="calibre2"><span id="calibre_link-128" class="calibre"></span>
<h2 id="calibre_link-129" class="calibre9">Exercises</h2>
<ol class="decimal" id="calibre_link-130">
<li id="calibre_link-131" class="calibre12"><span aria-label="142" type="pagebreak" id="calibre_link-132" role="doc-pagebreak" class="calibre10"></span>In the chapter, it was suggested that we should see if the 
<code class="calibre11">customer_purchases</code>
 data was collected for the same time frame as the 
<code class="calibre11">vendor_inventory</code>
 table. Write a query that gets the earliest and latest dates in the 
<code class="calibre11">customer_purchases</code>
 table.</li>
<li id="calibre_link-133" class="calibre12">There is a MySQL function 
<code class="calibre11">DAYNAME()</code>
 that returns the name of the day of the week for a date. Using the 
<code class="calibre11">DAYNAME</code>
 and 
<code class="calibre11">EXTRACT</code>
 functions on the 
<code class="calibre11">customer_purchases</code>
 table, select and group by the weekday and hour of the day, and count the distinct number of customers during each hour of the Wednesday and Saturday markets. See <a href="c06.xhtml" class="calibre6">Chapters 6</a>, “Aggregating Results for Analysis,” and <a href="c08.xhtml" class="calibre6">8</a>, “Date and Time Functions,” for information on the 
<code class="calibre11">COUNT DISTINCT</code>
 and 
<code class="calibre11">EXTRACT</code>
 functions.</li>
<li id="calibre_link-134" class="calibre12">What other questions haven't we yet asked about the data in these tables that you would be curious about? Write two more queries further exploring or summarizing the data in the 
<code class="calibre11">product</code>
, 
<code class="calibre11">vendor_inventory</code>
, or 
<code class="calibre11">customer_purchases</code>
 tables.</li>
</ol>
</section>
</section>
</div>


</body></html>