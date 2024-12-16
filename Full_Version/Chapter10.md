---
title: "SQL Chapter 10"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">
<section aria-labelledby="c10_1" type="chapter" role="doc-chapter" class="calibre2">
<header class="calibre3">
<h1 id="calibre_link-19" class="calibre4"><span aria-label="143" type="pagebreak" id="calibre_link-20" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-21" class="calibre6"></a><span class="calibre">CHAPTER 10</span><br class="calibre7" /><span class="calibre">Building SQL Datasets for Analytical Reporting</span></h1>
</header>
<section aria-label="chapter opening" class="calibre2"><span id="calibre_link-22" class="calibre"></span>
<p id="calibre_link-23" class="calibre8">In previous chapters, we covered basic SQL 
<code class="calibre9">SELECT</code>
 syntax and started to use SQL to construct datasets to answer specific questions. In the data analysis world, being asked questions, exploring a database, writing SQL statements to find and pull the data needed to determine the answers, and conducting the analysis of that data to calculate the answers to the questions, is called <i class="calibre10">ad-hoc reporting</i>.</p>
<p id="calibre_link-24" class="calibre8">I often say in my data science conference presentations that the process depicted in <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 10.1</a> is what is expected of any data analyst or data scientist: to be able to listen to a question from a business stakeholder, determine how it might be answered using data from the database, retrieve the data needed to answer it, calculate the answers, and present that result in a form that the business stakeholder can understand and use to make decisions.</p>
<figure class="calibre11">
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 10.1</a></span></p>
</figcaption>
</figure>
</section>
<section class="calibre2"><span id="calibre_link-25" class="calibre"></span>
<p id="calibre_link-26" class="calibre8">You now know enough SQL to answer some basic ad-hoc questions about what is occurring at the fictional farmer’s market using the demonstration database by filtering, joining, and summarizing the data.</p>
<p id="calibre_link-27" class="calibre8">In the remaining chapters, we'll take those skills to the next level and demonstrate how to think through multiple analysis questions, simulating what it might be like to write queries to answer a question posed by a business stakeholder. We'll design and develop analytical datasets that can be used repeatedly to facilitate ad-hoc reporting, build dashboards, and serve as inputs into predictive models.</p>
</section>
<section aria-labelledby="head-2-74" class="calibre2"><span id="calibre_link-28" class="calibre"></span>
<h2 id="calibre_link-29" class="calibre13">Thinking Through Analytical Dataset Requirements</h2>
<p id="calibre_link-30" class="calibre8"><span aria-label="144" type="pagebreak" id="calibre_link-31" role="doc-pagebreak" class="calibre14"></span>In this chapter, we'll walk through some examples of designing reusable datasets that can be queried to build many report variations. An experienced analyst who goes through the steps in <a href="#calibre_link-1" class="calibre6">Figure 10.1</a> won't only think about writing a query to answer the immediate question at hand but will think more generally about “Building Datasets for Analysis” (we're finally getting to this book's subtitle!) and designing SQL queries that combine and summarize data in a way that can then be used to answer many similar questions that might arise as offshoots of the original question.</p>
<p id="calibre_link-32" class="calibre8">You can think of a dataset as being like a table, already summarized at the desired level of detail, with multiple measures and dimensions that you can break down those metrics by. An analytical dataset is designed for use in reports and predictive models, and usually combines data from several tables summarized at a granularity (row level of detail) that lends itself to multiple analyses. If the dataset is meant to be used in a visual report or dashboard, it's a good idea to join in all fields that could be used for human-readable report labels (like vendor names instead of just IDs), measures (the numeric values you'll want to summarize and report on, such as sales), and dimensions (for grouping and slicing and dicing the measures, like product category).</p>
<p id="calibre_link-33" class="calibre8">Because I know that the first question I'm asked to answer with an ad-hoc query is almost never the only question, I will use any remaining project time to try to anticipate follow-up questions and design a dataset that may be useful for answering them. Adding additional relevant columns or calculations to a query also makes the resulting dataset reusable for future reporting purposes.</p>
<p id="calibre_link-34" class="calibre8">Anticipating potential follow-up questions is a skill that analysts develop over time, through experience. For example, if the manager of the farmer’s market asked me “What were the total sales at the market last week?” I would expect to be asked more questions after delivering the answer to that one, such as, <span aria-label="145" type="pagebreak" id="calibre_link-35" role="doc-pagebreak" class="calibre14"></span>“How many sales were at the Wednesday market versus the Saturday market last week?” or “Can you calculate the total sales over another time period?” or “Let's track these weekly market sales over time,” or “Now that we have the total sales for last week, can we break this down by vendor?” Given time, I could build a single dataset that could be imported into a reporting system like Tableau and used to answer all of these questions.</p>
<p id="calibre_link-36" class="calibre8">Since we're talking about summary sales and time, I would first think about all of the different time periods by which someone might want to “slice and dice” market sales. Someone could ask to summarize sales by minute, hour, day, week, month, year, and so on. Then I would think about dimensions other than time that people might want to filter or summarize sales by, such as vendor or customer zip code.</p>
<p id="calibre_link-37" class="calibre8">Whatever granularity I choose to make the dataset (at what level of detail I choose to summarize it) dictates the lowest level of detail at which I can then filter or summarize a report based on that dataset. So, for example, if I build a dataset that summarizes the data by week, I will not be able to produce a daily sales report from that dataset, because weeks are less granular than days.</p>
<p id="calibre_link-38" class="calibre8">Conversely, the granularity I choose means I will always need to write summary queries for any question that's at a higher level of aggregation than the dataset. For example, if I create a dataset that summarizes sales per minute, I will always have to use 
<code class="calibre9">GROUP BY</code>
 in the query, or use a reporting tool to summarize sets of rows, to answer any question that needs those minutes combined into a longer time period like hours or days.</p>
<p id="calibre_link-39" class="calibre8">If you are developing a dataset for use in a reporting tool that makes aggregation simple, such as Tableau, you may want to keep it as granular as possible, so you can drill down to as small of a time period as is available in the data. In Tableau, measures are automatically summarized by default as you build a report, and you have to instead break down the summary measures by adding dimensions. Summarizing by date in Tableau is as simple as dragging any datetime value into a report and choosing at what timescale you want to view that date field, regardless of whether the underlying dataset has one row per day or one row per second.</p>
<p id="calibre_link-40" class="calibre8">However, if you are primarily going to be querying the dataset with SQL, you will have to use 
<code class="calibre9">GROUP BY</code>
 and structure your queries to summarize at the desired level every time you use it to build a report. So, if you anticipate that you will frequently be summarizing by day, it would make sense to build a dataset that is already summarized to one row per day. You can always go back to the source data and write a custom query for that rare case when you're expected to report on sales per hour, but reuse the pre-summarized daily sales dataset as a shortcut for the more common report requests in this case.</p>
<p id="calibre_link-41" class="calibre8">Let's say that for this example, I can safely assume that most reports will be summarized at the daily or weekly level, so I will choose to build a dataset that has one row per day. In each row, I can summarize not only the total daily <span aria-label="146" type="pagebreak" id="calibre_link-42" role="doc-pagebreak" class="calibre14"></span>sales but also include other summary metrics and attributes that sales might be reported by. Once I think it through, I realize that the reports I'm asked to provide are often broken down by vendor, so I will revise my design to be one row per day per vendor. This will allow us to filter any report built on this dataset to any date range and to a specific vendor, or set of vendors.</p>
<p id="calibre_link-43" class="calibre8">In the Farmer's Market database, the sales are tracked in the 
<code class="calibre9">customer_purchases</code>
 table, which has one row per item purchased (with multiples of the same item potentially included in a single row, since there is a quantity column). Each row in 
<code class="calibre9">customer_purchases</code>
 contains the ID of the product purchased, the ID of the vendor the product was purchased from, the ID of the customer making the purchase, the transaction date and time, and the quantity and cost per quantity. Because I am designing my dataset to have one row per date and vendor, I do not need to include detailed information about the customers or products. And because my most granular time dimension is 
<code class="calibre9">market_date</code>
, I do not need to consider the field that tracks the time of purchase.</p>
<p class="calibre8">I will start by writing a 
<code class="calibre9">SELECT</code>
 statement that pulls only the fields I need, leaving out unnecessary information, and allowing me to summarize at the selected level of detail. I don't need to include the quantity of an item purchased in this dataset, only the final sale amount, so I'll multiply the 
<code class="calibre9">quantity</code>
 and the 
<code class="calibre9">cost_to_customer_per_qty</code>
 fields, like we did in <a href="c03.xhtml" class="calibre6">Chapter 3</a>, “The WHERE Clause”:</p>
<pre id="calibre_link-44" class="calibre15">
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> market_date, </code>
   
<code class="calibre9"> vendor_id, </code>
   
<code class="calibre9"> quantity * cost_to_customer_per_qty</code>
<code class="calibre9">FROM farmers_market.customer_purchases</code>
</pre>
<p class="calibre8">After reviewing the output without aggregation, to ensure that these are the fields and values I expected to see, I'll group and sort by 
<code class="calibre9">vendor_id</code>
 and 
<code class="calibre9">market_date</code>
, 
<code class="calibre9">SUM</code>
 the calculated cost column, round it to two decimal places, and give it an alias of 
<code class="calibre9">sales</code>
. The resulting output is shown in <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 10.2</a>.</p>
<figure class="calibre11">
<img alt="A table records market date, vendor id, and sales." class="center" src="images/000000.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 10.2</a></span></p>
</figcaption>
</figure>
<pre id="calibre_link-45" class="calibre15"><span aria-label="147" type="pagebreak" id="calibre_link-46" role="doc-pagebreak" class="calibre14"></span>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> market_date, </code>
   
<code class="calibre9"> vendor_id, </code>
   
<code class="calibre9"> ROUND(SUM(quantity * cost_to_customer_per_qty),2) AS sales</code>
<code class="calibre9">FROM farmers_market.customer_purchases</code>
<code class="calibre9">GROUP BY market_date, vendor_id</code>
<code class="calibre9">ORDER BY market_date, vendor_id</code>
</pre>
<p class="calibre8">This is a good time to review whether my results will allow me to answer the other questions I assumed I might be asked in the future and to determine what other information might be valuable to add to the dataset for reporting purposes. Let's go through each one:</p>
<ul class="square" id="calibre_link-47">
<li id="calibre_link-48" class="calibre16"><b class="calibre17">What were the total sales at the market last week?</b> There are multiple ways to accomplish this. A simple option is to filter the results by last week's date range and sum the sales. If we wanted the report to update dynamically as data is added, always adding up sales “over the last week,” we could have our query subtract 7 days from the current date and filter to only sales in rows with a 
<code class="calibre9">market_date</code>
 value that occurred after that date. We have the fields necessary to accomplish this.</li>
<li id="calibre_link-49" class="calibre16"><b class="calibre17">How many of last week's sales were at the Wednesday market versus the Saturday market?</b> In addition to the approach to answering the previous question, we can use the 
<code class="calibre9">DAYNAME()</code>
 function in MySQL to return the name of the day of the week of each market date. It might be a good idea to add the day of the week to the dataset so it's easily accessible for report labeling and grouping. If we explore the 
<code class="calibre9">market_date_info</code>
 table, we'll find that there is a field called 
<code class="calibre9">market_day</code>
 that already includes the weekday label of each market date, so there is actually no need to calculate it; we can join it in.</li>
<li id="calibre_link-50" class="calibre16"><b class="calibre17">Can we calculate the total sales over another time period?</b> Yes, we can filter the results of the existing query to any date range and sum up the sales.</li>
<li id="calibre_link-51" class="calibre16"><b class="calibre17">Can we track the weekly market sales over time?</b> We can use the MySQL functions 
<code class="calibre9">WEEK()</code>
 and 
<code class="calibre9">YEAR()</code>
 on the 
<code class="calibre9">market_date</code>
 field to pull those values into the dataset and group rows by week number (which could be especially useful for year over year comparisons) or by year and week (for a weekly summary time series). However, these values are also already in the 
<code class="calibre9">market_date_info</code>
 table, so we can join those in to allow reporting by that information.</li>
<li id="calibre_link-52" class="calibre16"><span aria-label="148" type="pagebreak" id="calibre_link-53" role="doc-pagebreak" class="calibre14"></span><b class="calibre17">Can we break down the weekly sales by vendor?</b> Because we included 
<code class="calibre9">vendor_id</code>
 in the output, we can group or filter any of the preceding approaches by vendor. It might be a good idea to join in the vendor name and type in this dataset in case we end up grouping by vendor in a report, so we can label those rows with more than just the numeric vendor ID.</li>
</ul>
<p class="calibre8">While evaluating whether the dataset includes the data that we need to answer the expected questions, we identified some additional information that might be valuable to have on hand for reporting from other tables, including 
<code class="calibre9">market_day</code>
, 
<code class="calibre9">market_week</code>
, and 
<code class="calibre9">market_year</code>
 from the 
<code class="calibre9">market_date_info</code>
 table (which also contains fields indicating properties of that date such as whether it was raining, which might be a dimension someone wants to summarize by in the future), and 
<code class="calibre9">vendor_name</code>
 and 
<code class="calibre9">vendor_type</code>
 from the 
<code class="calibre9">vendor</code>
 table. Let's 
<code class="calibre9">LEFT JOIN</code>
 those into our dataset, so we keep all of the existing rows, and add in more columns where available. A sample of the dataset generated by this query is shown in <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 10.3</a>:</p>
<figure class="calibre11">
<img alt="A table records market date, day, week, year, vendor id, name, type, and sales." class="center" src="images/000001.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 10.3</a></span></p>
</figcaption>
</figure>
<pre id="calibre_link-54" class="calibre15">
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> cp.market_date,</code>
   
<code class="calibre9"> md.market_day,</code>
   
<code class="calibre9"> md.market_week,</code>
   
<code class="calibre9"> md.market_year,</code>
   
<code class="calibre9"> cp.vendor_id, </code>
   
<code class="calibre9"> v.vendor_name,</code>
   
<code class="calibre9"> v.vendor_type,</code>
   
<code class="calibre9"> ROUND(SUM(cp.quantity * cp.cost_to_customer_per_qty),2) AS sales</code>
<code class="calibre9">FROM farmers_market.customer_purchases AS cp</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.market_date_info AS md</code>
      
<code class="calibre9"> ON cp.market_date = md.market_date</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.vendor AS v</code>
      
<code class="calibre9"> ON cp.vendor_id = v.vendor_id</code>
<code class="calibre9">GROUP BY cp.market_date, cp.vendor_id</code>
<code class="calibre9">ORDER BY cp.market_date, cp.vendor_id</code>
</pre>
<p id="calibre_link-55" class="calibre8">Now we can use this custom dataset to create reports and conduct further analysis.</p>
</section>
<section aria-labelledby="head-2-75" class="calibre2"><span id="calibre_link-56" class="calibre"></span>
<h2 id="calibre_link-57" class="calibre13">Using Custom Analytical Datasets in SQL: CTEs and Views</h2>
<p id="calibre_link-58" class="calibre8"><span aria-label="149" type="pagebreak" id="calibre_link-59" role="doc-pagebreak" class="calibre14"></span>There are multiple ways to store queries (and the results of queries) for reuse in reports and other analyses. Some techniques, such as creating new database tables, will be covered in later chapters. Here, we will cover two approaches for more easily querying from the results of custom dataset queries you build: <i class="calibre10">Common Table Expressions</i> and <i class="calibre10">views</i>.</p>
<p id="calibre_link-60" class="calibre8">Most database systems, including MySQL since version 8.0, support Common Table Expressions (CTEs), also known as “
<code class="calibre9">WITH</code>
 clauses.” CTEs allow you to create an alias for an entire query, which allows you to reference it in other queries like you would any database table.</p>
<p class="calibre8">The syntax for CTEs is:</p>
<ul class="none" id="calibre_link-61">
<li id="calibre_link-62" class="calibre18">
<code class="calibre9">WITH</code>
 [query_alias] 
<code class="calibre9">AS</code>
</li>
<li id="calibre_link-63" class="calibre18">
<code class="calibre9">(</code>
</li>
<li id="calibre_link-64" class="calibre18">[query]</li>
<li id="calibre_link-65" class="calibre18">
<code class="calibre9">),</code>
</li>
<li id="calibre_link-66" class="calibre18">[query_2_alias] 
<code class="calibre9">AS</code>
</li>
<li id="calibre_link-67" class="calibre18">
<code class="calibre9">(</code>
</li>
<li id="calibre_link-68" class="calibre18">[query_2]</li>
<li id="calibre_link-69" class="calibre18">
<code class="calibre9">)</code>
</li>
<li id="calibre_link-70" class="calibre18">
<code class="calibre9">SELECT</code>
 [column list]</li>
<li id="calibre_link-71" class="calibre18">
<code class="calibre9">FROM</code>
 [query_alias]</li>
<li id="calibre_link-72" class="calibre18"><i class="calibre10">
<code class="calibre9"><i class="calibre10">…</i></code>
</i> [remainder of query that references aliases created above]</li>
</ul>
<p id="calibre_link-73" class="calibre8">where “[query_alias]” is a placeholder for the name you want to use to refer to a query later, and “[query]” is a placeholder for the query you want to reuse. If you want to alias multiple queries in the 
<code class="calibre9">WITH</code>
 clause, you put each query inside its own set of parentheses, separated by commas. You only use the 
<code class="calibre9">WITH</code>
 keyword once at the top, and enter “[alias_name] 
<code class="calibre9">AS</code>
” before each new query you want to later reference. (The 
<code class="calibre9">AS</code>
 is not optional in this case.) Each query in the 
<code class="calibre9">WITH</code>
 clause can reference any query that preceded it, by using its alias.</p>
<p id="calibre_link-74" class="calibre8">Then below the 
<code class="calibre9">WITH</code>
 clause, you start your 
<code class="calibre9">SELECT</code>
 statement like you normally would, and use the query aliases to refer to the results of each of them. They are run before the rest of your queries that rely on their results, the same way they would be if you put them inside the 
<code class="calibre9">SELECT</code>
 statement as subqueries, which were covered in <a href="c07.xhtml" class="calibre6">Chapter 7</a>, “Window Functions Frequently Used by Data Scientists.”</p>
<p class="calibre8">For example, if we wanted to reuse the previous query we wrote to generate the dataset of sales summarized by date and vendor for a report that summarizes <span aria-label="150" type="pagebreak" id="calibre_link-75" role="doc-pagebreak" class="calibre14"></span>sales by market week, we could put that query inside a 
<code class="calibre9">WITH</code>
 clause, then query from it using another 
<code class="calibre9">SELECT</code>
 statement, like so:</p>
<pre id="calibre_link-76" class="calibre15">
<code class="calibre9">WITH sales_by_day_vendor AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> cp.market_date,</code>
       
<code class="calibre9"> md.market_day,</code>
       
<code class="calibre9"> md.market_week,</code>
       
<code class="calibre9"> md.market_year,</code>
       
<code class="calibre9"> cp.vendor_id, </code>
       
<code class="calibre9"> v.vendor_name,</code>
       
<code class="calibre9"> v.vendor_type,</code>
       
<code class="calibre9"> ROUND(SUM(cp.quantity * cp.cost_to_customer_per_qty),2) AS sales </code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases AS cp</code>
       
<code class="calibre9"> LEFT JOIN farmers_market.market_date_info AS md</code>
           
<code class="calibre9"> ON cp.market_date = md.market_date</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.vendor AS v</code>
           
<code class="calibre9"> ON cp.vendor_id = v.vendor_id</code>
   
<code class="calibre9"> GROUP BY cp.market_date, cp.vendor_id</code>
   
<code class="calibre9"> ORDER BY cp.market_date, cp.vendor_id</code>
<code class="calibre9">)</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> s.market_year, </code>
   
<code class="calibre9"> s.market_week,</code>
   
<code class="calibre9"> SUM(s.sales) AS weekly_sales</code>
<code class="calibre9">FROM sales_by_day_vendor AS s</code>
<code class="calibre9">GROUP BY s.market_year, s.market_week</code>
</pre>
<p id="calibre_link-77" class="calibre8">A subset of results of this query is shown in <a href="#calibre_link-7" id="calibre_link-8" class="calibre6">Figure 10.4</a>.<span aria-label="151" type="pagebreak" id="calibre_link-78" role="doc-pagebreak" class="calibre14"></span></p>
<figure class="calibre11">
<img alt="A table records market year, week, and sales." class="center" src="images/000002.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-8" id="calibre_link-7" role="doc-backlink" class="calibre6">Figure 10.4</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-79" class="calibre8">Notice how the 
<code class="calibre9">SELECT</code>
 statement at the bottom references the 
<code class="calibre9">sales_by_day_vendor</code>
 Common Table Expression using its alias, treating it just like a table, and even giving it an even shorter alias, 
<code class="calibre9">s</code>
. You can filter it, perform calculations, and do anything with its fields that you would do with a normal database table. By using a 
<code class="calibre9">WITH</code>
 statement instead of a subquery, it keeps this query at the bottom cleaner and easier to understand.</p>
<p id="calibre_link-80" class="calibre8">In most SQL editors, you can highlight the query within each set of parentheses to run just that code inside the 
<code class="calibre9">WITH</code>
 statement and view its results, so you know what data is available as you develop your 
<code class="calibre9">SELECT</code>
 query below the 
<code class="calibre9">WITH</code>
 statement. You can't highlight only the 
<code class="calibre9">SELECT</code>
 statement at the bottom to run it alone, however, because it references 
<code class="calibre9">sales_by_day_vendor</code>
, which is dynamically created by running the 
<code class="calibre9">WITH</code>
 statement above it along with it.</p>
<p id="calibre_link-81" class="calibre8">Another approach allows you to develop 
<code class="calibre9">SELECT</code>
 statements that depend on a custom dataset in their own SQL editor window, or inside other code such as a Python script, without first including the entire CTE. This involves storing the query as a database <i class="calibre10">view</i>. A view is treated just like a table in SQL, the only difference being that it has run when it's referenced to dynamically generate a result set (where a table stores the data instead of storing the query), so queries that reference views can take longer to run than queries that reference tables. However, the view is retrieving the latest data from the underlying tables each time it is run, so you are working with the freshest data available when you query from a view.</p>
<p id="calibre_link-82" class="calibre8">If you want to store your dataset as a view (assuming you have been granted database permissions to create a view in a schema), you simply precede your 
<code class="calibre9">SELECT</code>
 statement with “
<code class="calibre9">CREATE VIEW</code>
 [schema_name].[view_name] 
<code class="calibre9">AS</code>
”, replacing the bracketed statements with the actual schema name, and the name you are giving the view.</p>
<p class="calibre8">This is one query in this book that you will not be able to test using an online SQL editor and the sample database, because you won't have permissions to create a new database object there. The 
<code class="calibre9">vw_</code>
 prefix in the following view name serves as an indicator when writing a query that the object you're referencing is a view (stored query) and not a table (stored data):</p>
<pre id="calibre_link-83" class="calibre15">
<code class="calibre9">CREATE VIEW farmers_market.vw_sales_by_day_vendor AS</code>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> cp.market_date,</code>
   
<code class="calibre9"> md.market_day,</code>
   
<code class="calibre9"> md.market_week,</code>
   
<code class="calibre9"> md.market_year,</code>
   
<code class="calibre9"> cp.vendor_id, </code>
   
<code class="calibre9"> v.vendor_name,</code>
   
<code class="calibre9"> v.vendor_type,</code>
   
<code class="calibre9"> ROUND(SUM(cp.quantity * cp.cost_to_customer_per_qty),2) AS sales</code>
<span aria-label="152" type="pagebreak" id="calibre_link-84" role="doc-pagebreak" class="calibre14"></span>
<code class="calibre9">FROM farmers_market.customer_purchases AS cp</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.market_date_info AS md</code>
       
<code class="calibre9"> ON cp.market_date = md.market_date</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.vendor AS v</code>
       
