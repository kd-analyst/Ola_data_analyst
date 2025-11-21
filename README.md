# Ola Ride-Data Analysis Using SQL

## Project Overview
**Project Title**: OLA data analysis project  
**Level**: Beginner to intermediate
**Database**: `ola` 

The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

## Objectives

This project analyzes a ride-hailing dataset (similar to Uber/Ola) using SQL.
The dataset includes information about:
	- Bookings  
	- Customers  
	- Drivers  
	- Ride distances  
	- Ratings  
	- Payment data  
	- Cancellations  
	- TAT (Turnaround Time)  
	- Ride outcomes  
	
The goal of the project is to demonstrate proficiency in SQL analysis, including:   

✔ Joins  
✔ Window functions  
✔ Aggregations  
✔ CTEs  
✔ Subqueries  
✔ Ranking functions  
✔ KPI creation  
✔ Data quality checks  

This report includes insights, KPIs, and hard-level SQL queries solved using real-world analytical thinking.

## Dataset Schema

| Column Name                | Description                       |
| -------------------------- | --------------------------------- |
| Date                       | Booking date                      |
| Time                       | Booking time                      |
| Booking_ID                 | Unique ride ID                    |
| Booking_Status             | Completed / Canceled / Incomplete |
| Customer_ID                | Unique customer identifier        |
| Vehicle_Type               | Mini, Sedan, SUV, Bike, etc.      |
| Pickup_Location            | Start point                       |
| Drop_Location              | Destination                       |
| V_TAT                      | Vendor estimated turnaround time  |
| C_TAT                      | Actual customer turnaround time   |
| Canceled_Rides_by_Customer | Flag                              |
| Canceled_Rides_by_Driver   | Flag                              |
| Incomplete_Rides           | Flag                              |
| Incomplete_Rides_Reason    | Reason for incompletion           |
| Booking_Value              | Fare amount                       |
| Payment_Method             | UPI / Cash / Card                 |
| Ride_Distance              | Ride distance in km               |
| Driver_Ratings             | Rating given to driver            |
| Customer_Rating            | Rating given by driver            |


## Key Metrics (KPIs)

| KPI               | Description          |               |   |
| ----------------- | -------------------- | ------------- | - |
| Total Bookings    | Number of rides      |               |   |
| Completion Rate   | Completed / Total    |               |   |
| Revenue           | Sum of booking value |               |   |
| Cancellation Rate | Cancelled / Total    |               |   |
| Avg Ride Distance | Mean km              |               |   |
| Avg TAT Deviation |                      | V_TAT – C_TAT |   |
| Avg Driver Rating | Overall performance  |               |   |

## Project Structure

### 1. Database Setup
1. **Database Creation**: The project starts by creating a database named `ola`.
2. **Table Creation**: A table named `ola` is created to store the ola data analysis data. The table structure includes columns for date date, time, booking_id, booking_status, customer_id, vehicle_type, pickup_location, drop_location, v_tat, c_tat, canceled_rides_by_customer, canceled_rides_by_driver, incomplete_rides, incomplete_rides_reason, booking_value, payment_method, ride_distance, driver_ratings and customer_rating.
```sql
DROP DATABASE ola;
DROP TABLE IF EXISTS ola;

CREATE TABLE ola (
	date DATE,
	time TIME,
	booking_id VARCHAR (15),
	booking_status VARCHAR (25),
	customer_id VARCHAR(15),
	vehicle_type VARCHAR(15),
	pickup_location VARCHAR (25),
	drop_location VARCHAR(25),
	v_tat INT NULL,
	c_tat INT NULL,
	canceled_rides_by_customer VARCHAR(100) NULL,
	canceled_rides_by_driver VARCHAR(100) NULL,
	incomplete_rides VARCHAR(20) NULL,
	incomplete_rides_reason VARCHAR(25),
	booking_value INT, 
	payment_method VARCHAR (15),
	ride_distance INT,
	driver_ratings FLOAT NULL,
	customer_rating FLOAT NULL 
);
```

## Data Exploration

### BEGINNER LEVEL

1.**Retrieve all successful bookings**.
```sql
	SELECT * FROM ola
	WHERE booking_status = 'Success';
```
2.**Find the average ride distance for each vehicle type.**
```sql
SELECT 
	vehicle_type, 
	AVG(ride_distance) AS avg_ride_distance
FROM ola
GROUP BY vehicle_type
ORDER BY avg_ride_distance DESC;
```
3.**Get the total number of cancelled rides by customers.**
```sql
SELECT COUNT(*)
FROM ola
WHERE booking_status = 'Canceled by Customer';
```
4.**List the top 5 customers who booked the highest number of rides.**
```sql
SELECT customer_id, COUNT(booking_id) AS no_of_rides
FROM ola
WHERE booking_status = 'Success'
GROUP BY customer_id
ORDER BY no_of_rides DESC
LIMIT 5;
```

