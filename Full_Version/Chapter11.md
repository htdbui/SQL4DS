---
title: "SQL Chapter 11"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">
<section aria-labelledby="c11_1" type="chapter" role="doc-chapter" class="calibre2">
<header class="calibre3">
<h1 id="calibre_link-17" class="calibre4"><span aria-label="159" type="pagebreak" id="calibre_link-18" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-19" class="calibre6"></a><span class="calibre">CHAPTER 11</span><br class="calibre7" /><span class="calibre">More Advanced Query Structures</span></h1>
</header>
<section aria-label="chapter opening" class="calibre2"><span id="calibre_link-20" class="calibre"></span>
<p id="calibre_link-21" class="calibre8">Most of this book is targeted at beginners, but because beginners can quickly become more advanced in developing SQL, I wanted to give you some ideas of what is possible when you think a little more creatively and go beyond the simplest 
<code class="calibre9">SELECT</code>
 statements. SQL is a powerful way to shape and summarize data into a wide variety forms that can be used for many types of analyses. This chapter includes a few examples of more complex query structures.</p>
</section>
<section aria-labelledby="head-2-78" class="calibre2"><span id="calibre_link-22" class="calibre"></span>
<h2 id="calibre_link-23" class="calibre10">UNIONs</h2>
<p class="calibre8">One query structure that I haven't yet covered in this book, but which certainly deserves a mention, is the 
<code class="calibre9">UNION</code>
 query. Using a 
<code class="calibre9">UNION</code>
, you can combine any two queries that result in the same number of columns with the same data types. The columns must be in the same order in both queries. There are many possible use cases for 
<code class="calibre9">UNION</code>
 queries, but the syntax is simple: write two queries with the same number and type of fields, and put a 
<code class="calibre9">UNION</code>
 keyword between them:</p>
<pre id="calibre_link-24" class="calibre11">
<code class="calibre9">SELECT market_year, MIN(market_date) AS first_market_date</code>
<code class="calibre9">FROM farmers_market.market_date_info</code>
<code class="calibre9">WHERE market_year = '2019'</code>
<code class="calibre9"> </code>
<code class="calibre9">UNION</code>
<code class="calibre9"> </code>
<code class="calibre9">SELECT market_year, MIN(market_date) AS first_market_date</code>
<code class="calibre9">FROM farmers_market.market_date_info</code>
<code class="calibre9">WHERE market_year = '2020'</code>
</pre>
<p id="calibre_link-25" class="calibre8">Of course, this isn't a sensible use case, because you could just write one query, 
<code class="calibre9">GROUP BY market_year</code>
, and filter to 
<code class="calibre9">WHERE market_year IN (‘2019’,’2020’)</code>
 and get the same output. There are always multiple ways to write queries, but sometimes combining two queries with identical columns selected but different criteria or different aggregation is the quickest way to get the results you want.</p>
<p class="calibre8">For a more complex example combining CTEs and 
<code class="calibre9">UNION</code>
s, we'll build a report that shows the products with the largest quantities available at each market: the bulk product with the largest weight available, and the unit product with the highest count available:</p>
<pre id="calibre_link-26" class="calibre11">
<code class="calibre9">WITH</code>
<code class="calibre9">product_quantity_by_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> vi.market_date, </code>
       
<code class="calibre9"> vi.product_id, </code>
       
<code class="calibre9"> p.product_name,</code>
       
<code class="calibre9"> SUM(vi.quantity) AS total_quantity_available,</code>
       
<code class="calibre9"> p.product_qty_type</code>
   
<code class="calibre9"> FROM farmers_market.vendor_inventory vi</code>
       
<code class="calibre9"> LEFT JOIN farmers_market.product p</code>
           
<code class="calibre9"> ON vi.product_id = p.product_id</code>
   
<code class="calibre9"> GROUP BY market_date, product_id</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * FROM</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT</code>
       
<code class="calibre9"> market_date,</code>
       
<code class="calibre9"> product_id, </code>
       
<code class="calibre9"> product_name,</code>
       
<code class="calibre9"> total_quantity_available, </code>
       
<code class="calibre9"> product_qty_type,</code>
       
<code class="calibre9"> RANK() OVER (PARTITION BY market_date ORDER BY total_quantity_available DESC) AS quantity_rank</code>
   
<code class="calibre9"> FROM product_quantity_by_date</code>
   
<code class="calibre9"> WHERE product_qty_type = 'unit'</code>
   
<code class="calibre9"> </code>
   
<code class="calibre9"> UNION</code>
   
<code class="calibre9"> </code>
   
<code class="calibre9"> SELECT</code>
       
<code class="calibre9"> market_date,</code>
       
<code class="calibre9"> product_id, </code>
       
<code class="calibre9"> product_name,</code>
       
<code class="calibre9"> total_quantity_available, </code>
       
<code class="calibre9"> product_qty_type,</code>
<span aria-label="160" type="pagebreak" id="calibre_link-27" role="doc-pagebreak" class="calibre12"></span>
       <span aria-label="161" type="pagebreak" id="calibre_link-28" role="doc-pagebreak" class="calibre12"></span>
<code class="calibre9"> RANK() OVER (PARTITION BY market_date ORDER BY total_quantity_available DESC) AS quantity_rank</code>
   
<code class="calibre9"> FROM product_quantity_by_date</code>
   
<code class="calibre9"> WHERE product_qty_type = 'lbs'</code>
<code class="calibre9">) x</code>
   
<code class="calibre9"> </code>
<code class="calibre9">WHERE x.quantity_rank = 1</code>
<code class="calibre9">ORDER BY market_date</code>
</pre>
<p id="calibre_link-29" class="calibre8">The 
<code class="calibre9">WITH</code>
 statement (CTE) at the top of this query totals up the quantity of each product that is available at each market from the 
<code class="calibre9">vendor_inventory</code>
 table, and joins in helpful information from the product table such as product name and the type of quantity, which has 
<code class="calibre9">product_qty_type</code>
 values of “lbs” or “unit.”</p>
<p id="calibre_link-30" class="calibre8">The inner part of the bottom query contains two different queries of the same view created in the CTE, 
<code class="calibre9">product_quantity_by_date</code>
, 
<code class="calibre9">UNION</code>
ed together. Each ranks the information available in the CTE by 
<code class="calibre9">total_quantity_available</code>
 (the sum of the quantity field, aggregated in the 
<code class="calibre9">WITH</code>
 clause), as well as returning all of the available fields. Note that both queries return the fields in the exact same order. The only difference between the two queries is their 
<code class="calibre9">WHERE</code>
 clauses, which separate the results by 
<code class="calibre9">product_qty_type</code>
. In this case, we can't simply 
<code class="calibre9">GROUP BY</code>
 the 
<code class="calibre9">product_qty_type</code>
 and remove the 
<code class="calibre9">UNION</code>
 as was possible for the initial example query in this section, because the 
<code class="calibre9">RANK()</code>
 window function is ranking by quantity available in each query, and we want to see the top item per 
<code class="calibre9">product_qty_type</code>
, so want to return the top ranked item from each set separately.</p>
<p id="calibre_link-31" class="calibre8">The outer part of the bottom query selects the results of the union, and filters to only the top-ranked quantities, so we get one row per market date with the highest number of lbs, and one row per market date with the highest number of units. Some results of this query are shown in <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 11.1</a>. You can see that for the month of August 2019, the bulk product with the highest weight each week was organic jalapeno peppers, and the product sold by unit with the highest count each week was sweet corn.<span aria-label="162" type="pagebreak" id="calibre_link-32" role="doc-pagebreak" class="calibre12"></span></p>
<figure class="calibre13">
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 11.1</a></span></p>
</figcaption>
</figure>
<p class="calibre8">For the sake of instruction, and because I frequently say that there are multiple ways to construct queries in SQL that result in identical outputs, there is at least one other way to get the preceding output that doesn't require a 
<code class="calibre9">UNION</code>
. In the following query, the second query in the 
<code class="calibre9">WITH</code>
 clause queries from the first query in the 
<code class="calibre9">WITH</code>
 clause, and the final 
<code class="calibre9">SELECT</code>
 statement simply filters the result of the second query:</p>
<pre id="calibre_link-33" class="calibre11">
<code class="calibre9">WITH</code>
<code class="calibre9">product_quantity_by_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> vi.market_date, </code>
       
<code class="calibre9"> vi.product_id, </code>
       
<code class="calibre9"> p.product_name,</code>
       
<code class="calibre9"> SUM(vi.quantity) AS total_quantity_available,</code>
       
<code class="calibre9"> p.product_qty_type</code>
   
<code class="calibre9"> FROM farmers_market.vendor_inventory vi</code>
       
<code class="calibre9"> LEFT JOIN farmers_market.product p</code>
          
<code class="calibre9"> ON vi.product_id = p.product_id</code>
   
<code class="calibre9"> GROUP BY market_date, product_id</code>
<code class="calibre9">),</code>
<code class="calibre9">rank_by_qty_type AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT</code>
       
<code class="calibre9"> market_date, </code>
       
<code class="calibre9"> product_id, </code>
       
<code class="calibre9"> product_name,</code>
       
<code class="calibre9"> total_quantity_available, </code>
       
<code class="calibre9"> product_qty_type,</code>
       
<code class="calibre9"> RANK() OVER (PARTITION BY market_date, product_qty_type ORDER BY total_quantity_available DESC) AS quantity_rank</code>
   
<code class="calibre9"> FROM product_quantity_by_date</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * FROM rank_by_qty_type</code>
<code class="calibre9">WHERE quantity_rank = 1</code>
<code class="calibre9">ORDER BY market_date</code>
</pre>
<p id="calibre_link-34" class="calibre8">We were able to accomplish the same result without the 
<code class="calibre9">UNION</code>
 by partitioning by both the 
<code class="calibre9">market_date</code>
 and 
<code class="calibre9">product_qty_type</code>
 in the 
<code class="calibre9">RANK()</code>
 function, resulting in a ranking for each date and quantity type.</p>
<p id="calibre_link-35" class="calibre8">Because I have shown two examples of 
<code class="calibre9">UNION</code>
 queries that don't actually require 
<code class="calibre9">UNION</code>
s, I wanted to mention one case when a 
<code class="calibre9">UNION</code>
 is definitely required: when you have separate tables with the same columns, representing different time periods. This could happen, for example, when you have event logs (such as website traffic logs) that are stored across multiple files, and each file is loaded into its own table in the database. Or, the tables could be static snapshots of the same dynamic dataset from different points in time. Or, maybe the data was <span aria-label="163" type="pagebreak" id="calibre_link-36" role="doc-pagebreak" class="calibre12"></span>migrated from one system into another, and you need to pull data from tables in two different systems and combine them together into one view to see the entire history of records.</p>
</section>
<section aria-labelledby="head-2-79" class="calibre2"><span id="calibre_link-37" class="calibre"></span>
<h2 id="calibre_link-38" class="calibre10">Self-Join to Determine To-Date Maximum</h2>
<p id="calibre_link-39" class="calibre8">A <i class="calibre15">self-join</i> in SQL is when a table is joined to itself (you can think of it like two copies of the table joined together) in order to compare rows to one another.</p>
<p class="calibre8">You write the SQL for a self-join just like any other join, but reference the same table name twice. To differentiate the two “copies” of the table, give each one its own alias:</p>
<ul class="none" id="calibre_link-40">
<li id="calibre_link-41" class="calibre16">
<code class="calibre9">SELECT</code>
 t1.id1, t1.field2, t2.field2, t2.field3</li>
<li id="calibre_link-42" class="calibre16">
<code class="calibre9">FROM</code>
 mytable AS t1</li>
<li id="calibre_link-43" class="calibre16">
<code class="calibre9">&nbsp;&nbsp;&nbsp;LEFT JOIN</code>
 mytable AS t2</li>
<li id="calibre_link-44" class="calibre16">
<code class="calibre9">&nbsp;&nbsp;&nbsp;&nbsp;ON</code>
 t1.id1 = t2.id1</li>
</ul>
<p id="calibre_link-45" class="calibre8">This particular example is meant to demonstrate the syntax and not highlight a typical use case, because you would not normally be joining on a primary key and comparing a row to itself. One more realistic use case is using a comparison operator other than an equal sign to accomplish something such as joining every row to every previous row, as shown in the queries associated with <a href="#calibre_link-3" id="calibre_link-7" class="calibre6">Figures 11.3</a> through <a href="#calibre_link-4" id="calibre_link-10" class="calibre6">11.5</a> below.</p>
<p id="calibre_link-46" class="calibre8">Let's say we wanted to show an aggregate metric changing over time, comparing each value to all previous values. One reason you might want to compare a value to all previous values is to create a “record high to-date” indicator.</p>
<p id="calibre_link-47" class="calibre8">One possible use case might be the need to automatically determine when the count of positive COVID-19 tests for a region on a particular day set a record as the highest count to-date. One way you could use this data point is to create a visual indicator on a COVID-19 tracking dashboard that appears when the count of new cases hits a new record high. Building this indicator into your dataset allows you to look back at the reported positive case counts at any past point in time and know if the number that day had set a new record high at the time.</p>
<p id="calibre_link-48" class="calibre8">We can demonstrate an example of that type of query using the Farmer's Market database by creating a report showing whether the total sales on each market date were the highest for any market to-date. If we were always looking at data filtered to dates before a selected date, we could simply use the 
<code class="calibre9">SUM()</code>
 and 
<code class="calibre9">MAX()</code>
 functions to determine the highest total sales for the given date range. But, if we're looking back at a log of sales on all dates, and want to do the calculation using data from dates prior to each past date without filtering out the later dates from view, we need a different approach.</p>
<p class="calibre8"><span aria-label="164" type="pagebreak" id="calibre_link-49" role="doc-pagebreak" class="calibre12"></span>In this case, we want to determine whether there is any previous date that has a higher sales total than the “current” row we're looking at, and we can use a self-join to do that comparison. First, we'll need to summarize the sales by 
<code class="calibre9">market_date</code>
, which we have done previously. We'll put this query into a CTE (
<code class="calibre9">WITH</code>
 clause), and alias it 
<code class="calibre9">sales_per_market_date</code>
. If we select the first 10 rows of this query, we get the output shown in <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 11.2</a>:</p>
<figure class="calibre13">
<img alt="A table records market date and sales." class="center" src="images/000000.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 11.2</a></span></p>
</figcaption>
</figure>
<pre id="calibre_link-50" class="calibre11">
<code class="calibre9">WITH </code>
<code class="calibre9">sales_per_market_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> market_date, </code>
       
<code class="calibre9"> ROUND(SUM(quantity * cost_to_customer_per_qty),2) AS sales</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre9"> GROUP BY market_date</code>
   
<code class="calibre9"> ORDER BY market_date</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * </code>
<code class="calibre9">FROM sales_per_market_date</code>
<code class="calibre9">LIMIT 10</code>
</pre>
<p class="calibre8">We can select data from this “table” twice. The trick here is to join the table to itself using the 
<code class="calibre9">market_date</code>
 field&mdash;but we won't be using an equal sign in the join. In this case, we want to join every row to all other rows that have a date that occurred prior to the “current” row's date, so we'll use a less-than sign (
<code class="calibre9">&lt;</code>
) in the join. I'll use an alias of 
<code class="calibre9">cm</code>
 to represent the “current market date” row (the left side of the join), and an alias of 
<code class="calibre9">pm</code>
 to represent a “previous market date” row.</p>
<aside class="calibre17">
<div class="top"><hr class="calibre18" /></div>
<section class="feature">
<h3 class="calibre19">NOTE</h3>
<p id="calibre_link-51" class="calibre20">It's easy to make an error when building this kind of join, so be sure to check your results carefully to ensure you created the intended output, especially if other tables are also joined in.</p>
<div class="top"><hr class="calibre18" /></div>
</section>
</aside>
<p id="calibre_link-52" class="calibre8"><span aria-label="165" type="pagebreak" id="calibre_link-53" role="doc-pagebreak" class="calibre12"></span></p>
<p class="calibre8">So we're joining every row to every other row in the database that has a lower 
<code class="calibre9">market_date</code>
 value than it does. We'll filter this to view the row from April 13, 2019, and you can see it is joined to the rows representing earlier market dates in the output in <a href="#calibre_link-3" class="calibre6">Figure 11.3</a>:</p>
<figure class="calibre13">
<img alt="A table records market date, and sales." class="center" src="images/000001.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-7" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 11.3</a></span></p>
</figcaption>
</figure>
<pre id="calibre_link-54" class="calibre11">
<code class="calibre9">WITH </code>
<code class="calibre9">sales_per_market_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> market_date, </code>
       
<code class="calibre9"> ROUND(SUM(quantity * cost_to_customer_per_qty),2) AS sales</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre9"> GROUP BY market_date</code>
   
<code class="calibre9"> ORDER BY market_date</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * </code>
<code class="calibre9">FROM sales_per_market_date AS cm</code>
   
<code class="calibre9"> LEFT JOIN sales_per_market_date AS pm</code>
       
<code class="calibre9"> ON pm.market_date &lt; cm.market_date</code>
<code class="calibre9">WHERE cm.market_date = '2019-04-13'</code>
</pre>
<p class="calibre8">Now we'll use a 
<code class="calibre9">MAX()</code>
 function on the 
<code class="calibre9">pm.sales</code>
 field and 
<code class="calibre9">GROUP BY cm.market_date</code>
 to get the previous highest sales value. The output of this version of the query is shown in <a href="#calibre_link-8" id="calibre_link-9" class="calibre6">Figure 11.4</a>. If you compare this to <a href="#calibre_link-3" class="calibre6">Figure 11.3</a>, you'll recognize the sales total from April 6, 2019, which had the highest sales of any date prior to April 13, 2019:</p>
<pre id="calibre_link-55" class="calibre11">
<code class="calibre9">WITH </code>
<code class="calibre9">sales_per_market_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> market_date, </code>
       
<code class="calibre9"> ROUND(SUM(quantity * cost_to_customer_per_qty),2) AS sales</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre9"> GROUP BY market_date</code>
   
<code class="calibre9"> ORDER BY market_date</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> cm.market_date,</code>
<span aria-label="166" type="pagebreak" id="calibre_link-56" role="doc-pagebreak" class="calibre12"></span>
<code class="calibre9"> cm.sales,</code>
   
<code class="calibre9"> MAX(pm.sales) AS previous_max_sales</code>
<code class="calibre9">FROM sales_per_market_date AS cm</code>
   
<code class="calibre9"> LEFT JOIN sales_per_market_date AS pm</code>
      
<code class="calibre9"> ON pm.market_date &lt; cm.market_date</code>
<code class="calibre9">WHERE cm.market_date = '2019-04-13'</code>
<code class="calibre9">GROUP BY cm.market_date, cm.sales</code>
</pre>
<figure class="calibre13">
<img alt="A table records market date, sales, and previous max sales." class="center" src="images/000002.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-9" id="calibre_link-8" role="doc-backlink" class="calibre6">Figure 11.4</a></span></p>
</figcaption>
</figure>
<p class="calibre8">We can now remove the date filter in the 
<code class="calibre9">WHERE</code>
 clause to get the 
<code class="calibre9">previous_max_sales</code>
 for each date. Additionally, we can use a 
<code class="calibre9">CASE</code>
 statement to create a flag field that indicates whether the current sales are higher than the previous maximum sales, indicating whether each row's 
<code class="calibre9">market_date</code>
 set a sales record as of that date. This query's output is displayed in <a href="#calibre_link-4" class="calibre6">Figure 11.5</a>. Note that now that we can see the row for April 6, 2019, we can see that it is labeled as being a record sales day at the time, and its sales record is the value we saw displayed as the 
<code class="calibre9">previous_max_sales</code>
 in <a href="#calibre_link-8" class="calibre6">Figure 11.4</a>. You can see that after a new record is set on May 29, 2019, the value in the 
<code class="calibre9">previous_max_sales</code>
 field updates to the new record value:</p>
<pre id="calibre_link-57" class="calibre11">
<code class="calibre9">WITH </code>
<code class="calibre9">sales_per_market_date AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> market_date, </code>
       
<code class="calibre9"> ROUND(SUM(quantity * cost_to_customer_per_qty),2) AS sales</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre9"> GROUP BY market_date</code>
   
<code class="calibre9"> ORDER BY market_date</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> cm.market_date,</code>
   
