# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals and Process
My goals are to understand the dataset as completely as possible to provide the best answer possible for the questions. It is important to spend the majority of the time for this exercise exploring the entire dataset and understanding how everything is connected, making educated guesses, and planning out how to address the largest issues first. My main goals for this project are:

1. Import the data and view it from many angles by querying each categorical column to see how columns interact with one another. This is vital and necessary because there is no readme file explaining what each column contains.

2. Once each table is understood by itself, the next logical step is to understand if tables interact with one another, and how. This is necessary to establish and separate the fact table and the dimension tables, or determine if tables provided are queries themselves.

3. Lastly, I must work backwards from the questions to properly clean and query the tables to arrive at the most logical answer. This requires plainly commenting code blocks to describe exactly what is being done and why.

## Results

1. all_sessions and analytics tables 

The **all_sessions** table is a snapshot of the ecommerce websites activity from 01-Aug-2016 to 01-Aug-2017. It collects information for many categories of products, across different stages of the shopping process; from arriving at the site, all the way to thanking you for your purchase. It appears to associate a unique IP address with a unique visitor id, and each time you access the website, the table tracks that as a unique visit id while also noting what city and country you located in. 

The **analytics** table shares many characteristics of the **all_sessions** table, however, it does not collect information about the users location, the timeframe is from 01-May-2017 to 01-Aug-2017, and notes not only purchases, but also products that were viewed only. Additionally, although the timeframes overlap, there are some rows in the **analytics** which do not appear in the **all_sessions** table. 

There are sufficient columns shared by both that allow for the joining of these two datasets after removing the duplicates. This process is fully described in the starting_with_data.md file. Once the cleaned tables are joined, questions 1-4 in the assignment.md file can be answered with PSQL queries.

2. products, sales_report, and sales_by_sku tables

The other three datasets in this project appear to be fact and dimension tables for products that customers are able to purchase. The *product_sku / sku* column is a primary key for all three tables, and is the common link to the analytics table via a join with the all_sessions table. Although these three tables require less cleaning, and are more straight forward to understand, there is insufficient information in them in order associate and extract information and how it interacts with the analytics and all_sessions table because they lack customer and date values. These tables are useful for shippers, and distributers, but we cannot answer the questions in the assignment.md file with these tables.

## Challenges 
The two main challenges I faced in this project were understanding the *time / time_on_site*, and the *total_transaction_revenue / transaction_revenue* columns in the **analytics** and **all_sessions** tables.

1. *time / time_on_site*:

The format these data values were in were very difficult to understand. I was able to see that if time_on_site column in **analytics** was null, that meant the bounce column registered a value of 1, meaning that a prospective customer arrived at the ecommerce store main page, but left before progressing to a second page. This same column in the **all_sessions** table appears to be a datetime value, but all time values are within an 2 hours of one another, and also does not match the format in the analytics table making connections difficult.

2. t*otal_transaction_revenue / transaction_revenue*

## Future Goals

To start at the beginning of the data analysis process, I would put entire dataset in 3NF. This would be achieved by: 

1. Splitting the all_sessions and analytics tables into one table with customer information, and a second table that had order information. 

2. Add order information the the product related tables so that a clear foreign key link is created

3. Attempt to deduce what the dates are for the sales_report & sales_by_sku tables
