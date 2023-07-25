Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


**SQL Queries:**

**Start with countries as there is a more complete dataset for this**
SELECT DISTINCT(country) AS "Country", SUM(calculated_revenue) as "Total Revenue by Country" FROM final_table_1
GROUP BY country
ORDER BY 2 DESC
LIMIT 5;

**Repeat for cities, include null values**
SELECT DISTINCT(city) AS "City", SUM(calculated_revenue) as "Total Revenue by City" 
FROM final_table_2
GROUP BY city
ORDER BY 2 DESC
LIMIT 5;

**Repeat for cities, exclude null values**
SELECT DISTINCT(city) AS "City", SUM(calculated_revenue) as "Total Revenue by City" 
FROM (
	SELECT *
	FROM final_table_2
	WHERE city IS NOT NULL
	) not_null_city
GROUP BY city
ORDER BY 2 DESC
LIMIT 5;

**Repeat for cities and countries**
SELECT city, country, SUM(calculated_revenue) as "Total Revenue by City and Country"
FROM final_table_2
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5;

Answer(s):

**Countries Only**

"Country"	"Total Revenue by Country"
"United States"	1107428.42
"Czechia"		6118.59
"Mexico"		952.00
"Hong Kong"		857.61
"Canada"		788.83

**Repeat for cities, exclude null values**

"City"	"Total Revenue by City"
NULL			2328782.88
"Mountain View"	568860.12
"New York"		79153.23
"San Bruno"		75149.47
"Chicago"		47814.09

**Repeat for cities, exclude null values**

"City"	"Total Revenue by City"
"Mountain View"	565301.00
"San Bruno"	170708.27
"New York"	61311.46
"Chicago"	23945.70
"Sunnyvale"	15290.78

**Repeat for cities and countries**

"city"			"country"			"Total Revenue by City and Country"
NULL			"United States"		4048395.82
"San Bruno"		"United States"		96369.87
"Chicago"		"United States"		33579.22
"Mountain View"	"United States"		26982.48
"Sunnyvale"		"United States"		11430.02

**Question 2: What is the average number of products ordered from visitors in each city and country?**

SQL Queries:

**By country**
SELECT DISTINCT(country), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

**By city, null values included**
SELECT DISTINCT(city), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

**By city, null values excluded**
SELECT DISTINCT(city), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM (SELECT * FROM final_table_2 WHERE city IS NOT NULL) AS "city_not_null"
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

**By city or country**
SELECT city, country, ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5;

Answer:

**By country**
"country"		"Average Units Sold"
"United States"	21.11
"Czechia"		17.67
"Canada"		2.33
"Bulgaria"		2.00
"Mexico"		2.00

**By city, null values included**

"city"			"Average Units Sold"
"San Jose"		69.80
"San Bruno"		69.33
NULL			28.39
"Mountain View"	13.53
"Seattle"		13.38

**By city, null values excluded**
"city"			"Average Units Sold"
"San Bruno"		65.56
"Mountain View"	36.35
"San Jose"		32.45
"Seattle"		15.29
"Salem"			13.40

**By city or country**
"city"			"country"		"Average Units Sold"
"San Bruno"		"United States"	64.25
NULL			"United States"	42.40
"San Jose"		"United States"	39.44
"Mountain View"	"United States"	22.27
NULL			"Czechia"		17.67


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

SELECT * FROM final_table_2;

