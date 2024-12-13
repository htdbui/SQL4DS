---
title: "SQL Chapter 13"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">
<section aria-labelledby="c13_1" type="chapter" role="doc-chapter" class="calibre2">
<header class="calibre3">
<h1 id="calibre_link-45" class="calibre4"><span aria-label="191" type="pagebreak" id="calibre_link-46" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-47" class="calibre6"></a><span class="calibre">CHAPTER 13</span><br class="calibre7" /><span class="calibre">Analytical Dataset Development Examples</span></h1>
</header>
<section aria-label="chapter opening" class="calibre2"><span id="calibre_link-48" class="calibre"></span>
<p class="calibre8">In this chapter, I will walk through the development of datasets for answering different types of analytical questions. This involves combining multiple concepts from previous chapters into more complex queries and therefore is more advanced. Note that the example database doesn't currently contain enough data with correlations for actually doing the analyses that would follow the dataset development, so we won't be looking for trends in the output screenshots. The focus here is on how I would go about designing and building a dataset from our Farmer's Market database using SQL to answer each of the following analytical questions:</p>
<ul class="square" id="calibre_link-49">
<li id="calibre_link-50" class="calibre9">What factors correlate with fresh produce sales?</li>
<li id="calibre_link-51" class="calibre9">How do sales vary by customer zip code, market distance, and demographic data?</li>
<li id="calibre_link-52" class="calibre9">How does product price distribution affect market sales?</li>
</ul>
</section>
<section aria-labelledby="head-2-87" class="calibre2"><span id="calibre_link-53" class="calibre"></span>
<h2 id="calibre_link-54" class="calibre10">What Factors Correlate with Fresh Produce Sales?</h2>
<p id="calibre_link-55" class="calibre8">Let's say we're asked the analytical question “What factors are correlated with sales of fresh produce at the farmer's market?” So what we're being asked is to determine the relationships between a selection of different variables and a <span aria-label="192" type="pagebreak" id="calibre_link-56" role="doc-pagebreak" class="calibre11"></span>subset of market product sales. That means from a data perspective that we'll need to summarize different variables over periods of time and explore how sales during those same time periods change as each variable changes.</p>
<p id="calibre_link-57" class="calibre8">For example, “As the number of different available products at the market increases, do sales of fresh produce go up or down?” is a question exploring the relationship between two variables: product variety and sales. If sales go up when the product variety goes up, then the two variables are positively correlated. If sales go down when product variety goes up, then the two variables are negatively correlated.</p>
<p id="calibre_link-58" class="calibre8">I could choose to summarize each value per week and then create a scatterplot of the weekly pairs of numbers to visualize the relationship between them, for example. To do that for a variety of variables, I'll need to write a query that generates a dataset with one row per market week containing weekly summaries of each value to be explored.</p>
<p id="calibre_link-59" class="calibre8">I'll first need to determine what products are considered “fresh produce,” then calculate sales of those products per week, and pull in other variables (factors) summarized per week to explore in relation to those sales. Some ideas for values to compare to sales include: product availability (number of vendors carrying the products, volume of inventory available for purchase, or special high-demand product seasonal availability, to give a few examples), product cost, time of year/season, sales trends over time, and things that affect all sales at the market such as weather and the number of customers shopping at the market.</p>
<p class="calibre8">First, I will look at all of the different product categories to determine which make the most sense to use to answer this question. The result of the following query is shown in <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 13.1</a>:</p>
<pre id="calibre_link-60" class="calibre12">
<code class="calibre13">SELECT * FROM farmers_market.product_category</code>
</pre>
<figure class="calibre14">
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 13.1</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-61" class="calibre8">From the list in <a href="#calibre_link-1" class="calibre6">Figure 13.1</a>, I can see that product category 1 is “Fresh Fruits &amp; Vegetables,” which sounds like “fresh produce” to me. The “Plants &amp; Flowers” and “Eggs &amp; Meat” categories may contain products that the requester considers fresh produce, so I can generate a list of all products in categories 1, 5, and 6 and take this list to the requester to double-check that category 1 contains the types of products they would like me to analyze sales for. If they request analysis of a list of products instead of an entire category, I might mention that if this analysis <span aria-label="193" type="pagebreak" id="calibre_link-62" role="doc-pagebreak" class="calibre11"></span>is meant to be repeated over time, we will need to check for product additions and changes to assess the included product list every time we run the report (where if we simply filter to a category, any product added to that category in the future will be automatically included).</p>
<p class="calibre8">The output from this query is shown in <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 13.2</a>:</p>
<pre id="calibre_link-63" class="calibre12">
<code class="calibre13">SELECT * FROM farmers_market.product</code>
<code class="calibre13">WHERE product_category_id IN (1, 5, 6)</code>
<code class="calibre13">ORDER BY product_category_id</code>
</pre>
<figure class="calibre14">
<img alt="A table records product category id, name, size, and type." class="center" src="images/000000.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 13.2</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-64" class="calibre8">We'll assume that going with product category 1 was the correct guess. Now that I have the basic filter requirements, I can design the structure of my query. I know that I want some summary information about sales, which will need to come from the 
<code class="calibre13">customer_purchases</code>
 table. I'll need data about the availability, which will come from the 
<code class="calibre13">vendor_inventory</code>
 table. And I'll want some time-related information. Both the 
<code class="calibre13">customer_purchases</code>
 and 
<code class="calibre13">vendor_inventory</code>
 tables have market dates in them. Even though neither is directly related to the 
<code class="calibre13">market_date_info</code>
 table in the E-R diagram, I can join it to them by 
<code class="calibre13">market_date</code>
 to pull in additional information about that date.</p>
<p id="calibre_link-65" class="calibre8">Because this is a question about something related to sales over time, I'm going to start with the product sales part of the question, then join other information to the results of that query.</p>
<p class="calibre8">I need to select the details needed to summarize sales per week for products in the “Fresh Fruits &amp; Vegetables” category 1 by inner joining sales (
<code class="calibre13">customer_purchases</code>
) and products (
<code class="calibre13">product</code>
) by 
<code class="calibre13">product_id</code>
. If we were looking at products in multiple categories, it might also be worth joining in the 
<code class="calibre13">product_category</code>
 table to get the category names, but for now, I'll simplify and leave that out. Let's look at the details, shown in <a href="#calibre_link-5" id="calibre_link-7" class="calibre6">Figure 13.3</a>, first:</p>
<pre id="calibre_link-66" class="calibre12"><span aria-label="194" type="pagebreak" id="calibre_link-67" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13">SELECT * </code>
<code class="calibre13">FROM customer_purchases cp</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON cp.product_id = p.product_id</code>
<code class="calibre13">WHERE p.product_category_id = 1</code>
</pre>
<p id="calibre_link-68" class="calibre8">I used an 
<code class="calibre13">INNER JOIN</code>
 instead of a 
<code class="calibre13">LEFT JOIN</code>
, because for this sales calculation, I'm not interested in products that don't have purchases. At this stage, it's good to check the count of rows returned, to ensure it makes sense given your join selection. I also look at the details, to make sure I'm pulling the right data from the tables I meant to pull from, that I'm joining on the correct fields, and that my results are filtered the way I expect.</p>
<p id="calibre_link-69" class="calibre8">Since I'll be summarizing by week, I don't need the 
<code class="calibre13">transaction_time</code>
 field, and I don't currently have a need to know about the size of each product. We might eventually need the product quantity type and vendor information for when we start adding up how much product is available for purchase. However, do we need those fields from these tables? Total sales is the dependent variable&mdash;the value we're trying to correlate different values with. If we wanted to get a count of vendors who had the product for sale, we don't want to get that from the 
<code class="calibre13">customer_purchases</code>
 table, because there could be products available for sale that no one purchased, which means their existence wouldn't be recorded in the 
<code class="calibre13">customer_purchases</code>
 table. So, we'll want to get those pieces of data from the 
<code class="calibre13">vendor_inventory</code>
 table at a later step, and not from the 
<code class="calibre13">customer_purchases</code>
 table.</p>
<p class="calibre8">I can join in the 
<code class="calibre13">market_date_info</code>
 table to get the week number, to make summarization easier, as well as other date-related information such as the season and the weather. I decided to 
