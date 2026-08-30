# Retail Sales Analysis SQL Project
## Project Overview

**Project Title**: Retail Sales Analysis  

## Business Objectives

**Set up a retail sales database**

Create and populate a retail sales database with the provided sales data.

**Data Cleaning**
   - Missing value identification
   - NULL handling
   - Data type conversion

**Exploratory Data Analysis (EDA)**
  - Transaction volume
  - Customer count
  - Product categories
  - Sales distribution

**Business Analysis**
    1. Sales on a specific date
    2. Clothing purchases in November 2022
	3. Sales by category
	4. Average customer age by category
	5. High-value transactions
	6. Transactions by gender and category
	7. Best-performing month by year
	8. Top 5 customers
	9. Unique customers by category
	10. Sales by time-of-day shift

## SQL Skills Demonstrated
- SELECT
- WHERE
- GROUP BY
- ORDER BY
- CASE
- CTE
- Aggregate Functions
- Window Functions
- NULLIF
- ALTER TABLE
- Date Functions

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
create database retails_project;

use retails_project;

create table retail_sales_stg
		( 
			transactions_id INT PRIMARY KEY,
			sale_date DATE,
			sale_time TIME,
			customer_id INT,
			gender VARCHAR(10),
			age varchar(25),
			category VARCHAR(25),
			quantiy varchar(25),
			price_per_unit varchar(25),
			cogs varchar(25),
			total_sale varchar(25)
		);
-- ** A staging table was created to initially load raw CSV values as text, allowing missing/blank values to be standardized before converting columns to their appropriate analytical data types.**
create table retail_sales AS
select 
	transactions_id,
			sale_date,
			sale_time,
			customer_id,
			gender,
			NULLIF(age, '') AS age,
			category,
			NULLIF(quantiy, '') AS quantiy,
			NULLIF(price_per_unit, '') AS price_per_unit,
			NULLIF(cogs, '') AS cogs,
			NULLIF(total_sale, '') AS total_sale
from retail_sales_stg;

Alter table retail_sales
Modify column age INT,
Modify column quantiy INT,
Modify column price_per_unit FLOAT,
Modify column cogs FLOAT,
Modify column total_sale FLOAT;

-- The **quantity** column was initially created as **quantiy** and was later renamed and corrected to **quantity**.
Alter table retail_sales
Change column quantiy quantity INT;
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
Select * from retail_sales;

Select Count(Distinct(customer_id)) from retail_sales;

Select DISTINCT category from  retail_sales;

Select * from retail_sales 
where
  sale_date is NULL OR sale_time is NULL OR customer_id is NULL OR gender is NULL OR age is NULL OR category is NULL OR 
  quantity is NULL OR price_per_unit is NULL OR cogs is NULL OR total_sale is NULL;

Delete from retail_sales
where
  sale_date is NULL OR sale_time is NULL OR customer_id is NULL OR gender is NULL OR age is NULL OR category is NULL OR 
  quantity is NULL OR price_per_unit is NULL OR cogs is NULL OR total_sale is NULL;
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
Select * from retail_sales
where
sale_date = "2022-11-05";
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
Select * from retail_sales
where
	Category = "Clothing" 
	AND 
	sale_date >= '2022-11-01'
	AND 
	sale_date < '2022-12-01' 
	AND 
	Quantity > 4 ;
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
Select 
	category, 
	sum(total_sale) as total_sales, 
	COUNT(*) as total_orders 
from retail_sales
group by category;
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
Select 
	category, 
	round(avg(age),2) as avg_age 
from retail_sales 
where category = "Beauty"
group by category;
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
Select * from retail_sales
where total_sale > 1000;
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
Select 
	Category, 
	Gender, 
	Count(transactions_id) as Total_Transactions 
from retail_sales 
group by category, gender
order by Category;
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
Select 
	Year,
	Month,
	Avg_sales
from
(
	Select 
	year(sale_date) as Year,
	month(sale_date) as Month,
	round(avg(total_sale),2) as Avg_sales,
	rank() Over(
				Partition By year(sale_date)
				Order By avg(total_sale) Desc
	) as ranking
	from retail_sales
	Group by Year, Month
) as t1
where ranking = 1;
```

8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
Select 
	customer_id as Customers, 
	sum(total_sale) as Total_sales 
from retail_sales
group by 1
order by 2 desc
limit 5;
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
Select 
  Category, 
  count(distinct(customer_id)) as Unique_Cutomer 
from retail_sales
group by 1;
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql
With Hourly_sales
As
(
	Select *,
		Case
			When Extract(Hour from sale_time) < 12 Then "Morning"
			When Extract(Hour from sale_time) Between 12 AND 17 Then "Afternoon"
			Else "Evening"
		End as shift
	from retail_sales
)
Select 
    shift,
    COUNT(*) as total_orders
from Hourly_sales
Group By shift;
```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.


Thanks, looking forward to connect with you.
