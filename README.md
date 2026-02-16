# Brazil E-commerce SQL Analysis

## Project Overview

**Project Title**: Brazil E-commerce Analysis  
**Level**: Beginner  
**Database**: `TARGET_SQL`

SQL-based E-Commerce Sales & Operations Analysis project focused on revenue growth, customer distribution, payment behavior, freight costs, and delivery performance. This project uses advanced SQL concepts including joins, CTEs, window functions, and date analysis to generate real business insights.

## Tool used 
I used BigQuery because it is a serverless, cloud-based analytical database optimized for large-scale data processing and high-performance aggregation queries.

## Advantage of using BigQuery:
I used BigQuery also because there you don’t need to create the table structure manually. You can simply click on the three dots on the database, select the upload option, and easily use the table data. But in MySQL and other databases, you have to create the table schema manually and then insert the data. This is the main advantage of using BigQuery over others.

## Project Structure

### 1. Description of tables 

I have used a total of 8 tables in my project:
customers, geolocation, order_items, order_reviews, orders, payments, products, and sellers.

---

### **Customers Table**

* customer_id
* customer_unique_id
* customer_zip_code_prefix
* customer_city
* customer_state

---

### **Geolocation Table**

* geolocation_zip_code_prefix
* geolocation_lat
* geolocation_lng
* geolocation_city
* geolocation_state

---

### **Order_Items Table**

* order_id
* order_item_id
* product_id
* seller_id
* shipping_limit_date
* price
* freight_value

---

### **Order_Reviews Table**

* review_id
* review_score
* order_id
* review_comment_title
* review_creation_date
* review_answer_timestamp

---

### **Orders Table**

* order_id
* customer_id
* order_status
* order_purchase_timestamp
* order_delivered_carrier_date
* order_delivered_customer_date
* order_estimated_delivery_date

---

### **Payments Table**

* order_id
* payment_sequential
* payment_type
* payment_installments
* payment_value

---

### **Products Table**

* product_id
* product_category
* product_name_length
* product_description_length
* product_photos_qty
* product_weight_g
* product_length_cm
* product_height_cm
* product_width_cm

---

### **Sellers Table**

* seller_id
* seller_zip_code_prefix
* seller_city
* seller_state

---

1️⃣ Customers Table
Columns:

customer_id, customer_unique_id, customer_zip_code_prefix, customer_city, customer_state


2️⃣ Geolocation Table
Columns:
geolocation_zip_code_prefix, geolocation_lat, geolocation_lng, geolocation_city, geolocation_state
3️⃣ Order_Items Table
Columns:
order_id, order_item_id, product_id, seller_id, shipping_limit_date, price, freight_value
4️⃣ Order_Reviews Table
Columns:
review_id, review_score, order_id, review_comment_title, review_creation_date, review_answer_timestamp
5️⃣ Orders Table
Columns:
order_id, customer_id, order_status, order_purchase_timestamp, order_delivered_carrier_date, order_delivered_customer_date, order_estimated_delivery_date
6️⃣ Payments Table
Columns:
order_id, payment_sequential, payment_type, payment_installments, payment_value
7️⃣ Products Table
Columns:
product_id, product_category, product_name_length, product_description_length, product_photos_qty, product_weight_g, product_length_cm, product_height_cm, product_width_cm
8️⃣ Sellers Table
Columns:
seller_id, seller_zip_code_prefix, seller_city, seller_state
Final Professional Statement:
“We are going to build the project by using data from all these tables. By joining these tables using common keys like order_id, customer_id, product_id, and seller_id, we can perform detailed analysis such as sales trends, customer behavior, delivery performance, payment analysis, and product performance.”

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*) FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;

SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

DELETE FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

### 1. Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:

The following SQL queries were developed to answer specific business questions:

1. **2. Get the time range between which the orders were placed.**:
```sql
select 
min(order_purchase_timestamp) as start_time,
max(order_purchase_timestamp) as end_time
from `TARGET_SQL.orders`;
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) as total_trans
FROM retail_sales
GROUP 
    BY 
    category,
    gender
ORDER BY 1
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT 
       year,
       month,
    avg_sale
FROM 
(    
SELECT 
    EXTRACT(YEAR FROM sale_date) as year,
    EXTRACT(MONTH FROM sale_date) as month,
    AVG(total_sale) as avg_sale,
    RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1, 2
) as t1
WHERE rank = 1
```

8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
SELECT 
    category,    
    COUNT(DISTINCT customer_id) as cnt_unique_cs
FROM retail_sales
GROUP BY category
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql
WITH hourly_sale
AS
(
SELECT *,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) as total_orders    
FROM hourly_sale
GROUP BY shift
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

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.

## How to Use

1. **Clone the Repository**: Clone this project repository from GitHub.
2. **Set Up the Database**: Run the SQL scripts provided in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries provided in the `analysis_queries.sql` file to perform your analysis.
4. **Explore and Modify**: Feel free to modify the queries to explore different aspects of the dataset or answer additional business questions.

## Author - Zero Analyst

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

### Stay Updated and Join the Community

For more content on SQL, data analysis, and other data-related topics, make sure to follow me on social media and join our community:

- **YouTube**: [Subscribe to my channel for tutorials and insights](https://www.youtube.com/@zero_analyst)
- **Instagram**: [Follow me for daily tips and updates](https://www.instagram.com/zero_analyst/)
- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/najirr)
- **Discord**: [Join our community to learn and grow together](https://discord.gg/36h5f2Z5PK)

Thank you for your support, and I look forward to connecting with you!
