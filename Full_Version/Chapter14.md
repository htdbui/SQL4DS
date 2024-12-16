---
title: "SQL Chapter 14"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">

  <section aria-labelledby="c14_1" type="chapter" role="doc-chapter" class="calibre2">

    <header class="calibre3">

      <h1 id="calibre_link-9" class="calibre4"><span aria-label="229" type="pagebreak" id="calibre_link-10" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-11" class="calibre6"></a><span class="calibre">CHAPTER 14</span><br class="calibre7" /><span class="calibre">Storing and Modifying Data</span></h1>

    </header>

    <section aria-label="chapter opening" class="calibre2"><span id="calibre_link-12" class="calibre"></span>
<p id="calibre_link-13" class="calibre8">We have covered many aspects of developing datasets for machine learning that involve selecting data from a database and preparing it for machine learning models, but what do you do once you have designed your query and are ready to start analyzing the results? Your SQL editor will often allow you to write the results of your query to a CSV file to be imported into Business Intelligence (BI) software such as Tableau or machine learning scripts in a language like Python. However, sometimes for data governance, data security, teamwork, or file size and processing speed purposes, it is preferable to store the dataset within the database.</p>
<p id="calibre_link-14" class="calibre8">In this chapter, we'll cover some types of SQL queries beyond 
<code class="calibre9">SELECT</code>
 statements, such as 
<code class="calibre9">INSERT</code>
 statements, which allow you to store the results of your query in a new table in the database.</p>
</section>

    <section aria-labelledby="head-2-90" class="calibre2"><span id="calibre_link-15" class="calibre"></span>
<h2 id="calibre_link-16" class="calibre10">Storing SQL Datasets as Tables and Views</h2>
<p id="calibre_link-17" class="calibre8">In most databases, you can store the results of a query as either a table or a view. Storing results as a <i class="calibre11">table</i> takes a snapshot of whatever the results are at the time the query is run and saves the data returned as a new table object, or as new rows appended to an existing table, depending on how you write your SQL statement. A database <i class="calibre11">view</i> instead stores the SQL itself and runs it on-demand when you write a query that references the name of the view, to dynamically <span aria-label="230" type="pagebreak" id="calibre_link-18" role="doc-pagebreak" class="calibre12"></span>generate a new dataset based on the state of the referenced database objects at the time you run the query. (You may have also heard of the term <i class="calibre11">materialized view</i>, which is more like a stored snapshot and is not what I'm referring to here.)</p>
<p id="calibre_link-19" class="calibre8">If you have database storage space available, permissions to create tables or insert records into tables in your database, and it is not cost-prohibitive to do so, it can be good practice to store snapshots of the datasets you are using in your machine learning applications. You can check with your database administrator to determine whether you can and should create and modify tables, and which schema(s) you have permission to write to.</p>
<p id="calibre_link-20" class="calibre8">When you're iteratively testing various combinations of fields and parameters in your machine learning algorithm, you'll want to test multiple different approaches with the same static dataset, so you can be sure your input data isn't changing each time you run your script by writing the results to a table. You might also decide to store a copy of the dataset for later reference if the dataset you're querying could change over time and you want to keep a record of the exact values that were run through your model at the time you ran it.</p>
<p class="calibre8">One way to store the results of a query is to use a 
<code class="calibre9">CREATE TABLE</code>
 statement. The syntax is</p>
<ul class="none" id="calibre_link-21">
<li id="calibre_link-22" class="calibre13">
<code class="calibre9">CREATE TABLE</code>
 [schema_name].[new_table_name] 
<code class="calibre9">AS</code>
</li>
<li id="calibre_link-23" class="calibre13">
<code class="calibre9">(</code>
</li>
<li id="calibre_link-24" class="calibre13">&nbsp;&nbsp;&nbsp;[your query here]</li>
<li id="calibre_link-25" class="calibre13">
<code class="calibre9">)</code>
</li>
</ul>
<p id="calibre_link-26" class="calibre8">As with the 
<code class="calibre9">SELECT</code>
 statements, the indentation and line breaks in these queries don't matter to the database and are just used to format for readability. The table name used in a 
<code class="calibre9">CREATE TABLE</code>
 statement must be new and unique within the schema. If you try to run the same 
<code class="calibre9">CREATE TABLE</code>
 statement twice in a row, you will get an error stating that the table already exists.</p>
<p class="calibre8">Once you create the table, you can query it like any other table or view, referencing the new name you gave it. If you created a table by accident or want to re-create it with a different name or definition, you can 
<code class="calibre9">DROP</code>
 the table.</p>
<aside class="calibre14">
<div class="top"><hr class="calibre15" /></div>
<section class="feature">
<h3 class="calibre16">WARNING</h3>
<p id="calibre_link-27" class="calibre17">Be very careful when using the 
<code class="calibre9">DROP TABLE</code>
 statement, or you might accidentally delete something that should not have been be deleted! Depending on the database settings and backup frequency, the data you delete may not be recoverable! I usually ensure that I am only granted database permissions to create and drop tables in a personal schema, which is separate from the schema used to run applications or where tables that others are using are stored, so I can't accidentally delete a table I did not create or that is used in a production application.</p>
<div class="top"><hr class="calibre15" /></div>
</section>
</aside>
<p class="calibre8">The syntax for dropping a table is simply:</p>
<ul class="none" id="calibre_link-28">
<li id="calibre_link-29" class="calibre13">
<code class="calibre9">DROP TABLE</code>
 [schema_name].[table_name]</li>
</ul>
<p class="calibre8"><span aria-label="231" type="pagebreak" id="calibre_link-30" role="doc-pagebreak" class="calibre12"></span>So, to create, select from, and drop a table that contains a snapshot of the data that is currently in the Farmer's Market database 
<code class="calibre9">product</code>
 table, filtered to products with a quantity type “unit,” run the following three queries in sequence:</p>
<pre id="calibre_link-31" class="calibre18">
<code class="calibre9">CREATE TABLE farmers_market.product_units AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT * </code>
   
<code class="calibre9"> FROM farmers_market.product </code>
   
<code class="calibre9"> WHERE product_qty_type = "unit"</code>
<code class="calibre9">)</code>
<code class="calibre9">;</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * FROM farmers_market.product_units</code>
<code class="calibre9">;</code>
   
<code class="calibre9"> </code>
<code class="calibre9">DROP TABLE farmers_market.product_units</code>
<code class="calibre9">;</code>
</pre>
<p class="calibre8">The semicolons are used to separate multiple queries in the same file.</p>
<aside class="calibre14">
<div class="top"><hr class="calibre15" /></div>
<section class="feature">
<h3 class="calibre16">TIP</h3>
<p id="calibre_link-32" class="calibre17">If you don't want to accidentally run a 
<code class="calibre9">DROP TABLE</code>
 statement that is included in a file with other SQL statements (since many SQL editors have a “run all” command), comment it out immediately after running it, and save the file with that query commented out, so you don't accidentally run it the next time you open the file and drop a table you didn't intend to! In MySQL Workbench, you can comment out code by preceding each line with two dashes and a space or by surrounding a block of code with 
<code class="calibre9">/*</code>
 and 
<code class="calibre9">*/</code>
.</p>
<div class="top"><hr class="calibre15" /></div>
</section>
</aside>
<p class="calibre8">Database views are created and dropped the same exact way as tables, though when you create a view, you are not actually storing the data, but storing the query to be run when you query the view. So when you drop a view, you are not actually deleting any data, since the data isn't stored; you are just dropping the named reference to the query:</p>
<pre id="calibre_link-33" class="calibre18">
<code class="calibre9">CREATE VIEW farmers_market.product_units_vw AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT * </code>
   
<code class="calibre9"> FROM farmers_market.product </code>
   
<code class="calibre9"> WHERE product_qty_type = "unit"</code>
<code class="calibre9">)</code>
<code class="calibre9">;</code>
   
<code class="calibre9"> </code>
<code class="calibre9">SELECT * FROM farmers_market.product_units_vw</code>
<code class="calibre9">;</code>
   
<code class="calibre9"> </code>
<code class="calibre9">DROP VIEW farmers_market.product_units_vw</code>
<code class="calibre9">;</code>
</pre>
<p id="calibre_link-34" class="calibre8">Note that some database systems, like SQL Server, support a 
<code class="calibre9">SELECT INTO</code>
 syntax, which operates much like the 
<code class="calibre9">CREATE TABLE</code>
 statement previously <span aria-label="232" type="pagebreak" id="calibre_link-35" role="doc-pagebreak" class="calibre12"></span>demonstrated, and is often used to create backups of existing tables. Check your database's documentation online to determine which syntax to use.</p>
</section>

    <section aria-labelledby="head-2-91" class="calibre2"><span id="calibre_link-36" class="calibre"></span>
<h2 id="calibre_link-37" class="calibre10">Adding a Timestamp Column</h2>
<p id="calibre_link-38" class="calibre8">When you create or modify a database table, you might want to keep a record of when each row in the table was created or last modified. You can do this by adding a timestamp column to your 
<code class="calibre9">CREATE TABLE</code>
 or 
<code class="calibre9">UPDATE</code>
 statement. The syntax for creating a timestamp varies by database system, but in MySQL, the function that returns the current date and time is called 
<code class="calibre9">CURRENT_TIMESTAMP</code>
. You can give the timestamp column an alias like any calculated column.</p>
<p id="calibre_link-39" class="calibre8">Keep in mind that the timestamp is generated by the database server, so if the database is in another time zone, the timestamp returned by the function may be different than the current time at your physical location. Many databases use Coordinated Universal Time (UTC) as their default timestamp time, which is a global time standard that is synchronized using atomic clocks, aligns in hour offset with the Greenwich Mean Time time zone, and doesn't change for Daylight Savings Time. Eastern Standard Time, which is observed in the Eastern Time Zone in North America during the winter, can be signified as UTC-05:00, meaning it is five hours behind UTC. Eastern Daylight Time, observed during the summer, has an offset of UTC-04:00, because the Eastern Time Zone observes Daylight Savings Time and shifts by an hour, while UTC does not shift. You can see how time zone math can quickly get complicated, which is why many databases simplify by using a standard UTC clock time instead of a developer's local time.</p>
<p class="calibre8">We can modify the preceding 
<code class="calibre9">CREATE TABLE</code>
 example to include a timestamp column as follows:</p>
<pre id="calibre_link-40" class="calibre18">
<code class="calibre9">CREATE TABLE farmers_market.product_units AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT p.*,</code>
    
<code class="calibre9">   CURRENT_TIMESTAMP AS snapshot_timestamp</code>
   
<code class="calibre9"> FROM farmers_market.product AS p</code>
   
<code class="calibre9"> WHERE product_qty_type = "unit"</code>
<code class="calibre9">)</code>
</pre>
<p id="calibre_link-41" class="calibre8">Example output from this query is shown in <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 14.1</a>.<span aria-label="233" type="pagebreak" id="calibre_link-42" role="doc-pagebreak" class="calibre12"></span></p>
<figure class="calibre19">
<figcaption class="calibre20">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 14.1</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-92" class="calibre2"><span id="calibre_link-43" class="calibre"></span>
<h2 id="calibre_link-44" class="calibre10">Inserting Rows and Updating Values in Database Tables</h2>
<p id="calibre_link-45" class="calibre8">If you want to modify data in an existing database table, you can use an 
<code class="calibre9">INSERT</code>
 statement to add a new row or an 
<code class="calibre9">UPDATE</code>
 statement to modify an existing row of data in a table.</p>
<p id="calibre_link-46" class="calibre8">In this chapter, we're specifically inserting results of a query into another table, which is a specific kind of 
<code class="calibre9">INSERT</code>
 statement called 
<code class="calibre9">INSERT INTO SELECT</code>
.</p>
<p class="calibre8">The syntax is</p>
<ul class="none" id="calibre_link-47">
<li id="calibre_link-48" class="calibre13">INSERT INTO [schema_name].[table_name] ([comma-separated list of column names])</li>
<li id="calibre_link-49" class="calibre13">[your SELECT query here]</li>
</ul>
<p class="calibre8">So if we wanted to add rows to our 
<code class="calibre9">product_units</code>
 table created earlier, we would write:</p>
<pre id="calibre_link-50" class="calibre18">
<code class="calibre9">INSERT INTO farmers_market.product_units (product_id, product_name, product_size, product_category_id, product_qty_type, snapshot_timestamp)</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> product_id, </code>
       
<code class="calibre9"> product_name,</code>
       
<code class="calibre9"> product_size, </code>
       
<code class="calibre9"> product_category_id, </code>
       
<code class="calibre9"> product_qty_type,</code>
       
<code class="calibre9"> CURRENT_TIMESTAMP</code>
   
<code class="calibre9"> FROM farmers_market.product AS p</code>
   
<code class="calibre9"> WHERE product_id = 23</code>
</pre>
<p id="calibre_link-51" class="calibre8">It is important that the columns in both queries are in the same order. The corresponding fields may not have identical names, but the system will attempt to insert the returned values from the 
<code class="calibre9">SELECT</code>
 statement in the column order listed in parentheses.</p>
