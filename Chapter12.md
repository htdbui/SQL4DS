---
title: "SQL Chapter 12"
author: "db"
---






















<br><br><br><br><br><br><br><br><br><br><br><br>
<html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><link href="style.css" rel="stylesheet" type="text/css" /><title>SQL for Data Scientists: A Beginner's Guide for Building Datasets for Analysis</title></head><body><div type="bodymatter" class="calibre1" id="calibre_link-0">
<section aria-labelledby="c12_1" type="chapter" role="doc-chapter" class="calibre2">
<header class="calibre3">
<h1 id="calibre_link-15" class="calibre4"><span aria-label="173" type="pagebreak" id="calibre_link-16" role="doc-pagebreak" class="calibre5"></span><a id="calibre_link-17" class="calibre6"></a><span class="calibre">CHAPTER 12</span><br class="calibre7" /><span class="calibre">Creating Machine Learning Datasets Using SQL</span></h1>
</header>
<section aria-label="chapter opening" class="calibre2"><span id="calibre_link-18" class="calibre"></span>
<p id="calibre_link-19" class="calibre8">In previous chapters, we introduced SQL concepts and walked through some analytical reporting examples, but we have not yet focused on the specifics of dataset design for predictive modeling applications. In this chapter, we'll discuss the development of datasets for two types of algorithms: classification and time series models.</p>
<p id="calibre_link-20" class="calibre8">A <i class="calibre9">binary classification model</i> predicts whether a record belongs to one category or another. For example, a heart disease classification model might analyze data from a patient's medical history to determine whether they're likely to develop heart disease, or not. A weather model could use past and current temperature, precipitation, pressure, and wind measurements, as well as those from surrounding geographic areas, to predict whether or not it will rain in the next 24 hours. In a retail scenario like a farmer's market, the seller may want to predict whether a customer will return to make another purchase within a certain time frame, or not.</p>
<p id="calibre_link-21" class="calibre8">In order to make predictions, the model needs to be trained. Binary classifiers are a type of <i class="calibre9">supervised learning</i> model, which means they are trained by passing example rows of data (also called instances, observations, or feature vectors) labeled with each of the possible outcomes into the algorithm, so it can detect patterns and identify characteristics that are more strongly associated with one result or the other. Some example instances are set aside to test the trained model, feeding them into the algorithm to generate predictions we can compare to the known actual outcomes in order to check the model's performance and determine in what ways the model is incorrect, so we can make adjustments to it.</p>
<p id="calibre_link-22" class="calibre8"><span aria-label="174" type="pagebreak" id="calibre_link-23" role="doc-pagebreak" class="calibre10"></span>A <i class="calibre9">time series model</i> performs statistical operations on a series of measurements over time to forecast what the measurement might be at some point in time in the future. The training data for a time series model is a running log of data measurements from past points in time. Someone could use an hourly history of a stock's prices to attempt to predict the value of an investment at the end of the day. A college might use historical counts of applications, admission offers, enrolled students, and deposits paid per week to generate a weekly forecast of incoming freshman class enrollment. A farmer's market may use purchases over time to detect seasonal product sales trends and growth in the customer base, or try to predict how many ears of corn will sell per week next month.</p>
<p id="calibre_link-24" class="calibre8">Each type of model requires a different type of dataset, and we'll review some approaches for preparing datasets for these two common types of models using SQL.</p>
</section>
<section aria-labelledby="head-2-83" class="calibre2"><span id="calibre_link-25" class="calibre"></span>
<h2 id="calibre_link-26" class="calibre11">Datasets for Time Series Models</h2>
<p id="calibre_link-27" class="calibre8">The simplest type of time series forecasting uses a single variable measured over specified time intervals to predict the value of that same variable at a future point in time. A dataset for training a simple time series model can consist of just two columns: a column with dates or datetime values that indicate the time of measurement, and a column with the value being measured.</p>
<p id="calibre_link-28" class="calibre8">For example, a model to predict the high temperature in a location tomorrow could have a dataset with years’ worth of daily high temperatures measured at that location. The dataset would have one row per day, with one column for the date, and another for the daily high temperature measured. A time series algorithm could detect seasonal temperature patterns, long-term trends, and the most recent daily high temperatures to predict what the high temperature might be tomorrow.</p>
<p id="calibre_link-29" class="calibre8">Let's create a dataset that allows us to plot a time series of farmer's market sales per week. Note that this will be a simplistic view of sales, because it will not take into consideration changes in vendors over time, available inventory at different times of year, or external economic factors.</p>
<p id="calibre_link-30" class="calibre8">In <a href="c10.xhtml" class="calibre6">Chapter 10</a>, “Building SQL Datasets for Analytical Reporting,” we created a dataset that summarized sales per market date. Here, we'll further summarize that data to a weekly level. Because we have joined the 
<code class="calibre12">customer_purchases</code>
 table to the 
<code class="calibre12">market_date_info</code>
 table and there is a 
<code class="calibre12">market_week</code>
 field available, you may assume that we want to group by that field. However, remember that 
<code class="calibre12">market_week</code>
 is a number that represents the week of the year, and every year has week numbers 1 through 52. So, if you group by 
<code class="calibre12">market_week</code>
 only, you will be adding sales from the same calendar weeks from different years together! Therefore, we will need to 
<code class="calibre12">GROUP BY</code>
 both 
<code class="calibre12">market_year</code>
 and 
<code class="calibre12">market_week</code>
. However, <span aria-label="175" type="pagebreak" id="calibre_link-31" role="doc-pagebreak" class="calibre10"></span>we want only a single field indicating the time period in our dataset, so we don't want to output the 
<code class="calibre12">market_year</code>
 and 
