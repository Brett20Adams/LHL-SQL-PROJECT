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
LIMIT 10;

Answer:




**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

-- By country
SELECT DISTINCT(country), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- By city, null values included
SELECT DISTINCT(city), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- By city, null values excluded
SELECT DISTINCT(city), ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM (SELECT * FROM final_table_2 WHERE city IS NOT NULL) AS "city_not_null"
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- By city or country
SELECT city, country, ROUND(AVG(units_sold), 2) AS "Average Units Sold"
FROM final_table_2
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5;

Answer:





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





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:


Answer:







