# SQL Project: Data Analysis for Zomato - A Food Delivery Company

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India.
The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

## Project Structure

- **Database Setup:** Creation of the `zomato_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.

- ## Database Setup
```sql
CREATE DATABASE zomato_db;
```
###  Dropping Existing Tables
```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;

### Creating Tables
create table customers(
	customer_id int primary key,
	customer_name varchar(25) not null,
	reg_date date
	
);

create table restaurants(
	restaurant_id int primary key,
	restaurant_name varchar(55),
	city varchar(25),
	opening_hours varchar(55)
);

create table riders(
	rider_id int primary key,
	rider_name varchar(55),
	sign_up date
);

create table orders(
	order_id int primary key,
	customer_id int,
	restaurant_id int,
	order_item varchar(55).
	order_date date,
	order_time time,
	order_status varchar(25),
	total_amount numeric,
	foreign key (customer_id) references customers(customer_id),
	foreign key (restaurant_id) references restaurants(restaurant_id)
);

create table deliveries(
	delivery_id int primary key,
	order_id int,
	delivery_status varchar(35),
	delivery_time time,
	rider_id int,
	foreign Key (order_id) references orders(order_id),
	foreign key (rider_id) references riders(rider_id
);

## Data Import

## Data Cleaning and Handling Null Values

select count(*) from customers
where 
	customer_name is null
	or
	reg_date is null

select count(*) from restaurants
where
	restaurant_name is null
	or 
	city is null
	or 
	opening_hours is null

select * from orders
where 
	order_item is null
	or
	order_date is null
	or 
	order_time is null
	or
	order_status is null
	or 
	total_amount is null

select count(*) from riders
where
	rider_name is null
	or 
	sign_up is null

select count(*) from riders
where
	order_id is null
	or
	delivery_status


Before performing analysis, I ensured that the data was clean and free from null values where necessary.
For instance:

```sql
UPDATE orders
SET total_amount = COALESCE(total_amount, 0);
```

## Business Problems Solved

### Top 5 Most Frequently Ordered Dishes
### 1. Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.
```sql
with ranking_cte as(
select
	c.customer_id,
	c.customer_name,
	o.order_item,
	count(o.order_item) as total_orders,
	dense_rank() over(order by count(o.order_item) desc) as rn
from orders as o 
join customers as c 
on o.customer_id = c.customer_id 
where c.customer_name = 'Arjun Mehta' and order_date between '2023-09-02' and '2024-09-02'
group by 1,2,3
order by 4 desc)
select
	customer_id,
	customer_name,
	order_item,
	total_orders
from ranking_cte
where rn <= 5
```
### Popular Time Slots
### 2. Identify the time slots during which the most orders are placed, based on 2-hour intervals



```sql
-- Approach 1
select 
	case
	when extract(hour from order_time) between 0 and 1 then   '00:00 - 02:00'
	when extract(hour from order_time) between 2 and 3 then   '02:00- 04:00'
	when extract(hour from order_time) between 4 and 5 then   '04:00 - 06:00'
	when extract(hour from order_time) between 6 and 7 then   '06:00 - 08:00'
	when extract(hour from order_time) between 8 and 9 then   '08:00 - 10:00'
	when extract(hour from order_time) between 10 and 11 then '10:00 - 12:00'
	when extract(hour from order_time) between 12 and 13 then '12:00 - 14:00'
	when extract(hour from order_time) between 14 and 15 then '14:00 - 16:00'
	when extract(hour from order_time) between 16 and 17 then '16:00 - 18:00'
	when extract(hour from order_time) between 18 and 19 then '18:00 - 20:00'
	when extract(hour from order_time) between 20 and 21 then '20:00 - 22:00'
	when extract(hour from order_time) between 22 and 23 then '22:00 - 24:00'
	end as time_slot,
	count(order_id) as order_count
from orders
group by time_slot
order by order_count desc
```

```sql
-- Approach 2

select
	floor(extract(hour from order_time)/2)*2  as start_time,
	floor(extract(hour from order_time/2))*2 + 2 as end_time,
	count(order_id) as order_count
from orders
group by start_time, end_time
order by order_count desc
```

### Order Value Analysis
### Q.3 Find the average order value (AOV) per customer who has placed more than 750 orders.
-- Return: customer_name, aov (average order value).

```sql
select
	c.customer_name,
	avg(o.total_amount) as aov
from orders as o 
join customers as c 
on o.customer_id = c.customer_id
group by c.customer_name
having count(o.order_id) > 750
```

### High-Value Customers

### Q.4 List the customers who have spent more than 100K in total on food orders.
-- Return: customer_name, customer_id.

```sql
select
	c.customer_name,
	c.customer_id
from orders as o 
join customers as c
on o.customer_id = c.customer_id
group by 1,2
having sum(total_amount) > 100000
```

### Orders Without Delivery

### Q.5 Write a query to find orders that were placed but not delivered.
-- Return: restaurant_name, city, and the number of not delivered orders

-- Approach 1

```sql
select
	r.restaurant_name,
	r.city,
	count(o.order_id) as cnt_not_delivered_orders
from orders as o
left join restaurants as r
on o.restaurant_id = r.restaurant_id 
left join deliveries as d 
on o.order_id = d.delivery_id
where d.delivery_id is null
group by 1,2
order by 3 desc
```

-- Approach 2
```sql
select
	r.restaurant_name,
	r.city,
	count(o.order_id) as cnt_not_delivered_orders
from orders as o 
left join restaurants as r
on o.restaurant_id = r.restaurant_id
where
	o.order_id not in (select order_id from deliveries)
group by 1, 2
order by 3 desc
```
### Restaurant Revenue Ranking

### Q.6 Rank restaurants by their total revenue from the last year.
-- Return: restaurant_name, total_revenue, and their rank within their city.

```sql
select
	r.restaurant_name,
	r.city,
	sum(total_amount) as total_revenue,
	rank() over(partition by r.city order by sum(total_amount) desc) as rn
from orders as o 
join restaurants as r 
on o.restaurant_id = r.restaurant_id
where o.order_date between '2023-09-02' and '2024-09-02'
group by 1,2
```

### Most Popular Dish by City

### Q.7 Identify the most popular dish in each city based on the number of orders.

```sql
select
	*
from(select 
	r.city,
	o.order_item as dish,
	count(order_id) as total_orders,
	rank() over(partition by r.city order by count(order_id) desc) as rn
from orders as o 
join restaurants as r 
on r.restaurant_id = o.restaurant_id
group by 1,2) as t1
where rn = 1
```

### Customer Churn

### Q.8 Find customers who haven’t placed an order in 2024 but did in 2023.


```sql

select
distinct o.customer_id,
c.customer_name
from orders as o
join customers as c 
on o.customer_id = c.customer_id
where 
	extract(year from order_date) = 2023
	and
	o.customer_id not in (select distinct customer_id from orders where extract(year from order_date) = 2024)
```

### Cancellation Rate Comparison
### Q.9 Calculate and compare the order cancellation rate for each restaurant between the current year and the previous year.

--Approach 1


```sql
with cancel_ratio_2023 as(
select
	o.restaurant_id,
	count(o.order_id) as total_orders,
	count(case when d.delivery_id is null then 1 end) as not_delivered
from orders as o 
left join deliveries as d 
on o.order_id = d.order_id
where extract(year from order_date) = 2023
group by 1),
last_year as(
	select
		restaurant_id,
		total_orders,
		not_delivered,
		round((not_delivered::numeric / total_orders::numeric) * 100,2) as  cancel_rate
	from cancel_ratio_2023
),
cancel_ratio_2024 as(
select
	o.restaurant_id,
	count(o.order_id) as total_orders,
	count(case when d.delivery_id is null then 1 end) as not_delivered
from orders as o 
left join deliveries as d 
on o.order_id = d.order_id
where extract(year from order_date) = 2024
group by 1),
current_year as(
	select
		restaurant_id,
		total_orders,
		not_delivered,
		round((not_delivered::numeric / total_orders::numeric) * 100,2) as  cancel_rate
	from cancel_ratio_2024
)
select
	c.restaurant_id as rest_id,
	c.cancel_rate as cs_ratio,
	l.cancel_rate as ls_cs_ratio
from current_year as c 
join last_year as l
on c.restaurant_id = l.restaurant_id
```
-- Approach 2
```sql
with cancel_ratio as(
select
	o.restaurant_id,
	extract(year from o.order_date) as order_year,
	count(o.order_id) as total_orders,
	count(case when d.delivery_id is null then 1 end) as not_delivered
from orders as o 
left join deliveries as d 
on o.order_id = d.order_id
where extract(year from o.order_date) in (2023, 2024)
group by 1, 2),
pivoted as(
	select
	restaurant_id,
	sum(case when order_year = 2023 then total_orders end) as total_orders_2023,
	sum(case when order_year = 2023 then not_delivered end) as not_delivered_2023,
	sum(case when order_year = 2024 then total_orders end) as total_orders_2024,
	sum(case when order_year = 2024 then not_delivered end) as not_delivered_2024
	from cancel_ratio
	group by 1
)
select
	restaurant_id,
	round((not_delivered_2023::numeric)/nullif(total_orders_2023,0),2) * 100 as ls_cancel_ratio,
	round((not_delivered_2024::numeric)/nullif(total_orders_2024,0),2) * 100 as cr_cancel_ratio
from pivoted
```

### Rider Average Delivery Time
### Q.10 Determine each rider's average delivery time

```sql
select
	d.rider_id,
	avg(extract(epoch from (d.delivery_time - o.order_time + case when d.delivery_time < o.order_time then interval ' 1 day' else interval '0 day' end))/60) as avg_time
from orders as o 
join deliveries as d 
on o.order_id = d.order_id
where d.delivery_status = 'Delivered'
group by 1
```

### Monthly Restaurant Growth Ratio
### Q.11 Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining
```sql
with monthly_growth_cte as(
select
	o.restaurant_id,
	extract(month from o.order_date) as month,
	extract(year from o.order_date) as year,
	count(o.order_id) as cr_mnth_orders,
	lag(count(o.order_id), 1, null) over(partition by o.restaurant_id order by extract(year from o.order_date),extract(month from o.order_date) ) as prev_month_orders
from orders as o
join deliveries as d 
on o.order_id = d.delivery_id 
where d.delivery_status = 'Delivered'
group by 1, 3,2
order by  1,3,2)
select
	restaurant_id,
	month,
	cr_mnth_orders,
	prev_month_orders,
	round((cr_mnth_orders::numeric - prev_month_orders::numeric)/nullif(prev_month_orders::numeric,0) * 100,2) as growth_ratio
from monthly_growth_cte
```

### Q.12 Customer Segmentation
### Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the
-- average order value (AOV). If a customer's total spending exceeds the AOV, label them as 'Gold'; otherwise, label them as 'Silver'.
-- Return: The total number of orders and total revenue for each segment.

```sql
with segg_cte as(
select
	customer_id,
	count(orders) as total_orders,
	sum(total_amount) as total_amount,
	case
	when sum(total_amount) > (select avg(total_amount) from orders) then 'Gold'
	else 'Silver'
	end buckets
from orders
group by 1)
select
	buckets,
	sum(total_orders),
	sum(total_amount)
from segg_cte
group by 1
```

### Rider Monthly Earnings
### Q.13 Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount
```sql
select
	d.rider_id,
	extract(year from o.order_date) as year,
	extract(month from o.order_date) as month,
	0.08 * sum(total_amount)::numeric as total_earnings
from orders as o
join deliveries as d
on o.order_id = d.delivery_id 
group by 1,2,3
order by  1,2,3
```

### Q.14 Rider Ratings Analysis
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has.
-- Riders receive ratings based on delivery time:
-- 5-star: Delivered in less than 15 minutes
-- 4-star: Delivered between 15 and 20 minutes
-- 3-star: Delivered after 20 minutes



-- Approach 1
```sql
with order_time_for_delivery as (
select
	o.order_id,
	d.rider_id,
	extract(epoch from (d.delivery_time - o.order_time + case when d.delivery_time < o.order_time then interval '1 day' else interval '0 day' end))/60 as time_order_delivered
from orders as o 
join deliveries as d 
on o.order_id = d.order_id
where d.delivery_status = 'Delivered')
select
	rider_id,
	count(case when time_order_delivered < 15 then 1 else null end) as five_star,
	count(case when time_order_delivered >= 15 and time_order_delivered <= 20 then 1 else null end) as four_star,
	count(case when time_order_delivered  > 20 then 1 else null end) as three_star
from order_time_for_delivery
group by 1
order by 1
```

-- Approach 2
```sql
select
	rider_id,
	stars,
	count(*) as total_stars
from(select
	rider_id,
	delivery_took_time,
	case
	when delivery_took_time < 15 then '5 star' 
	when delivery_took_time between 15 and 20 then '4 star'
	else '3 star'
	end as stars
from(select
	o.order_id,
	o.order_time,
	d.delivery_time,
	extract(epoch from (d.delivery_time - o.order_time + case when d.delivery_time < o.order_time then interval '1 day' else interval '0 day' end))/60 as delivery_took_time,
	d.rider_id
from orders as o
join deliveries as d
on  o.order_id = d.order_id
where d.delivery_status ='Delivered') as t1
) as t2
group by 1,2 
order by 1
```

### Order Frequency by Day
### Q.15 Analyze order frequency per day of the week and identify the peak day for each restaurant

```sql
with cte as(
select
	r.restaurant_name,
	to_char(o.order_date, 'day') as day_of_the_week,
	-- extract(dow from o.order_date) as dow,
	count(o.order_id) as total_orders,
	rank() over(partition by r.restaurant_name order by count(o.order_id) desc) as rn
from orders as o 
join restaurants as r
on o.restaurant_id = r.restaurant_id
group by 1,2
order by 1)
select
	restaurant_name,
	day_of_the_week,
	total_orders
from cte
where rn = 1
```

 ### Customer Lifetime Value (CLV)
 ### Q.16 Calculate the total revenue generated by each customer over all their orders.


```sql
select
	c.customer_id,
	c.customer_name,
	sum(total_amount) as CLV
from orders as o 
join customers as c 
on o.customer_id = c.customer_id
group by 1,2
order by 1
```

### Monthly Sales Trends
### Q.17 Identify sales trends by comparing each month's total sales to the previous month.

```sql
select
	r.restaurant_name,
	extract(year from o.order_date) as year,
	extract(month from o.order_date) as month,
	sum(total_amount) as cr_mnth_sales,
	lag(sum(total_amount),1,null) over(partition by r.restaurant_name order by extract(year from o.order_date), extract(month from o.order_date) asc ) pr_mnth_sales,
	sum(total_amount) - lag(sum(total_amount),1,null) over(partition by r.restaurant_name order by extract(year from o.order_date), extract(month from o.order_date) asc ) as sales_trend
from orders as o 
join restaurants as r 
on o.restaurant_id = r.restaurant_id
group by 1, 2, 3
```

### Rider Efficiency
### Q.18 Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.

```sql
with t1 as
(select
	d.rider_id,
	avg(extract(epoch from (d.delivery_time - o.order_time + case when d.delivery_time  < o.order_time then interval '1 day' else interval ' 0 day' end))/60) as avg_delivery_time
from orders as o 
join deliveries as d 
on o.order_id = d.order_id
where d.delivery_status = 'Delivered'
group by 1 
)
(select
	rider_id,
	avg_delivery_time
from t1
order by avg_delivery_time asc
limit 1)
union all
(select
	rider_id,
	avg_delivery_time
from t1
order by avg_delivery_time desc
limit 1)
```

### Order Item Popularity
### Q.19 Track the popularity of specific order items over time and identify seasonal demand spikes.

```sql
select
	order_item,
	case 
		when extract(month from order_date) between 4 and 6 then 'spring'
		when extract(month from order_date) > 6 and extract(month from order_date) < 9 then 'summer'
		else 'winter'
		end as season,
	count(order_id)
	 
from orders as o 
group by 1,2
order by 1, 3 desc
```
### City Revenue Ranking
### Q.20 Rank each city based on the total revenue for the last year (2023).

```sql
select
	r.city,
	sum(total_amount) as total_revenue,
	rank() over(order by sum(total_amount) desc) as rn
from orders as  o
join restaurants as r
on o.restaurant_id = r.restaurant_id 
where extract(year from o.order_date) = 2023

```

--End of Reports

## Conclusion

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Zomato.
The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 

All customer names and data used in this project are computer-generated using AI and random functions.
They do not represent real data associated with Zomato or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.




