<code class="calibre12">market_week</code>
 fields in the results.</p>
<p class="calibre8">Because so many time series algorithms are designed to use calendar dates to indicate when an event occurred or a measurement was taken, we're going to use the first market date of each week to label the weekly intervals. So, we'll find the minimum market date per week and output that, with the column alias 
<code class="calibre12">first_market_date_of_week</code>
:</p>
<pre id="calibre_link-32" class="calibre13">
<code class="calibre12">SELECT </code>
   
<code class="calibre12"> MIN(cp.market_date) AS first_market_date_of_week,</code>
   
<code class="calibre12"> ROUND(SUM(cp.quantity * cp.cost_to_customer_per_qty),2) AS weekly_sales</code>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
   
<code class="calibre12"> LEFT JOIN farmers_market.market_date_info AS md</code>
      
<code class="calibre12"> ON cp.market_date = md.market_date</code>
<code class="calibre12">GROUP BY md.market_year, md.market_week</code>
<code class="calibre12">ORDER BY md.market_year, md.market_week</code>
</pre>
<p id="calibre_link-33" class="calibre8"><a href="#calibre_link-1" id="calibre_link-2" class="calibre6">Figure 12.1</a> shows the last 13 rows of data generated by this query.</p>
<figure class="calibre14">
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-2" id="calibre_link-1" role="doc-backlink" class="calibre6">Figure 12.1</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-34" class="calibre8">Tableau has some built-in time series forecasting functions, including one that uses a method called exponential smoothing. We'll import the results of our query into Tableau and have it use the weekly sales data to generate a sales forecast for the eight weeks beyond the last date in our dataset.</p>
<p id="calibre_link-35" class="calibre8">The line chart in <a href="#calibre_link-3" id="calibre_link-4" class="calibre6">Figure 12.2</a> shows a visualization of the numbers generated by the preceding query, in the lighter gray color labeled “Actual” in the legend. These are the actual sales per week from our query. The forecasted sales for the next eight weeks are plotted in a darker gray color. These “Estimate” values are surrounded by a shaded area that represents a 90% confidence interval for each forecasted value.<span aria-label="176" type="pagebreak" id="calibre_link-36" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="Graph plots time series forecast of weekly Farmer&apos;s market sales." class="center" src="images/000000.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-4" id="calibre_link-3" role="doc-backlink" class="calibre6">Figure 12.2</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-37" class="calibre8">By summarizing past sales into a dataset that has one row per week, with a date column and a weekly sales total column, we enabled Tableau's forecast to identify patterns in past weekly sales and use those to forecast future weekly sales. However, we didn't provide it with enough granularity to forecast daily sales. There is no information in our dataset indicating that each week in our dataset actually represents sales from two market dates (as opposed to, say, seven different days of sales summarized by week). And if we asked Tableau to forecast monthly sales using this dataset, it only has the date of the first market per week, so if a new month started between two market dates we grouped into a single week, the second market's sales will be categorized into the wrong month, creating erroneous training data for the forecasting algorithm.</p>
<p id="calibre_link-38" class="calibre8">It's important to know the intended use of a dataset in order to make the correct choices when designing it. It's also important to provide documentation to accompany a dataset, so others who might use it later are aware of its original purpose, and any caveats that might be important to know when using it for analysis.</p>
</section>
<section aria-labelledby="head-2-84" class="calibre2"><span id="calibre_link-39" class="calibre"></span>
<h2 id="calibre_link-40" class="calibre11">Datasets for Binary Classification</h2>
<p id="calibre_link-41" class="calibre8">Classification algorithms detect patterns in training datasets, which contain example records from the past that fall into known categories, and use those detected patterns to then categorize new data. Binary classification algorithms categorize inputs into one of two outcomes, which are determined according to the purpose of the model and the available data. For example: Given the medical <span aria-label="177" type="pagebreak" id="calibre_link-42" role="doc-pagebreak" class="calibre10"></span>history of a patient, is it likely that they have heart disease, or not? Given the current and summarized past weather data, is it likely to rain within the next 24 hours, or not? Many of these algorithms can also output a probability or likelihood score, so you can also determine how “sure” the algorithm is that the instance should be classified into one category or the other (how well it matches the patterns detected for training examples in either class).</p>
<p id="calibre_link-43" class="calibre8">These algorithms need training data that is in the same form as the data to be classified. For example, if you want the algorithm to take a patient's medical history, including current vital measurements, as input, then the training data has to be at the same granularity and level of summary. Based on the dataset it was trained on, your classification model may expect one row of data per patient, with input fields such as a patient's age, sex, cholesterol as of five years ago, cholesterol as of one year ago, cholesterol measured today (the day of diagnosis), number of years that the patient has smoked cigarettes (as of the day of diagnosis), resting blood pressure as of five years ago, resting blood pressure as of one year ago, resting blood pressure measured today, resting ECG results, chest pain level indicator, and other summary metrics. In this case, each training “instance” (row of data, or vector) should have the data as of the diagnosis date, as well as the measurements or cholesterol and blood pressure from one and five years prior to the diagnosis date, so the duration between those two data points in the training data is as similar as possible to the duration between those two data points in the data you are passing through the trained model to be classified. The conditions under which the model will be applied need to be considered when designing the dataset.</p>
<p id="calibre_link-44" class="calibre8">Say you trained your model on data from a study that was structured that way, collecting data over a five-year period, but it's unlikely that you will have a five-year history of those measurements for current patients you want this algorithm to make a prediction for. You might not want to include the data from five years ago in the training dataset, since the model might not be good at classifying records with NULL values in the “resting blood pressure as of five years ago” field and other fields requiring past data, if all of the training instances included those values. Alternatively, training a model on data collected five years ago with outcomes as of the current year would be ideal if the thing you're trying to predict is whether a patient will develop heart disease five years from now.</p>
<p id="calibre_link-45" class="calibre8">Every classification model requires a <i class="calibre9">target variable</i>, which is the thing you're trying to predict. In binary classifiers, you can usually input the target as a binary value of 1 or 0, with each number representing each outcome. The target variable in the preceding example is a binary flag indicating a diagnosis of heart disease (or no heart disease) as of the date of the latest cholesterol and blood pressure measurements.</p>
<p id="calibre_link-46" class="calibre8"><span aria-label="178" type="pagebreak" id="calibre_link-47" role="doc-pagebreak" class="calibre10"></span>We won't get into the details of how much past data with known outcomes is required for training and testing a model here, as it depends on the type of model, the number of columns, the variation in the data values, and many other factors beyond the scope of this book. We also won't be training classification models in this book. However, we will discuss how to structure the datasets needed for binary classification model training and prediction, and how to use SQL to pull the data you need to train and run a classifier.</p>
<p id="calibre_link-48" class="calibre8">When thinking about how to structure a dataset for binary classification, the first thing you'll need to determine is the target variable, meaning the categories you're building a model to classify records into. Often, the target variable needs a time bounding, so instead of predicting “Will this patient develop heart disease?” the outcome being predicted could be “Will this patient be diagnosed with heart disease in the next five years?” The time bounding will affect choices you make when designing the training dataset.</p>
<section class="calibre2"><span id="calibre_link-49" class="calibre"></span>
<h3 id="calibre_link-50" class="calibre16">Creating the Dataset</h3>
<p id="calibre_link-51" class="calibre8">To have a concrete example to discuss, we'll build a dataset that could be used to train a model that can predict an answer to the question “Will this customer who just made a purchase return to make another purchase within the next month?” The binary target variable is “makes another purchase within 30 days,” with the values 1 for “yes” and 0 for “no.” With this time-limited target variable, we can create a training dataset that summarizes information about each customer as of the date of each purchase, and flag that row with a 1 if that customer made another purchase within a month, and 0 if they did not. Now, instead of having one training example per customer, we have one training example per customer per purchase date.</p>
<p id="calibre_link-52" class="calibre8">The benefit of having multiple records per customer is that a lot more training instances are available for the model to use to detect patterns in the data. Additionally, a person's behavior can change over time, so having a snapshot record of their summary activity every time they make a purchase, and a record of whether they came back within a month of making that purchase, can help the algorithm determine what impact certain activities may have on behavior. One effect of this approach to be aware of is that frequent customers will be over-represented in the dataset, which could have different impacts depending on the model, and could lead to overfitting the model to that type of customer. (Because a one-time customer will only be in the training dataset one time, while a frequent customer will be in the training dataset many times.)</p>
<p id="calibre_link-53" class="calibre8">Setting up your query to produce a dataset that is at the correct granularity with a target variable that is time-bound can be the most complicated step of the query design process, and it's worth taking the time to make sure it is calculated correctly before pulling in any other data fields. When I first designed <span aria-label="179" type="pagebreak" id="calibre_link-54" role="doc-pagebreak" class="calibre10"></span>this example and the dataset to pair with it, I set it up to determine whether each customer makes a purchase in the next calendar month (for example: if the purchase record is from April, will the customer make a purchase in May?). That is one way to approach this model that would be perfectly valid, if that's the type of prediction you wanted to make. However, I quickly realized that the time duration for the target variable wouldn't be consistent. Depending on when in April the customer made a purchase, the target variable could be 1 (yes) if the next purchase date was one day after the initial purchase, up to almost two months later if the initial purchase was on April 1 and the second purchase was on May 30. So, I decided to instead put a dynamic one-month time limit after each purchase for determining the returning customer flag value. So in the following queries, the target variable 
<code class="calibre12">purchased_again_within_30_days</code>
 represents whether the customer returned within 30 days of making a purchase, without considering the calendar month.</p>
<p id="calibre_link-55" class="calibre8">With this approach, we can start by creating a CTE that has a row for every purchase, like the query we developed in <a href="c11.xhtml" class="calibre6">Chapter 11</a>, “More Advanced Query Structures,” that was associated with <a href="c11.xhtml#c11-fig-0005" id="calibre_link-56" class="calibre6">Figure 11.5</a>. We can reuse the 
<code class="calibre12">customer_markets_attended</code>
 CTE in the following query to look up whether a customer made another purchase within 30 days after the date of each purchase.</p>
<p class="calibre8">Before we look at the calculated columns in the following query, let's look at its 
<code class="calibre12">FROM</code>
, 
<code class="calibre12">WHERE</code>
, and 
<code class="calibre12">GROUP BY</code>
 clauses, which determine which table(s) we're selecting from, how they're filtered, and the granularity of the final result. You can see that we're selecting from the 
<code class="calibre12">customer_purchases</code>
 table, which has one row per customer per product purchased. There is no 
<code class="calibre12">WHERE</code>
 clause, so we're returning all rows. The 
<code class="calibre12">GROUP BY</code>
 clause includes the 
<code class="calibre12">customer_id</code>
 and 
<code class="calibre12">market_date</code>
 from the 
<code class="calibre12">customer_purchases</code>
 table, so we'll end up with one row per customer per market date at which they made a purchase:</p>
<pre id="calibre_link-57" class="calibre13">
<code class="calibre12">WITH</code>
<code class="calibre12">customer_markets_attended AS</code>
<code class="calibre12">(</code>
   
<code class="calibre12"> SELECT DISTINCT</code>
       
<code class="calibre12"> customer_id,</code>
       
<code class="calibre12"> market_date</code>
   
<code class="calibre12"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre12"> ORDER BY customer_id, market_date</code>
<code class="calibre12">)</code>
   
<code class="calibre12"> </code>
<code class="calibre12">SELECT </code>
   
<code class="calibre12"> cp.market_date,</code>
   
<code class="calibre12"> cp.customer_id,</code>
   
<code class="calibre12"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS purchase_total,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.vendor_id) AS vendors_patronized,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.product_id) AS different_products_purchased,</code>
   
<code class="calibre12"> (SELECT MIN(cma.market_date) </code>
   
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
   <span aria-label="180" type="pagebreak" id="calibre_link-58" role="doc-pagebreak" class="calibre10"></span>
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
       
<code class="calibre12"> AND cma.market_date&gt; cp.market_date</code>
   
<code class="calibre12"> GROUP BY cma.customer_id) AS customer_next_market_date,</code>
   
<code class="calibre12"> DATEDIFF(</code>
       
<code class="calibre12"> (SELECT MIN(cma2.market_date) </code>
       
<code class="calibre12"> FROM customer_markets_attended AS cma2</code>
       
<code class="calibre12"> WHERE cma2.customer_id = cp.customer_id </code>
           
<code class="calibre12"> AND cma2.market_date&gt; cp.market_date</code>
       
<code class="calibre12"> GROUP BY cma2.customer_id),</code>
   
<code class="calibre12"> cp.market_date) AS days_until_customer_next_market_date,</code>
   
<code class="calibre12"> CASE WHEN</code>
       
<code class="calibre12"> DATEDIFF(</code>
            
<code class="calibre12"> (SELECT MIN(cma3.market_date) </code>
            
<code class="calibre12"> FROM customer_markets_attended AS cma3</code>
            
<code class="calibre12"> WHERE cma3.customer_id = cp.customer_id </code>
                
<code class="calibre12"> AND cma3.market_date&gt; cp.market_date</code>
            
<code class="calibre12"> GROUP BY cma3.customer_id),</code>
       
<code class="calibre12"> cp.market_date) &lt;=30 </code>
   
<code class="calibre12"> THEN 1 </code>
   
<code class="calibre12"> ELSE 0 END AS purchased_again_within_30_days</code>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
<code class="calibre12">GROUP BY cp.customer_id, cp.market_date</code>
<code class="calibre12">ORDER BY cp.customer_id, cp.market_date</code>
</pre>
<p id="calibre_link-59" class="calibre8">The 
<code class="calibre12">purchase_total</code>
 column should be familiar by now, multiplying the quantity and cost of each item purchased and summing that up to get the total spent by each customer at each market date. The 
<code class="calibre12">vendors_patronized</code>
 column is a distinct count of how many different vendors the customer made purchases from that day, and the 
<code class="calibre12">different_products_purchased</code>
 column is a distinct count of how many different kinds of products the customer purchased. These calculated columns are also called <i class="calibre9">engineered features</i> when you're talking about datasets for machine learning, and we're including them so we can explore the relationship between these values and the target variable. Maybe the more vendors the customer makes purchases from, the more likely they are to purchase an item they like so much that they'll return within 30 days to buy more. Including this column in the dataset enables the exploration of the relationship between this feature and the target variable.</p>
<p id="calibre_link-60" class="calibre8">The next column, 
<code class="calibre12">customer_next_market_date</code>
, is generated by a subquery that references our CTE. Look at all of the code inside parentheses after 
<code class="calibre12">different_products_purchased</code>
, and before 
<code class="calibre12">customer_next_market_date</code>
. This subquery selects the minimum market date a customer attended, which occurs after the current row's 
<code class="calibre12">market_date</code>
 value. In other words, we're finding the date of this customer's next purchase. In the 
<code class="calibre12">WHERE</code>
 clause of this subquery, we're matching up the subquery's 
<code class="calibre12">customer_id</code>
 with the main query's 
<code class="calibre12">customer_id</code>
 (to ensure we're looking at a single customer's trips to the market). We're also limiting the subquery rows to those where the market date occurs <span aria-label="181" type="pagebreak" id="calibre_link-61" role="doc-pagebreak" class="calibre10"></span>after the main query's market date, with the 
<code class="calibre12">cma.market_date &gt; cp.market_date</code>
 filter. In effect, we're pulling a list of all of this customer's future market dates, and then only returning the minimum date from that list, and aliasing it 
<code class="calibre12">customer_next_market_date</code>
.</p>
<p id="calibre_link-62" class="calibre8">The next column, aliased 
<code class="calibre12">days_until_customer_next_market_date</code>
, uses the same subquery, but this time calculates the difference between the current row's date and that calculated next market date. Then the last column, aliased 
<code class="calibre12">purchased_again_within_30_days</code>
, uses the same calculation and wraps it inside a 
<code class="calibre12">CASE</code>
 statement that returns a binary flag (1 or 0) value indicating whether that next purchase was made within 30 days. If you review these three calculated columns, you will see that the subqueries are all the same, and we're just performing different calculations on the result. Some example rows generated by this query are shown in <a href="#calibre_link-5" id="calibre_link-6" class="calibre6">Figure 12.3</a>.</p>
<figure class="calibre14">
<img alt="A table records market date and products purchased within 30 days." class="center" src="images/000001.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-6" id="calibre_link-5" role="doc-backlink" class="calibre6">Figure 12.3</a></span></p>
</figcaption>
</figure>
<p id="calibre_link-63" class="calibre8">I aliased the 
<code class="calibre12">customer_markets_attended</code>
 CTE reference differently in each of the subqueries, as 