<code class="calibre13">RIGHT JOIN</code>
 it to the other tables, because I want to know whether there are market dates with no fresh produce sales at all, and the 
<code class="calibre13">RIGHT JOIN</code>
 will still pull in market dates with no corresponding records in the 
<code class="calibre13">customer_purchases</code>
 table, as shown in <a href="#calibre_link-6" id="calibre_link-8" class="calibre6">Figure 13.4</a>:</p>
<pre id="calibre_link-70" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> cp.market_date, </code>
   
<code class="calibre13"> cp.customer_id, </code>
   
<code class="calibre13"> cp.quantity, </code>
   
<code class="calibre13"> cp.cost_to_customer_per_qty, </code>
   
<code class="calibre13"> p.product_category_id,</code>
   
<code class="calibre13"> mdi.market_date, </code>
   
<code class="calibre13"> mdi.market_week, </code>
   
<code class="calibre13"> mdi.market_year, </code>
   
<code class="calibre13"> mdi.market_rain_flag, </code>
   
<code class="calibre13"> mdi.market_snow_flag</code>
<code class="calibre13">FROM customer_purchases cp</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON cp.product_id = p.product_id</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = cp.market_date</code>
<code class="calibre13">WHERE p.product_category_id = 1</code>
<span aria-label="195" type="pagebreak" id="calibre_link-71" role="doc-pagebreak" class="calibre11"></span><span aria-label="196" type="pagebreak" id="calibre_link-72" role="doc-pagebreak" class="calibre11"></span><span aria-label="197" type="pagebreak" id="calibre_link-73" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records product category id, name, size, and type." class="center" src="images/000001.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-7" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 13.3</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records product category id, name, size, and type." class="center" src="images/000002.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-8" id="calibre_link-6" role="doc-backlink" class="calibre6">Figure 13.4</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-74" class="calibre8">But look what happened. Even though I'm right joining the 
<code class="calibre13">market_date_info</code>
 table to the 
<code class="calibre13">customer_purchases</code>
 table, I'm not seeing any market dates that don't have sales in this category, even though I know they exist in the data! This is a common SQL design error. You might think that the solution is to rearrange all of the joins, but if that's the only change you make, you will still have the same issue. What's happening is that our 
<code class="calibre13">WHERE</code>
 clause is filtering the results to only rows with a customer purchase from product category 1. So if there are no sales on a market date, there are no product categories associated with that date, so we are filtering it out, defeating the purpose of the 
<code class="calibre13">RIGHT JOIN</code>
. (This is also a good reason to look at the data in each table before joining and write some quality control queries such as distinct counts of market dates, so you are aware if some expected values are missing after you join the tables together.)</p>
<p class="calibre8">The solution to this filter issue is to put the product category filter in the 
<code class="calibre13">JOIN ON</code>
 clause instead of in the 
<code class="calibre13">WHERE</code>
 clause, which is something we haven't covered previously. I can join to the 
<code class="calibre13">product</code>
 table on the 
<code class="calibre13">product_id</code>
 and 
<code class="calibre13">product_category_id</code>
 fields, and filter the 
<code class="calibre13">product_category_id</code>
 in the 
<code class="calibre13">ON</code>
 clause. This makes the filter only apply to the data from the product table (and now the 
<code class="calibre13">customer_purchases</code>
 table, since they're inner joined), and not to the results set, the way the 
<code class="calibre13">WHERE</code>
 clause does. So now all of our market dates will be returned. I moved the 
<code class="calibre13">market_date_info</code>
 fields to appear first, and modified the join to include the filter. You can now see that we're joining on the 
<code class="calibre13">product_id</code>
 and filtering the 
<code class="calibre13">product_category_id</code>
 in the 
<code class="calibre13">ON</code>
 section of the 
<code class="calibre13">JOIN</code>
. Note that now there is no 
<code class="calibre13">WHERE</code>
 clause, but we are still filtering the results of one of the tables being joined into the dataset! The output of this query is displayed in <a href="#calibre_link-9" id="calibre_link-10" class="calibre6">Figure 13.5</a>:</p>
<pre id="calibre_link-75" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_date, </code>
   
<code class="calibre13"> mdi.market_week, </code>
   
<code class="calibre13"> mdi.market_year, </code>
   
<code class="calibre13"> mdi.market_rain_flag, </code>
   
<code class="calibre13"> mdi.market_snow_flag,</code>
   
<code class="calibre13"> cp.market_date, </code>
   
<code class="calibre13"> cp.customer_id, </code>
   
<code class="calibre13"> cp.quantity, </code>
   
<code class="calibre13"> cp.cost_to_customer_per_qty, </code>
   
<code class="calibre13"> p.product_category_id</code>
<code class="calibre13">FROM customer_purchases cp</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON cp.product_id = p.product_id</code>
         
<code class="calibre13"> AND p.product_category_id = 1</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = cp.market_date</code>
<span aria-label="198" type="pagebreak" id="calibre_link-76" role="doc-pagebreak" class="calibre11"></span><span aria-label="199" type="pagebreak" id="calibre_link-77" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records product category id, name, size, and type." class="center" src="images/000003.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-9" role="doc-backlink" class="calibre6">Figure 13.5</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-78" class="calibre8">Now we can see rows for market dates with no fresh produce purchases.</p>
<p class="calibre8">I can summarize the customer purchases to one row per week to get sales per week by grouping on 
<code class="calibre13">market_year</code>
 and 
<code class="calibre13">market_week</code>
, and we don't need most of the other columns with additional details about the purchases, so we can remove them from our query. The sales calculation is the same one we used in <a href="c12.xhtml" class="calibre6">Chapter 12</a>, “Creating Machine Learning Datasets Using SQL,” with a slight addition. 
<code class="calibre13">COALESCE</code>
 is a function that returns the first non-NULL value in a list of values. In this case, when the query returns a market date with no sales in product category 1, the 
<code class="calibre13">weekly_category1_sales</code>
 value would be NULL. If we want it to be 0 instead, we can use the syntax</p>
<ul class="none" id="calibre_link-79">
<li id="calibre_link-80" class="calibre16">
<code class="calibre13">COALESCE(</code>
[value 1]
<code class="calibre13">, 0)</code>
</li>
</ul>
<p id="calibre_link-81" class="calibre8">which will return a 0 if “value 1” is NULL, and will otherwise return the calculated value. We wrap our 
<code class="calibre13">SUM</code>
 function in this 
<code class="calibre13">COALESCE</code>
 function, then 
<code class="calibre13">ROUND</code>
 the result of the 
<code class="calibre13">COALESCE</code>
 function to two digits after the decimal point. To summarize, in that final line before the 
<code class="calibre13">FROM</code>
 clause, we're adding up the sales, converting the result to 0 if there are no sales, then rounding the numeric result to two digits.</p>
<p class="calibre8">I'm also returning the 
<code class="calibre13">MAX</code>
 of the snow and rain flags, because if there was precipitation at either of the markets during the week, I want to return a 1 in this field. And I returned the minimum 
<code class="calibre13">market_season</code>
 value, so only one value is returned if a week happens to be split across two seasons. The updated result using the following query is displayed in <a href="#calibre_link-11" id="calibre_link-12" class="calibre6">Figure 13.6</a>:</p>
<pre id="calibre_link-82" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week,</code>
   
<code class="calibre13"> MAX(mdi.market_rain_flag) AS market_week_rain_flag, </code>
   
<code class="calibre13"> MAX(mdi.market_snow_flag) AS market_week_snow_flag,</code>
   
<code class="calibre13"> MIN(mdi.market_min_temp) AS minimum_temperature,</code>
   
<code class="calibre13"> MAX(mdi.market_max_temp) AS maximum_temperature,</code>
   
<code class="calibre13"> MIN(mdi.market_season) AS market_season,</code>
   
<code class="calibre13"> ROUND(COALESCE(SUM(cp.quantity * cp.cost_to_customer_per_qty), 0), 2) AS weekly_category1_sales</code>
<code class="calibre13">FROM customer_purchases cp</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON cp.product_id = p.product_id</code>
         
<code class="calibre13"> AND p.product_category_id = 1</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = cp.market_date</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week </code>
<span aria-label="200" type="pagebreak" id="calibre_link-83" role="doc-pagebreak" class="calibre11"></span><span aria-label="201" type="pagebreak" id="calibre_link-84" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records product category id, name, size, and type." class="center" src="images/000004.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-12" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 13.6</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-85" class="calibre8">So now we have total sales by week.</p>
<p id="calibre_link-86" class="calibre8">Some of the other aggregate values that could be added to this dataset include the number of vendors carrying products in the category, the volume of inventory available for purchase, and special high-demand product seasonal availability. These are values that come from the 
<code class="calibre13">vendor_inventory</code>
 table, because I want to know what the vendors brought to market, regardless of whether people purchased the items.</p>
<p class="calibre8">We can set up the query for 
<code class="calibre13">vendor_inventory</code>
 just like we did for 
<code class="calibre13">customer_purchases</code>
, joining it to the 
<code class="calibre13">product</code>
 and 
<code class="calibre13">market_date_info</code>
 tables the same way, and filtering to 
<code class="calibre13">product_category_id = 1</code>
 in the 
<code class="calibre13">JOIN</code>
 statement as we did previously. The results of this query are shown in <a href="#calibre_link-13" id="calibre_link-15" class="calibre6">Figure 13.7</a>:</p>
<pre id="calibre_link-87" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_date,</code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week,</code>
   
<code class="calibre13"> vi.*,</code>
   
<code class="calibre13"> p.*</code>
<code class="calibre13">FROM vendor_inventory vi</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
        
<code class="calibre13"> AND p.product_category_id = 1</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = vi.market_date</code>
</pre>
<p class="calibre8">Removing the fields we don't need for our weekly summary, we can narrow down the 
<code class="calibre13">vendor_inventory</code>
 table to keep the features we need for calculating the number of vendors (
<code class="calibre13">vendor_id</code>
), the number of products (
<code class="calibre13">product_id</code>
), and volume of products (
<code class="calibre13">quantity</code>
). We can also use the 
<code class="calibre13">product_id</code>
 to flag the existence of certain products. Let's say that we suspect that when the sweet corn vendors are at the market, some customers come that don't come at any other time of year, just to get the locally famous corn on the cob. We want to know if overall fresh produce sales go up during the weeks when corn is available, so we'll create a product availability flag for product 16, sweet corn, called 
<code class="calibre13">corn_available_flag</code>
, as shown in the following query and in <a href="#calibre_link-14" id="calibre_link-16" class="calibre6">Figure 13.8</a>:</p>
<pre id="calibre_link-88" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week,</code>
   
<code class="calibre13"> COUNT(DISTINCT vi.vendor_id) AS vendor_count,</code>
   
<code class="calibre13"> COUNT(DISTINCT vi.product_id) AS unique_product_count,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS unit_products_qty,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type = 'lbs' THEN vi.quantity ELSE 0 END) AS bulk_products_lbs,</code>
   
