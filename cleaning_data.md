What issues will you address by cleaning the data?

Table: sales_report

1. 1. Null Values: There are 78 null values in the ratio column. Upon exploring these null values, the null values can be replaced with 0 as the ratio is total_ordered:stock_level; all null values for ratio have a value of 0 for either of these columns.

SELECT COUNT(*) FROM sales_report WHERE ratio IS NULL; -- 78 null values
-- Explore the 78 rows which have null ratio values
SELECT * FROM sales_report
WHERE ratio IS NULL;
-- Note: All 78 rows have a value of 0 for total_ordered and stock_level
UPDATE sales_report
SET ratio = 0 
WHERE ratio IS NULL;

1. 2. There are leading spaces in the name column of some rows, these will be removed with the TRIM() function

SELECT TRIM(name)
FROM sales_report;

Table: sales_by_sku

2. 1. Check for null values

SELECT COUNT(*) FROM sales_by_sku WHERE product_sku IS NULL; -- no null values
SELECT COUNT(*) FROM sales_by_sku WHERE total_ordered IS NULL; -- no null values

2. 2. Check for distinct values in product_sku column
SELECT COUNT(DISTINCT(product_sku)) FROM sales_by_sku; -- 462 distinct; declare PK

2. 3. Check total_ordered to see if all values are > 0
SELECT COUNT(*) FROM sales_by_sku WHERE total_ordered = 0; -- 156 rows

2. 4. Check if total_ordered in "sales_by_sku" matches total_ordered in "sales_report" for sku "GGOEGOAQ012899" (456)
SELECT * FROM sales_report WHERE product_sku = 'GGOEGOAQ012899'; -- total_ordered = 456

**Nothing to clean in this table**

Table: products

3. 1. Check for null values
SELECT COUNT(*) FROM products WHERE sku IS NULL; -- no null values
SELECT COUNT(*) FROM products WHERE name IS NULL; -- no null values
SELECT COUNT(*) FROM products WHERE ordered_quantity IS NULL; -- no null values
SELECT COUNT(*) FROM products WHERE stock_level IS NULL; -- no null values
SELECT COUNT(*) FROM products WHERE restocking_lead_time IS NULL; -- no null values
SELECT COUNT(*) FROM products WHERE sentiment_score IS NULL; -- 1 null value
SELECT COUNT(*) FROM products WHERE sentiment_magnitude IS NULL; -- 1 null value

3. 2. Check for duplicate in product_sku which is assumed PK
SELECT COUNT(DISTINCT(sku)) from products; -- No duplicates; declare PK

3. 3. Explore products.names column
SELECT COUNT(DISTINCT(name)) FROM products; -- 313 distinct names; many products can have the same name

-- Explore names by grouping
SELECT sku, name FROM products
GROUP BY 1, 2
ORDER BY 2;
-- Example: there are 5 products which are named "Baby Essentials Set"

Table: analytics

4. 1. Check for null values

SELECT COUNT(*) FROM analytics WHERE visit_number IS NULL; -- no null values
SELECT COUNT(*) FROM analytics WHERE visit_id IS NULL; -- no null values
SELECT COUNT(*) FROM analytics WHERE visit_start_time IS NULL; -- No null values
SELECT COUNT(*) FROM analytics WHERE date IS NULL; -- No null values
SELECT COUNT(*) FROM analytics WHERE full_visitor_id IS NULL; -- No null values
SELECT COUNT(*) FROM analytics WHERE user_id IS NULL; -- ALL NULL VALUES
SELECT COUNT(*) FROM analytics WHERE channel_grouping IS NULL; -- no null values
SELECT COUNT(*) FROM analytics WHERE social_engagement_type IS NULL; -- no null values
SELECT COUNT(*) FROM analytics WHERE units_sold IS NULL; -- 4,205,975 null values
SELECT COUNT(*) FROM analytics WHERE page_views IS NULL; -- 72 null values -> These could be errors, explore further
SELECT COUNT(*) FROM analytics WHERE time_on_site IS NULL; -- 477,465 null values -> These could be errors
SELECT COUNT(*) FROM analytics WHERE bounces IS NULL; -- 3,826,283 null values -> These could be zero (no bounce happened)
SELECT COUNT(*) FROM analytics WHERE revenue IS NULL; -- 4,285,767 null rows -> Investigate further
SELECT COUNT(*) FROM analytics WHERE unit_price IS NULL; -- no null values

4. 2. Determine what columns are required in order to make a primary key. Columns with the highest # of distinct values are clues.

-- Explore for potential primary key(s) by checking distinct values
SELECT COUNT(DISTINCT(visit_number)) from analytics; -- 222 distinct values
SELECT COUNT(DISTINCT(visit_id)) from analytics; -- 148,642 distinct values
SELECT COUNT(DISTINCT(visit_start_time)) from analytics; -- 148,853 distinct values
SELECT COUNT(DISTINCT(date)) from analytics; -- 93 distinct values
SELECT COUNT(DISTINCT(full_visitor_id)) from analytics; -- 120,018 distinct values