<code class="calibre12">cma</code>
, 
<code class="calibre12">cma2</code>
, and 
<code class="calibre12">cma3</code>
 for clarity, though that isn't actually necessary. The table alias only applies within the subquery. Also note that each of these subqueries are aggregates that return only one value to be inserted into each row of the dataset.</p>
<p id="calibre_link-64" class="calibre8">The 
<code class="calibre12">customer_next_market_date</code>
 and 
<code class="calibre12">days_until_customer_next_market</code>
 are useful for validating the output of the preceding query, but once we have checked that the target variable 
<code class="calibre12">purchased_again_within_30_days</code>
 looks correct in context, we can remove those two columns from our machine learning dataset. All of the columns other than the target variable will be available as inputs to our algorithm, so we don't want to encode that value indicating whether or not they returned within 30 days in any other field, to avoid <i class="calibre9">data leakage</i>. Data leakage occurs when data that would not have been known as of the snapshot in time that the row represents is fed into the algorithm, artificially improving the predictions.</p>
</section>
<section class="calibre2"><span id="calibre_link-65" class="calibre"></span>
<h3 id="calibre_link-66" class="calibre16">Expanding the Feature Set</h3>
<p class="calibre8">What other features might we include that might contain “signal (detectable patterns)” for the model to detect? Maybe particular vendors have more loyal customers, or sell items that need to be replenished more frequently, so shopping at a particular vendor is an indicator that a shopper might return sooner. Or, the number of days since a customer's last purchase could be an indicator that they <span aria-label="182" type="pagebreak" id="calibre_link-67" role="doc-pagebreak" class="calibre10"></span>are a frequent shopper and have a higher likelihood to come back again soon. Let's add some columns that indicate which vendors each customer shopped at on each market day and flip the 
<code class="calibre12">days_until_customer_next_market_date</code>
 calculation to instead indicate how long it's been since the customer last shopped before the visit represented by the row:</p>
<pre id="calibre_link-68" class="calibre13">
<code class="calibre12">WITH</code>
<code class="calibre12">customer_markets_attended AS</code>
<code class="calibre12">(</code>
   
<code class="calibre12"> SELECT DISTINCT</code>
       
<code class="calibre12"> customer_id,</code>
       
<code class="calibre12"> market_date</code>
   
<code class="calibre12"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre12"> ORDER BY customer_id, market_date</code>
<code class="calibre12">)</code>
   
<code class="calibre12"> </code>
<code class="calibre12">SELECT cp.market_date,</code>
   
<code class="calibre12"> cp.customer_id,</code>
   
<code class="calibre12"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS purchase_total,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.vendor_id) AS vendors_patronized,</code>
   
<code class="calibre12"> MAX(CASE WHEN cp.vendor_id = 7 THEN 1 ELSE 0 END) AS purchased_from_vendor_7,</code>
   
<code class="calibre12"> MAX(CASE WHEN cp.vendor_id = 8 THEN 1 ELSE 0 END) AS purchased_from_vendor_8,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.product_id) AS different_products_purchased,</code>
   
<code class="calibre12"> DATEDIFF(cp.market_date,</code>
        
<code class="calibre12"> (SELECT MAX(cma.market_date) </code>
        
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
        
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
            
<code class="calibre12"> AND cma.market_date &lt; cp.market_date</code>
        
<code class="calibre12"> GROUP BY cma.customer_id)) AS days_since:last_customer_market_date,</code>
        
<code class="calibre12"> CASE WHEN</code>
        
<code class="calibre12"> DATEDIFF(</code>
             
<code class="calibre12"> (SELECT MIN(cma.market_date) </code>
             
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
             
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
                 
<code class="calibre12"> AND cma.market_date&gt; cp.market_date</code>
             
<code class="calibre12"> GROUP BY cma.customer_id),</code>
             
<code class="calibre12"> cp.market_date) &lt;=30 THEN 1 ELSE 0 END AS purchased_again_within_30_days</code>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
<code class="calibre12">GROUP BY cp.customer_id, cp.market_date</code>
<code class="calibre12">ORDER BY cp.customer_id, cp.market_date</code>
</pre>
<p id="calibre_link-69" class="calibre8">You can see in the preceding query that we added a couple demonstration columns indicating whether each customer purchased from vendors 7 or 8. This technique could be repeated to create a column for every vendor. We also flipped the greater-than sign in the date comparison to a less-than sign, to find <span aria-label="183" type="pagebreak" id="calibre_link-70" role="doc-pagebreak" class="calibre10"></span>how many days it had been since the customer last made a purchase, in the feature aliased 
<code class="calibre12">days_since:last_customer_market_date</code>
.</p>
<p id="calibre_link-71" class="calibre8">Another type of aggregate value that might be useful to input into a predictive model is some type of representation of the customer's entire history of farmer's market shopping, up to the date the row represents. For example, how many times has the customer shopped at the farmer's market before? A long-time shopper might be more likely to return than a brand-new shopper.</p>
<p id="calibre_link-72" class="calibre8">The 
<code class="calibre12">ROW_NUMBER</code>
 window function is one way to calculate this value, counting how many prior rows exist for each customer, but you have to be careful where you put it in the query, because 
<code class="calibre12">ROW_NUMBER</code>
 only counts the rows returned by the query. So, if we wanted to count how many times the customer has shopped at the market before as of the market date in the current row, but our main query is filtered to only return data from the year 2019, then our 
<code class="calibre12">ROW_NUMBER</code>
 window function will only be able to count previous purchases in 2019 and not the customer's entire shopping history.</p>
<p id="calibre_link-73" class="calibre8">One solution for our use case is to put the 
<code class="calibre12">ROW_NUMBER</code>
 function in the 