<code class="calibre13"> ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS total_product_value,</code>
<span aria-label="202" type="pagebreak" id="calibre_link-89" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13"> MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_flag</code>
<code class="calibre13">FROM vendor_inventory vi</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = vi.market_date</code>
   
<code class="calibre13"> GROUP BY </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week </code>
</pre>
<p class="calibre8">Now that I see these results, I realize that I would like to have a count of vendors selling and products available at the entire market, in addition to the product availability for product category 1. To avoid developing another query that will need to be joined in, I will remove the 
<code class="calibre13">product_category_id</code>
 filter and use 
<code class="calibre13">CASE</code>
 statements to create a set of fields that provides the same metrics, but only for products in the category. Then, the existing fields will turn into a count for all vendors and products at the market:</p>
<pre id="calibre_link-90" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week,</code>
   
<code class="calibre13"> COUNT(DISTINCT vi.vendor_id) AS vendor_count,</code>
   
<code class="calibre13"> COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.vendor_id ELSE NULL END) AS vendor_count_product_category1,</code>
   
<code class="calibre13"> COUNT(DISTINCT vi.product_id) AS unique_product_count,</code>
   
<code class="calibre13"> COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.product_id ELSE NULL END) AS unique_product_count_product_category1,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS unit_products_qty,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS unit_products_qty_product_category1,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type = 'lbs' THEN vi.quantity ELSE 0 END) AS bulk_products_lbs,</code>
   
<code class="calibre13"> SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type = 'lbs' THEN vi.quantity ELSE 0 END) AS bulk_products_lbs_product_category1,</code>
   
<code class="calibre13"> ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS total_product_value,</code>
   
<code class="calibre13"> ROUND(COALESCE(SUM(CASE WHEN p.product_category_id = 1 THEN vi.quantity * vi.original_price ELSE 0 END), 0), 2) AS total_product_value_product_category1, </code>
   
<code class="calibre13"> MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_flag</code>
<code class="calibre13">FROM vendor_inventory vi</code>
   
<code class="calibre13"> INNER JOIN product p</code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
      
<code class="calibre13"> ON mdi.market_date = vi.market_date</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> mdi.market_week</code>
<span aria-label="203" type="pagebreak" id="calibre_link-91" role="doc-pagebreak" class="calibre11"></span><span aria-label="204" type="pagebreak" id="calibre_link-92" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000005.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-15" id="calibre_link-13" role="doc-backlink" class="calibre6">Figure 13.7</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000006.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-16" id="calibre_link-14" role="doc-backlink" class="calibre6">Figure 13.8</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-93" class="calibre8"><a href="#calibre_link-17" id="calibre_link-20" class="calibre6">Figures 13.9</a>, <a href="#calibre_link-18" id="calibre_link-21" class="calibre6">13.10</a>, and <a href="#calibre_link-19" id="calibre_link-22" class="calibre6">13.11</a> display the columns resulting from this query, with each figure compressing the width of the previously displayed columns so examples of all columns in the output can be shown.</p>
<p id="calibre_link-94" class="calibre8">One easy quality check to do on the output here is to make sure that every field with the 
<code class="calibre13">_category1</code>
 suffix has an equal or lower value than the corresponding field without the suffix, since the totals include product category 1, so the category value should never come out to be higher than the overall total.</p>
<p class="calibre8">Now I can combine the results of these two queries. I'll alias each of them in the 
<code class="calibre13">WITH</code>
 clause (CTE), then join the views:</p>
<pre id="calibre_link-95" class="calibre12">
<code class="calibre13">WITH</code>
<code class="calibre13">my_customer_purchases AS</code>
<code class="calibre13">(</code>
   
<code class="calibre13"> SELECT </code>
       
<code class="calibre13"> mdi.market_year,</code>
       
<code class="calibre13"> mdi.market_week,</code>
       
<code class="calibre13"> MAX(mdi.market_rain_flag) AS market_week_rain_flag, </code>
       
<code class="calibre13"> MAX(mdi.market_snow_flag) AS market_week_snow_flag,</code>
       
<code class="calibre13"> MIN(mdi.market_min_temp) AS minimum_temperature,</code>
       
<code class="calibre13"> MAX(mdi.market_max_temp) AS maximum_temperature,</code>
       
<code class="calibre13"> MIN(mdi.market_season) AS market_season,</code>
       
<code class="calibre13"> ROUND(COALESCE(SUM(cp.quantity * cp.cost_to_customer_per_qty), 0), 2) AS weekly_category1_sales</code>
   
<code class="calibre13"> FROM customer_purchases cp</code>
       
<code class="calibre13"> INNER JOIN product p</code>
       <span aria-label="205" type="pagebreak" id="calibre_link-96" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13"> ON cp.product_id = p.product_id</code>
          
<code class="calibre13"> AND p.product_category_id = 1</code>
   
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
         
<code class="calibre13"> ON mdi.market_date = cp.market_date</code>
   
<code class="calibre13"> GROUP BY </code>
       
<code class="calibre13"> mdi.market_year,</code>
       
<code class="calibre13"> mdi.market_week </code>
<code class="calibre13">),</code>
<code class="calibre13">my_vendor_inventory AS</code>
<code class="calibre13">(</code>
   
<code class="calibre13"> SELECT </code>
       
<code class="calibre13"> mdi.market_year,</code>
       
<code class="calibre13"> mdi.market_week,</code>
       
<code class="calibre13"> COUNT(DISTINCT vi.vendor_id) AS vendor_count,</code>
       
<code class="calibre13"> COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.vendor_id ELSE NULL END) AS vendor_count_product_category1,</code>
       
