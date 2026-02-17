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

I have used a total of 4 tables in my project:
customers, order_items, orders, and payments.

---

### **1️⃣Customers Table**

* customer_id
* customer_unique_id
* customer_zip_code_prefix
* customer_city
* customer_state

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

We are going to build the project by using data from all these tables. By joining these tables using common keys like order_id, customer_id, product_id, and seller_id, we can perform detailed analysis such as sales trends, customer behavior, delivery performance, payment analysis, and product performance.


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
### 2. In-depth Exploration:
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

3. **During what time of the day, do the Brazilian customers mostly place
-- their orders? (Dawn, Morning, Afternoon or Night)
-- ■ 0-6 hrs : Dawn
-- ■ 7-12 hrs : Mornings
-- ■ 13-18 hrs : Afternoon
-- ■ 19-23 hrs : Night**:
```sql
select 
extract(hour from order_purchase_timestamp) as time,
count(order_id) as order_num 
from `TARGET_SQL.orders`
group by extract(hour from order_purchase_timestamp) 
order by order_num desc;
```

### 3. Evolution of E-commerce orders in the Brazil region:
1. **Get the month on month no. of orders placed.**:
```sql
select 
extract(month from order_purchase_timestamp) as month,
extract(year from order_purchase_timestamp) as year,
count(*) as num_orders 
from `TARGET_SQL.orders`
group by year, month 
order by year, month; 
```

2. **Get the month on month no. of orders placed in each state.**:
```sql
select 
extract(month from order_purchase_timestamp) as month,
extract(year from order_purchase_timestamp) as year,
customer_state as state,
count(*) as num_orders
from `TARGET_SQL.orders` as o 
join `TARGET_SQL.customers` as c
on o.customer_id = c.customer_id
group by year, month, state 
order by num_orders desc;
```

3. **How are the customers distributed across all the states?**:
```sql
select
customer_city as city, customer_state as state,
count(distinct customer_id) as num_customers
from `TARGET_SQL.customers` 
group by city, state
order by num_customers desc;
```


### 4. Impact on Economy: Analyze the money movement by e-commerce by looking at order prices, freight and others:
1. **Get the % increase in the cost of orders from year 2017 to 2018
-- (include months between Jan to Aug only).
-- You can use the "payment_value" column in the payments table to get
-- the cost of orders.
--step 1: Calculate total payments per year**:
   
--step 1: Calculate total payments per year
```sql
with yearly_totals as (
  select 
  extract(year from o.order_purchase_timestamp) as year,
  sum(p.payment_value) as total_payment
  from `TARGET_SQL.payments` as p
  join `TARGET_SQL.orders` as o
  on p.order_id = o.order_id
  where extract(year from o.order_purchase_timestamp) in (2017, 2018)
  and extract(month from o.order_purchase_timestamp) between 1 and 8
  group by extract(year from o.order_purchase_timestamp)
),
```
-- Step 2: Use LEAD window function to compare each year's payments with the pervious year 
```sql
wyearly_comparisons as (
  select 
  year, 
  total_payment,
  lead(total_payment) over (order by year desc) as prev_year_payment
  from yearly_totals
)
```
-- Step 3: Calculate % increase
```sql
select 
round(((total_payment - prev_year_payment) / prev_year_payment)*100, 2)
from yearly_comparisons;
```

2. **Calculate the Total & Average value of order price for each state.**:
```sql
select 
c.customer_state as state,
round(sum(p.price), 2) as total_value, 
round(avg(p.price), 2) as avg_value 
from `TARGET_SQL.orders` as o
join `TARGET_SQL.customers` as c
on o.customer_id = c.customer_id
join `TARGET_SQL.order_items` as p
on o.order_id = p.order_id
group by state
order by total_value desc;
```

3. **Calculate the Total & Average value of order freight for each state.**:
```sql
select 
c.customer_state as state,
round(sum(oi.freight_value), 2) as total_freight,
round(avg(oi.freight_value), 2) as avg_freight
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
join `TARGET_SQL.order_items` as oi 
on o.order_id = oi.order_id
group by state
order by total_freight, avg_freight desc;
```

### 5. Analysis based on sales, freight and delivery time.:
1. **Find the no. of days taken to deliver each order from the order’s purchase date as delivery time.
-- Also, calculate the difference (in days) between the estimated & actual delivery date of an order.
-- Do this in a single query.
-- You can calculate the delivery time and the difference between the estimated & actual delivery date using the given formula:
   1.time_to_deliver = order_delivered_customer_date - order_purchase_timestamp
   2.diff_estimated_delivery = order_delivered_customer_date - order_estimated_delivery_date**