<p id="calibre_link-52" class="calibre8">Now when we query the 
<code class="calibre9">product_units</code>
 table, we'll have a snapshot of the same product row at two different times, as shown in <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 14.2</a>.</p>
<figure class="calibre19">
<img alt="A table records product id, name, size, category id, quantity type, and snapshot timestamp." class="center" src="images/000000.png" />
<figcaption class="calibre20">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 14.2</a></span></p>
</figcaption>
</figure>
<p class="calibre8">If you make a mistake when inserting a row and want to delete it, the syntax is simply</p>
<ul class="none" id="calibre_link-53">
<li id="calibre_link-54" class="calibre13">
<code class="calibre9">DELETE FROM</code>
 [schema_name].[table_name]</li>
<li id="calibre_link-55" class="calibre13">
<code class="calibre9">WHERE</code>
 [set of conditions that uniquely identifies the row]</li>
</ul>
<p id="calibre_link-56" class="calibre8"><span aria-label="234" type="pagebreak" id="calibre_link-57" role="doc-pagebreak" class="calibre12"></span>You may want to start with 
<code class="calibre9">SELECT *</code>
 instead of 
<code class="calibre9">DELETE</code>
 so you can see what rows will be deleted before running the 
<code class="calibre9">DELETE</code>
 statement!</p>
<p class="calibre8">The 
<code class="calibre9">product_id</code>
 and 
<code class="calibre9">snapshot_timestamp</code>
 uniquely identify rows in the 
<code class="calibre9">product_units</code>
 table, so we can run the following statement to delete the row added by our previous 
<code class="calibre9">INSERT INTO</code>
:</p>
<pre id="calibre_link-58" class="calibre18">
<code class="calibre9">DELETE FROM farmers_market.product_units</code>
<code class="calibre9">WHERE product_id = 23</code>
   
<code class="calibre9"> AND snapshot_timestamp = '2021-04-18 00:49:24'</code>
</pre>
<p class="calibre8">Sometimes you want to update a value in an existing row instead of inserting a totally new row. The syntax for an 
<code class="calibre9">UPDATE</code>
 statement is as follows:</p>
<ul class="none" id="calibre_link-59">
<li id="calibre_link-60" class="calibre13">
<code class="calibre9">UPDATE</code>
 [schema_name].[table_name]</li>
<li id="calibre_link-61" class="calibre13">
<code class="calibre9">SET</code>
 [column_name] = [new value]</li>
<li id="calibre_link-62" class="calibre13">
<code class="calibre9">WHERE</code>
 [set of conditions that uniquely identifies the rows you want to change]</li>
</ul>
<p id="calibre_link-63" class="calibre8">Let's say that you've already entered all of the farmer's market vendor booth assignments for the next several months, but vendor 4 informs you that they can't make it on October 10, so you decide to upgrade vendor 8 to vendor 4's booth, which is larger and closer to the entrance, for the day.</p>
<p class="calibre8">Before making any changes, let's snapshot the existing vendor booth assignments, along with the vendor name and booth type, into a new table using the following SQL:</p>
<pre id="calibre_link-64" class="calibre18">
<code class="calibre9">CREATE TABLE farmers_market.vendor_booth_log AS</code>
<code class="calibre9">(</code>
   
<code class="calibre9"> SELECT vba.*, </code>
       
<code class="calibre9"> b.booth_type, </code>
       
<code class="calibre9"> v.vendor_name,</code>
       
<code class="calibre9"> CURRENT_TIMESTAMP AS snapshot_timestamp </code>
   
<code class="calibre9"> FROM farmers_market.vendor_booth_assignments vba</code>
       
<code class="calibre9"> INNER JOIN farmers_market.vendor v</code>
           
<code class="calibre9"> ON vba.vendor_id = v.vendor_id</code>
       
<code class="calibre9"> INNER JOIN farmers_market.booth b</code>
           
<code class="calibre9"> ON vba.booth_number = b.booth_number</code>
   
<code class="calibre9"> WHERE market_date&gt;= '2020-10-01'</code>
<code class="calibre9">)</code>
</pre>
<p id="calibre_link-65" class="calibre8">Selecting all records from this new log table produces the results shown in <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 14.3</a>.<span aria-label="235" type="pagebreak" id="calibre_link-66" role="doc-pagebreak" class="calibre12"></span></p>
<figure class="calibre19">
<img alt="A table records vendor id, booth number, booth type, vendor name, and snapshot timestamp." class="center" src="images/000001.png" />
<figcaption class="calibre20">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 14.3</a></span></p>
</figcaption>
</figure>
<p class="calibre8">To update vendor 8's booth assignment, we can run the following SQL:</p>
<pre id="calibre_link-67" class="calibre18">
<code class="calibre9">UPDATE farmers_market.vendor_booth_assignments</code>
<code class="calibre9">SET booth_number = 7 </code>
<code class="calibre9">WHERE vendor_id = 8 and market_date = '2020-10-10'</code>
</pre>
<p class="calibre8">And we can delete vendor 4's booth assignment with the following SQL:</p>
<pre id="calibre_link-68" class="calibre18">
<code class="calibre9">DELETE FROM farmers_market.vendor_booth_assignments</code>
<code class="calibre9">WHERE vendor_id = 4 and market_date = '2020-10-10'</code>
</pre>
<p class="calibre8">Now, when we query the 
<code class="calibre9">vendor_booth_assignments</code>
 table, there is no record that vendor 4 had a booth assignment on that date, or that vendor 8's booth assignment used to be different. But we do have a record of the previous assignments in the 