<code class="calibre13"> COUNT(DISTINCT vi.product_id) unique_product_count,</code>
       
<code class="calibre13"> COUNT(DISTINCT CASE WHEN p.product_category_id = 1 THEN vi.product_id ELSE NULL END) AS unique_product_count_product_category1,</code>
       
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS unit_products_qty,</code>
       
<code class="calibre13"> SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type = 'unit' THEN vi.quantity ELSE 0 END) AS unit_products_qty_product_category1,</code>
       
<code class="calibre13"> SUM(CASE WHEN p.product_qty_type &lt;&gt; 'unit' THEN vi.quantity ELSE 0 END) AS bulk_products_qty,</code>
       
<code class="calibre13"> SUM(CASE WHEN p.product_category_id = 1 AND p.product_qty_type &lt;&gt; 'unit' THEN vi.quantity ELSE 0 END) AS bulk_products_qty_product_category1,</code>
       
<code class="calibre13"> ROUND(COALESCE(SUM(vi.quantity * vi.original_price), 0), 2) AS total_product_value,</code>
       
<code class="calibre13"> ROUND(COALESCE(SUM(CASE WHEN p.product_category_id = 1 THEN vi.quantity * vi.original_price ELSE 0 END), 0), 2) AS total_product_value_product_category1, </code>
       
<code class="calibre13"> MAX(CASE WHEN p.product_id = 16 THEN 1 ELSE 0 END) AS corn_available_flag</code>
       
<code class="calibre13"> FROM vendor_inventory vi</code>
           
<code class="calibre13"> INNER JOIN product p</code>
               
<code class="calibre13"> ON vi.product_id = p.product_id</code>
           
<code class="calibre13"> RIGHT JOIN market_date_info mdi</code>
               
<code class="calibre13"> ON mdi.market_date = vi.market_date</code>
   
<code class="calibre13"> GROUP BY </code>
        
<code class="calibre13"> mdi.market_year,</code>
        
<code class="calibre13"> mdi.market_week </code>
<code class="calibre13">)</code>
   
<code class="calibre13"> </code>
<code class="calibre13">SELECT * </code>
<code class="calibre13">FROM my_vendor_inventory</code>
   
<code class="calibre13"> LEFT JOIN my_customer_purchases</code>
       
<code class="calibre13"> ON my_vendor_inventory.market_year = my_customer_purchases.market_year</code>
           
<code class="calibre13"> AND my_vendor_inventory.market_week = my_customer_purchases.market_week</code>
<code class="calibre13">ORDER BY my_vendor_inventory.market_year, my_vendor_inventory.market_week</code>
<span aria-label="206" type="pagebreak" id="calibre_link-97" role="doc-pagebreak" class="calibre11"></span><span aria-label="207" type="pagebreak" id="calibre_link-98" role="doc-pagebreak" class="calibre11"></span><span aria-label="208" type="pagebreak" id="calibre_link-99" role="doc-pagebreak" class="calibre11"></span><span aria-label="209" type="pagebreak" id="calibre_link-100" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000007.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-20" id="calibre_link-17" role="doc-backlink" class="calibre6">Figure 13.9</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000008.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-21" id="calibre_link-18" role="doc-backlink" class="calibre6">Figure 13.10</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000009.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-22" id="calibre_link-19" role="doc-backlink" class="calibre6">Figure 13.11</a></span></p>
</figcaption>
</figure>
<p class="calibre8">I can alter the final 
<code class="calibre13">SELECT</code>
 statement to include the prior week's product category 1 sales, too, because the prior week's sales might be a good indicator of what to expect this week. I can use the 
<code class="calibre13">LAG</code>
 window function that was introduced in <a href="c07.xhtml" class="calibre6">Chapter 7</a>, ”Window Functions and Subqueries.” And I'll go ahead and list all of the column names to avoid showing the duplicate 
<code class="calibre13">market_year</code>
 and 
<code class="calibre13">market_week</code>
 columns that are in both CTEs which therefore show in the output twice when we use 
<code class="calibre13">*</code>
. (This query should be preceded by the same CTE/
<code class="calibre13">WITH</code>
 clause as the previous query, but removed here to save space.)</p>
<pre id="calibre_link-101" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mvi.market_year, </code>
   
<code class="calibre13"> mvi.market_week,</code>
   
<code class="calibre13"> mcp.market_week_rain_flag,</code>
   
<code class="calibre13"> mcp.market_week_snow_flag,</code>
   
<code class="calibre13"> mcp.minimum_temperature,</code>
   
<code class="calibre13"> mcp.maximum_temperature,</code>
   
<code class="calibre13"> mcp.market_season,</code>
   
<code class="calibre13"> mvi.vendor_count,</code>
   
<code class="calibre13"> mvi.vendor_count_product_category1,</code>
   
<code class="calibre13"> mvi.unique_product_count,</code>
   
<code class="calibre13"> mvi.unique_product_count_product_category1,</code>
   
<code class="calibre13"> mvi.unit_products_qty,</code>
   
<code class="calibre13"> mvi.unit_products_qty_product_category1,</code>
   
<code class="calibre13"> mvi.bulk_products_qty,</code>
   
<code class="calibre13"> mvi.bulk_products_qty_product_category1,</code>
   
<code class="calibre13"> mvi.total_product_value,</code>
   
<code class="calibre13"> mvi.total_product_value_product_category1,</code>
   
<code class="calibre13"> LAG(mcp.weekly_category1_sales, 1) OVER (ORDER BY mvi.market_year, mvi.market_week) AS previous_week_category1_sales,</code>
   
<code class="calibre13"> mcp.weekly_category1_sales</code>
<code class="calibre13">FROM my_vendor_inventory mvi</code>
   
<code class="calibre13"> LEFT JOIN my_customer_purchases mcp</code>
      
<code class="calibre13"> ON mvi.market_year = mcp.market_year</code>
         
<code class="calibre13"> AND mvi.market_week = mcp.market_week</code>
<code class="calibre13">ORDER BY mvi.market_year, mvi.market_week</code>
</pre>
<p id="calibre_link-102" class="calibre8">Now we have a detailed dataset summarized to one row per week that we could use to explore the relationships between the market weather, product availability, and fresh produce sales. Some of the columns in <a href="#calibre_link-23" id="calibre_link-24" class="calibre6">Figure 13.12</a> that were shown in previous figures have been condensed so the newly added 
<code class="calibre13">LAG</code>
 column is visible.</p>
<p id="calibre_link-103" class="calibre8">Before you read this book, a query like this might have looked large and indecipherable, but now you know that it's made by using various combinations of straightforward SQL statements you learned in other chapters, which are then combined into datasets with a progressively larger number of columns to use in analysis.</p>
<figure class="calibre14">
<img alt="A table records market date, year, week, date, quantity, and product details." class="center" src="images/000010.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-24" id="calibre_link-23" role="doc-backlink" class="calibre6">Figure 13.12</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-88" class="calibre2"><span id="calibre_link-104" class="calibre"></span>
<h2 id="calibre_link-105" class="calibre10">How Do Sales Vary by Customer Zip Code, Market Distance, and Demographic Data?</h2>
<p id="calibre_link-106" class="calibre8"><span aria-label="210" type="pagebreak" id="calibre_link-107" role="doc-pagebreak" class="calibre11"></span><span aria-label="211" type="pagebreak" id="calibre_link-108" role="doc-pagebreak" class="calibre11"></span>We have a couple of core questions here. How do sales vary by customer zip code and distance from the market? Can we integrate demographic data into this analysis? We will need to pull together several pieces of information to answer these questions. We could group all sales by zip code, but we might get more meaningful answers if we group sales by customer, then look at the summary statistics and distributions of per-customer sales totals by zip code.</p>
<p id="calibre_link-109" class="calibre8">We have the 5-digit zip (postal) code of each customer in the 
<code class="calibre13">customer</code>
 table, but we don't have their full address or 9-digit zip code, so the closest we can get to a “distance from the market” calculation is by calculating the distances between the market location and some location associated with each zip code, such as a centrally located latitude and longitude (keeping in mind that zip codes can be any kind of shape, so the “center” of the area isn't a great representation of the location of most residences in the postal code). One source of latitudes and longitudes by zip is <a class="code" href="https://public.opendatasoft.com/explore/dataset/us-zip-code-latitude-and-longitude/table/?q=22821">https://public.opendatasoft.com/explore/dataset/us-zip-code-latitude-and-longitude/table/?q=22821</a>. If we import this data into our database, we can join it to our queries to add a latitude and longitude per zip code to our dataset.</p>
<p id="calibre_link-110" class="calibre8">There is also plenty of demographic data online related to zip codes, such as census data summarized by ZCTAs (Zip Code Tabulation Areas), so we can pull in age distributions, wealth statistics, and other demographics summarized by zip.</p>
<p id="calibre_link-111" class="calibre8">For this example, I decided to summarize the sales per customer first, then join in the demographic data to every customer's record, even though it's not customer-specific. Then, if I were to train a model based on the behavior of customers, I could use the zip code data as an input into the customer-level model, or as a dimension to summarize the other per-customer fields by for reporting purposes.</p>
<p class="calibre8">So I'll first summarize sales per customer, including for how long they've been a customer, the count of market dates at which they've made a purchase, and the total each customer has spent to date. I'll also join the purchase summary to the customer table, in order to include each customer's zip code. The output of the following query is shown in <a href="#calibre_link-25" id="calibre_link-26" class="calibre6">Figure 13.13</a>:</p>
<pre id="calibre_link-112" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> c.customer_id,</code>
   
<code class="calibre13"> c.customer_zip,</code>
   
<code class="calibre13"> DATEDIFF(MAX(market_date), MIN(market_date)) customer_duration_days,</code>
   
<code class="calibre13"> COUNT(DISTINCT market_date) number_of_markets,</code>
   
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty), 2) total_spent,</code>
   <span aria-label="212" type="pagebreak" id="calibre_link-113" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT market_date), 2) average_spent_per_market</code>
<code class="calibre13">FROM farmers_market.customer c</code>
   
<code class="calibre13">LEFT JOIN farmers_market.customer_purchases cp</code>
      
<code class="calibre13"> ON cp.customer_id = c.customer_id</code>
<code class="calibre13">GROUP BY c.customer_id</code>
</pre>
<figure class="calibre14">
<img alt="A table records the calculation of the average spend per market." class="center" src="images/000011.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-26" id="calibre_link-25" role="doc-backlink" class="calibre6">Figure 13.13</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records the details of zip codes and the latitude and longitude points." class="center" src="images/000012.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-27" id="calibre_link-28" role="doc-backlink" class="calibre6">Figure 13.14</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-114" class="calibre8">Note that because of the nature of the sample data, the customers in this case have pretty similar values in this output, which isn't typical when you look at data collected from real-world scenarios.</p>
<p class="calibre8">I have loaded some example demographic data into a new table in the database I called 
<code class="calibre13">zip_data</code>
, which is shown in <a href="#calibre_link-28" id="calibre_link-27" class="calibre6">Figure 13.14</a>:</p>
<pre id="calibre_link-115" class="calibre12">
<code class="calibre13">SELECT * FROM zip_data</code>
</pre>
<p class="calibre8">I will join this table to my existing query by the zip code fields, so every customer in a zip code will have the same demographic data added to their record, as shown in <a href="#calibre_link-29" id="calibre_link-31" class="calibre6">Figure 13.15</a>. I condensed the columns that were already shown in <a href="#calibre_link-25" class="calibre6">Figure 13.13</a> in order to fit the newly added columns in view:</p>
<pre id="calibre_link-116" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> c.customer_id,</code>
   
<code class="calibre13"> DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days,</code>
   
<code class="calibre13"> COUNT(DISTINCT market_date) AS number_of_markets,</code>
   
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent,</code>
   
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT market_date), 2) AS average_spent_per_market,</code>
   
<code class="calibre13"> c.customer_zip,</code>
   
<code class="calibre13"> z.median_household_income AS zip_median_household_income,</code>
   
<code class="calibre13"> z.percent_high_income AS zip_percent_high_income,</code>
   
<code class="calibre13"> z.percent_under_18 AS zip_percent_under_18,</code>
   
<code class="calibre13"> z.percent_over_65 AS zip_percent_over_65,</code>
   
<code class="calibre13"> z.people_per_sq_mile AS zip_people_per_sq_mile,</code>
   
<code class="calibre13"> z.latitude,</code>
   
<code class="calibre13"> z.longitude</code>
<code class="calibre13">FROM farmers_market.customer c</code>
   
<code class="calibre13"> LEFT JOIN farmers_market.customer_purchases cp</code>
      
<code class="calibre13"> ON cp.customer_id = c.customer_id</code>
   
<code class="calibre13"> LEFT JOIN zip_data z</code>
      
<code class="calibre13"> ON c.customer_zip = z.zip_code_5</code>
<code class="calibre13">GROUP BY c.customer_id</code>
<span aria-label="213" type="pagebreak" id="calibre_link-117" role="doc-pagebreak" class="calibre11"></span><span aria-label="214" type="pagebreak" id="calibre_link-118" role="doc-pagebreak" class="calibre11"></span></pre>
<p class="calibre8">One part of the analysis question was about distance from the market, which the data we have doesn't exactly tell us. There is a calculation for distance between two latitudes and longitudes that I found on Dayne Batten's blog at <a class="code" href="https://daynebatten.com/2015/09/latitude-longitude-distance-sql/">https://daynebatten.com/2015/09/latitude-longitude-distance-sql/</a>. If the Farmer's Market is at the coordinates 38.4463, &ndash;78.8712, the calculation for distance between the latitude and longitude fields of a record in the dataset and the Farmer's Market location is:</p>
<pre id="calibre_link-119" class="calibre12">
<code class="calibre13">ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((latitude - 38.4463) / 2)),2) + COS(RADIANS(38.4463)) * COS(RADIANS(latitude)) * POWER((SIN(RADIANS((longitude - -78.8712) / 2))), 2))))</code>
</pre>
<p class="calibre8">Don't worry about understanding how that calculation works, just know that it returns the distance between two pairs of latitude and longitude rounded to the nearest mile. Replacing the latitude and longitude fields in our query with this calculation, we now have the query:</p>
<pre id="calibre_link-120" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> c.customer_id,</code>
   
<code class="calibre13"> DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days,</code>
   
<code class="calibre13"> COUNT(DISTINCT market_date) AS number_of_markets,</code>
   
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent,</code>
   
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT market_date), 2) AS average_spent_per_market,</code>
   
<code class="calibre13"> c.customer_zip,</code>
   
<code class="calibre13"> z.median_household_income AS zip_median_household_income,</code>
   
<code class="calibre13"> z.percent_high_income AS zip_percent_high_income,</code>
   
<code class="calibre13"> z.percent_under_18 AS zip_percent_under_18,</code>
   
<code class="calibre13"> z.percent_over_65 AS zip_percent_over_65,</code>
   
<code class="calibre13"> z.people_per_sq_mile AS zip_people_per_sq_mile,</code>
   
<code class="calibre13"> ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((z.latitude - 38.4463) / 2)),2) + COS(RADIANS(38.4463)) * COS(RADIANS(z.latitude)) * POWER((SIN(RADIANS((z.longitude - -78.8712) / 2))), 2)))) AS zip_miles_from_market</code>
<code class="calibre13">FROM farmers_market.customer AS c</code>
   
<code class="calibre13"> LEFT JOIN farmers_market.customer_purchases AS cp</code>
      
<code class="calibre13"> ON cp.customer_id = c.customer_id</code>
   
<code class="calibre13"> LEFT JOIN zip_data AS z</code>
      