```sql
select order_id,
date_diff(date(order_delivered_customer_date), date(order_purchase_timestamp), day) as days_to_delivery,
date_diff(date(order_delivered_customer_date), date(order_estimated_delivery_date), day) as diff_estimated_delivery
from `TARGET_SQL.orders`;
```

2. **Find out the top 5 states with the average freight value.**:
```sql
select 
c.customer_state as state,
round(avg(oi.freight_value), 2) as avg_value
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
join `TARGET_SQL.order_items` as oi 
on o.order_id = oi.order_id
group by state
order by avg_value desc
limit 5;
```

3. **Find out the top 5 states with the highest & lowest average freight value.**:
```sql
with state_freight_value as (
select 
c.customer_state as state,
round(avg(oi.freight_value), 2) as avg_value
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
join `TARGET_SQL.order_items` as oi 
on o.order_id = oi.order_id
group by state
)

(select 
state,
avg_value
from state_freight_value
order by avg_value desc
limit 5)
UNION ALL
(select
state, 
avg_value 
from state_freight_value
order by avg_value asc
limit 5);
```

4. **Find out the top 5 states with average delivery time.**:
```sql
select 
c.customer_state as state,
avg(extract(date from o.order_delivered_customer_date)- extract(date from o.order_purchase_timestamp)) as avg_time_to_delivery
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
group by state
order by avg_time_to_delivery desc
limit 5;
```

-- Another Query:
```sql
select 
c.customer_state as state,
avg(date_diff(date(order_delivered_customer_date), date(order_purchase_timestamp), day)) as avg_time_to_delivery
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
group by state
order by avg_time_to_delivery desc
limit 5;
```
5. **Find out the top 5 states with the highest & lowest average delivery time.**:
```sql
with average_delivery as (
select 
c.customer_state as state,
avg(date_diff(date(o.order_delivered_customer_date), date(o.order_purchase_timestamp), day)) as avg_time_to_delivery 
from `TARGET_SQL.customers` as c
join `TARGET_SQL.orders` as o
on c.customer_id = o.customer_id
group by state)
(select 
state, 
avg_time_to_delivery
from average_delivery
order by avg_time_to_delivery desc
limit 5)
union all 
(select 
state, 
avg_time_to_delivery
from average_delivery 
order by avg_time_to_delivery asc
limit 5);
```

6. **Find out the top 5 states where the order delivery is really fast as
-- compared to the estimated date of delivery.
-- You can use the difference between the averages of actual & estimated
-- delivery date to figure out how fast the delivery was for each state.**:
```sql
select 
c.customer_state as state,
round(avG(date_diff(date(o.order_delivered_customer_date), date(o.order_estimated_delivery_date), day)), 2) as com_delivery_date
from `TARGET_SQL.orders`  as o
join `TARGET_SQL.customers` as c
on o.customer_id = c.customer_id
group by state
order by com_delivery_date asc 
limit 5;
```
-- Another Query:
```sql
with fast_delivery as (
  select c.customer_state as state,
  date_diff(date(o.order_delivered_customer_date),date(o.order_estimated_delivery_date), day) as delivery_deff
  from `TARGET_SQL.customers` as c
  join `TARGET_SQL.orders` as o
  on c.customer_id = o.customer_id
)
select 
fast_delivery.state,
round(avg(fast_delivery.delivery_deff), 2) as avg_del
from fast_delivery 
group by fast_delivery.state
order by avg_del asc
limit 5;
```

### 6. Analysis based on the payments:
1. **Find the month on month no. of orders placed using different payment types.**:
```sql
select 
p.payment_type as payment,
extract(year from o.order_purchase_timestamp) as year,
extract(month from o.order_purchase_timestamp) as month,
count(distinct o.order_id) as order_count
from `TARGET_SQL.orders` as o
join `TARGET_SQL.payments` as p
on o.order_id = p.order_id
group by payment, year, month
order by payment, year, month;
```

2. **Find the no. of orders placed on the basis of the payment installments that have been paid.**:
```sql
select 
payment_installments,
count(distinct order_id) as order_count
from `TARGET_SQL.payments`
where payment_installments = 0
group by payment_installments
order by order_count;
```

3. **orders with installments**:
```sql
select 
payment_installments as installment,
count(distinct order_id) as order_count
from `TARGET_SQL.payments` 
group by installment;
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
