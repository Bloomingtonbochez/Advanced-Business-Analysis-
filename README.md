# Advanced-Business-Analysis-
This project showcases an advanced business analytics solution designed to transform raw transactional data into strategic business intelligence. The analysis goes beyond traditional reporting by uncovering trends, measuring performance, and revealing actionable insights that support data-driven decision-making

## Business Objective 
the primary objective of this project is to help business stakeholders understand
-how sales change over time 
-which period generated the highest revenue
-whether business performance is improving year after year 
-which products consistently out performed others
-which product categories contribute the most to totalsales
-how products can be segmented by cost 
-how customers can be classiffied according to their purchasing behaviour
-which custimer metrics are most valuable for long term decision making

## Poject Workflow


Raw Sales Data 
Data Preparation
Advanced SQL Analysis

    │
    
    ├── Change Over Time Analysis
    
    ├── Cumulative Analysis 
    
    ├── Product Performance Analysis
    
    ├── Year-over-Year Comparison
    
    ├── Whole-to-Part Analysis
    
    ├── Product Segmentation
    
    ├── Customer Segmentation
    
    └── Customer Reporting
    
      │
      
      ▼ 
  Business Insights
  Strategic Decision Support

 # Analytical Techniques Used
 1. Change Over Time Analysis

Understanding business performance over time is one of the most important aspects of business analytics.

This section analyzes sales performance on both yearly and monthly levels to uncover long-term growth patterns and seasonal behavior.

The analysis answers questions such as

Is the business growing over time?
Which months generate the highest sales?
Which months experience slow performance?
Are there seasonal buying patterns?

--Analyze sales performance overtime
--YEARLY BASIS
```

 SELECT  
YEAR (order_date) AS order_year,
MONTH (order_date) AS order_month,
SUM(sales_amount) AS total_sales,
COUNT(DISTINCT customer_key) AS total_customers,
SUM(quantity) AS Total_quantity
FROM fact_sales
WHERE order_date IS NOT NULL AND YEAR(order_date) = 2014
GROUP BY YEAR( order_date),MONTH (order_date)
ORDER BY YEAR(order_date),MONTH (order_date)

```
/* report 
from the the sales date were are able to see that sales increase over time expect for the year 2014 
this because it records the first month that we are in which is january */
-- MONTHLY BASIS
```

SELECT  
MONTH(order_date) AS order_year,
SUM(sales_amount) AS total_sales,
COUNT(DISTINCT customer_key) AS total_customers,
SUM(quantity) AS Total_quantity
FROM fact_sales
WHERE order_date IS NOT NULL 
GROUP BY MONTH( order_date)
ORDER BY MONTH(order_date)

```
## 2. Cumulative Analysis

Looking at individual monthly sales only tells part of the story.

Running totals provide a clearer understanding of how the business accumulates revenue over time.

Using SQL window functions, this project calculates:

Running Sales Total
Moving Average Price

These metrics allow stakeholders to evaluate long-term business growth rather than isolated monthly performance.

Business Value

Cumulative metrics help determine whether growth is sustainable and whether revenue is consistently increasing over time.

-- Calculate the total sales per month 
-- and the running total of sales over time
```
SELECT
order_date,
total_sales,
SUM(total_sales) OVER(ORDER BY order_date) AS total_running_total,
AVG(avg_price) OVER ( ORDER BY order_date) AS moving_average_price
FROM(
 
	SELECT
	DATETRUNC(year, order_date) As order_date,
	SUM(sales_amount) as total_sales,
	AVG(price) AS avg_price
	FROM fact_sales
	WHERE  order_date IS NOT NULL 
	GROUP BY DATETRUNC(year, order_date)

	) t

```
## 3. Product Performance Analysis

Every business needs to know which products are driving success.

This section evaluates yearly product performance by comparing current sales against:

Historical average sales
Previous year's sales

Window functions such as AVG() and LAG() are used to identify performance changes over time.

Products are automatically classified as:

Above Average
Average
Below Average

Year-over-year comparisons further identify whether product performance has increased, decreased, or remained stable.

Business Value

This analysis highlights high-performing products, detects declining products early, and supports inventory and marketing decisions.


