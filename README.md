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

### Description of tables 

I have used a total of 8 tables in my project:
customers, geolocation, order_items, order_reviews, orders, payments, products, and sellers.

---

### **1️⃣Customers Table**

* customer_id
* customer_unique_id
* customer_zip_code_prefix
* customer_city
* customer_state

---

### **2️⃣Geolocation Table**

* geolocation_zip_code_prefix
* geolocation_lat
* geolocation_lng
* geolocation_city
* geolocation_state

---

### **3️⃣Order_Items Table**

* order_id
* order_item_id
* product_id
* seller_id
* shipping_limit_date
* price
* freight_value

---

### **4️⃣Order_Reviews Table**

* review_id
* review_score
* order_id
* review_comment_title
* review_creation_date
* review_answer_timestamp

---

### **5️⃣Orders Table**

* order_id
* customer_id
* order_status
* order_purchase_timestamp
* order_delivered_carrier_date
* order_delivered_customer_date
* order_estimated_delivery_date

---

### **6️⃣Payments Table**

* order_id
* payment_sequential
* payment_type
* payment_installments
* payment_value

---

### **7️⃣Products Table**

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

### **8️⃣Sellers Table**

* seller_id
* seller_zip_code_prefix
* seller_city
* seller_state

---

“We are going to build the project by using data from all these tables. By joining these tables using common keys like order_id, customer_id, product_id, and seller_id, we can perform detailed analysis such as sales trends, customer behavior, delivery performance, payment analysis, and product performance.”


### 1. Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:

The following SQL queries were developed to answer specific business questions:

1. **Get the time range between which the orders were placed.**:
```sql
select 
min(order_purchase_timestamp) as start_time,
max(order_purchase_timestamp) as end_time
from `TARGET_SQL.orders`;
```

2. **Display the details of Cities & States of customers who ordered during the given period.**:
```sql
select 
c.customer_city, c.customer_state
from `TARGET_SQL.orders` as o 
join `TARGET_SQL.customers` as c
on o.customer_id = c.customer_id
where extract(year from o.order_purchase_timestamp) = 2018
and extract(month from o.order_purchase_timestamp) between 1 and 3;
```
### 1. 2. In-depth Exploration:
1. **Is there a growing trend in the no. of orders placed over the past years?**:
```sql
select 
extract(year from order_purchase_timestamp) as year, 
count(order_id) as order_num
from `TARGET_SQL.orders`
group by extract(year from order_purchase_timestamp)
order by order_num desc; 
```

2. **Can we see some kind of monthly seasonality in terms of the no. of orders being placed?**:
```sql
select 
extract(month from order_purchase_timestamp) as month,
count(order_id) as order_num
from `TARGET_SQL.orders` 
group by extract(month from order_purchase_timestamp)
order by order_num desc;
```



## Findings

- **Sales Trend**: Orders showed consistent year-over-year growth with noticeable seasonal patterns.
- **Regional Performance**: Revenue contribution and freight costs varied significantly across different states.
- **Payment Behavior**: Customers showed diverse payment preferences, including installment-based purchasing patterns.
- **Delivery Performance Differences**: Delivery timelines differed by region, highlighting variations in logistics efficiency.

## Reports

- **Sales Summary**: Yearly and monthly order trends.
- **Regional Performance Report**: State-wise revenue, freight, and delivery performance.
- **Payment Report**: Payment types and installment behavior analysis.

## Conclusion

This project demonstrates how SQL and BigQuery can be used to analyze large-scale e-commerce data and generate meaningful business insights. By integrating multiple tables, we evaluated sales trends, customer distribution, payment behavior, freight costs, and delivery performance. The analysis identified growth patterns, seasonal trends, and regional performance differences. Logistics efficiency was measured by comparing actual and estimated delivery timelines. Overall, the project showcases practical data analytics skills aligned with real-world business decision-making.

Thank you for your support, and I look forward to connecting with you!