<code class="calibre9"> cm.sales,</code>
   
<code class="calibre9"> MAX(pm.sales) AS previous_max_sales,</code>
   
<code class="calibre9"> CASE WHEN cm.sales&gt; MAX(pm.sales)</code>
       
<code class="calibre9"> THEN "YES"</code>
       
<code class="calibre9"> ELSE "NO"</code>
   
<code class="calibre9"> END sales_record_set</code>
<code class="calibre9">FROM sales_per_market_date AS cm</code>
   
<code class="calibre9"> LEFT JOIN sales_per_market_date AS pm</code>
      
<code class="calibre9"> ON pm.market_date &lt; cm.market_date</code>
<code class="calibre9">GROUP BY cm.market_date, cm.sales</code>
<span aria-label="167" type="pagebreak" id="calibre_link-58" role="doc-pagebreak" class="calibre12"></span></pre>
<figure class="calibre13">
<img alt="A table records market date, sales, previous max sales, and sales record set." class="center" src="images/000003.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-4" role="doc-backlink" class="calibre6">Figure 11.5</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-80" class="calibre2"><span id="calibre_link-59" class="calibre"></span>
<h2 id="calibre_link-60" class="calibre10">Counting New vs. Returning Customers by Week</h2>
<p id="calibre_link-61" class="calibre8">Another common report has to do with summarizing customers by time period. The manager of the farmer's market might want to monitor how many customers are visiting the market per week, and how many of those are new, making a purchase for the first time.</p>
<p id="calibre_link-62" class="calibre8">Remember that in this case, a customer is only counted if they make a purchase. So, if the farmer's market manager asks what percentage of visitors to the market end up making a purchase, we can't give them that information using the data in this database. We can only report on customers who purchased items, and they are identified using their loyalty card with each purchase (we assume for the sake of simplicity that 100% of customers are identifiable at purchase).</p>
<p class="calibre8">We have built queries and used functions to summarize customers per week before, but how do we determine whether a customer is new? One way is to compare each purchase date to the minimum purchase date per customer. If a customer's minimum purchase date is today, then the customer made their first purchase today, and therefore is new. Let's get a summary of every market date attended by every customer, and determine their first purchase dates. This query finds the customer's first purchase date by using 
<code class="calibre9">MIN()</code>
 as a window function, partitioned by 
<code class="calibre9">customer_id</code>
:</p>
<pre id="calibre_link-63" class="calibre11">
<code class="calibre9">SELECT DISTINCT </code>
   
<code class="calibre9"> customer_id,</code>
<span aria-label="168" type="pagebreak" id="calibre_link-64" role="doc-pagebreak" class="calibre12"></span>
<code class="calibre9"> market_date,</code>
   
<code class="calibre9"> MIN(market_date) OVER(PARTITION BY cp.customer_id) AS first_purchase_date</code>
<code class="calibre9">FROM farmers_market.customer_purchases cp</code>
</pre>
<p id="calibre_link-65" class="calibre8">The 
<code class="calibre9">DISTINCT</code>
 has been added because there is a row in the 
<code class="calibre9">customer_purchases</code>
 table for each item purchased, and we only need one row per 
<code class="calibre9">market_date</code>
 per 
<code class="calibre9">customer_id</code>
. You might have noticed that we didn't need a 
<code class="calibre9">GROUP BY</code>
 here. Because the window function does its own partitioning, and the 
<code class="calibre9">DISTINCT</code>
 ensures we don't return duplicate rows, no further grouping is needed. A portion of the results from the preceding query is shown in <a href="#calibre_link-11" id="calibre_link-12" class="calibre6">Figure 11.6</a>. You can see that each row depicts a date that customer shopped at the farmer's market alongside the customer's first purchase date.</p>
<figure class="calibre13">
<img alt="A table records customer id, market date, and first purchase date." class="center" src="images/000004.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-12" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 11.6</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Now we can put that query inside a 
<code class="calibre9">WITH</code>
 clause and query its results with some calculations added. We also need to join it to the 
<code class="calibre9">market_date_info</code>
 table to get the year and week of each 
