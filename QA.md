What are your risk areas? Identify and describe them.

1. Ensure that the time column is in the correct format and if a sale is made, the time can't be zero.

2. Ensure that the date is correct

2. Sufficient information is not stored when a purchase is made, such as quantity of units, or denoting the row as '6' in the ecommerce_action_type is consistent with page_title = 'Checkout Confirmation' 



QA Process:
Describe your QA process and include the SQL queries used to execute it.

First, we must start with the analysis of products being purchased and then go from there.

SELECT full_visitor_id, t