```
--Analyze the yearly performance of product by comparing each product sales to both its average sales performance and the
--previous sales to both its average sales performance and previous year's sales
-- year over year  
WITH yearly_product_sales AS (
SELECT
YEAR(f.order_date) AS order_year, 
p.product_name,
SUM(f.sales_amount) Current_sales 
FROM fact_sales f
LEFT JOIN dbo.[gold.report_products] p
ON f.product_key =p.product_key
WHERE order_date IS NOT NULL 
GROUP BY YEAR(f.order_date),p.product_name 
)
SELECT 
order_year,
product_name,    
Current_sales,
AVG(current_sales) OVER (PARTITION BY product_name) avg_sales, 
Current_sales - AVG(current_sales) OVER (PARTITION BY product_name) diff_avg,
CASE WHEN Current_sales - AVG(current_sales) OVER (PARTITION BY product_name) > 0 THEN 'Above_avg'
	WHEN Current_sales - AVG(current_sales) OVER (PARTITION BY product_name) < 0 THEN 'Below_avg'
	ELSE 'Avg'
	END avg_change,
LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) previous_sales,
current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) as diff_previuos_sales,
CASE WHEN current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year)> 0 THEN 'Increase'
	WHEN current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) <0 THEN 'Deacrease'
	ELSE 'No change'
	END Previous_year_change
FROM yearly_product_sales

```
## 4. Whole-to-Part Analysis

Not every product category contributes equally to total business revenue.

This analysis calculates the proportional contribution of each product category to total sales.

Instead of looking at absolute sales values, stakeholders can quickly understand:

Which categories dominate revenue.
Which categories have the greatest business impact.
Which areas present opportunities for future investment.
Business Value

Understanding category contribution helps businesses prioritize resources where they generate the greatest return.

```
WITH category_sales AS (
SELECT
category,
SUM(sales_amount) total_sales
FROM fact_sales s
LEFT JOIN [gold.report_products] p
ON s.product_key = p.product_key
GROUP BY category)
SELECT
category,
total_sales,
CONCAT(ROUND((CAST(total_sales AS FLOAT) / SUM(total_sales) OVER ()*100),2),'%') as percentage_of_total 
FROM category_sales
ORDER BY total_sales DESC

```

 ## 5. Product Segmentation

Products are grouped into meaningful cost ranges using SQL conditional logic.

This allows the business to understand its product portfolio across different pricing levels.

Example segments include:

Below 100
100–500
500–1000
Above 1000
Business Value

Product segmentation assists pricing strategy, inventory planning, and product portfolio optimization.

```
/*segment products into cost range and count
how many products fall into each segment*/
WITH product_segment AS(
SELECT
product_key,
product_name,
cost,
CASE WHEN cost < 100 THEN 'Below 100'
	WHEN cost BETWEEN 100 AND 500 THEN 'Below 100-500'
	WHEN cost BETWEEN 100 AND 1000THEN 'Below 100'
	ELSE 'Above 1000'
	END cost_range
FROM [gold.report_products])
SELECT
cost_range,
COUNT(product_key) total_product
FROM product_segment
GROUP BY cost_range
ORDER BY total_product DESC


```

## 6. Customer Segmentation

Customer value is rarely equal across the customer base.

This project segments customers according to:

Total spending
Purchasing lifespan
Transaction history

Customers are classified into:

 VIP

Long-term customers with high spending.

 Regular

Long-term customers with moderate spending.


New customers with limited purchasing history.

Business Value

Customer segmentation enables targeted marketing campaigns, loyalty programs, and customer retention strategies.
```
WITH customer_spending AS (
SELECT 
c.customer_key,
SUM(s.sales_amount) AS Total_spending,
MIN(order_date) AS first_order,
MAX(order_date) AS last_order,
DATEDIFF(month,MIN(order_date),MAX(order_date)) AS lifespan
FROM fact_sales s
LEFT JOIN [gold.report_customers] c
ON s.customer_key = c.customer_key
GROUP BY c.customer_key
)
SELECT
Customer_segment,
COUNT(customer_key) total_customers
FROM
	(SELECT
	customer_key,
	CASE WHEN lifespan >= 12 AND Total_spending > 5000 THEN 'VIP'
		WHEN lifespan >= 12 AND Total_spending <= 5000 THEN 'Regular'
		ELSE 'New'
	END  Customer_segment
	FROM customer_spending)t
GROUP BY customer_segment
ORDER BY total_customers

```

## SQL Concepts Demonstrated

Throughout this project, a wide range of advanced SQL techniques are applied, including:

Common Table Expressions (CTEs)
Window Functions
Running Totals
Moving Averages

LAG()

AVG() OVER()

SUM() OVER()

Aggregate Functions

CASE Expressions

LEFT JOIN

GROUP BY

ORDER BY

Date Functions

Customer Segmentation Logic

Business Impact

This project demonstrates how SQL can move beyond querying data to solving real business problems.

The analyses support strategic decision-making by identifying growth trends, evaluating product performance, understanding customer behavior, measuring category contribution, and generating business-ready customer insights.

Rather than presenting isolated metrics, the project transforms transactional data into a comprehensive analytical framework that helps stakeholders make informed, data-driven decisions.


Developed as part of an Advanced Business Analytics portfolio to demonstrate practical SQL skills in solving real-world business problems through data analysis, performance measurement, and insight generation.