<code class="calibre9">market_date</code>
. We're going to group by week, so if a customer made purchases at both markets within that week, they will have two rows to be grouped into each year-week combination. So let's do two types of counts. The field we alias, 
<code class="calibre9">customer_visit_count</code>
, will count each of those rows, so a customer shopping at both markets in a week would be counted twice in that summary. We'll create a second calculation that counts unique 
<code class="calibre9">customer_id</code>
 values using 
<code class="calibre9">DISTINCT</code>
 and alias that 
<code class="calibre9">distinct_customer_count</code>
, which will only count unique customers per week without double-counting anyone. The results of the following counts are shown in <a href="#calibre_link-13" id="calibre_link-14" class="calibre6">Figure 11.7</a>:</p>
<pre id="calibre_link-66" class="calibre11">
<code class="calibre9">WITH</code>
<code class="calibre9">customer_markets_attended AS</code>
<code class="calibre9">(</code>
<span aria-label="169" type="pagebreak" id="calibre_link-67" role="doc-pagebreak" class="calibre12"></span>
<code class="calibre9"> SELECT DISTINCT</code>
   
<code class="calibre9"> customer_id,</code>
   
<code class="calibre9"> market_date,</code>
   
<code class="calibre9"> MIN(market_date) OVER(PARTITION BY cp.customer_id) AS first_purchase_date</code>
<code class="calibre9">FROM farmers_market.customer_purchases cp</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT</code>
   
<code class="calibre9"> md.market_year,</code>
   
<code class="calibre9"> md.market_week,</code>
   
<code class="calibre9"> COUNT(customer_id) AS customer_visit_count,</code>
   
<code class="calibre9"> COUNT(DISTINCT customer_id) AS distinct_customer_count</code>
<code class="calibre9">FROM customer_markets_attended AS cma</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.market_date_info AS md</code>
      
<code class="calibre9"> ON cma.market_date = md.market_date</code>
<code class="calibre9">GROUP BY md.market_year, md.market_week</code>
<code class="calibre9">ORDER BY md.market_year, md.market_week</code>
</pre>
<figure class="calibre13">
<img alt="A table records market year, week, customer visit count, and distinct customer count." class="center" src="images/000005.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-14" id="calibre_link-13" role="doc-backlink" class="calibre6">Figure 11.7</a></span></p>
</figcaption>
</figure>
<p class="calibre8">The results are something we could've achieved with a much simpler query, so what was the point of the query we turned into a CTE? The data is now in a form that facilitates performing further calculations, and we also have access to each customer's first purchase date, which we haven't made use of yet. We also want to get a count of new customers per week, so let's add a column displaying what percent of each week's customers are new. This requires adding two more fields to the query. The first looks like this:</p>
<pre id="calibre_link-68" class="calibre11">
<code class="calibre9">COUNT( </code>
   
<code class="calibre9"> DISTINCT </code>
   
<code class="calibre9"> CASE WHEN cma.market_date = cma.first_purchase_date </code>
       
<code class="calibre9"> THEN customer_id </code>
       
<code class="calibre9"> ELSE NULL </code>
   
<code class="calibre9"> END</code>
   
<code class="calibre9"> ) AS new_customer_count</code>
<span aria-label="170" type="pagebreak" id="calibre_link-69" role="doc-pagebreak" class="calibre12"></span></pre>
<p id="calibre_link-70" class="calibre8">Inside the 
<code class="calibre9">COUNT()</code>
 function is a 
<code class="calibre9">CASE</code>
 statement. Can you tell what it does? It is looking for rows in the results of the 
<code class="calibre9">customer_markets_attended</code>
 CTE where the 
<code class="calibre9">market_date</code>
 (the date when the customer made the purchase) is equal to the customer's first purchase date. If those values match, the 
<code class="calibre9">CASE</code>
 statement returns a 
<code class="calibre9">customer_id</code>
 to count. If not, the 
<code class="calibre9">CASE</code>
 statement returns NULL. So, the result is a distinct count of customers that made their first purchase that week.</p>
<p class="calibre8">The second field, which is the last listed in the following full query, then divides that same value by the total distinct count of customer IDs, giving us a percentage. The result of this query is shown in <a href="#calibre_link-15" id="calibre_link-16" class="calibre6">Figure 11.8</a>.</p>
<pre id="calibre_link-71" class="calibre11">
<code class="calibre9">WITH</code>
<code class="calibre9">customer_markets_attended AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT DISTINCT</code>
       
<code class="calibre9"> customer_id,</code>
       
<code class="calibre9"> market_date,</code>
       
<code class="calibre9"> MIN(market_date) OVER(PARTITION BY cp.customer_id) AS first_purchase_date</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases cp</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT</code>
   
<code class="calibre9"> md.market_year,</code>
   
<code class="calibre9"> md.market_week,</code>
   
<code class="calibre9"> COUNT(customer_id) AS customer_visit_count,</code>
   
<code class="calibre9"> COUNT(DISTINCT customer_id) AS distinct_customer_count,</code>
   
<code class="calibre9"> COUNT(DISTINCT </code>
       
<code class="calibre9"> CASE WHEN cma.market_date = cma.first_purchase_date </code>
          
<code class="calibre9"> THEN customer_id </code>
          
<code class="calibre9"> ELSE NULL </code>
       
<code class="calibre9"> END)AS new_customer_count,</code>
   
<code class="calibre9"> COUNT(DISTINCT </code>
          
<code class="calibre9"> CASE WHEN cma.market_date = cma.first_purchase_date </code>
              
<code class="calibre9"> THEN customer_id </code>
              
<code class="calibre9"> ELSE NULL </code>
          
<code class="calibre9"> END) </code>
   
<code class="calibre9"> / COUNT(DISTINCT customer_id)</code>
   
<code class="calibre9"> AS new_customer_percent</code>
<code class="calibre9">FROM customer_markets_attended AS cma</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.market_date_info AS md</code>
      
<code class="calibre9"> ON cma.market_date = md.market_date</code>
<code class="calibre9">GROUP BY md.market_year, md.market_week</code>
<code class="calibre9">ORDER BY md.market_year, md.market_week</code>
<span aria-label="171" type="pagebreak" id="calibre_link-72" role="doc-pagebreak" class="calibre12"></span></pre>
<figure class="calibre13">
<img alt="A table records market year, week, customer visit count, and distinct customer count." class="center" src="images/000006.png" />
<figcaption class="calibre14">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-16" id="calibre_link-15" role="doc-backlink" class="calibre6">Figure 11.8</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-73" class="calibre8">It makes sense that on the first market date, 100% of the customers are new. With the data that's currently entered into the Farmer's Market database as of the time of this writing, there are no new customers added after week 18 of 2019.</p>
</section>
<section aria-labelledby="head-2-81" class="calibre2"><span id="calibre_link-74" class="calibre"></span>
<h2 id="calibre_link-75" class="calibre10">Summary</h2>
<p id="calibre_link-76" class="calibre8">These are just a few examples of more complicated query structures that can be created with SQL. I hope by demonstrating these approaches that it gives you a sense of the wide variety of analyses and datasets you can create with different combinations of the SQL you have learned in this book. In the next chapter, we'll reuse some of these concepts to generate datasets designed specifically to be used as inputs into machine learning and forecasting algorithms.</p>
</section>
<section aria-labelledby="head-2-82" class="calibre2"><span id="calibre_link-77" class="calibre"></span>
<h2 id="calibre_link-78" class="calibre10">Exercises</h2>
<ol class="decimal" id="calibre_link-79">
<li id="calibre_link-80" class="calibre21">Starting with the query associated with <a href="#calibre_link-4" class="calibre6">Figure 11.5</a>, put the larger 
<code class="calibre9">SELECT</code>
 statement in a second CTE, and write a query that queries from its results to display the current record sales and associated market date. Can you think of another way to generate the same results?</li>
<li id="calibre_link-81" class="calibre21">Modify the “New vs. Returning Customers Per Week” report (associated with <a href="#calibre_link-15" class="calibre6">Figure 11.8</a>) to summarize the counts by vendor by week.</li>
<li id="calibre_link-82" class="calibre21">Using a 
<code class="calibre9">UNION</code>
, write a query that displays the market dates with the highest and lowest total sales.</li>
</ol>
</section>
</section>
</div>


</body></html>