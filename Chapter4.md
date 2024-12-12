# Introduction
- In Chapters 2 and 3, you learned to specify columns and rows to pull from a database using the WHERE clause to filter rows. 
- But what if you want a column or value based on a conditional statement? For example, instead of filtering purchases over $50, you want to flag each purchase as above or below $50. Or, you need to encode categorical strings into numeric values for a machine learning algorithm. These are called "derived columns" or "calculated fields" in SQL, and creating new columns with different values is known as "feature engineering." This is where CASE statements come in.
- If you know "if" statements in languages like Python, SQL handles conditional logic similarly, just with different syntax.
# CASE Statement Syntax















<br><br><br><br><br><br><br><br><br>
    <section aria-label="chapter opening" class="calibre2"><span id="calibre_link-26" class="calibre"></span>
<p id="calibre_link-27" class="calibre8">In <a href="c02.xhtml" class="calibre6">Chapters 2</a>, �The SELECT Statement,� and <a href="c03.xhtml" class="calibre6">3</a>, �The WHERE Clause,� you learned how to specify which columns and rows you want to pull from a database table into your dataset. We used the WHERE clause to filter rows using conditional statements that must evaluate to TRUE in order for a row to be returned.</p>
<p class="calibre8">But what if, instead of using conditional statements to filter rows, you want a column or value in your dataset to be based on a conditional statement? For example, instead of filtering your results to purchases over $50, say you just want to return all rows and create a new column that flags each purchase as being above or below $50? Or, maybe the machine learning algorithm you want to use can't accept a categorical string column as an input feature, so you want to encode those categories into numeric values. These are a version of what SQL developers call �derived columns� or �calculated fields,� and creating new columns that present the values differently is what data scientists call �feature engineering.� This is where CASE statements come in.</p>
<aside class="calibre9">
<div class="top"><hr class="calibre10" /></div>
<section class="feature">
<h3 class="calibre11">NOTE</h3>
<p id="calibre_link-28" class="calibre12">If you're familiar with other scripting languages like Python that use �if� statements, you'll find that SQL handles conditional logic somewhat similarly, just with different syntax.</p>
<div class="top"><hr class="calibre10" /></div>
</section>
</aside>
<p id="calibre_link-29" class="calibre8"><span aria-label="50" type="pagebreak" id="calibre_link-30" role="doc-pagebreak" class="calibre13"></span></p>
</section>

    <section aria-labelledby="head-2-40" class="calibre2"><span id="calibre_link-31" class="calibre"></span>
<h2 id="calibre_link-32" class="calibre14">CASE Statement Syntax</h2>
<p class="calibre8">You use conditional reasoning in your daily life any time you think �If [one condition] is true, then [take this action]. Otherwise, [take this other action].� �If the weather forecast predicts it will rain today, then I'll take an umbrella with me. Otherwise, I'll leave the umbrella at home.� In SQL, the code to delineate this type of logic is called a CASE statement, which uses the following syntax:</p>
<pre id="calibre_link-33" class="calibre15">
<code class="calibre16">CASE</code>
   
<code class="calibre16"> WHEN [first conditional statement] </code>
     
<code class="calibre16"> THEN [value or calculation]</code>
   
<code class="calibre16"> WHEN [second conditional statement] </code>
     
<code class="calibre16"> THEN [value or calculation]</code>
   
<code class="calibre16"> ELSE [value or calculation]</code>
<code class="calibre16">END</code>
</pre>
<p class="calibre8">This statement indicates that you want a column to contain different values under different conditions. If we put the umbrella example into this form:</p>
<pre id="calibre_link-34" class="calibre15">
<code class="calibre16">CASE</code>
   
<code class="calibre16"> WHEN weather_forecast = 'rain'</code>
     
<code class="calibre16"> THEN 'take umbrella'</code>
   