SELECT MIN(date), MAX(date) FROM analytics -- this table is for 93 days from 01-May-2017 to 01-Aug-2017

4. 3. Update column types after the import process

ALTER TABLE analytics
ALTER COLUMN time_on_site TYPE integer USING(time_on_site::integer)
ALTER TABLE analytics
ALTER COLUMN revenue TYPE numeric USING(revenue::numeric)
ALTER TABLE analytics
ALTER COLUMN unit_price TYPE numeric USING(unit_price::numeric)
ALTER TABLE analytics
ALTER COLUMN units_sold TYPE integer USING(units_sold::integer)

4. 4. Remove duplicates once primary key columns are determined.

--Remove rows with duplicate unit price for analytics then create view for performing analysis
CREATE OR REPLACE VIEW analysis_no_duplicates AS (
WITH duplicates AS (
	SELECT *, ROW_NUMBER() OVER (
				PARTITION BY date, full_visitor_id, visit_number, visit_id, unit_price) AS Duplicate_Count
	FROM analytics)
SELECT *
FROM duplicates
WHERE Duplicate_Count = 1);

-- Confirm that there are no duplicates in analysis_no_duplicates
SELECT *, COUNT(*) FROM analysis_no_duplicates
GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15
HAVING COUNT(*) > 1;
-- Confirmed, no duplicates

The rank function will increase by 1 each time a duplicate is encountered across the partition by statements. By limiting the query to the first instance, we will select only one instance.

4. 5. Create view from the duplicate free analytics table that only contains rows in which an order was placed.

CREATE OR REPLACE VIEW analysis_cleaned_1 AS (
	SELECT visit_id, date, full_visitor_id, units_sold, ROUND(unit_price/1000000, 2) as "unit_price", ROUND(revenue/1000000, 2) as "raw_revenue", units_sold * (ROUND(unit_price/1000000, 2)) as "calculated_revenue"
	FROM analysis_no_duplicates
	WHERE units_sold is not null
	ORDER BY date)

Table: all_sessions

5. 1. Check for null values

SELECT COUNT(*) FROM all_sessions WHERE full_visitor_id IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE channel_grouping IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE all_sessions.time IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE country IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE city IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE total_transaction_revenue IS NULL; -- 15,053 null values
SELECT COUNT(*) FROM all_sessions WHERE transactions IS NULL; -- 15,053 null values (matches above)
SELECT COUNT(*) FROM all_sessions WHERE time_on_site IS NULL; -- 3,300 null values
SELECT COUNT(*) FROM all_sessions WHERE pageviews IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE session_quality_dimension IS NULL; -- 13,906 null values
SELECT COUNT(*) FROM all_sessions WHERE all_sessions.date IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE visit_id IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE all_sessions.type IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE product_refund_amount IS NULL; -- ALL VALUES NULL
SELECT COUNT(*) FROM all_sessions WHERE product_quantity IS NULL; -- 15,081 null values
SELECT COUNT(*) FROM all_sessions WHERE product_price IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE product_revenue IS NULL; -- 15,130 null values
SELECT COUNT(*) FROM all_sessions WHERE product_sku IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE v2_product_name IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE v2_product_category IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE product_variant IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE currency_code IS NULL; -- 272 null values
SELECT COUNT(*) FROM all_sessions WHERE item_quantity IS NULL; -- ALL VALUES NULL
SELECT COUNT(*) FROM all_sessions WHERE item_revenue IS NULL; -- ALL VALUES NULL
SELECT COUNT(*) FROM all_sessions WHERE transaction_revenue IS NULL; -- 15,130 null values
SELECT COUNT(*) FROM all_sessions WHERE transaction_id IS NULL; -- 15,125 null values
SELECT COUNT(*) FROM all_sessions WHERE page_title IS NULL; -- 1 null values
SELECT COUNT(*) FROM all_sessions WHERE search_keyword IS NULL; -- ALL VALUES NULL
SELECT COUNT(*) FROM all_sessions WHERE path_page_level_1 IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE ecommerce_action_type IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE ecommerce_action_step IS NULL; -- No null values
SELECT COUNT(*) FROM all_sessions WHERE ecommerce_action_options IS NULL; -- 15,103 null values

5. 2. Check for potential primary keys by exploring columns with high # of distinct values
SELECT COUNT(DISTINCT(full_visitor_id)) FROM all_sessions; -- 14,223 distinct values
SELECT COUNT(DISTINCT(date)) FROM all_sessions; -- 366 distinct values, one year of data from 01-Aug-2016 to 01-Aug-2017
SELECT COUNT(DISTINCT(time)) FROM all_sessions; -- 9,600 distinct values
SELECT COUNT(DISTINCT(visit_id)) FROM all_sessions; -- 14,556 distinct values
SELECT COUNT(DISTINCT(product_sku)) FROM all_sessions; -- 536 distinct values

5. 3. Clean/update columns

city:
    UPDATE all_sessions
    SET city = NULL
    WHERE city = 'not available in demo dataset'
    UPDATE all_sessions
    SET city = NULL
    WHERE city = '(not set)'
    UPDATE all_sessions
    SET city = 'San Francisco'
    WHERE city = 'South San Francisco'

Update product_quantity product type to integer
    ALTER TABLE all_sessions
    ALTER COLUMN product_quantity TYPE integer USING(product_quantity::integer)   

5. 2. Determine what columns are required in order to make a primary key. The four columns that can be used to generate a primary key are date, full_visitor_id, visit_id, and transaction_id. This can be proved as a unique visit_id is generated for a unique full_visitor_id each date, and since we are concerned in the questions for transactions, we filter by rows with a transaction_id to separate purchases from views. Remove duplicates once primary key columns are determined.

CREATE OR REPLACE VIEW all_sessions_no_duplicates AS (
WITH dup_all_sessions AS (
	SELECT *, ROW_NUMBER() OVER (
			PARTITION BY date, full_visitor_id, visit_id, transaction_id) AS Duplicate_Count
	FROM all_sessions)
SELECT *
FROM dup_all_sessions
WHERE Duplicate_Count = 1);

5. 3. Confirm that there are no duplicates in all_sessions_no_duplicates
SELECT *, COUNT(*) FROM all_sessions_no_duplicates
GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33
HAVING COUNT(*) > 1;
-- Confirmed, no duplicates

5. 4. Create view of the all_sessions table with duplicates removed that contains all columns needed for questions. This view only has data for sales made, which is denoted by the ecommerce_action_type = '6'

CREATE OR REPLACE VIEW all_sessions_cleaned_2 AS (
	SELECT visit_id, date, full_visitor_id, city, country, product_sku, v2_product_name, v2_product_category, CAST(product_quantity AS integer), ROUND(product_price/1000000, 2) as "unit_price", CAST(product_revenue AS numeric)/1000000 as "raw_revenue"
	FROM all_sessions_no_duplicates
	WHERE ecommerce_action_type = '6')

 This contains all 6 rows that have information confirming that an order was placed.   

**Queries:**
Below, provide the SQL queries you used to clean your data.

Queries are underneath their descriptions for ease of reading.

**Process to join the all_sessions and analytics table:**

-- Create view of columns missing in analytics table that will be added via join.

CREATE OR REPLACE VIEW all_sessions_countries_cities_product_info AS (
	SELECT DISTINCT(full_visitor_id), country, city, product_sku, v2_product_name, v2_product_category
	FROM all_sessions_no_duplicates
	GROUP BY 1, 2, 3, 4, 5, 6)

-- Join the cleaned analysis view with all_sessions_countries_cities_product_info to link orders to their city, country, and product info.

CREATE OR REPLACE VIEW analysis_countries_cities_product_info AS (
	SELECT ac1.date, ac1.visit_id, ac1.full_visitor_id, alscpi.city, alscpi.country, alscpi.product_sku, alscpi.v2_product_name, alscpi.v2_product_category, ac1.units_sold, ac1.unit_price, ac1.calculated_revenue
	FROM analysis_cleaned_1 ac1
	JOIN all_sessions_countries_cities_product_info alscpi ON
	ac1.full_visitor_id = alscpi.full_visitor_id
	ORDER BY ac1.date)

-- Join analysis_cleaned_1 (ac1) to all_sessions_cleaned_2 (asc2) to add the units sold from ac1 that are missing in asc2

CREATE OR REPLACE VIEW all_sessions_units_2 AS (
	SELECT asc2.date, asc2.visit_id, asc2.full_visitor_id, asc2.city, asc2.country, asc2.product_sku, asc2.v2_product_name, asc2.v2_product_category, asc2.product_quantity AS "units_sold", asc2.unit_price, asc2.product_quantity * asc2.unit_price as "calculated_revenue"
	FROM all_sessions_cleaned_2 asc2
	LEFT JOIN analysis_cleaned_1 ac1 ON
	asc2.full_visitor_id = ac1.full_visitor_id AND
	asc2.visit_id = ac1.visit_id AND
	asc2.unit_price = ac1.unit_price
	WHERE asc2.product_quantity IS NOT NULL
	ORDER BY asc2.date)

-- Union the all_sessions_units_2 with the analysis_countries_cities_product_info table

CREATE OR REPLACE VIEW final_table_2 AS (
	SELECT * FROM all_sessions_units_2
	UNION ALL
	SELECT * FROM analysis_countries_cities_product_info
	ORDER BY date)

**final_table_2 used for all questions**   