<code class="calibre13"> ON c.customer_zip = z.zip_code_5</code>
<code class="calibre13">GROUP BY c.customer_id</code>
</pre>
<p id="calibre_link-121" class="calibre8">Again, I have condensed the fields shown in <a href="#calibre_link-25" class="calibre6">Figures 13.13</a> and <a href="#calibre_link-29" class="calibre6">13.15</a> in order to display the new fields fully in <a href="#calibre_link-30" id="calibre_link-32" class="calibre6">Figure 13.16</a>.</p>
<p id="calibre_link-122" class="calibre8">This will allow me to analyze customer metrics like total amount spent and number of markets attended by customer zip code or by calculated distance from the market. I could now build a scatterplot of 
<code class="calibre13">zip_miles_from_market</code>
 <span aria-label="215" type="pagebreak" id="calibre_link-123" role="doc-pagebreak" class="calibre11"></span><span aria-label="216" type="pagebreak" id="calibre_link-124" role="doc-pagebreak" class="calibre11"></span>versus 
<code class="calibre13">total_spent</code>
 (though all of the customer data points in each zip code would overlap on the distance axis, so I would have to add some jitter or size the dots in order to indicate how many people were represented by that point).</p>
<figure class="calibre14">
<img alt="A table records the details of zip codes and the latitude and longitude points." class="center" src="images/000013.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-31" id="calibre_link-29" role="doc-backlink" class="calibre6">Figure 13.15</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records the details of zip codes and the latitude and longitude points." class="center" src="images/000014.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-32" id="calibre_link-30" role="doc-backlink" class="calibre6">Figure 13.16</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-125" class="calibre8">With the newly added information per zip, I could create a rural versus urban flag based on the population density of the zip code and look at customer behavior based on that value. I could assess customer longevity by zip code, or look at the distributions of amount spent or customer durations by zip code. I could correlate the percent of high-income residents per zip code with the customers’ total amount spent at the market. There are a wide variety of analyses I could do with just this dataset.</p>
<p id="calibre_link-126" class="calibre8">Some other ideas of additional customer summary fields that could be added to this dataset if we also joined in details about the products include total purchases by product category, top vendor purchased from, number of different vendors purchased from, number of different products purchased, most frequent time of day of purchase, etc.</p>
<p class="calibre8">Here is one example usage of this dataset where I put the preceding query into a CTE and select from it to get the count of customers and the average total spent per customer for each zip code. Note that in this case, the 
<code class="calibre13">zip_miles_from_market</code>
 is the same per row, so I could take the min, max, or average and would get the same value returned because all of the values are identical per zip, which we're grouping by. I could also add 
<code class="calibre13">zip_miles_from_market</code>
 to the 
<code class="calibre13">GROUP BY</code>
 statement, or even change the structure of my query so I'm only joining in the zip code data at this point, when I'm summarizing the customers by zip, instead of joining the zip code data to the query inside the CTE. For the purposes of summary, either approach is fine. The output of the following approach is shown in <a href="#calibre_link-33" id="calibre_link-34" class="calibre6">Figure 13.17</a>:</p>
<pre id="calibre_link-127" class="calibre12">
<code class="calibre13">WITH customer_and_zip_data AS</code>
<code class="calibre13">(</code>
   
<code class="calibre13"> SELECT </code>
        
<code class="calibre13"> c.customer_id,</code>
        
<code class="calibre13"> DATEDIFF(MAX(market_date), MIN(market_date)) AS customer_duration_days,</code>
        
<code class="calibre13"> COUNT(DISTINCT market_date) AS number_of_markets,</code>
        
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent,</code>
        
<code class="calibre13"> ROUND(SUM(quantity * cost_to_customer_per_qty) / COUNT(DISTINCT market_date), 2) AS average_spent_per_market,</code>
        
<code class="calibre13"> c.customer_zip,</code>
        
<code class="calibre13"> z.median_household_income AS zip_median_household_income,</code>
        
<code class="calibre13"> z.percent_high_income AS zip_percent_high_income,</code>
        
<code class="calibre13"> z.percent_under_18 AS zip_percent_under_18,</code>
        
<code class="calibre13"> z.percent_over_65 AS zip_percent_over_65,</code>
        
<code class="calibre13"> z.people_per_sq_mile AS zip_people_per_sq_mile,</code>
        <span aria-label="217" type="pagebreak" id="calibre_link-128" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13"> ROUND(2 * 3961 * ASIN(SQRT(POWER(SIN(RADIANS((z.latitude - 38.4463) / 2)),2) + COS(RADIANS(38.4463)) * COS(RADIANS(z.latitude)) * POWER((SIN(RADIANS((z.longitude - -78.8712) / 2))), 2)))) AS zip_miles_from_market</code>
   
<code class="calibre13"> FROM farmers_market.customer AS c</code>
       
<code class="calibre13"> LEFT JOIN farmers_market.customer_purchases AS cp</code>
           
<code class="calibre13"> ON cp.customer_id = c.customer_id</code>
       
<code class="calibre13"> LEFT JOIN zip_data AS z</code>
           
<code class="calibre13"> ON c.customer_zip = z.zip_code_5</code>
   
<code class="calibre13"> GROUP BY c.customer_id</code>
<code class="calibre13">)</code>
   
<code class="calibre13"> </code>
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> cz.customer_zip,</code>
   
<code class="calibre13"> COUNT(cz.customer_id) AS customer_count,</code>
   
<code class="calibre13"> ROUND(AVG(cz.total_spent)) AS average_total_spent,</code>
   
<code class="calibre13"> MIN(cz.zip_miles_from_market) AS zip_miles_from_market</code>
<code class="calibre13">FROM customer_and_zip_data AS cz</code>
<code class="calibre13">GROUP BY cz.customer_zip</code>
</pre>
<figure class="calibre14">
<img alt="A table records customer zip, count, average total spent, and zip miles from market." class="center" src="images/000015.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-34" id="calibre_link-33" role="doc-backlink" class="calibre6">Figure 13.17</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-129" class="calibre8">One caveat to this analysis that we would want to inform the recipient about is that we only store the customer's current zip code in this database, so if a customer used to live in a different zip code, that past connection is lost, and all of their purchase history is now associated with the current zip code on their customer record. To maintain the changes in zip code over time, we would need to add another table to the database in which to store the customer zip code history.</p>
</section>
<section aria-labelledby="head-2-89" class="calibre2"><span id="calibre_link-130" class="calibre"></span>
<h2 id="calibre_link-131" class="calibre10">How Does Product Price Distribution Affect Market Sales?</h2>
<p id="calibre_link-132" class="calibre8">What if a manager of the market asks, “What does our distribution of product prices look like at the market? Are our low-priced items or high-priced items generating more sales for the market?” You might have guessed that in order to answer these questions, I'm going to be using window functions.</p>
<p id="calibre_link-133" class="calibre8"><span aria-label="218" type="pagebreak" id="calibre_link-134" role="doc-pagebreak" class="calibre11"></span>One clarifying question I would have for the requester in this case before trying to answer these questions is related to time, because I know that the distribution of prices can change over time, and the answer to the second question might have changed at some point as well. So, should we answer these questions only for the most recent market season? Compare year over year? Or just look at all sales for the entire history we have tracked, ignoring any possible changes over time?</p>
<p id="calibre_link-135" class="calibre8">Let's say that the requester replied to clarify that they want to look at product price distributions for each market season over time (because the types of products sold can be very different in the heat of summer versus at the winter holidays, for example, as well as changing over the years).</p>
<p id="calibre_link-136" class="calibre8">So first, I want to get the product pricing details prior to completing the level of summarization required to answer the questions.</p>
<p class="calibre8">The first step in this analysis is to get raw data on the price per product per market date. But that seemingly simple task then raises another question: What do we mean by “product”? Is each 
<code class="calibre13">product_id</code>
 in the database a product? Like “Carrots” sold by weight? Or do the products differ enough by vendor that I should consider each 
<code class="calibre13">product_id</code>
 sold by each vendor as a separate “product”? We were asked to look at the distribution of product prices over time, and different vendors do charge different amounts for the same products if we go by 
<code class="calibre13">product_id</code>
, so I will choose to look at the average price per product per vendor per season. I will start with the 
<code class="calibre13">original_price</code>
 per product specified by the vendor in the 
<code class="calibre13">vendor_inventory</code>
 table, and won't consider special discounts given to customers, which would appear in the 
<code class="calibre13">customer_purchases</code>
 table. This query's results are shown in <a href="#calibre_link-35" id="calibre_link-36" class="calibre6">Figure 13.18</a>:</p>
<pre id="calibre_link-137" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> vi.market_date, </code>
   
<code class="calibre13"> SUM(vi.quantity), </code>
   
<code class="calibre13"> AVG(vi.original_price)</code>
<code class="calibre13">FROM product AS p</code>
   
<code class="calibre13"> LEFT JOIN vendor_inventory AS vi </code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> vi.market_date</code>
<span aria-label="219" type="pagebreak" id="calibre_link-138" role="doc-pagebreak" class="calibre11"></span><span aria-label="220" type="pagebreak" id="calibre_link-139" role="doc-pagebreak" class="calibre11"></span></pre>
<figure class="calibre14">
<img alt="A table records the details of the products and the average value of them." class="center" src="images/000016.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-36" id="calibre_link-35" role="doc-backlink" class="calibre6">Figure 13.18</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Next, since it was determined that we're looking at prices per season over time, I need to pull in the 
<code class="calibre13">market_season</code>
 from the 
<code class="calibre13">market_date_info</code>
 table. I can also use the year so we're talking about seasons over time. And here's another challenge I can anticipate might arise later when I'm building the reports: The seasons are strings, so how do I know what order they should go in when sorting by season and year? I will keep the minimum month value in the dataset so it can later be used to sort the seasons in the correct order. See <a href="#calibre_link-37" id="calibre_link-38" class="calibre6">Figure 13.19</a> for the output of this query:</p>
<pre id="calibre_link-140" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> MIN(MONTH(vi.market_date)) AS month_market_season_sort, </code>
   
<code class="calibre13"> mdi.market_season, </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> SUM(vi.quantity) AS quantity_available, </code>
   
<code class="calibre13"> AVG(vi.original_price) AS avg_original_price</code>
<code class="calibre13">FROM product AS p</code>
   
<code class="calibre13"> LEFT JOIN vendor_inventory AS vi </code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
   
<code class="calibre13"> LEFT JOIN market_date_info AS mdi </code>
      
<code class="calibre13"> ON vi.market_date = mdi.market_date</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> mdi.market_year, </code>
   
<code class="calibre13"> mdi.market_season</code>
</pre>
<p id="calibre_link-141" class="calibre8">One thing you might have noticed is that my attempt to pull in a sortable value to order the market seasons has resulted in multiple 
<code class="calibre13">month_market_season_sort</code>
 values per season, because we are getting the minimum month per season per product, and some products aren't offered in the earliest month of each season. These differing values could have detrimental effects to our results later on, depending on how we use the sorting field, so we'll have to be careful how items group and sort with this value involved. We could also use a window function to get the minimum month per 
<code class="calibre13">market_season</code>
 across all rows for that season, ignoring the 
<code class="calibre13">product_id</code>
 values, which we'll switch to in the next version of the query.</p>
<figure class="calibre14">
<img alt="A table records the details of the products and the average value of them." class="center" src="images/000017.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-38" id="calibre_link-37" role="doc-backlink" class="calibre6">Figure 13.19</a></span></p>
</figcaption>
</figure>
<p class="calibre8"><span aria-label="221" type="pagebreak" id="calibre_link-142" role="doc-pagebreak" class="calibre11"></span><span aria-label="222" type="pagebreak" id="calibre_link-143" role="doc-pagebreak" class="calibre11"></span>At this point, I also realized that I do need to pull in the 
<code class="calibre13">customer_purchases</code>
 data, because the second question is about sales, and the 
<code class="calibre13">vendor_inventory</code>
 table only contains available items, not items sold or the total amount generated by sales of each item. So I'll join that table in and use it to calculate the aggregate sales data for the second question. Note that we need to join on all three matching keys: 
<code class="calibre13">product_id</code>
, 
<code class="calibre13">vendor_id</code>
, and 
<code class="calibre13">market_date</code>
, as shown here, and reflected in the results in <a href="#calibre_link-39" id="calibre_link-40" class="calibre6">Figure 13.20</a>:</p>
<pre id="calibre_link-144" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS month_market_season_sort, </code>
   
<code class="calibre13"> mdi.market_season, </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> AVG(vi.original_price) AS avg_original_price,</code>
   
<code class="calibre13"> SUM(cp.quantity) AS quantity_sold,</code>
   
<code class="calibre13"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales</code>
<code class="calibre13">FROM product AS p</code>
   
<code class="calibre13"> LEFT JOIN vendor_inventory AS vi </code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
   
<code class="calibre13"> LEFT JOIN market_date_info AS mdi </code>
      
<code class="calibre13"> ON vi.market_date = mdi.market_date</code>
   
<code class="calibre13"> LEFT JOIN customer_purchases AS cp </code>
      
<code class="calibre13"> ON vi.product_id = cp.product_id </code>
      
<code class="calibre13"> AND vi.vendor_id = cp.vendor_id </code>
      
<code class="calibre13"> AND vi.market_date = cp.market_date</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> p.product_id, </code>
   
<code class="calibre13"> p.product_name, </code>
   
<code class="calibre13"> p.product_category_id, </code>
   
<code class="calibre13"> p.product_qty_type,</code>
   
<code class="calibre13"> vi.vendor_id, </code>
   
<code class="calibre13"> mdi.market_year, </code>
   
<code class="calibre13"> mdi.market_season</code>
</pre>
<figure class="calibre14">
<img alt="A table records the details of the products and the total value of them." class="center" src="images/000018.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-40" id="calibre_link-39" role="doc-backlink" class="calibre6">Figure 13.20</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-145" class="calibre8">It would be good at this point to pick a few products and look through the detailed availability and purchase data, to quality check these summarized results before continuing.</p>
<p id="calibre_link-146" class="calibre8"><span aria-label="223" type="pagebreak" id="calibre_link-147" role="doc-pagebreak" class="calibre11"></span><span aria-label="224" type="pagebreak" id="calibre_link-148" role="doc-pagebreak" class="calibre11"></span>Now that I have a sale price of each item (the average price each vendor offered each product for each season), I could start exploring the distribution of prices. When developing these queries, I actually tried several different approaches at this point and landed on this one, partly because the small number of products in the database can expose an issue with using 
<code class="calibre13">NTILE</code>
s: two different items of the same price can end up in different 
<code class="calibre13">NTILE</code>
s if the number of 
<code class="calibre13">NTILE</code>
s you choose splits the set at that point. After attempting multiple 
<code class="calibre13">NTILE</code>
 number options, I realized that if I wanted to end up with high and low price points in order to answer the second question, I probably shouldn't be ranking the products anyway. I should be ranking the prices.</p>
<p id="calibre_link-149" class="calibre8">In that case, I decided to modify my approach to group the records by 
<code class="calibre13">market_year</code>
, 
<code class="calibre13">market_season</code>
, and 
<code class="calibre13">original_price</code>
. I used an 
<code class="calibre13">NTILE</code>
 value of 3, which then gives me groupings of the top, middle, and bottom 1/3 of prices. I can then summarize the sales of products that fall into each of these price points.</p>
<p class="calibre8">Here's the query that creates three price groupings per season:</p>
<pre id="calibre_link-150" class="calibre12">
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> mdi.market_season, </code>
   
<code class="calibre13"> mdi.market_year,</code>
   
<code class="calibre13"> MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS month_market_season_sort,</code>
   
<code class="calibre13"> vi.original_price, </code>
   
<code class="calibre13"> NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price) AS price:ntile,</code>
   
<code class="calibre13"> NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price DESC) AS price:ntile_desc,</code>
   
<code class="calibre13"> COUNT(DISTINCT CONCAT(vi.product_id, vi.vendor_id)) product_count,</code>
   
<code class="calibre13"> SUM(cp.quantity) AS quantity_sold,</code>
   
<code class="calibre13"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales</code>
<code class="calibre13">FROM product AS p</code>
   
<code class="calibre13"> LEFT JOIN vendor_inventory AS vi </code>
      
