# The SELECT Statement
- 










<head>
  <title>Chapter 2 The SELECT Statement</title>
  <link href="WileyTemplate_v5.5.css" rel="stylesheet" type="text/css"/>
  <meta content="urn:uuid:337bf635-a175-4f47-b1e5-fd6cea377a42" name="Adept.expected.resource"/>
</head>

<body epub:type="bodymatter">

  <section aria-labelledby="c02_1" epub:type="chapter" role="doc-chapter">

    <header>

      <h1 id="c02_1"><span aria-label="15" epub:type="pagebreak" id="Page_15" role="doc-pagebreak"></span><a id="c02"></a><span class="chapterNumber">CHAPTER 2</span><br/><span class="chapterTitle">The SELECT Statement</span></h1>

    </header>

    <section aria-label="chapter opening"><span id="c02-sec-0001"></span>
<p id="c02-para-0001">Before we can discuss designing analytical datasets, you first need to understand the syntax of Structured Query Language (SQL), so the next several chapters will cover SQL basics. Throughout the book, you will see many variations on the themes introduced here, combining these basic concepts with increasing complexity, because even the most complex SQL queries boil down to the same basic concepts.</p>
</section>

    <section aria-labelledby="head-2-22"><span id="c02-sec-0002"></span>
<h2 id="head-2-22">The SELECT Statement</h2>
<p>The majority of the queries in this book will be SELECT statements. A SELECT statement is SQL code that retrieves data from the database. When used in combination with other SQL keywords, 
<code>SELECT</code>
 can be used to view data from a set of columns in a database table, combine data from multiple tables, filter the results, perform calculations, and more.</p>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0003">You'll often see the word 
<code>SELECT</code>
 capitalized in SQL code, and I chose to follow that formatting standard in this book, because 
<code>SELECT</code>
 is a reserved SQL keyword, meaning it is a special instructional word that the code interpreter uses to execute your query. Capitalizing it visually differentiates the keyword from other text in your query, such as field names.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p id="c02-para-0002"><span aria-label="16" epub:type="pagebreak" id="Page_16" role="doc-pagebreak"></span></p>
</section>

    <section aria-labelledby="head-2-23"><span id="c02-sec-0004"></span>
<h2 id="head-2-23">The Fundamental Syntax Structure of a SELECT Query</h2>
<p>SQL SELECT queries follow this basic syntax, though most of the clauses are optional:</p>
<ul class="none" id="c02-list-0001">
<li id="c02-li-0001">
<code>SELECT</code>
 [columns to return]</li>
<li id="c02-li-0002">
<code>FROM</code>
 [schema.table]</li>
<li id="c02-li-0003">
<code>WHERE</code>
 [conditional filter statements]</li>
<li id="c02-li-0004">
<code>GROUP BY</code>
 [columns to group on]</li>
<li id="c02-li-0005">
<code>HAVING</code>
 [conditional filter statements that are run after grouping]</li>
<li id="c02-li-0006">
<code>ORDER BY</code>
 [columns to sort on]</li>
</ul>
<p id="c02-para-0005">The 
<code>SELECT</code>
 and 
<code>FROM</code>
 clauses are generally required, because those indicate which columns to select and from what table. The words in brackets are placeholders, and as you go through the next several chapters of this book, you will learn what to put in each section.</p>
</section>

    <section aria-labelledby="head-2-24"><span id="c02-sec-0005"></span>
<h2 id="head-2-24">Selecting Columns and Limiting the Number of Rows Returned</h2>
<p>The simplest SELECT statement is</p>
<ul class="none" id="c02-list-0002">
<li id="c02-li-0007">
<code>SELECT * FROM</code>
 [schema.table]</li>
</ul>
<p>where [schema.table] is the name of the database schema and table you want to retrieve data from.</p>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0008">In <a href="c05.xhtml">Chapter 5</a>, “SQL JOINs,” you'll learn the syntax for pulling data from multiple tables at once.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p>For example,</p>
<pre id="c02-code-0001">
<code>SELECT * FROM farmers_market.product</code>
</pre>
<p id="c02-para-0010">can be read as “Select everything from the product table in the farmers_market schema.” The asterisk really represents “all columns,” so technically it's “Select all columns from the product table in the farmers_market schema,” but since there are no filters in this query (no WHERE clause, which you'll learn about in <a href="c03.xhtml">Chapter 3</a>, “The WHERE Clause”), it will also return all rows, hence “everything.”</p>
<p id="c02-para-0011">There is another optional clause I didn't include in the basic SELECT syntax in the previous section: the LIMIT clause. (The syntax is different in some database systems. See the note about the 
<code>TOP</code>
 keyword and WHERE clause.) I frequently use 
<code>LIMIT</code>
 while developing queries. 
<code>LIMIT</code>
 sets the maximum number of rows <span aria-label="17" epub:type="pagebreak" id="Page_17" role="doc-pagebreak"></span>that are returned, in cases when you don't want to see all of the results you would otherwise get. It is especially useful when you're developing queries that pull from a large database, when queries take a long time to run, since the query will stop executing and show the output once it's generated the set number of rows.</p>
<p>By using 
<code>LIMIT</code>
, you can get a preview of your current query's results without having to wait for all of the results to be compiled. For example,</p>
<pre id="c02-code-0002">
<code>SELECT * </code>
<code>FROM farmers_market.product</code>
<code>LIMIT 5</code>
</pre>
<p id="c02-para-0013">returns all columns for the first 5 rows of the 
<code>product</code>
 table, as shown in <a href="#c02-fig-0001" id="R_c02-fig-0001">Figure 2.1</a>.</p>
<figure>
<img alt="Schematic illustration of a command returning all columns for the first 5 rows of the product table." class="center" src="images/c02f001.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0001" id="c02-fig-0001" role="doc-backlink">Figure 2.1</a></span></p>
</figcaption>
</figure>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0015">When querying MS SQL Server databases, there is a keyword similar to MySQL's 
<code>LIMIT</code>
 called 
<code>TOP</code>
 (which goes before the 
<code>SELECT</code>
 statement). For Oracle databases, this is accomplished in the WHERE clause using “
<code>WHERE rownum &lt;=</code>
 [number]”. This is one example of the slight differences in SQL syntax across database systems. However, the basic concepts are the same. So, if you learn SQL for one type of database, you will know enough to know what to look up to find the equivalents for other databases. In MySQL syntax, the keyword is 
<code>LIMIT</code>
, which goes at the end of the query.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p id="c02-para-0016">You might have noticed that I put the 
<code>FROM</code>
 clause on the second line in the 
<code>SELECT</code>
 query in the preceding example, while the first 
<code>SELECT</code>
 query was written as a single line. Line breaks and tabs don't matter to SQL code execution and are treated like spaces, so you can indent your SQL and break it into multiple lines for readability without affecting the output.</p>
<p>To specify which columns you want returned, list the column names immediately after 
<code>SELECT</code>
, separated by commas, instead of using the asterisk. The following query lists five product IDs and their associated product names from the 
<code>product</code>
 table, as displayed in <a href="#c02-fig-0002" id="R_c02-fig-0002">Figure 2.2</a>.</p>
<pre id="c02-code-0003">
<code>SELECT product_id, product_name</code>
<code>FROM farmers_market.product</code>
<code>LIMIT 5</code>
<span aria-label="18" epub:type="pagebreak" id="Page_18" role="doc-pagebreak"></span></pre>
<figure>
<img alt="A table depicts query lists five product IDs and their associated product names from the product table." class="center" src="images/c02f002.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0002" id="c02-fig-0002" role="doc-backlink">Figure 2.2</a></span></p>
</figcaption>
</figure>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>TIP</h3>
<p id="c02-para-0019">Even if you want all columns returned, it's good practice to list the names of the columns instead of using the asterisk, especially if the query will be used as part of a data pipeline (if the query is automatically run nightly and the results are used as an input into the next step of a series of code functions without human review, for example). This is because returning “all” columns may result in a different output if the underlying table is modified, such as when a new column is added, or columns appear in a different order, which could break your automated data pipeline.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p>The following query lists five rows of farmer's market vendor booth assignments, displaying the market date, vendor ID, and booth number from the 
<code>vendor_booth_assignments</code>
 table, depicted in <a href="#c02-fig-0003" id="R_c02-fig-0003">Figure 2.3</a>:</p>
<figure>
<img alt="A table depicts query lists five rows of Farmer’s Market vendor booth assign ments, displaying the market date, vendor ID, and booth number from the vendor_booth_assignments table." class="center" src="images/c02f003.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0003" id="c02-fig-0003" role="doc-backlink">Figure 2.3</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0004">
<code>SELECT market_date, vendor_id, booth_number </code>
<code>FROM farmers_market.vendor_booth_assignments</code>
<code>LIMIT 5</code>
</pre>
<p id="c02-para-0021">But this output would make a lot more sense if it were sorted by the market date, right? In the next section, we'll learn how to set the sort order of query results.</p>
</section>

    <section aria-labelledby="head-2-25"><span id="c02-sec-0009"></span>
<h2 id="head-2-25">The ORDER BY Clause: Sorting Results</h2>
<p>The ORDER BY clause is used to sort the output rows. In it, you list the columns you want to sort the results by, in order, separated by commas. You can also specify whether you want the sorting to be in ascending (
<code>ASC</code>
) or descending (
<code>DESC</code>
) order. 
<code>ASC</code>
 sorts text alphabetically and numeric values from low to high, and 
<code>DESC</code>
 sorts them in the reverse order. In MySQL, NULL values appear first when sorting in default ascending order.<span aria-label="19" epub:type="pagebreak" id="Page_19" role="doc-pagebreak"></span></p>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0023">The sort order is ascending by default, so if you want your values sorted in ascending order, the 
<code>ASC</code>
 keyword is optional.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p>The following query sorts the results by product name, even though the product ID is listed first in the output, shown in <a href="#c02-fig-0004" id="R_c02-fig-0004">Figure 2.4</a>:</p>
<figure>
<img alt="A table depicts the results by product name, even though the product ID is listed first in the output." class="center" src="images/c02f004.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0004" id="c02-fig-0004" role="doc-backlink">Figure 2.4</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0005">
<code>SELECT product_id, product_name</code>
<code>FROM farmers_market.product</code>
<code>ORDER BY product_name</code>
<code>LIMIT 5</code>
</pre>
<p>And the following modification to the 
<code>ORDER BY</code>
 clause changes the query to now sort the results by product ID, highest to lowest, as shown in <a href="#c02-fig-0005" id="R_c02-fig-0005">Figure 2.5</a>:</p>
<figure>
<img alt="A table depicts the sorting of the results by product ID, highest to lowest." class="center" src="images/c02f005.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0005" id="c02-fig-0005" role="doc-backlink">Figure 2.5</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0006">
<code>SELECT product_id, product_name</code>
<code>FROM farmers_market.product</code>
<code>ORDER BY product_id DESC</code>
<code>LIMIT 5</code>
</pre>
<p id="c02-para-0026">Note that the rows returned display a different set of products than we saw in the previous query. That's because the 
<code>ORDER BY</code>
 clause is executed before the 
<code>LIMIT</code>
 is imposed. So in addition to limiting the number of rows returned for quicker development (or to conserve space on the page of a book!), a LIMIT clause can also be combined with an ORDER BY statement to sort the results and return the top <i>x</i> number of results, for reporting or validation purposes.</p>
<p id="c02-para-0027">To sort the output of the last query in the previous section, we can add an 
<code>ORDER BY</code>
 line, specifying that we want it to sort the output first by market date, then by vendor ID.</p>
<p>In <a href="#c02-fig-0006" id="R_c02-fig-0006">Figure 2.6</a>, we only see rows with the earliest market date available in the database, since we sorted by 
<code>market_date</code>
 first, there are more than 5 records <span aria-label="20" epub:type="pagebreak" id="Page_20" role="doc-pagebreak"></span>in the table for that same date, and we limited this query to only return the first 5 rows. After sorting by market date, the records are then sorted by vendor ID in ascending order:</p>
<pre id="c02-code-0007">
<code>SELECT market_date, vendor_id, booth_number </code>
<code>FROM farmers_market.vendor_booth_assignments</code>
<code>ORDER BY market_date, vendor_id</code>
<code>LIMIT 5</code>
</pre>
<figure>
<img alt="A table depicts the market date, vendor id, and booth number." class="center" src="images/c02f006.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0006" id="c02-fig-0006" role="doc-backlink">Figure 2.6</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-26"><span id="c02-sec-0011"></span>
<h2 id="head-2-26">Introduction to Simple Inline Calculations</h2>
<p id="c02-para-0029">In this section, you will see examples of calculations being performed on the data in some columns. We'll dive deeper into calculations throughout later chapters, but this will give you a sense of how calculations look in the basic SELECT query syntax.</p>
<p id="c02-para-0030">Let's say we wanted to do a calculation using the data in two different columns in each row. In the 
<code>customer_purchases</code>
 table, we have a 
<code>quantity</code>
 column and a 
<code>cost_to_customer_per_qty</code>
 column, so we can multiply those to get a price.</p>
<p>In <a href="#c02-fig-0007" id="R_c02-fig-0007">Figure 2.7</a>, you can see how the raw data in the selected columns of the 
<code>customer_purchases</code>
 table looks prior to adding any calculations to the following query:</p>
<pre id="c02-code-0008">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> quantity,</code>
    
<code> cost_to_customer_per_qty </code>
<code>FROM farmers_market.customer_purchases</code>
<code>LIMIT 10</code>
<span aria-label="21" epub:type="pagebreak" id="Page_21" role="doc-pagebreak"></span></pre>
<figure>
<img alt="A table depicts the raw data in the selected columns of the customer_purchases." class="center" src="images/c02f007.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0007" id="c02-fig-0007" role="doc-backlink">Figure 2.7</a></span></p>
</figcaption>
</figure>
<p>The following query demonstrates how the values in two different columns can be multiplied by one another by putting an asterisk between them. When used this way, the asterisk represents a multiplication sign. The results of the calculation are shown in the last column in <a href="#c02-fig-0008" id="R_c02-fig-0008">Figure 2.8</a>.</p>
<figure>
<img alt="A table depicts the results of a calculation." class="center" src="images/c02f008.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0008" id="c02-fig-0008" role="doc-backlink">Figure 2.8</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0009">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> quantity, </code>
    
<code> cost_to_customer_per_qty,</code>
    
<code> quantity * cost_to_customer_per_qty </code>
<code>FROM farmers_market.customer_purchases</code>
<code>LIMIT 10</code>
</pre>
<p id="c02-para-0033">To give the calculated column a meaningful name, we can create an <i>alias</i> by adding the keyword 
<code>AS</code>
 after the calculation and then specifying the new name. If your alias includes spaces, it should be surrounded by single quotes. I prefer not to use spaces in my aliases, to avoid having to remember to surround them with quotes if I need to reference them later.</p>
<p>Here, we'll give the alias “
<code>price</code>
” to the result of the “
<code>quantity</code>
 times 
<code>cost_to_customer_per_qty</code>
” calculation. There is no need for the columns used for the calculation to also be included individually like they were in the previous query, so we can remove them in this version.</p>
<pre id="c02-code-0010">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> quantity * cost_to_customer_per_qty AS price </code>
<code>FROM farmers_market.customer_purchases</code>
<code>LIMIT 10</code>
</pre>
<p id="c02-para-0035">The column alias is shown in the header of the last column in <a href="#c02-fig-0009" id="R_c02-fig-0009">Figure 2.9</a>.<span aria-label="22" epub:type="pagebreak" id="Page_22" role="doc-pagebreak"></span></p>
<figure>
<img alt="A table depicts the calculation of price." class="center" src="images/c02f009.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0009" id="c02-fig-0009" role="doc-backlink">Figure 2.9</a></span></p>
</figcaption>
</figure>
<p>In MySQL syntax, the 
<code>AS</code>
 keyword is actually optional, so the following query will return the same results as the previous query. For clarity, we will use the 
<code>AS</code>
 convention for assigning column aliases in this book.</p>
<pre id="c02-code-0011">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> quantity * cost_to_customer_per_qty price </code>
<code>FROM farmers_market.customer_purchases</code>
<code>LIMIT 10</code>
</pre>
<p id="c02-para-0037">A sensible next step would be to calculate transaction totals (how much the customer paid for all of the products they purchased on that day from each vendor). This could be accomplished by adding up the price per customer per market vendor per market date. In <a href="c06.xhtml">Chapter 6</a>, “Aggregating Results for Analysis,” you will learn about aggregate calculations, which calculate summaries across multiple rows. Because we aren't aggregating our results yet, the calculation in the preceding query is applied to the values in each row, as displayed in <a href="#c02-fig-0009">Figure 2.9</a>, and does not calculate across rows or summarize multiple rows.</p>
</section>

    <section aria-labelledby="head-2-27"><span id="c02-sec-0012"></span>
<h2 id="head-2-27">More Inline Calculation Examples: Rounding</h2>
<p>A SQL function is a piece of code that takes inputs that you give it (which are called parameters), performs some operation on those inputs, and returns a value. You can use functions inline in your query to modify the raw values from the database tables before displaying them in the output. A SQL function call uses the following syntax:</p>
<ul class="none" id="c02-list-0003">
<li id="c02-li-0008">FUNCTION_NAME([parameter 1],[parameter 2], … .[parameter n])</li>
</ul>
<p id="c02-para-1038">Each bracketed item shown is a placeholder for an input parameter. Parameters go inside the parentheses following the function name and are separated by commas. The input parameter might be a field name or a value that gives the <span aria-label="23" epub:type="pagebreak" id="Page_23" role="doc-pagebreak"></span>function further instructions. To determine what input parameters a particular function requires, you can look it up in the database documentation, which you can find at <a class="code" href="http://dev.mysql.com/doc">dev.mysql.com/doc</a> for MySQL.</p>
<p id="c02-para-0039">To give an example of how functions are used in SELECT statements, we'll use the 
<code>ROUND()</code>
 function to round a number. In the query from the previous section, the “
<code>price</code>
” field was displayed with four digits after the decimal point. Let's say we wanted to display the number rounded to the nearest penny (in US dollars), which is two digits after the decimal. That can be accomplished using the 
<code>ROUND()</code>
 function.</p>
<p id="c02-para-0040">The syntax of the 
<code>ROUND()</code>
 function is for the first parameter (the first item inside the parentheses) to represent the value to be rounded, followed by a comma, then the second parameter indicating the desired number of digits after the decimal. So 
<code>ROUND(</code>
[column name]
<code>, 3)</code>
 will round the values in the specified column to 3 digits after the decimal.</p>
<p>We can update our query from the previous section to put the price calculation inside the 
<code>ROUND()</code>
 function:</p>
<pre id="c02-code-0012">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> ROUND(quantity * cost_to_customer_per_qty, 2) AS price </code>
<code>FROM farmers_market.customer_purchases</code>
<code>LIMIT 10</code>
</pre>
<p id="c02-para-0042">The result of this rounding is shown in the final column of the output, displayed in <a href="#c02-fig-0010" id="R_c02-fig-0010">Figure 2.10</a>.</p>
<figure>
<img alt="A table depicts the results of rounding the price." class="center" src="images/c02f010.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0010" id="c02-fig-0010" role="doc-backlink">Figure 2.10</a></span></p>
</figcaption>
</figure>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>TIP</h3>
<p id="c02-para-0044">The 
<code>ROUND()</code>
 function can also accept negative numbers for the second parameter, to round digits that are to the left of the decimal point. For example, 
<code>SELECT ROUND(1245, -2)</code>
 will return a value of 1200.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<p id="c02-para-0043"><span aria-label="24" epub:type="pagebreak" id="Page_24" role="doc-pagebreak"></span></p>
</section>

    <section aria-labelledby="head-2-28"><span id="c02-sec-0014"></span>
<h2 id="head-2-28">More Inline Calculation Examples: Concatenating Strings</h2>
<p>In addition to performing numeric operations, there are also inline functions that can be used to modify string values in SQL, as well. In our 
<code>customer</code>
 table, there are separate columns for each customer's first and last names, as shown in <a href="#c02-fig-0011" id="R_c02-fig-0011">Figure 2.11</a>.</p>
<figure>
<img alt="A table depicts the data of customers with different columns." class="center" src="images/c02f011.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0011" id="c02-fig-0011" role="doc-backlink">Figure 2.11</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0013">
<code>SELECT *</code>
<code>FROM farmers_market.customer</code>
<code>LIMIT 5</code>
</pre>
<p>Let's say we wanted to merge each customer's name into a single column that contains the first name, then a space, and then the last name. We can accomplish that by using the 
<code>CONCAT()</code>
 function. The list of string values you want to merge together are entered into the 
<code>CONCAT()</code>
 function as parameters. A space can be included by surrounding it with quotes. You can see the result of this concatenation in <a href="#c02-fig-0012" id="R_c02-fig-0012">Figure 2.12</a>.</p>
<figure>
<img alt="A table depicts the customers id and name." class="center" src="images/c02f012.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0012" id="c02-fig-0012" role="doc-backlink">Figure 2.12</a></span></p>
</figcaption>
</figure>
<pre id="c02-code-0014">
<code>SELECT </code>
    
<code> customer_id,</code>
    
<code> CONCAT(customer_first_name, " ", customer_last_name) AS customer_name</code>
<code>FROM farmers_market.customer</code>
<code>LIMIT 5</code>
</pre>
<p id="c02-para-0047">Note that we can still add an ORDER BY clause and sort by last name first, even though the columns are merged together in the output, as shown in <a href="#c02-fig-0013" id="R_c02-fig-0013">Figure 2.13</a>.<span aria-label="25" epub:type="pagebreak" id="Page_25" role="doc-pagebreak"></span></p>
<figure>
<img alt="A table depicts the customers id and name." class="center" src="images/c02f013.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0013" id="c02-fig-0013" role="doc-backlink">Figure 2.13</a></span></p>
</figcaption>
</figure>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0049">As we discussed earlier in the “Selecting Columns and Limiting the Number of Rows Returned” section, the LIMIT clause determines how many results we will display. Because more than five names are stored in our table, and we are limiting our results to 5, you see the names change from <a href="#c02-fig-0012">Figure 2.12</a> to <a href="#c02-fig-0013">Figure 2.13</a> as the sort order changes.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
<pre id="c02-code-0015">
<code>SELECT </code>
 
<code> customer_id,</code>
 
<code> CONCAT(customer_first_name, " ", customer_last_name) AS customer_name</code>
 
<code>FROM farmers_market.customer</code>
 
<code>ORDER BY customer_last_name, customer_first_name</code>
 
<code>LIMIT 5</code>
 </pre>
<p>It's also possible to nest functions inside other functions, which are executed by the SQL interpreter from the “inside” to the “outside.” 
<code>UPPER()</code>
 is a function that capitalizes string values. We can enclose the 
<code>CONCAT()</code>
 function inside it to uppercase the full name. Let's also change the order of the concatenation parameters to put the last name first, and add a comma after the last name (note the comma before the space inside the double quotes). The result is shown in <a href="#c02-fig-0014" id="R_c02-fig-0014">Figure 2.14</a>.</p>
<pre id="c02-code-0016">
<code>SELECT </code>
    
<code> customer_id,</code>
    
<code> UPPER(CONCAT(customer_last_name, ", ", customer_first_name)) AS customer_name</code>
<code>FROM farmers_market.customer</code>
<code>ORDER BY customer_last_name, customer_first_name</code>
<code>LIMIT 5</code>
</pre>
<p>Because the 
<code>CONCAT()</code>
 function is contained inside the parentheses of the 
<code>UPPER()</code>
 function, the concatenation is performed first, and then the combined string is uppercased.<span aria-label="26" epub:type="pagebreak" id="Page_26" role="doc-pagebreak"></span></p>
<figure>
<img alt="A table depicts the customers id and name." class="center" src="images/c02f014.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0014" id="c02-fig-0014" role="doc-backlink">Figure 2.14</a></span></p>
</figcaption>
</figure>
<aside>
<div class="top hr"><hr/></div>
<section class="feature3">
<h3>NOTE</h3>
<p id="c02-para-0052">Note that we did not sort on the new derived column alias 
<code>customer_name</code>
 here, but on columns that exist in the 