<code class="calibre9"> ON cp.vendor_id = v.vendor_id</code>
<code class="calibre9">GROUP BY cp.market_date, cp.vendor_id</code>
<code class="calibre9">ORDER BY cp.market_date, cp.vendor_id</code>
</pre>
<p id="calibre_link-85" class="calibre8">No results will be displayed when you run this query, other than a confirmation message indicating that the view was created. But if you were to select the data from this view, it would be identical to the data in <a href="#calibre_link-5" class="calibre6">Figure 10.3</a>.</p>
<p class="calibre8">Since this dataset we created has one row per market date per vendor, we can filter this view by 
<code class="calibre9">vendor_id</code>
 and a range of 
<code class="calibre9">market_date</code>
 values, just like we could if it were a table. Our stored view query is then run to retrieve and summarize data from the underlying tables, then those results are modified by the 
<code class="calibre9">SELECT</code>
 statement, as shown in <a href="#calibre_link-9" id="calibre_link-10" class="calibre6">Figure 10.5</a>.</p>
<pre id="calibre_link-86" class="calibre15">
<code class="calibre9">SELECT *</code>
<code class="calibre9">FROM farmers_market.vw_sales_by_day_vendor AS s</code>
<code class="calibre9">WHERE s.market_date BETWEEN '2020-04-01' AND '2020-04-30'</code>
   
<code class="calibre9"> AND s.vendor_id = 7</code>
<code class="calibre9">ORDER BY market_date</code>
</pre>
<figure class="calibre11">
<img alt="A table records market date, day, week, year, vendor id, name, type, and sales." class="center" src="images/000003.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-9" role="doc-backlink" class="calibre6">Figure 10.5</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-87" class="calibre8">Because the results of CTEs and views are not stored, they pull the data dynamically each time they are referenced. So, if you use the preceding SQL to report on weekly sales using the 
<code class="calibre9">vw_sales_by_day_vendor</code>
 view, each time you run the query, it will include the latest week for which data exists in the 
<code class="calibre9">customer_purchases</code>
 table, which the view code references.</p>
<p id="calibre_link-88" class="calibre8">We can also paste the SQL that generates this dataset into Business Intelligence software such as Tableau, effectively pulling our summary dataset into a visual drag-and-drop reporting and dashboarding interface.</p>
<p id="calibre_link-89" class="calibre8">The Tableau reports in <a href="#calibre_link-11" id="calibre_link-13" class="calibre6">Figures 10.6</a> and <a href="#calibre_link-12" id="calibre_link-14" class="calibre6">10.7</a> were built using the same dataset query we developed earlier, and are data visualizations that filter and summarize data using the same fields and values displayed in <a href="#calibre_link-7" class="calibre6">Figures 10.4</a> and <a href="#calibre_link-9" class="calibre6">10.5</a>.<span aria-label="153" type="pagebreak" id="calibre_link-90" role="doc-pagebreak" class="calibre14"></span></p>
<figure class="calibre11">
<img alt="Graphs titled, year over year sales by week." class="center" src="images/000004.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-13" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 10.6</a></span></p>
</figcaption>
</figure>
<figure class="calibre11">
<img alt="Graph titled, Marco&apos;s peppers sales by market date." class="center" src="images/000005.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-14" id="calibre_link-12" role="doc-backlink" class="calibre6">Figure 10.7</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-76" class="calibre2"><span id="calibre_link-91" class="calibre"></span>
<h2 id="calibre_link-92" class="calibre13">Taking SQL Reporting Further</h2>
<p id="calibre_link-93" class="calibre8">Now that you have seen one example of the development of a reusable analytical dataset for reporting, let's walk through another example, starting with the dataset we built at the end of <a href="c09.xhtml" class="calibre6">Chapter 9</a>, “Exploratory Data Analysis with SQL.” The last query in that chapter creates a dataset that has one row per 
<code class="calibre9">market_date</code>
, 
<code class="calibre9">vendor_id</code>
, and 
<code class="calibre9">product_id</code>
, and includes information about the vendor and product, including the total inventory brought to market and the total sales for that day. This is an example of an analytical dataset that can be reused for many report variations.</p>
<p class="calibre8">Some examples of questions that could be answered with that dataset include:</p>
<ul class="square" id="calibre_link-94">
<li id="calibre_link-95" class="calibre16">What quantity of each product did each vendor sell per market/week/month/year?</li>
<li id="calibre_link-96" class="calibre16">When are certain products in season (most available for sale)?</li>
<li id="calibre_link-97" class="calibre16"><span aria-label="154" type="pagebreak" id="calibre_link-98" role="doc-pagebreak" class="calibre14"></span>What percentage of each vendor's inventory is selling per time period?</li>
<li id="calibre_link-99" class="calibre16">Did the prices of any products change over time?</li>
<li id="calibre_link-100" class="calibre16">What are the total sales per vendor for the season?</li>
<li id="calibre_link-101" class="calibre16">How frequently do vendors discount their product prices?</li>
<li id="calibre_link-102" class="calibre16">Which vendor sold the most tomatoes last week?</li>
</ul>
<p id="calibre_link-103" class="calibre8">We can't answer questions about any time periods shorter than a day, because the timestamp of the sale isn't included. We also don't have any detailed information about customers, but because we have date, vendor, and product dimensions, we can slice and dice different metrics by those values.</p>
<p class="calibre8">We can add some calculated fields to the query for reporting purposes without pulling in any additional columns. The following query adds fields calculating the percentage of product quantity sold and the total discount and aliases them 
<code class="calibre9">percent_of_available_sold</code>
 and 
<code class="calibre9">discount_amount</code>
, respectively. Let's store this dataset as a view and use it to answer a business question:</p>
<pre id="calibre_link-104" class="calibre15">
<code class="calibre9">CREATE VIEW farmers_market.vw_sales_per_date_vendor_product AS</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> vi.market_date, </code>
   
<code class="calibre9"> vi.vendor_id, </code>
   
<code class="calibre9"> v.vendor_name, </code>
   
<code class="calibre9"> vi.product_id, </code>
   
<code class="calibre9"> p.product_name,</code>
   
<code class="calibre9"> vi.quantity AS quantity_available,</code>
   
<code class="calibre9"> sales.quantity_sold,</code>
   
<code class="calibre9"> ROUND((sales.quantity_sold / vi.quantity) * 100, 2) AS percent_of_available_sold,</code>
   
<code class="calibre9"> vi.original_price, </code>
   
<code class="calibre9"> (vi.original_price * sales.quantity_sold) - sales.total_sales AS discount_amount,</code>
   
<code class="calibre9"> sales.total_sales</code>
<code class="calibre9">FROM farmers_market.vendor_inventory AS vi</code>
   
<code class="calibre9"> LEFT JOIN</code>
   
<code class="calibre9"> (</code>
   
<code class="calibre9"> SELECT market_date, </code>
       
<code class="calibre9"> vendor_id, </code>
       
<code class="calibre9"> product_id, </code>
       
<code class="calibre9"> SUM(quantity) quantity_sold, </code>
       
<code class="calibre9"> SUM(quantity * cost_to_customer_per_qty) AS total_sales</code>
   
<code class="calibre9"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre9"> GROUP BY market_date, vendor_id, product_id</code>
   
<code class="calibre9"> ) AS sales</code>
      
<code class="calibre9"> ON vi.market_date = sales.market_date </code>
          
<code class="calibre9"> AND vi.vendor_id = sales.vendor_id</code>
          
<code class="calibre9"> AND vi.product_id = sales.product_id</code>
   
<code class="calibre9"> LEFT JOIN farmers_market.vendor v </code>
          
<code class="calibre9"> ON vi.vendor_id = v.vendor_id</code>
<span aria-label="155" type="pagebreak" id="calibre_link-105" role="doc-pagebreak" class="calibre14"></span>
<code class="calibre9"> LEFT JOIN farmers_market.product p</code>
          
<code class="calibre9"> ON vi.product_id = p.product_id</code>
<code class="calibre9">ORDER BY vi.vendor_id, vi.product_id, vi.market_date</code>
</pre>
<p class="calibre8">To clarify what's happening in the calculation for 
<code class="calibre9">discount_amount</code>
:</p>
<ul class="none" id="calibre_link-106">
<li id="calibre_link-107" class="calibre18">
<code class="calibre9">(vi.original_price * sales.quantity_sold) - sales.total_sales</code>
</li>
</ul>
<p id="calibre_link-108" class="calibre8">we're taking the 
<code class="calibre9">original_price</code>
 of a product and multiplying it by the quantity of that product that was sold, to get the total value of the products sold, and subtracting from it the actual sales for the day, so we get a total for how much potential profit the vendor gave away in the form of discounts for that product on that day.</p>
<p id="calibre_link-109" class="calibre8">If a vendor asked, “What percent of my sales at each market came from each product I brought?” you could use this dataset to build a report to answer the question, because you have a summary of sales per product per vendor per market date. You will need to use a window function to get the answer, because the total you are dividing is the sum of all of a vendor's sales per date, which means adding up the 
<code class="calibre9">total_sales</code>
 values across multiple rows of the dataset. Once you have the 
<code class="calibre9">total_sales</code>
 for a vendor on a market date, then you can divide each row's total sales (remember there is one row per product per vendor per market date) into the vendor's total sales of all products for the day.</p>
