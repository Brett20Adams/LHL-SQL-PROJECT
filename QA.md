What are your risk areas? Identify and describe them.

1. Ensure that the time column is in the correct format and if a sale is made, the time can't be zero.

2. Ensure that the date is correct

3. Sufficient information is not stored when a purchase is made, such as quantity of units, or denoting the row as '6' in the ecommerce_action_type is consistent with page_title = 'Checkout Confirmation' 

QA Process:
Describe your QA process and include the SQL queries used to execute it.

SELECT full_visitor_id, 
	CASE 
		WHEN time NOT LIKE '%1[0-2]|0?[1-9])%:%[0-5][0-9]%' THEN NULL
		ELSE time
	END AS time,
	CASE
		WHEN date > '2023-07-24' THEN NULL
		WHEN date < '1900-01-01' THEN NULL
		ELSE date
	END AS date,
	CASE
		WHEN ecommerce_action_type = 6 AND item_quantity IS NULL THEN NULL
		ELSE ecommerce_action_type = 6
	END AS ecommerce_action_type
FROM all_sessions_no_duplicates;

Majority of QA was done as part of the data cleaning. Please refer to that file in combination with this one.