<code>customer</code>
 table. In some cases (depending on what database system you're using, which functions are used, and the execution order of your query) you can't reuse aliases in other parts of the query. It is possible to put some functions or calculations in the ORDER BY clause, to sort by the resulting value. Some other options for referencing derived values will be covered in later chapters.</p>
<div class="bottom hr"><hr/></div>
</section>
</aside>
</section>

    <section aria-labelledby="head-2-29"><span id="c02-sec-0017"></span>
<h2 id="head-2-29">Evaluating Query Output</h2>
<p id="c02-para-0053">When you are developing a SQL SELECT statement, how do you know if the result will include the rows and columns you expect, in the form you expect?</p>
<p id="c02-para-0054">As previously demonstrated, one method is to run the query with a LIMIT each time you make a modification. This gives a quick preview of the first <i>x</i> number of rows to ensure the changes you expect to see are returned, and you can inspect the column names and format of a few output values to verify that they look the way you intended.</p>
<p id="c02-para-0055">However, you still might want to confirm how many rows would have been returned if you hadn't placed the LIMIT on the results. Similarly, there's the concern that your function might not perform as expected on some values that didn't appear in your limited results preview.</p>
<p id="c02-para-0056">I therefore use the query editor to help me review the results of my query a bit further. This method doesn't provide a full quality control of the output (which should be done before putting any query into production), but it can give me a sanity check that the output looks correct enough for me to continue on with my work. To demonstrate, we'll use the “rounded price” query from the earlier “More Inline Calculation Examples: Rounding” section.</p>
<p id="c02-para-0057">First, I remove the 
<code>LIMIT</code>
. Note that your query editor might have a built-in limit (such as 2000 rows) to prevent you from generating a gigantic dataset by accident, so you might need to go into the settings and turn off any pre-set row limits to actually return the full dataset. <a href="#c02-fig-0015" id="R_c02-fig-0015">Figure 2.15</a> shows the “Don't Limit” option available in MySQL Workbench, under the Query menu.</p>
<p>Then, I'll run the query to generate the output for inspection:</p>
<pre id="c02-code-0017">
<code>SELECT </code>
    
<code> market_date, </code>
    
<code> customer_id, </code>
    
<code> vendor_id, </code>
    
<code> ROUND(quantity * cost_to_customer_per_qty, 2) AS price </code>
<code>FROM farmers_market.customer_purchases</code>
<span aria-label="27" epub:type="pagebreak" id="Page_27" role="doc-pagebreak"></span></pre>
<figure>
<img alt="Snapshot shows choosing the limit rows." class="center" src="images/c02f015.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0015" id="c02-fig-0015" role="doc-backlink">Figure 2.15</a></span></p>
</figcaption>
</figure>
<p id="c02-para-0059">The first thing I'll look at in the output is the total count of rows returned, to see if it matches my expectations. This is often displayed at the bottom of the results window or in the output window. In this prototype version of the Farmer's Market database, there are only 21 rows in the 
<code>customer_purchases</code>
 table, which is indicated in the Message column of the Output section of MySQL Workbench, shown in the lower right of <a href="#c02-fig-0016" id="R_c02-fig-0016">Figure 2.16</a>. (Note that there will be more rows when you run this query yourself, after a more realistic volume of data has been added to the database).</p>
<p id="c02-para-0060">Next, I'll look at the resulting dataset that was generated (which is called the “Result Grid” in MySQL Workbench). I check the column headers, to see if I need to change any aliases. Then, I scroll through and spot-check a few of the values in the output, looking for anything that stands out as incorrect or surprising. If I included an ORDER BY clause (which I did not in this case), I'll also ensure the results are sorted the way I intended.</p>
<p id="c02-para-0061">Then, I can use the editor to manually sort each column first in one direction and then the other. For example, <a href="#c02-fig-0017" id="R_c02-fig-0017">Figures 2.17</a> and <a href="#c02-fig-0018" id="R_c02-fig-0018">2.18</a> show the query results manually sorted in the Result Grid by 
<code>market_date</code>
 and 
<code>vendor_id</code>
, respectively. This allows me to look at the minimum and maximum values in each, because that's often where “edge cases” exist, such as unexpected NULLs, strings that <span aria-label="28" epub:type="pagebreak" id="Page_28" role="doc-pagebreak"></span>start with spaces or numbers, or data entry or calculation errors that increase numeric values by a large factor. I can explore anything that looks strange, and I might also spot if there is an unusual value present, because series of frequent values will appear side by side in the sorted results, making both frequent and unique values noticeable as you scroll down the sorted column.</p>
<figure>
<img alt="A table depicts the query results manually sorted in the Result Grid by market_date." class="center" src="images/c02f016.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0016" id="c02-fig-0016" role="doc-backlink">Figure 2.16</a></span></p>
</figcaption>
</figure>
<figure>
<img alt="A table depicts the query results manually sorted in the Result Grid by and vendor_id." class="center" src="images/c02f017.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0017" id="c02-fig-0017" role="doc-backlink">Figure 2.17</a></span></p>
</figcaption>
</figure>
<p id="c02-para-0062">In <a href="c06.xhtml">Chapter 6</a>, you will learn about aggregate queries, which will provide more options for inspecting your results, but these are a few simple steps you <span aria-label="29" epub:type="pagebreak" id="Page_29" role="doc-pagebreak"></span>can take, without writing a more complicated query, to make sure your output looks sensible.</p>
<figure>
<img alt="A table depicts the query results manually sorted in the Result Grid by and vendor_id." class="center" src="images/c02f018.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0018" id="c02-fig-0018" role="doc-backlink">Figure 2.18</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-30"><span id="c02-sec-0018"></span>
<h2 id="head-2-30">SELECT Statement Summary</h2>
<p id="c02-para-0063">In this chapter, you learned basic SQL SELECT statement syntax and how to pull your desired columns from a single table and sort the output. You also learned how simple inline calculations look.</p>
<p>Note that every query in this chapter, even the ones that started to look more complex by including calculations, all followed this basic syntax:</p>
<ul class="none" id="c02-list-0004">
<li id="c02-li-0009">
<code>SELECT</code>
 [columns to return]</li>
<li id="c02-li-0010">
<code>FROM</code>
 [schema.table]</li>
<li id="c02-li-0011">
<code>ORDER BY</code>
 [columns to sort on]</li>
</ul>
<p>You should now be able to describe what the following two queries do. The results for the following query are shown in <a href="#c02-fig-0019" id="R_c02-fig-0019">Figure 2.19</a>:</p>
<pre id="c02-code-0018">
<code>SELECT * FROM farmers_market.vendor</code>
</pre>
<p>The results for the following query are shown in <a href="#c02-fig-0020" id="R_c02-fig-0020">Figure 2.20</a>:</p>
<pre id="c02-code-0019">
<code>SELECT </code>
    
<code> vendor_name,</code>
    
<code> vendor_id,</code>
    
<code> vendor_type</code>
<code>FROM farmers_market.vendor</code>
<code>ORDER BY vendor_name</code>
<span aria-label="30" epub:type="pagebreak" id="Page_30" role="doc-pagebreak"></span></pre>
<figure>
<img alt="A table depicts the results of select star from farmers market query." class="center" src="images/c02f019.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0019" id="c02-fig-0019" role="doc-backlink">Figure 2.19</a></span></p>
</figcaption>
</figure>
<figure>
<img alt="A table depicts the vendor's name, id, and type." class="center" src="images/c02f020.png"/>
<figcaption>
<p><span class="figureLabel"><a href="#R_c02-fig-0020" id="c02-fig-0020" role="doc-backlink">Figure 2.20</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-31"><span id="c02-sec-0019"></span>
<h2 id="head-2-31">Exercises Using the Included Database</h2>
<p>The following exercises refer to the 
<code>customer</code>
 table. The columns contained in the 
<code>customer</code>
 table, and some example rows with data values, are shown in <a href="#c02-fig-0011">Figure 2.11</a>.</p>
<ol class="decimal" id="c02-list-0005">
<li id="c02-li-0012">Write a query that returns everything in the 
<code>customer</code>
 table.</li>
<li id="c02-li-0013">Write a query that displays all of the columns and 10 rows from the 
<code>customer</code>
 table, sorted by 
<code>customer_last_name</code>
, then 
<code>customer_first_name</code>
.</li>
<li id="c02-li-0014">Write a query that lists all customer IDs and first names in the 
<code>customer</code>
 table, sorted by 
<code>first_name</code>
.</li>
</ol>
</section>

  </section>

</body>