<code class="calibre16"> ELSE 'leave umbrella at home'</code>
<code class="calibre16">END</code>
</pre>
<p id="calibre_link-35" class="calibre8">the 
<code class="calibre16">WHEN</code>
s are evaluated in order, from top to bottom, and the first time a condition evaluates to TRUE, the corresponding 
<code class="calibre16">THEN</code>
 part of the statement is executed, and no other 
<code class="calibre16">WHEN</code>
 conditions are evaluated.</p>
<p class="calibre8">To illustrate, consider this nonsense query:</p>
<pre id="calibre_link-36" class="calibre15">
<code class="calibre16">SELECT</code>
   
<code class="calibre16"> CASE </code>
       
<code class="calibre16"> WHEN 1=1 THEN 'Yes'</code>
       
<code class="calibre16"> WHEN 2=2 THEN 'No'</code>
   
<code class="calibre16"> END</code>
</pre>
<p id="calibre_link-37" class="calibre8">This query will always evaluate to �Yes,� because 1=1 is always TRUE, and therefore the 2=2 conditional statement is never evaluated, even though it is also true.</p>
<p id="calibre_link-38" class="calibre8">The 
<code class="calibre16">ELSE</code>
 part of the statement is optional, and that value or calculation result is returned if none of the conditional statements above it evaluate to TRUE. If the 
<code class="calibre16">ELSE</code>
 is not included and none of the 
<code class="calibre16">WHEN</code>
 conditionals evaluate to TRUE, the resulting value will be NULL.</p>
<p id="calibre_link-39" class="calibre8">You should always alias columns that contain 
<code class="calibre16">CASE</code>
 statements so the resulting column headers are readable, as demonstrated in the queries in this chapter.</p>
<p id="calibre_link-40" class="calibre8"><span aria-label="51" type="pagebreak" id="calibre_link-41" role="doc-pagebreak" class="calibre13"></span>Let's say that we want to know which vendors primarily sell fresh produce and which don't. <a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 4.1</a> shows the vendor types currently in our Farmer's Market database.</p>
<figure class="calibre17">
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 4.1</a></span></p>
</figcaption>
</figure>
<p class="calibre8">The vendors we want to label as �Fresh Produce� have the word �Fresh� in the 
<code class="calibre16">vendor_type</code>
 column. We can use a 
<code class="calibre16">CASE</code>
 statement and the 
<code class="calibre16">LIKE</code>
 operator that was covered in <a href="c03.xhtml" class="calibre6">Chapter 3</a> to create a new column, which we'll alias 
<code class="calibre16">vendor_type_condensed</code>
, that condenses the vendor types to just �Fresh Produce� or �Other�:</p>
<pre id="calibre_link-42" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> vendor_id,</code>
   
<code class="calibre16"> vendor_name,</code>
   
<code class="calibre16"> vendor_type,</code>
   
<code class="calibre16"> CASE </code>
      
<code class="calibre16"> WHEN LOWER(vendor_type) LIKE '%fresh%'</code>
        
<code class="calibre16"> THEN 'Fresh Produce'</code>
      
<code class="calibre16"> ELSE 'Other' </code>
   
<code class="calibre16"> END AS vendor_type_condensed</code>
<code class="calibre16">FROM farmers_market.vendor</code>
</pre>
<p id="calibre_link-43" class="calibre8">In the last two columns of <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 4.2</a>, you can see how the vendor types were converted to condensed vendor types.</p>
<figure class="calibre17">
<img alt="A table records vendor id, vendor type, vendor name, and vendor type condensed." class="center" src="images/000000.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 4.2</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-44" class="calibre8">We're using the 
<code class="calibre16">LOWER()</code>
 function (which does the opposite of the 
<code class="calibre16">UPPER()</code>
 function demonstrated in <a href="c02.xhtml" class="calibre6">Chapter 2</a>) to lowercase the vendor type string, because we don't want the comparison to fail because of capitalization. 
<code class="calibre16">UPPER()</code>
 would have also worked, if we then made the comparison string all caps: 