<code class="calibre12">customer_markets_attended</code>
 CTE, which doesn't need to be filtered by date, even if you wanted to filter the final output, so we can calculate the number of past markets attended using a similar approach we used to determine the previous purchase date, referencing that CTE. This time, instead of returning the maximum market date that's less than the 
<code class="calibre12">market_date</code>
 in the row, we'll return the row number that's associated with the current 
<code class="calibre12">market_date</code>
 from that same query. In the following query, this value has the alias 
<code class="calibre12">market_count</code>
 in the CTE and is summarized as 
<code class="calibre12">customer_markets_attended_count</code>
 in the main query.</p>
<p id="calibre_link-74" class="calibre8">One important note is that when we add the 
<code class="calibre12">ROW_NUMBER</code>
 to the 
<code class="calibre12">customer_markets_attended</code>
 query in the 
<code class="calibre12">WITH</code>
 clause, we have to modify it to use a 
<code class="calibre12">GROUP BY</code>
 instead of a 
<code class="calibre12">COUNT DISTINCT</code>
 to summarize per 
<code class="calibre12">customer_id</code>
 and 
<code class="calibre12">market_date</code>
. The first eight rows of the output of the 
<code class="calibre12">COUNT DISTINCT</code>
 approach are shown in <a href="#calibre_link-7" id="calibre_link-9" class="calibre6">Figure 12.4</a>, and the first eight rows of the output of the 
<code class="calibre12">GROUP BY</code>
 approach are shown in <a href="#calibre_link-8" id="calibre_link-10" class="calibre6">Figure 12.5</a>. The reason the 
<code class="calibre12">ROW_NUMBER</code>
 returns much higher counts when the query is summarized using 
<code class="calibre12">COUNT DISTINCT</code>
 is that the window function is calculated on the dataset before the 
<code class="calibre12">DISTINCT</code>
, so since the results aren't grouped, it's returning one row per customer per product purchased, not one row per customer per 
<code class="calibre12">market_date</code>
, then numbering every row in the 
<code class="calibre12">customer_purchases</code>
 table for that customer. Grouping by 
<code class="calibre12">market_date</code>
 solves that issue because the window function is calculated after the data is aggregated by the 
<code class="calibre12">GROUP BY</code>
. Sometimes it takes trial and error to get the order of operations correct, and this is why it's important to view the details of the underlying data prior to aggregating, so you know whether the resulting summary data is correct.<span aria-label="184" type="pagebreak" id="calibre_link-75" role="doc-pagebreak" class="calibre10"></span></p>
<figure class="calibre14">
<img alt="A table records customer id, market date, and market count." class="center" src="images/000002.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-9" id="calibre_link-7" role="doc-backlink" class="calibre6">Figure 12.4</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records customer id, market date, and market count." class="center" src="images/000003.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-10" id="calibre_link-8" role="doc-backlink" class="calibre6">Figure 12.5</a></span></p>
</figcaption>
</figure>
<p class="calibre8">Like with most SQL queries, there are actually several ways to accomplish the same result. The following approach, which moves the 
<code class="calibre12">ROW_NUMBER()</code>
 into the 
<code class="calibre12">WHERE</code>
 clause, will return the same count for 
<code class="calibre12">customer_markets_attended_count</code>
 as a version that has it in the main query, even if the main query is filtered to a date range that doesn't include a customer's entire purchase history:</p>
<pre id="calibre_link-76" class="calibre13">
<code class="calibre12">WITH</code>
<code class="calibre12">customer_markets_attended AS</code>
<code class="calibre12">(</code>
   
<code class="calibre12"> SELECT </code>
       
<code class="calibre12"> customer_id,</code>
       
<code class="calibre12"> market_date,</code>
       
<code class="calibre12"> ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date) AS market_count</code>
   
<code class="calibre12"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre12"> GROUP BY customer_id, market_date</code>
   
<code class="calibre12"> ORDER BY customer_id, market_date</code>
<code class="calibre12">)</code>
<code class="calibre12">select cp.customer_id, cp.market_date, </code>
   
<code class="calibre12"> (SELECT MAX(market_count) </code>
       
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
       
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
       
<code class="calibre12"> AND cma.market_date &lt;= cp.market_date) AS customer_markets_attended_count</code>
       <span aria-label="185" type="pagebreak" id="calibre_link-77" role="doc-pagebreak" class="calibre10"></span>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
       
<code class="calibre12">GROUP BY cp.customer_id, cp.market_date</code>
       
<code class="calibre12">ORDER BY cp.customer_id, cp.market_date</code>
</pre>
<p class="calibre8">One feature we could add that is likely predictive of whether a customer returns in the next 30 days is how many times they shopped in the previous 30 days. So, we can create another column that puts a time range on the calculation for past markets attended and counts the market dates in the last 30 days. This is demonstrated in the following query by the calculation aliased 
<code class="calibre12">customer_markets_attended_30days_count</code>
. You might extrapolate from this calculation one of the aforementioned alternative ways of determining the total count of markets attended:</p>
<pre id="calibre_link-78" class="calibre13">
<code class="calibre12">WITH</code>
<code class="calibre12">customer_markets_attended AS</code>
<code class="calibre12">(</code>
   
<code class="calibre12"> SELECT </code>
       
<code class="calibre12"> customer_id,</code>
       
<code class="calibre12"> market_date,</code>
   
<code class="calibre12"> ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date) AS market_count</code>
   
<code class="calibre12"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre12"> GROUP BY customer_id, market_date </code>
   
<code class="calibre12"> ORDER BY customer_id, market_date</code>
<code class="calibre12">)</code>
<code class="calibre12">select cp.customer_id, cp.market_date, </code>
   