<p id="calibre_link-110" class="calibre8">The query needed to generate this report is pictured in <a href="#calibre_link-15" id="calibre_link-16" class="calibre6">Figure 10.8</a>, because the window functions make the calculations quite long, and the syntax highlighting provided by the IDE helps make the various sections of the SQL statement clearer.</p>
<figure class="calibre11">
<img alt="Snapshot shows the generation of a report." class="center" src="images/000006.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-16" id="calibre_link-15" role="doc-backlink" class="calibre6">Figure 10.8</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-111" class="calibre8">Let's walk through these calculations step by step. First, note that we're querying from the view created by the previous query, 
<code class="calibre9">vw_sales_per_date_vendor_product</code>
. We give the total sales, which is summarized per market date, vendor, and product in the view, an alias of 
<code class="calibre9">vendor_product_sales_on_market_date</code>
, and round it to have two digits after the decimal point, since this is a report and we want to format everything nicely.</p>
<p id="calibre_link-112" class="calibre8"><span aria-label="156" type="pagebreak" id="calibre_link-113" role="doc-pagebreak" class="calibre14"></span>In the next line of the SQL statement in <a href="#calibre_link-15" class="calibre6">Figure 10.8</a>, we are summing up each vendor's sales (of all of their products) on each market date, using a window function that partitions sales by 
<code class="calibre9">market_date</code>
 and 
<code class="calibre9">vendor_id</code>
. (See <a href="c07.xhtml" class="calibre6">Chapter 7</a> for more information about window functions.) Then we give that sum an alias of 
<code class="calibre9">vendor_total_sales_on_market_date</code>
 and round it to two decimal places.</p>
<p id="calibre_link-114" class="calibre8">We now have the total sales for the vendor for the day on each row, and we already had the total sales of each product the vendor sold that day. The calculation in the next line is that first dollar amount divided by the second dollar amount, which calculates the percentage of the vendor's sales on that market date represented by each product.</p>
<p id="calibre_link-115" class="calibre8">In the pictured rows of output at the bottom of <a href="#calibre_link-15" class="calibre6">Figure 10.8</a>, you can see that Marco's Peppers is only selling one product on 4/22/2020, so the sales on that row represent 100% of Marco's sales for the day. Annie's Pies is selling three different products, and you can see in the final column what portion of Annie's total sales was contributed by each product.</p>
<p class="calibre8">We can write additional queries against this reusable dataset to build other reports in SQL, too. To use SQL to get the same data summary that is shown in <a href="c09.xhtml#c09-fig-0018" class="calibre6">Figure 9.18</a>, which was grouped and visualized in Tableau, we can query the view as follows:</p>
<pre id="calibre_link-116" class="calibre15">
<code class="calibre9">SELECT </code>
   
<code class="calibre9"> market_date, </code>
   
<code class="calibre9"> vendor_name, </code>
   
<code class="calibre9"> product_name, </code>
   
<code class="calibre9"> quantity_available, </code>
   
<code class="calibre9"> quantity_sold</code>
<code class="calibre9">FROM farmers_market.vw_sales_per_date_vendor_product AS s</code>
<code class="calibre9">WHERE market_date BETWEEN '2020-06-01' AND '2020-07-31'</code>
   
<code class="calibre9"> AND vendor_name = 'Marco''s Peppers'</code>
   
<code class="calibre9"> AND product_id IN (2, 4)</code>
<code class="calibre9">ORDER BY market_date, product_id</code>
</pre>
<p id="calibre_link-117" class="calibre8">A partial view of the output of this query is in <a href="#calibre_link-17" id="calibre_link-18" class="calibre6">Figure 10.9</a>, and you can compare the numbers to those in the bar chart in <a href="c09.xhtml#c09-fig-0018" class="calibre6">Figure 9.18</a>. One benefit of saving queries that generate summary datasets so they are available to reuse as needed, is that any tool you use to pull the data will be referencing the underlying data, table joins, and calculated values. As long as the data isn't changing between the report generation times, everyone using the defined dataset can get the same results.<span aria-label="157" type="pagebreak" id="calibre_link-118" role="doc-pagebreak" class="calibre14"></span></p>
<figure class="calibre11">
<img alt="A table records market date, vendor name, product name, quantity available, and sold." class="center" src="images/000007.png" />
<figcaption class="calibre12">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-18" id="calibre_link-17" role="doc-backlink" class="calibre6">Figure 10.9</a></span></p>
</figcaption>
</figure>
</section>
<section aria-labelledby="head-2-77" class="calibre2"><span id="calibre_link-119" class="calibre"></span>
<h2 id="calibre_link-120" class="calibre13">Exercises</h2>
<ol class="decimal" id="calibre_link-121">
<li id="calibre_link-122" class="calibre16">Using the view created in this chapter called 
<code class="calibre9">farmers_market.vw_sales_by_day_vendor</code>
, referring to <a href="#calibre_link-5" class="calibre6">Figure 10.3</a> for a preview of the data in the dataset, write a query to build a report that summarizes the sales per vendor per market week.</li>
<li id="calibre_link-123" class="calibre16">Rewrite the query associated with <a href="c07.xhtml#c07-fig-0011" class="calibre6">Figure 7.11</a> using a CTE (
<code class="calibre9">WITH</code>
 clause).</li>
<li id="calibre_link-124" class="calibre16">If you were asked to build a report of total and average market sales by vendor booth type, how might you modify the query associated with <a href="#calibre_link-5" class="calibre6">Figure 10.3</a> to include the information needed for your report?</li>
</ol>
</section>
</section>
</div>


</body></html>