<code class="calibre16">'%FRESH%'</code>
.</p>
<p id="calibre_link-45" class="calibre8"><span aria-label="52" type="pagebreak" id="calibre_link-46" role="doc-pagebreak" class="calibre13"></span>If a new vendor type is added to the database that includes the word �fresh,� this query using the 
<code class="calibre16">LIKE</code>
 comparison would automatically categorize it as �Fresh Produce� in the 
<code class="calibre16">vendor_type_condensed</code>
 column. If we only wanted existing vendor types to be labeled using this logic, we could instead use the 
<code class="calibre16">IN</code>
 keyword and explicitly list the existing vendor types we want to label with the �Fresh Produce� category. As a data analyst or data scientist building a dataset that may be refreshed as new data is added to the database, you should always consider what might happen to your transformed columns if the underlying data changes.</p>
</section>

    <section aria-labelledby="head-2-41" class="calibre2"><span id="calibre_link-47" class="calibre"></span>
<h2 id="calibre_link-48" class="calibre14">Creating Binary Flags Using CASE</h2>
<p class="calibre8">A CASE statement can be used to create a �binary flag field,� which is a type of field that's often found in machine learning datasets. A binary flag field contains only 1s or 0s, usually indicating a �Yes� or �No� or �exists� or �doesn't exist� type of value. For example, the Farmer's Markets in our database all occur on Wednesday evenings or Saturday mornings. Many machine learning algorithms won't know what to do with the words �Wednesday� and �Saturday� that appear in our database, as shown in the 
<code class="calibre16">market_day column</code>
 of <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 4.3</a>:</p>
<pre id="calibre_link-49" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> market_date,</code>
   
<code class="calibre16"> market_day</code>
<code class="calibre16">FROM farmers_market.market_date_info</code>
<code class="calibre16">LIMIT 5</code>
</pre>
<figure class="calibre17">
<img alt="A table records market date and market day." class="center" src="images/000001.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 4.3</a></span></p>
</figcaption>
</figure>
<p class="calibre8">But, the algorithm could use a numeric value as an input. So, how might we turn this string column into a number? One approach we can take to including the market day in our dataset is to generate a binary flag field that indicates whether it's a weekday or weekend market. We can do this with a CASE statement, making a new column that contains a 1 if the market occurs on a Saturday or Sunday, and a 0 if it doesn't, calling the field �
<code class="calibre16">weekend_flag</code>
,� as shown in <a href="#calibre_link-7" id="calibre_link-8" class="calibre6">Figure 4.4</a>.</p>
<pre id="calibre_link-50" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> market_date,</code>
   <span aria-label="53" type="pagebreak" id="calibre_link-51" role="doc-pagebreak" class="calibre13"></span>
<code class="calibre16"> CASE </code>
       
<code class="calibre16"> WHEN market_day = 'Saturday' OR market_day = 'Sunday' </code>
           
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0</code>
   
<code class="calibre16"> END AS weekend_flag</code>
<code class="calibre16">FROM farmers_market.market_date_info</code>
<code class="calibre16">LIMIT 5</code>
</pre>
<figure class="calibre17">
<img alt="A table records market date and weekend flag." class="center" src="images/000002.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-8" id="calibre_link-7" role="doc-backlink" class="calibre6">Figure 4.4</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-52" class="calibre8">You may have noticed that I included �Sunday� in the 
<code class="calibre16">OR</code>
 statement, even though we said earlier that our farmer's markets currently occur on Wednesday evenings and Saturday mornings. I had decided to call the field �
<code class="calibre16">weekend_flag</code>
� instead of �saturday_flag� because when creating this example, I imagined an analytical question that could be asked: �Do farmers sell more produce at our weekend market or at our weekday market?� If the farmer's market ever changes or expands its schedule to hold a market on a Sunday, this CASE statement will still correctly flag it as a weekend market for the analysis. There is not much downside to making the field aliased �
<code class="calibre16">weekend_flag</code>
� actually mean what it's called (except for the tiny additional computation done to check the second 
<code class="calibre16">OR</code>
 condition when necessary, which is unlikely to make any noticeable difference for data on the scale most farmer's markets could collect) and planning for the future possibility of other market days when designing a dataset to answer this question.</p>
</section>

    <section aria-labelledby="head-2-42" class="calibre2"><span id="calibre_link-53" class="calibre"></span>
<h2 id="calibre_link-54" class="calibre14">Grouping or Binning Continuous Values Using CASE</h2>
<p class="calibre8">In <a href="c03.xhtml" class="calibre6">Chapter 3</a>, we had a query that filtered to only customer purchases where an item or quantity of an item cost over $50, by putting a conditional statement in the 
<code class="calibre16">WHERE</code>
 clause. But let's say we wanted to return all rows, and instead of using that value as a filter, only indicate whether the cost was over $50 or not. We could write the query like this:</p>
<pre id="calibre_link-55" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> market_date, </code>
   
<code class="calibre16"> customer_id, </code>
   
<code class="calibre16"> vendor_id,</code>
   
<code class="calibre16"> ROUND(quantity * cost_to_customer_per_qty, 2) AS price,</code>
   <span aria-label="54" type="pagebreak" id="calibre_link-56" role="doc-pagebreak" class="calibre13"></span>
<code class="calibre16"> CASE </code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty&gt; 50 </code>
         
<code class="calibre16"> THEN 1</code>
      
<code class="calibre16"> ELSE 0</code>
   
<code class="calibre16"> END AS price:over_50</code>
<code class="calibre16">FROM farmers_market.customer_purchases</code>
<code class="calibre16">LIMIT 10</code>
</pre>
<p id="calibre_link-57" class="calibre8">The final column in <a href="#calibre_link-9" id="calibre_link-10" class="calibre6">Figure 4.5</a> now contains a flag indicating which item purchases were over $50. The price is displayed here for explanatory purposes, but could be left out if only the flag indicator were important.</p>
<figure class="calibre17">
<img alt="A table records market date, customer id, vendor id, price, and price over 50." class="center" src="images/000003.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-9" role="doc-backlink" class="calibre6">Figure 4.5</a></span></p>
</figcaption>
</figure>
<p class="calibre8">CASE statements can also be used to �bin� a continuous variable, such as price. Let's say we wanted to put the line-item customer purchases into bins of under $5.00, $5.00&ndash;$9.99, $10.00&ndash;$19.99, or $20.00 and over. We could accomplish that with a 
<code class="calibre16">CASE</code>
 statement in which we surround the values after the 
<code class="calibre16">THEN</code>
s in single quotes to generate a column that contains a string label, as shown in <a href="#calibre_link-11" id="calibre_link-12" class="calibre6">Figure 4.6</a>:</p>
<pre id="calibre_link-58" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> market_date, </code>
   
<code class="calibre16"> customer_id, </code>
   
<code class="calibre16"> vendor_id, </code>
   
<code class="calibre16"> ROUND(quantity * cost_to_customer_per_qty, 2) AS price,</code>
   
<code class="calibre16"> CASE </code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 5.00 </code>
          
<code class="calibre16"> THEN 'Under $5'</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 10.00 </code>
          
<code class="calibre16"> THEN '$5-$9.99'</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 20.00 </code>
          
<code class="calibre16"> THEN '$10-$19.99'</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty&gt;= 20.00 </code>
          
<code class="calibre16"> THEN '$20 and Up'</code>
   
<code class="calibre16"> END AS price:bin</code>
<code class="calibre16">FROM farmers_market.customer_purchases</code>
<code class="calibre16">LIMIT 10</code>
<span aria-label="55" type="pagebreak" id="calibre_link-59" role="doc-pagebreak" class="calibre13"></span></pre>
<figure class="calibre17">
<img alt="A table records market date, customer id, vendor id, price, and price bin." class="center" src="images/000004.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-12" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 4.6</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Or, if the result needs to be numeric, a different approach is to output the bottom end of the numeric range, as shown in <a href="#calibre_link-13" id="calibre_link-14" class="calibre6">Figure 4.7</a>:</p>
<pre id="calibre_link-60" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> market_date, </code>
   