<code class="calibre13"> ON vi.product_id = p.product_id</code>
   
<code class="calibre13"> LEFT JOIN market_date_info AS mdi </code>
      
<code class="calibre13"> ON vi.market_date = mdi.market_date</code>
   
<code class="calibre13"> LEFT JOIN customer_purchases AS cp </code>
      
<code class="calibre13"> ON vi.product_id = cp.product_id </code>
      
<code class="calibre13"> AND vi.vendor_id = cp.vendor_id </code>
      
<code class="calibre13"> AND vi.market_date = cp.market_date</code>
<code class="calibre13">WHERE market_year IS NOT NULL</code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> mdi.market_year, </code>
   
<code class="calibre13"> mdi.market_season, </code>
   
<code class="calibre13"> vi.original_price</code>
</pre>
<p id="calibre_link-151" class="calibre8">The results of this query are shown in <a href="#calibre_link-41" id="calibre_link-42" class="calibre6">Figure 13.21</a>.<span aria-label="225" type="pagebreak" id="calibre_link-152" role="doc-pagebreak" class="calibre11"></span><span aria-label="226" type="pagebreak" id="calibre_link-153" role="doc-pagebreak" class="calibre11"></span></p>
<figure class="calibre14">
<img alt="A table records market season, year, season sort, and original price." class="center" src="images/000019.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-42" id="calibre_link-41" role="doc-backlink" class="calibre6">Figure 13.21</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-154" class="calibre8">Note that the 
<code class="calibre13">price:ntile</code>
 break points vary by season. In Summer 2019 and Summer 2020, a $4.00 item is in the second of the three 
<code class="calibre13">NTILE</code>
 groups, so it's in the middle price grouping. In Spring, the distribution of product prices changes, so a $4.00 item is in 
<code class="calibre13">price:ntile</code>
 1, or the low price grouping.</p>
<p id="calibre_link-155" class="calibre8">Also note that I created a column using the 
<code class="calibre13">NTILE</code>
 window function with the same number of groups as 
<code class="calibre13">price:ntile</code>
, but sorting the price descending. This way, if you are using the output and don't know how many groups there are, you can filter to 
<code class="calibre13">price:ntile = 1</code>
 to get the lowest price group, and 
<code class="calibre13">price:ntile_desc = 1</code>
 to get the highest price group.</p>
<p id="calibre_link-156" class="calibre8">Another thing I want to point out in the previous query is that I didn't 
<code class="calibre13">COUNT(DISTINCT product_id)</code>
 values, but first concatenated 
<code class="calibre13">product_id</code>
 and 
<code class="calibre13">vendor_id</code>
 and did a distinct count of the combined values. The reason is because of how we're defining “product,” where the same 
<code class="calibre13">product_id</code>
 sold by different vendors, possibly for different prices, is considered a different product for our purposes.</p>
<p id="calibre_link-157" class="calibre8">One caveat with these results is that we're summing up different types of quantities, so we're counting an ounce, pound, or unit product as “an item sold.” So our quantity isn't exactly apples-to-apples across seasons, but gives us a quick sales volume measure for rough comparison.</p>
<p id="calibre_link-158" class="calibre8">The output in <a href="#calibre_link-41" class="calibre6">Figure 13.21</a> also illustrates why I wanted to create a 
<code class="calibre13">month_market_season_sort</code>
, because alphabetically, the seasons sort out of order as “Late Fall,” “Spring,” and “Summer.” We will make use of the sort values in the next query.</p>
<p class="calibre8">Now we'll use the previous query as a CTE and summarize it:</p>
<pre id="calibre_link-159" class="calibre12">
<code class="calibre13">WITH product_prices AS</code>
<code class="calibre13">(</code>
   
<code class="calibre13"> SELECT </code>
       
<code class="calibre13"> mdi.market_season, </code>
       
<code class="calibre13"> mdi.market_year,</code>
       
<code class="calibre13"> MIN(MONTH(vi.market_date)) OVER (PARTITION BY market_season) AS month_market_season_sort,</code>
       
<code class="calibre13"> vi.original_price, </code>
       
<code class="calibre13"> NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price) AS price:ntile,</code>
       
<code class="calibre13"> NTILE(3) OVER (PARTITION BY market_year, market_season ORDER BY original_price DESC) AS price:ntile_desc,</code>
       
<code class="calibre13"> COUNT(DISTINCT CONCAT(vi.product_id, vi.vendor_id)) AS product_count,</code>
       
<code class="calibre13"> SUM(cp.quantity) AS quantity_sold,</code>
       
<code class="calibre13"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_sales</code>
       
<code class="calibre13"> FROM product AS p</code>
       
<code class="calibre13"> LEFT JOIN vendor_inventory AS vi </code>
          
<code class="calibre13"> ON vi.product_id = p.product_id</code>
       
<code class="calibre13"> LEFT JOIN market_date_info AS mdi </code>
          
<code class="calibre13"> ON vi.market_date = mdi.market_date</code>
       <span aria-label="227" type="pagebreak" id="calibre_link-160" role="doc-pagebreak" class="calibre11"></span>
<code class="calibre13"> LEFT JOIN customer_purchases AS cp </code>
          
<code class="calibre13"> ON vi.product_id = cp.product_id </code>
          
<code class="calibre13"> AND vi.vendor_id = cp.vendor_id </code>
          
<code class="calibre13"> AND vi.market_date = cp.market_date</code>
   
<code class="calibre13"> WHERE market_year IS NOT NULL</code>
   
<code class="calibre13"> GROUP BY </code>
       
<code class="calibre13"> mdi.market_year, </code>
       
<code class="calibre13"> mdi.market_season, </code>
       
<code class="calibre13"> vi.original_price</code>
<code class="calibre13">)</code>
   
<code class="calibre13"> </code>
<code class="calibre13">SELECT </code>
   
<code class="calibre13"> market_year, </code>
   
<code class="calibre13"> market_season, </code>
   
<code class="calibre13"> price:ntile, </code>
   
<code class="calibre13"> SUM(product_count) AS product_count,</code>
   
<code class="calibre13"> SUM(quantity_sold) AS quantity_sold,</code>
   
<code class="calibre13"> MIN(original_price) AS min_price,</code>
   
<code class="calibre13"> MAX(original_price) AS max_price,</code>
   
<code class="calibre13"> SUM(total_sales) AS total_sales</code>
<code class="calibre13">FROM product_prices </code>
<code class="calibre13">GROUP BY </code>
   
<code class="calibre13"> market_year, </code>
   
<code class="calibre13"> market_season, </code>
   
<code class="calibre13"> price:ntile</code>
<code class="calibre13">ORDER BY </code>
   
<code class="calibre13"> market_year, </code>
   
<code class="calibre13"> month_market_season_sort, </code>
   
<code class="calibre13"> price:ntile</code>
</pre>
<p id="calibre_link-161" class="calibre8">In <a href="#calibre_link-43" id="calibre_link-44" class="calibre6">Figure 13.22</a>, you'll see that we're now sorting the seasons in the correct order (even though we're not outputting the sort value), and we're displaying the minimum and maximum price per grouping, as well as the 
<code class="calibre13">total_sales</code>
. This gives us the data to answer the second question, and we can see that with this sample data, the 
<code class="calibre13">total_sales</code>
 is highest for 
<code class="calibre13">price:ntile</code>
 3 in every season, so the higher-priced items are generating the most sales for the market.<span aria-label="228" type="pagebreak" id="calibre_link-162" role="doc-pagebreak" class="calibre11"></span></p>
<figure class="calibre14">
<img alt="A table records market season, year, season sort, and original price." class="center" src="images/000020.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-44" id="calibre_link-43" role="doc-backlink" class="calibre6">Figure 13.22</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-163" class="calibre8">Hopefully these dataset creation walkthroughs have given you a sense of the variety of ways you can combine and summarize data to create a dataset to answer analytical questions, and the kinds of clarifying questions an experienced analyst might ask while going through this process. Keep in mind that each dataset can now be refreshed to pull in the latest data by rerunning the queries, and the results can be reused to answer many questions, not just the ones initially asked!</p>
</section>
</section>
</div>


</body></html>