5.**Get the number of rides cancelled by drivers due to personal and car-related issues.**
```sql
SELECT COUNT(*) FROM ola
WHERE 
	booking_status = 'Canceled by Driver'
	AND canceled_rides_by_driver = 'Personal & Car related issue';
```
6.**Find the maximum and minimum driver ratings for Prime Sedan bookings.**
```sql
SELECT 
	MAX(driver_ratings) AS maximum_rating,
	MIN(driver_ratings) AS minimum_rating
FROM ola
	WHERE vehicle_type = 'Prime Sedan';
```
7.**Retrieve all rides where payment was made using UPI**.
```sql
SELECT * FROM ola
WHERE payment_method = 'UPI';
```
8.**Find the average customer rating per vehicle type.**
```sql
SELECT 
	vehicle_type, 
	AVG(customer_rating) AS avg_customer_rating
FROM ola
GROUP BY vehicle_type
ORDER BY avg_customer_rating DESC;
```
9.**Calculate the total booking value of rides completed successfully.**
```sql
SELECT 
	SUM(booking_value) AS total_booking_value
	FROM ola
	WHERE booking_status = 'Success';
```
10.**List all incomplete rides along with the reason.**
```sql
SELECT
	booking_id, 
	incomplete_rides_reason
FROM ola
WHERE incomplete_rides = 'Yes';
```

### MEDIUM LEVEL QUERIES

1.**Count of rides canceled by customer vs driver**
```sql
SELECT 
	COUNT(canceled_rides_by_customer) AS cancelled_by_customer, 
	COUNT(canceled_rides_by_driver) AS cancelled_by_driver
FROM ola
WHERE booking_status <> 'Success';
```
2.**Average ride distance per pickup location**
```sql
SELECT
	pickup_location, 
	AVG(ride_distance) AS avg_ride_distance
FROM ola
GROUP BY pickup_location
ORDER BY avg_ride_distance DESC;
```
3.**Get bookings with TAT (turnaround time) violations**
```sql
SELECT * FROM ola
WHERE v_tat>c_tat;
```
4.**Most common payment methods by city (pickup location)**
```sql
SELECT
	pickup_location,
	payment_method,
	COUNT(*) AS payment_method_count
FROM ola
GROUP BY 1,2
ORDER BY 1,2 DESC;
```
5.**Get the top 3 longest rides by distance**
```sql
SELECT booking_id, ride_distance
FROM ola
ORDER BY ride_distance DESC
LIMIT 3;
```
6.**Daily revenue trend**
```sql
SELECT 
	date,
	SUM(booking_value) AS total_revenue
FROM ola
WHERE booking_status = 'Success'
GROUP BY date
ORDER BY date ASC;
```
7.**Find repeat customers (customers with more than 3 rides).**
```sql
SELECT customer_id, COUNT(*) AS ride_count
FROM ola
GROUP BY customer_id
HAVING COUNT(*) > 3;
```
8.**Use a window function to rank drivers by rating**
```sql
SELECT 
    booking_id,
    driver_ratings,
    RANK() OVER (ORDER BY driver_ratings DESC) AS rating_rank
FROM ola;
```

### INTERMEDIATE LEVEL

1.**Find the top customer per day by booking value**
```sql
WITH daily_orders AS (
	SELECT 
		date, 
		customer_id,
		SUM(booking_value) AS total_booking_per_day,
		RANK() OVER (PARTITION BY date ORDER BY SUM(booking_value)DESC) AS ranks
	FROM ola
	GROUP BY date, customer_id)

SELECT * FROM daily_orders
WHERE ranks = 1;
```
2.**Detect unusually long rides (distance above daily mean + 2 SD)**
```sql
WITH stats AS (
    SELECT 
        Date,
        AVG(ride_distance) AS avg_distance,
        STDDEV(ride_distance) AS sd_distance
    FROM ola
    GROUP BY Date
	ORDER BY 1 ASC
)
SELECT r.*
FROM ola r
JOIN stats s USING (Date)
WHERE r.ride_distance > s.avg_distance + 2 * s.sd_distance;
```
3.**Identify peak booking time windows (hourly)**
```sql
SELECT 
    EXTRACT(HOUR FROM time) AS hour_of_day,
    COUNT(*) AS total_bookings
FROM ola
GROUP BY EXTRACT(HOUR FROM time)
ORDER BY total_bookings DESC;
```
4.**Revenue contribution: top 20% customers contributing to 80% revenue (Pareto)**
```sql
WITH revenue AS (
    SELECT 
        customer_id,
        SUM(booking_value) AS total_revenue
    FROM ola
    GROUP BY customer_id
),
ordered AS (
    SELECT *,
        SUM(total_revenue) OVER (ORDER BY total_revenue DESC) 
          / SUM(total_revenue) OVER () AS cumulative_share
    FROM revenue
)
SELECT *
FROM ordered
WHERE cumulative_share <= 0.80;
```
5.**Detect customers who frequently give low ratings (<3.5)**
```sql
SELECT
	customer_id,
	COUNT(customer_id) AS total_no_of_ratings
FROM ola
WHERE customer_rating < 3.5
GROUP BY 1
ORDER BY 1 DESC;
```
6.**Compare average distance between completed and canceled rides**
```sql
SELECT 
    booking_status,
    AVG(ride_distance) AS avg_distance
FROM ola
WHERE ride_distance IS NOT NULL
GROUP BY booking_status;
```
7.**Find rides where customer and driver rating differ significantly (>= 2 points)**
```sql
SELECT 
	booking_id,
	driver_ratings,
	customer_rating,
	(driver_ratings - customer_rating) AS difference
FROM ola
WHERE (driver_ratings - customer_rating) >= 2;
```
8.**Identify the top drop locations for each pickup location (ranked).**
```sql
SELECT 
    pickup_location,
    drop_location,
    COUNT(*) AS ride_count,
    RANK() OVER (PARTITION BY pickup_Location ORDER BY COUNT(*) DESC) AS rank_loc
FROM ola
GROUP BY pickup_location, drop_location;
```
9.**Track customer growth month-over-month**
```sql
WITH monthly_customers AS (
    SELECT
        DATE_TRUNC('month', Date) AS month,
        COUNT(DISTINCT Customer_ID) AS unique_customers
    FROM ola
    GROUP BY DATE_TRUNC('month', Date)
)
SELECT 
    month,
    unique_customers,
    unique_customers - LAG(unique_customers, 1) 
        OVER (ORDER BY month) AS mom_growth
FROM monthly_customers;
```
10.**ind bookings where estimated TAT was severely inaccurate (differences > 20%)**
```sql
SELECT
	booking_id, 
	v_tat,
	c_tat,
	ABS(v_tat - c_tat) AS tat_difference,
	ABS(v_tat - c_tat)/ c_tat * 100 AS percentage_error
FROM ola
WHERE ABS(v_tat - c_tat)/ c_tat * 100 > 20;
```
11.**Determine customer lifetime value (CLTV) (total spend + avg rating weight)**
```sql
SELECT 
    customer_id,
    SUM(booking_value) AS total_spend,
    AVG(customer_rating) AS avg_rating,
    SUM(booking_value) * (1 + (AVG(customer_rating) / 5.0)) AS cltv_score
FROM ola
GROUP BY customer_id;
```
12.**dentify unusually high-value bookings using window percentile**
```sql
WITH ranked AS (
    SELECT
        booking_id,
        booking_value,
        PERCENT_RANK() OVER (ORDER BY booking_value) AS pct_rank
    FROM ola
)
SELECT *
FROM ranked
WHERE pct_rank >= 0.95;
-- top 5% high-value rides
```

## Insights Summary
✔ High-value customers contribute a majority of revenue (Pareto rule applies).  
✔ Certain pickup→drop routes have significantly higher demand.  
✔ Cancellations are heavily driven by customer behavior, not drivers.  
✔ TAT deviations strongly correlate with incomplete rides.  
✔ Drivers with consistently high ratings form a very small % (top tier).  
✔ Customer ratings show high variability → strong opportunity for service improvement.  

## Tools Used
	- SQL (PostgreSQL / MySQL / SQL Server compatible)
	- GitHub

## Reports
A comprehensive SQL-driven analysis of ride-hailing data that highlights key metrics such as revenue, cancellations, TAT deviations, and customer/driver performance. The project showcases advanced analytical SQL skills through real-world problem-solving and data insights.
**Sales Summary**: A detailed report summarizing total sales, customer demographics, and vehicle performance.
**Trend Analysis**: Insights into sales trends across different months and vehicles.
**Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion
The analysis shows that a small group of high-value customers drives most revenue, customer cancellations dominate overall failures, and TAT deviations strongly impact ride completion. Insights from the data highlight clear opportunities to improve service efficiency and customer experience.

## Author - Koushik Das
This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

## About Me
- **Name**: Koushik Das
- **Education**: B.Sc. Physics (Hons.)
- **LinkedIn**: [click here to redirect to my linked account](https://www.linkedin.com/in/koushik-das-0047b836a?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=android_app)
- **Contact**: 9911568488

Thank you for your support, and I look forward to connecting with you!

***END OF THE PROJECT***