<code class="calibre16"> customer_id, </code>
   
<code class="calibre16"> vendor_id, </code>
   
<code class="calibre16"> ROUND(quantity * cost_to_customer_per_qty, 2) AS price,</code>
   
<code class="calibre16"> CASE </code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 5.00</code>
          
<code class="calibre16"> THEN 0</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 10.00</code>
          
<code class="calibre16"> THEN 5</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty &lt; 20.00 </code>
          
<code class="calibre16"> THEN 10</code>
      
<code class="calibre16"> WHEN quantity * cost_to_customer_per_qty&gt;= 20.00 </code>
          
<code class="calibre16"> THEN 20</code>
  
<code class="calibre16"> END AS price:bin_lower_end</code>
<code class="calibre16">FROM farmers_market.customer_purchases</code>
<code class="calibre16">LIMIT 10</code>
</pre>
<figure class="calibre17">
<img alt="A table records market date, customer id, vendor id, price, and price bin lower end." class="center" src="images/000005.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-14" id="calibre_link-13" role="doc-backlink" class="calibre6">Figure 4.7</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-61" class="calibre8">One of these queries generates a new column of strings, and one generates a new column of numbers. You might actually want to include both columns in <span aria-label="56" type="pagebreak" id="calibre_link-62" role="doc-pagebreak" class="calibre13"></span>your query if you were building it to be used in a report, because the 
<code class="calibre16">price:bin</code>
 column is a more explanatory label for the bin, but will sort alphabetically instead of in bin value order. With both available to use in your report, you could use the numeric version of the column to sort the bins correctly, and the string version to label the bins.</p>
<p id="calibre_link-63" class="calibre8">Remember that because neither of the preceding queries included an 
<code class="calibre16">ELSE</code>
 inside the 
<code class="calibre16">CASE</code>
 statement, the output will be NULL if the quantity field is blank or the calculation can't be completed with the available values for whatever reason.</p>
<p id="calibre_link-64" class="calibre8">If there is a mis-entered price, or perhaps a record of a refund, and the value in the price column turns out to be negative in one row, what do you think will happen? In the preceding queries, the first condition is �less than 5,� so negative values will end up in the �Under $5,� or 0, bin. Therefore, the name 
<code class="calibre16">price:bin_lower_end</code>
 is a misnomer, since 0 might not actually represent the lowest value possible in the first bin. It's important when writing CASE statements for analytical purposes to determine what the result will be if there end up being unexpected values in any of the referenced database fields.</p>
</section>

    <section aria-labelledby="head-2-43" class="calibre2"><span id="calibre_link-65" class="calibre"></span>
<h2 id="calibre_link-66" class="calibre14">Categorical Encoding Using CASE</h2>
<p id="calibre_link-67" class="calibre8">When developing datasets for machine learning, you will often need to �encode� categorical string variables as numeric variables, in order for a mathematical algorithm to be able to use them as input.</p>
<p class="calibre8">If the categories represent something that can be sorted in a rank order, it might make sense to convert the string variables into numeric values that represent that rank order. For example, the vendor booths at the farmer's market are rented out at different costs, depending on their size and proximity to the entrance. These booth price levels are labeled with the letters �A,� �B,� and �C,� in order by increasing price, which could be converted into either numeric values 1, 2, 3 or the actual booth prices. The following 
<code class="calibre16">CASE</code>
 statement converts the booth price levels into numeric values, and the results are shown in <a href="#calibre_link-15" id="calibre_link-16" class="calibre6">Figure 4.8</a>:</p>
<pre id="calibre_link-68" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> booth_number,</code>
   
<code class="calibre16"> booth_price:level,</code>
   
<code class="calibre16"> CASE </code>
       
<code class="calibre16"> WHEN booth_price:level = 'A' THEN 1</code>
       
<code class="calibre16"> WHEN booth_price:level = 'B' THEN 2</code>
       