<code class="calibre9">vendor_booth_log</code>
 we created! Now we can insert new records into the log table to record the latest changes:</p>
<pre id="calibre_link-69" class="calibre18">
<code class="calibre9">INSERT INTO farmers_market.vendor_booth_log (vendor_id, booth_number, market_date, booth_type, vendor_name, snapshot_timestamp)</code>
   
<code class="calibre9"> SELECT </code>
       
<code class="calibre9"> vba.vendor_id, </code>
       
<code class="calibre9"> vba.booth_number, </code>
       
<code class="calibre9"> vba.market_date,</code>
       
<code class="calibre9"> b.booth_type, </code>
       
<code class="calibre9"> v.vendor_name,</code>
       
<code class="calibre9"> CURRENT_TIMESTAMP AS snapshot_timestamp </code>
   
<code class="calibre9"> FROM farmers_market.vendor_booth_assignments vba</code>
       
<code class="calibre9"> INNER JOIN farmers_market.vendor v</code>
           
<code class="calibre9"> ON vba.vendor_id = v.vendor_id</code>
       
<code class="calibre9"> INNER JOIN farmers_market.booth b</code>
           
<code class="calibre9"> ON vba.booth_number = b.booth_number</code>
   
<code class="calibre9"> WHERE market_date&gt;= '2020-10-01'</code>
</pre>
<p id="calibre_link-70" class="calibre8">So now even though the original 
<code class="calibre9">vendor_booth_assignments</code>
 table doesn't contain the original booth assignments for these two vendors on October 10, if we had run an analysis at an earlier date and wanted to see what the database <span aria-label="236" type="pagebreak" id="calibre_link-71" role="doc-pagebreak" class="calibre12"></span>values were at that time, we could query this 
<code class="calibre9">vendor_booth_log</code>
 table to look at the values at different points time, as shown in <a href="#calibre_link-7" id="calibre_link-8" class="calibre6">Figure 14.4</a>.</p>
<figure class="calibre19">
<img alt="A table records vendor id, booth number, market date, and snapshot timestamp." class="center" src="images/000002.png" />
<figcaption class="calibre20">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-8" id="calibre_link-7" role="doc-backlink" class="calibre6">Figure 14.4</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-93" class="calibre2"><span id="calibre_link-72" class="calibre"></span>
<h2 id="calibre_link-73" class="calibre10">Using SQL Inside Scripts</h2>
<p id="calibre_link-74" class="calibre8">Importing the datasets you develop into your machine learning script is beyond the scope of this book, but you can search the internet for the combination of SQL and your chosen scripting language and packages to find tutorials. For example, searching “import SQL python pandas dataframe” will lead you to tutorials for connecting to a database from within your Python script, running a SQL query, and importing the results into a pandas dataframe for analysis. You can usually either paste the SQL query into your script and store it in a string variable or reference the SQL stored in a text document.</p>
<p id="calibre_link-75" class="calibre8">Keep in mind that some special characters in your query will need to be “escaped.” For example, if you surround your SQL query in double quotes to store it in a string variable in Python, it will interpret any double quotes within your query as ending the string and raise an error. To use quotes inside quoted strings in Python, you can precede them with a backslash.</p>
<p class="calibre8">Representing this SQL query as a string in Python</p>
<pre id="calibre_link-76" class="calibre18">
<code class="calibre9">SELECT * FROM farmers_market.product WHERE product_qty_type = "unit"</code>
</pre>
<p class="calibre8">requires escaping the internal double quotes, like so:</p>
<pre id="calibre_link-77" class="calibre18">
<code class="calibre9">my_query = "SELECT * FROM farmers_market.product WHERE product_qty_type = \"unit\""</code>
</pre>
<p id="calibre_link-78" class="calibre8">This is common in programming languages where strings are surrounded by quotes, since many strings contain quotes in the enclosed text, so you can search for “string escape characters” along with your chosen machine learning script tool or language to find out how to modify your SQL for use within your script.</p>
<p id="calibre_link-79" class="calibre8">You can also write data from your script back to the database using SQL, but the approach varies based on the scripting language you're using and the type of database you're connecting to. In Python, for example, there are packages <span aria-label="237" type="pagebreak" id="calibre_link-80" role="doc-pagebreak" class="calibre12"></span>available to help you connect and write to a variety of databases without even needing to write dynamic SQL 
<code class="calibre9">INSERT</code>
 statements&mdash;the packages generate the SQL for you to insert values from an object in Python like a dataframe into a database table for persistent storage.</p>