<code class="calibre12"> (SELECT COUNT(market_date) </code>
       
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
       
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
           
<code class="calibre12"> AND cma.market_date &lt; cp.market_date</code>
           
<code class="calibre12"> AND DATEDIFF(cp.market_date, cma.market_date) &lt;= 30) AS customer_markets_attended_30days_count</code>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
<code class="calibre12">GROUP BY cp.customer_id, cp.market_date</code>
<code class="calibre12">ORDER BY cp.customer_id, cp.market_date</code>
</pre>
</section>
<section class="calibre2"><span id="calibre_link-79" class="calibre"></span>
<h3 id="calibre_link-80" class="calibre16">Feature Engineering</h3>
<p id="calibre_link-81" class="calibre8">This process of creating different input values that might be helpful to the prediction algorithm is called <i class="calibre9">feature engineering</i>. Most binary classification algorithms require numeric inputs, so sometimes features are engineered to convert another data type into a numeric representation. Other types of features you might create include sets of one-hot encoded flag columns (as mentioned in <a href="c04.xhtml" class="calibre6">Chapter 4</a>, “CASE Statements”), converting categorical text columns to numeric values, aggregate <span aria-label="186" type="pagebreak" id="calibre_link-82" role="doc-pagebreak" class="calibre10"></span>high or low metrics (such as the maximum ever spent by the customer at a market to-date), other incrementing totals (such as the length of time the person has been a customer of the market), and other summaries for different time periods.</p>
<p id="calibre_link-83" class="calibre8">One important factor when engineering features is that each of these feature values is only what would be knowable as of the date represented by the row&mdash;the 
<code class="calibre12">market_date</code>
 in this case. We want to train the model on examples of customers with a variety of traits as of specific points in time that can be correlated with a specific outcome or target variable relative to that time. In fact, I've been outputting the 
<code class="calibre12">market_date</code>
 in each of the previous queries for verification purposes, but I wouldn't input the full date into a predictive model, because then the training data would all be tied to past dates, when only the relative dates for the events of interest (the time between purchases) are important. If we ran a current customer's record through the model to try to make a prediction based on data collected this week, but the model had been trained on full date values from the past, the model wouldn't know what to do with the 
<code class="calibre12">market_date</code>
, because there will have been no training examples with similar dates. So, when I train my classification model using this dataset, I will not input the 
<code class="calibre12">customer_id</code>
 or 
<code class="calibre12">market_date</code>
 into the algorithm. I will only use them as unique identifiers, or index values, so the predictions the model outputs can be tied back to their respective rows.</p>
<p class="calibre8">However, the month of the market date is likely predictive, because the market closes for the months of January and February. So, customers shopping in December would have a lower likelihood of returning in the next 30 days than customers in other months. The final version of this example classification dataset query will include a column representing the month, which can be seen in <a href="#calibre_link-11" id="calibre_link-13" class="calibre6">Figures 12.6</a> and <a href="#calibre_link-12" id="calibre_link-14" class="calibre6">12.7</a>. The number of columns is too wide to fit into one figure, so the output is split into two figures, with the 
<code class="calibre12">customer_id</code>
 and 
<code class="calibre12">market_date</code>
 index visible in both sections so you can find the continuation of each row in the second block:</p>
<pre id="calibre_link-84" class="calibre13">
<code class="calibre12">WITH</code>
<code class="calibre12">customer_markets_attended AS</code>
<code class="calibre12">(</code>
   
<code class="calibre12"> SELECT </code>
       
<code class="calibre12"> customer_id,</code>
       
<code class="calibre12"> market_date,</code>
       
<code class="calibre12"> ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date) AS market_count</code>
   
<code class="calibre12"> FROM farmers_market.customer_purchases</code>
   
<code class="calibre12"> GROUP BY customer_id, market_date </code>
   
<code class="calibre12"> ORDER BY customer_id, market_date</code>
<code class="calibre12">)</code>
   
<code class="calibre12"> </code>
<code class="calibre12">SELECT </code>
   
<code class="calibre12"> cp.customer_id,</code>
   
<code class="calibre12"> cp.market_date,</code>
   
<code class="calibre12"> EXTRACT(MONTH FROM cp.market_date) as market_month,</code>
   
<code class="calibre12"> SUM(cp.quantity * cp.cost_to_customer_per_qty) AS purchase_total,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.vendor_id) AS vendors_patronized,</code>
   
<code class="calibre12"> MAX(CASE WHEN cp.vendor_id = 7 THEN 1 ELSE 0 END) purchased_from_vendor_7,</code>
   
<code class="calibre12"> MAX(CASE WHEN cp.vendor_id = 8 THEN 1 ELSE 0 END) purchased_from_vendor_8,</code>
   
<code class="calibre12"> COUNT(DISTINCT cp.product_id) AS different_products_purchased,</code>
   
<code class="calibre12"> DATEDIFF(cp.market_date,</code>
     
<code class="calibre12"> (SELECT MAX(cma.market_date) </code>
         
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
         
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
             
<code class="calibre12"> AND cma.market_date &lt; cp.market_date</code>
         
<code class="calibre12"> GROUP BY cma.customer_id)</code>
     
<code class="calibre12"> ) days_since:last_customer_market_date,</code>
   
<code class="calibre12"> (SELECT MAX(market_count) </code>
       
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
       
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
           
<code class="calibre12"> AND cma.market_date &lt;= cp.market_date) AS customer_markets_attended_count,</code>
   
<code class="calibre12"> (SELECT COUNT(market_date) </code>
       
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
       
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
           
<code class="calibre12"> AND cma.market_date &lt; cp.market_date</code>
           
<code class="calibre12"> AND DATEDIFF(cp.market_date, cma.market_date) &lt;= 30) AS customer_markets_attended_30days_count,</code>
   