WITH top_categories_by_country AS (
	SELECT country, v2_product_category, SUM(units_sold), 
	RANK() OVER (
		PARTITION BY country
		ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2
	GROUP BY 1, 2
	)	
SELECT *
FROM top_categories_by_country
WHERE rank = 1;

-- Repeat to see which categories are the top

WITH top_categories_by_country AS (
	SELECT country, v2_product_category, SUM(units_sold), 
	RANK() OVER (
		PARTITION BY country
		ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2
	GROUP BY 1, 2
	)	
SELECT * 
FROM (
SELECT v2_product_category, COUNT(*)
FROM (SELECT * FROM top_categories_by_country WHERE rank = 1) AS "rank_1"
GROUP BY 1
ORDER BY 2 DESC) AS "grouped_rank";

Answer:

The top 3 categories are apparel, accessories, electronics and drinkwear if you exclude null values.

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

SQL Queries:

**Top selling product by country**
WITH ranked_product_sales_by_country AS (
	SELECT country, v2_product_name AS "product_name", SUM(units_sold) AS "units_sold", 
	RANK() OVER (
		PARTITION BY country
		ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2
	GROUP BY 1, 2
	)
SELECT *
FROM ranked_product_sales_by_country
WHERE rank = 1;

**Top selling product by city, including null values**
WITH ranked_product_sales_by_city AS (
	SELECT city, v2_product_name AS "product_name", SUM(units_sold) AS "units_sold", 
	RANK() OVER (
	PARTITION BY city
	ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2
	GROUP BY 1, 2
	)
SELECT *
FROM ranked_product_sales_by_city
WHERE rank = 1;

**Top selling product by city, excluding null values**
WITH ranked_product_sales_by_city_not_null AS (
	SELECT city, v2_product_name AS "product_name", SUM(units_sold) AS "units_sold", 
	RANK() OVER (
	PARTITION BY city
	ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2 
	GROUP BY 1, 2
	HAVING city IN (SELECT city FROM final_table_2 WHERE city IS NOT NULL)
	)
SELECT *
FROM ranked_product_sales_by_city_not_null
WHERE rank = 1;

**Patterns for top selling product by country**
WITH ranked_product_sales_by_country AS (
	SELECT country, v2_product_name AS "product_name", SUM(units_sold) AS "units_sold", 
	RANK() OVER (
	PARTITION BY country
	ORDER BY SUM(units_sold) DESC) as "rank"
	FROM final_table_2
	GROUP BY 1, 2
	)
SELECT product_name, COUNT(*) FROM ranked_product_sales_by_country
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

Answer:

**Top selling product by country**

"country"				"product_name"													"units_sold"	"rank"
"Belgium"				"Google 17 oz Double Wall Stainless Steel Insulated Bottle"					1		1
"Bulgaria"				"YouTube Men's Vintage Tank"												2		1
"Canada"				"YouTube Leatherette Notebook Combo"									   20		1
"Chile"					"Sport Bag"																	1		1
"Colombia"				"YouTube Men's 3/4 Sleeve Henley"											3		1
"Czechia"				"Google Snapback Hat Black"												   94		1
"Dominican Republic"	"Google Men's Vintage Badge Tee Black"										1		1
"Egypt"					"Android RFID Journal"														3		1
"France"				"Android Wool Heather Cap Heather/Black"									1		1
"France"				"Seat Pack Organizer"														1		1
"Germany"				"Google 5-Panel Cap"														1		1
"Guatemala"				"YouTube Twill Cap"															1		1
"Hong Kong"				"Google Device Stand"													   38		1
"India"					"Google Youth Short Sleeve T-shirt Royal Blue"								1		1
"Japan"					"Google Lunch Bag"															5		1
"Japan"					"Rubber Grip Ballpoint Pen 4 Pack"											5		1
"Mexico"				"Nest® Cam Indoor Security Camera - USA"									4		1
"Netherlands"			"Android Men's Vintage Tank"												1		1
"Romania"				"8 pc Android Sticker Sheet"												1		1
"Russia"				"YouTube Luggage Tag"														2		1
"Singapore"				"Google Men's Short Sleeve Performance Badge Tee Navy"						1		1
"Singapore"				"Android Stretch Fit Hat Black"												1		1
"South Korea"			"Waze Dress Socks"															1		1
"Sweden"				"Google Women's Short Sleeve Hero Tee Sky Blue"								1		1
"Switzerland"			"YouTube Men's 3/4 Sleeve Henley"											1		1
"Taiwan"				"Google Lunch Bag"															1		1
"Taiwan"				"Google 5-Panel Snapback Cap"												1		1
"Taiwan"				"Spiral Notebook and Pen Set"												1		1
"United Kingdom"		"Google Men's Lightweight Microfleece Jacket Black"							2		1
"United Kingdom"		"Google Rucksack"															2		1
"United States"			"Waze Baby on Board Window Decal"										 3311		1
"Uruguay"				"Google Vintage Henley Grey/Black"											1		1

**Top selling product by city, excluding null values**

"city"				"product_name"										"units_sold"	"rank"
"Ahmedabad"			"Google Youth Short Sleeve T-shirt Royal Blue"					1	1
"Ann Arbor"			"Google Men's Vintage Tank"										5	1
"Atlanta"			"Google Onesie Green"											1	1
"Austin"			"YouTube RFID Journal"											4	1
"Bogota"			"YouTube Men's 3/4 Sleeve Henley"								3	1
"Cambridge"			"Galaxy Screen Cleaning Cloth"									16	1
"Charlotte"			"Google Lunch Bag"												114	1
"Chicago"			"Nest® Learning Thermostat 3rd Gen-USA - Copper"				110	1
"Dallas"			"YouTube Leatherette Notebook Combo"							1	1
"Denver"			"Google Twill Cap"												4	1
"Fremont"			"Nest® Protect Smoke + CO White Battery Alarm-USA"				2	1
"Helsinki"			"Android Onesie Gold"											1	1
"Hong Kong"			"Google Device Stand"											40	1
"Houston"			"Google Sunglasses"												2	1
"Jersey City"		"Google Laptop Tech Backpack"									33	1
"Kirkland"			"Google Women's Long Sleeve Tee Lavender"						8	1
"London"			"Google Men's Lightweight Microfleece Jacket Black"				2	1
"London"			"Google Rucksack"												2	1
"Los Angeles"		"Basecamp Explorer Powerbank Flashlight"						31	1
"Madrid"			"Nest® Protect Smoke + CO White Wired Alarm-USA"				1	1
"Milpitas"			"Google Women's Short Sleeve Hero Dark Grey"					3	1
"Minato"			"Google 5-Panel Snapback Cap"									3	1
"Mountain View"		"Waze Baby on Board Window Decal"							2487	1
"Munich"			"Google Men's 100% Cotton Short Sleeve Hero Tee Red"			1	1
"New York"			"Metal Texture Roller Pen"										41	1
"Osaka"				"Google Infant Short Sleeve Tee Red"							1	1
"Palo Alto"			"Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel"		2	1
"Palo Alto"			"Nest® Learning Thermostat 3rd Gen-USA - White"					2	1
"Paris"				"Google Women's Quilted Insulated Vest Black"					14	1
"Phoenix"			"Google Men's Short Sleeve Hero Tee Heather"					1	1
"Salem"				"Google Men's  Zip Hoodie"										16	1
"San Bruno"			"22 oz YouTube Bottle Infuser"									436	1
"San Francisco"		"Google Men's Short Sleeve Hero Tee Light Blue"					6	1
"San Jose"			"22 oz Android Bottle"											347	1
"Seattle"			"Google Luggage Tag"											100	1
"Seoul"				"Waze Dress Socks"												2	1
"Singapore"			"Google Men's Short Sleeve Performance Badge Tee Navy"			1	1
"Stockholm"			"Android Rise 14 oz Mug"										1	1
"Sunnyvale"			"Google Hard Cover Journal"										501	1
"Tel Aviv-Yafo"		"Google Men's Vintage Tank"										1	1
"Toronto"			"Google Men's 3/4 Sleeve Raglan Henley Grey"					10	1
"Zhongli District"	"Spiral Notebook and Pen Set"									1	1
"Zurich"			"YouTube Men's 3/4 Sleeve Henley"								1	1
NULL				"Straw Beach Mat"											1099	1

**Patterns for top selling product by country**

"product_name"											"count"
"Google Lunch Bag"											3
"Google Rucksack"											3
"Google Snapback Hat Black"									3
"Google Men's 100% Cotton Short Sleeve Hero Tee White"		2
"Google Device Stand"										2
"Google 5-Panel Snapback Cap"								2
"Android Stretch Fit Hat Black"								2
"22 oz YouTube Bottle Infuser"								2
"Clip-on Compact Charger"									2
"Android Wool Heather Cap Heather/Black"					2


**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

I am unsure how this number is generated or how it is different from question 1. 

Answer:







