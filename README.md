# Ola_data_analyst

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

## BEGINNER LEVEL

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

## MEDIUM LEVEL QUERIES

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

## INTERMEDIATE LEVEL

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