<code class="calibre12"> CASE WHEN</code>
       
<code class="calibre12"> DATEDIFF(</code>
            
<code class="calibre12"> (SELECT MIN(cma.market_date) </code>
                
<code class="calibre12"> FROM customer_markets_attended AS cma</code>
                
<code class="calibre12"> WHERE cma.customer_id = cp.customer_id </code>
                    
<code class="calibre12"> AND cma.market_date&gt; cp.market_date</code>
                
<code class="calibre12"> GROUP BY cma.customer_id),</code>
       
<code class="calibre12"> cp.market_date) &lt;=30 </code>
   
<code class="calibre12"> THEN 1 </code>
   
<code class="calibre12"> ELSE 0 </code>
   
<code class="calibre12"> END AS purchased_again_within_30_days</code>
<code class="calibre12">FROM farmers_market.customer_purchases AS cp</code>
<code class="calibre12">GROUP BY cp.customer_id, cp.market_date</code>
<code class="calibre12">ORDER BY cp.customer_id, cp.market_date</code>
<span aria-label="187" type="pagebreak" id="calibre_link-85" role="doc-pagebreak" class="calibre10"></span><span aria-label="188" type="pagebreak" id="calibre_link-86" role="doc-pagebreak" class="calibre10"></span><span aria-label="189" type="pagebreak" id="calibre_link-87" role="doc-pagebreak" class="calibre10"></span></pre>
<figure class="calibre14">
<img alt="A table records customer id, market date, and market count." class="center" src="images/000004.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-13" id="calibre_link-11" role="doc-backlink" class="calibre6">Figure 12.6</a></span></p>
</figcaption>
</figure>
<figure class="calibre14">
<img alt="A table records customer id, market date, and market count." class="center" src="images/000005.png" />
<figcaption class="calibre15">
<p class="calibre8"><span class="figurelabel"><a href="#calibre_link-14" id="calibre_link-12" role="doc-backlink" class="calibre6">Figure 12.7</a></span></p>
</figcaption>
</figure>
</section>
</section>
<section aria-labelledby="head-2-85" class="calibre2"><span id="calibre_link-88" class="calibre"></span>
<h2 id="calibre_link-89" class="calibre11">Taking Things to the Next Level</h2>
<p id="calibre_link-90" class="calibre8">In this chapter, we have built datasets that could be used as inputs to train time series models and binary classification models. Sometimes you will be joining in data from additional tables to add columns to your dataset, and you'll have to be careful not to change the granularity as you do so. In this case, we engineered features using only data in the 
<code class="calibre12">customer_purchases</code>
 table in the Farmer's Market database, aggregating data from the same table in many ways. You can use the SQL you have learned in this book to engineer a wide variety of features, improving your model by providing it with many different signals to correlate with the target variable.</p>
<p id="calibre_link-91" class="calibre8">Some people do feature engineering in their model-building script or other software. Tools like the pandas package in Python do make certain types of feature engineering straightforward to include in your machine learning script. Benefits of conducting some feature engineering in the SQL code as a separate step in your data pipeline include the ability to easily store your results in a database table to use repeatedly during training (without having to regenerate the calculated columns each time your script is run) or to share with others. Additionally, some types of summarization are more efficient to do in SQL at the point of data extraction from the database than in other coding environments. If you need it to run more quickly, you could ask an experienced data engineer to help make your SQL more computationally efficient, once you have it returning the results you want. Now that you know how to build your own dataset, you can provide them with a query that generates the results you need instead of having to explain the granularity and define every column. And you don't have to rely on a data engineer to simply add a column to an existing dataset, since you can now read SQL that someone else has developed and modify it yourself.</p>
<p id="calibre_link-92" class="calibre8">The next step after building the dataset will be conducting Exploratory Data Analysis (EDA) on it to better understand the relationship between your input features and the target variable. Then you will go through a training and testing process with the portion of the dataset that contains known outcomes from the past. Once your model is trained, you can feed in current data summarized using the same query, but without values in the target variable column, and have it predict what those values will be. Then, after evaluating your model's performance, you'll likely be right back here engineering more features or joining in more data in order to improve your model's predictions!</p>
</section>
<section aria-labelledby="head-2-86" class="calibre2"><span id="calibre_link-93" class="calibre"></span>
<h2 id="calibre_link-94" class="calibre11">Exercises</h2>
<ol class="decimal" id="calibre_link-95">
<li id="calibre_link-96" class="calibre17">Add a column to the final query in the chapter that counts how many markets were attended by each customer in the past 14 days.</li>
<li id="calibre_link-97" class="calibre17"><span aria-label="190" type="pagebreak" id="calibre_link-98" role="doc-pagebreak" class="calibre10"></span>Add a column to the final query in the chapter that contains a 1 if the customer purchased an item that cost over $10, and a 0 if not. HINT: The calculation will follow the same form as the 
<code class="calibre12">purchased_from_vendor_x</code>
 flags.</li>
<li id="calibre_link-99" class="calibre17">Let's say that the farmer's market started a customer reward program that gave customers a market goods gift basket and branded reusable market bag when they had spent at least $200 total. Create a flag field (with a 1 or 0) that indicates whether the customer has reached this loyal customer status. HINT: One way to accomplish this involves modifying the CTE (
<code class="calibre12">WITH</code>
 clause) to include purchase totals, and adding a column to the main query with a similar structure to the one that calculates 
<code class="calibre12">customer_markets_attended_count</code>
, to calculate a running total spent.</li>
</ol>
</section>
</section>
</div>


</body></html>