<p id="calibre_link-81" class="calibre8">Another approach is to programmatically create a file to temporarily store your data, transfer the file to a location that is accessible from your script and your database, and load the results from the file into your table. For example, you might use Python to write data from a pandas dataframe to a CSV file, transfer the CSV file to an Amazon Web Services S3 bucket, then access the file from the database and copy the records into an existing table in a Redshift database. All of these steps can be automated from your script.</p>
<p id="calibre_link-82" class="calibre8">One machine learning use case for writing data from your script to the database is if you want to store your transformed dataset after you have completed feature engineering and data preprocessing steps in your script that weren't completed in the original dataset-generating SQL.</p>
<p id="calibre_link-83" class="calibre8">Another use case for writing values generated within your script back to the database is when you want to store the results that your predictive model generates and associate them with the original dataset. You can create a table that stores the unique identifiers for the rows in your input dataset for which you have scores, a timestamp, the name or ID of your model, and the score or classification generated by the model for each record. You can insert new rows into the table each time you refresh your model scores. Then, use a SQL query to filter this model score log table to a specific date and model identifier and join it to the table with your input dataset, joining on the unique row identifiers. This will allow you to analyze the results of the model alongside the input data used to generate the predictions at the time.</p>
<p id="calibre_link-84" class="calibre8">There are many ways to connect to and interact with data from a database in your other scripts that may or may not require SQL.</p>
</section>

    <section aria-labelledby="head-2-94" class="calibre2"><span id="calibre_link-85" class="calibre"></span>
<h2 id="calibre_link-86" class="calibre10">In Closing</h2>
<p id="calibre_link-87" class="calibre8">Now that you know SQL basics, you should have the foundation needed to create datasets for your machine learning models, even if you need to search the internet for functions and syntax that were not covered in this book.</p>
<p id="calibre_link-88" class="calibre8">I have been a data scientist for five years now, and all of the queries I have written to generate my datasets for machine learning have been variations of the SQL I originally learned in school 20 years ago. I hope that this book has given you the SQL skills you need to achieve your analysis goals faster and more independently, and that you find pulling and modifying your own datasets as empowering as I do!</p>
</section>

    <section aria-labelledby="head-2-95" class="calibre2"><span id="calibre_link-89" class="calibre"></span>
<h2 id="calibre_link-90" class="calibre10">Exercises</h2>
<ol class="decimal" id="calibre_link-91">
<li id="calibre_link-92" class="calibre21"><span aria-label="238" type="pagebreak" id="calibre_link-93" role="doc-pagebreak" class="calibre12"></span>If you include a 
<code class="calibre9">CURRENT_TIMESTAMP</code>
 column when you create a view, what would you expect the values of that column to be when you query the view?</li>
<li id="calibre_link-94" class="calibre21">Write a query to determine what the data from the 
<code class="calibre9">vendor_booth_assignment</code>
 table looked like on October 3, 2020 by querying the 
<code class="calibre9">vendor_booth_log</code>
 table created in this chapter. (Assume that records have been inserted into the log table any time changes were made to the 
<code class="calibre9">vendor_booth_assignment</code>
 table.)</li>
</ol>
</section>

  </section>

</div>



</body></html>