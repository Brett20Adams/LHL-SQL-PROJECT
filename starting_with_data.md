Question 1: What user has viewed the most products but has not made a purchase in the analytics table?

SQL Queries: 

select full_visitor_id, COUNT(*)
FROM (SELECT * FROM analysis_no_duplicates WHERE units_sold IS NULL) units_sold_null
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

Answer: 

"full_visitor_id"	"count"
"0232377434237234751"	737
"4215347458239853405"	464
"7477634267430924328"	449
"4038076683036146727"	424
"4518084412353571470"	336
"3641847080616677863"	306
"5112369122544987822"	299
"1389357748324401805"	296
"7861250785102078988"	271
"7054908618767753222"	261


Question 2: What country has the most page views and could be a proxy for time spent on the site?

SQL Queries:

select country, MAX(pageviews) FROM all_sessions_no_duplicates
GROUP BY country
ORDER BY 2 DESC
LIMIT 10;

Answer:

"country"	  "Max Pageview"
"San Marino"	         19
"Jersey"	             11
"NULL"              	  9
55 others                 9  

Question 3: What products have been ordered but not enough units in stock and have the highest differential?

SQL Queries: 

SELECT *, total_ordered - stock_level AS "Order Differential" FROM sales_report
WHERE total_ordered > stock_level
ORDER BY (total_ordered - stock_level) DESC;

Answer:

"name"	                                    "total_ordered"	"stock_level"	"Order Differential"
"Android Infant Short Sleeve Tee Pewter"	        7	           2	            5
" Youth Baseball Raglan Heather/Black"	            3	           2	            1