<code class="calibre16"> WHEN booth_price:level = 'C' THEN 3</code>
   
<code class="calibre16"> END AS booth_price:level_numeric</code>
<code class="calibre16">FROM farmers_market.booth</code>
<code class="calibre16">LIMIT 5</code>
<span aria-label="57" type="pagebreak" id="calibre_link-69" role="doc-pagebreak" class="calibre13"></span></pre>
<figure class="calibre17">
<img alt="A table records booth number, booth price level, booth price level numeric." class="center" src="images/000006.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-16" id="calibre_link-15" role="doc-backlink" class="calibre6">Figure 4.8</a></span></p>
</figcaption>
</figure>
<p class="calibre8">If the categories aren't necessarily in any kind of rank order, like our vendor type categories, we might use a method called �one-hot encoding.� This helps us avoid inadvertently indicating a sort order when none exists. One-hot encoding means that we create a new column representing each category, assigning it a binary value of 1 if a row falls into that category, and a 0 otherwise. These columns are sometimes called �dummy variables.� The following CASE statement one-hot encodes our vendor type categories, and the results are demonstrated in <a href="#calibre_link-17" id="calibre_link-18" class="calibre6">Figure 4.9</a>:</p>
<pre id="calibre_link-70" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> vendor_id,</code>
   
<code class="calibre16"> vendor_name,</code>
   
<code class="calibre16"> vendor_type,</code>
   
<code class="calibre16"> CASE WHEN vendor_type = 'Arts &amp; Jewelry' </code>
       
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0 </code>
   
<code class="calibre16"> END AS vendor_type_arts_jewelry,</code>
   
<code class="calibre16"> CASE WHEN vendor_type = 'Eggs &amp; Meats' </code>
       
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0 </code>
   
<code class="calibre16"> END AS vendor_type_eggs_meats,</code>
   
<code class="calibre16"> CASE WHEN vendor_type = 'Fresh Focused' </code>
       
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0 </code>
   
<code class="calibre16"> END AS vendor_type_fresh_focused, </code>
   
<code class="calibre16"> CASE WHEN vendor_type = 'Fresh Variety: Veggies &amp; More' </code>
       
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0 </code>
   
<code class="calibre16"> END AS vendor_type_fresh_variety,</code>
   
<code class="calibre16"> CASE WHEN vendor_type = 'Prepared Foods' </code>
       
<code class="calibre16"> THEN 1</code>
       
<code class="calibre16"> ELSE 0 </code>
   
<code class="calibre16"> END AS vendor_type_prepared</code>
<code class="calibre16">FROM farmers_market.vendor</code>
<span aria-label="58" type="pagebreak" id="calibre_link-71" role="doc-pagebreak" class="calibre13"></span><span aria-label="59" type="pagebreak" id="calibre_link-72" role="doc-pagebreak" class="calibre13"></span></pre>
<figure class="calibre17">
<img alt="A table records encoding vendor type categories." class="center" src="images/000007.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-18" id="calibre_link-17" role="doc-backlink" class="calibre6">Figure 4.9</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-73" class="calibre8">A situation to be aware of when manually encoding one-hot categorical variables this way is that if a new category is added (a new type of vendor in this case), there will be no column in your dataset for the new vendor type until you add another CASE statement.</p>
</section>

    <section aria-labelledby="head-2-44" class="calibre2"><span id="calibre_link-74" class="calibre"></span>
<h2 id="calibre_link-75" class="calibre14">CASE Statement Summary</h2>
<p id="calibre_link-76" class="calibre8">In this chapter, you learned SQL CASE statement syntax for creating new columns with values based on conditions. You also learned how to consolidate categorical values into fewer categories, create binary flags, bin continuous values, and encode categorical values.</p>
<p class="calibre8">You should now be able to describe what the following two queries do. The results for the first query are displayed in <a href="#calibre_link-19" id="calibre_link-20" class="calibre6">Figure 4.10</a>:</p>
<pre id="calibre_link-77" class="calibre15">
<code class="calibre16">SELECT </code>
   
<code class="calibre16"> customer_id,</code>
   
<code class="calibre16"> CASE </code>
       
<code class="calibre16"> WHEN customer_zip = '22801' THEN 'Local'</code>
       
<code class="calibre16"> ELSE 'Not Local' </code>
   
<code class="calibre16"> END customer_location_type</code>
<code class="calibre16">FROM farmers_market.customer</code>
<code class="calibre16">LIMIT 10</code>
</pre>
<figure class="calibre17">
<img alt="A table records customer id and customer location type." class="center" src="images/000008.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-20" id="calibre_link-19" role="doc-backlink" class="calibre6">Figure 4.10</a></span></p>
</figcaption>
</figure>
<p class="calibre8">The results for the following query are displayed in <a href="#calibre_link-21" id="calibre_link-22" class="calibre6">Figure 4.11</a>:</p>
<pre id="calibre_link-78" class="calibre15">
<code class="calibre16">SELECT </code>
    
<code class="calibre16"> booth_number,</code>
    
<code class="calibre16"> CASE WHEN booth_price:level = 'A' </code>
        
<code class="calibre16"> THEN 1 </code>
        
<code class="calibre16"> ELSE 0 </code>
    
<code class="calibre16"> END booth_price:level_A,</code>
    
<code class="calibre16"> CASE WHEN booth_price:level = 'B' </code>
        
<code class="calibre16"> THEN 1 </code>
        
<code class="calibre16"> ELSE 0 </code>
    <span aria-label="60" type="pagebreak" id="calibre_link-79" role="doc-pagebreak" class="calibre13"></span>
<code class="calibre16"> END booth_price:level_B,</code>
    
<code class="calibre16"> CASE WHEN booth_price:level = 'C' </code>
        
<code class="calibre16"> THEN 1 </code>
        
<code class="calibre16"> ELSE 0 </code>
    
<code class="calibre16"> END booth_price:level_C</code>
<code class="calibre16">FROM farmers_market.booth</code>
<code class="calibre16">LIMIT 5</code>
</pre>
<figure class="calibre17">
<img alt="A table records booth number, booth price level A, B, and C." class="center" src="images/000009.png" />
<figcaption class="calibre18">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-22" id="calibre_link-21" role="doc-backlink" class="calibre6">Figure 4.11</a></span></p>
</figcaption>
</figure>
</section>

    <section aria-labelledby="head-2-45" class="calibre2"><span id="calibre_link-80" class="calibre"></span>
<h2 id="calibre_link-81" class="calibre14">Exercises Using the Included Database</h2>
<p class="calibre8">Look back at <a href="c02.xhtml#c02-fig-0001" class="calibre6">Figure 2.1</a> in <a href="c02.xhtml" class="calibre6">Chapter 2</a> for sample data and column names for the 
<code class="calibre16">product</code>
 table referenced in these exercises.</p>
<ol class="decimal" id="calibre_link-82">
<li id="calibre_link-83" class="calibre19">Products can be sold by the individual unit or by bulk measures like lbs. or oz. Write a query that outputs the 
<code class="calibre16">product_id</code>
 and 
<code class="calibre16">product_name</code>
 columns from the 
<code class="calibre16">product</code>
 table, and add a column called 
<code class="calibre16">prod_qty_type_condensed</code>
 that displays the word �unit� if the 
<code class="calibre16">product_qty_type</code>
 is �unit,� and otherwise displays the word �bulk.�</li>
<li id="calibre_link-84" class="calibre19">We want to flag all of the different types of pepper products that are sold at the market. Add a column to the previous query called 
<code class="calibre16">pepper_flag</code>
 that outputs a 1 if the 
<code class="calibre16">product_name</code>
 contains the word �pepper� (regardless of capitalization), and otherwise outputs 0.</li>
<li id="calibre_link-85" class="calibre19">Can you think of a situation when a pepper product might not get flagged as a pepper product using the code from the previous exercise?</li>
</ol>
</section>

  </section>

</div>