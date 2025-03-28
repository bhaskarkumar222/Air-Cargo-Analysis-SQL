# Air Cargo Analysis SQL


## Situation  
Air Cargo is an airline company that transports both passengers and goods. It partners with other airlines to offer various services and wants to improve its passenger services and cargo operations by analyzing key factors such as:

* frequent passengers
* Busiest flight routes
* Regular passengers for loyalty programs
* Ticket sales and revenue breakdown

## Task 
As a Data Analyst your job is to identify frequent customers to offer them special deals analyze the busiest routes to help decide if more flights are needed and review ticket sales to improve business operations. This will help the company run more efficiently and provide a better experience for travelers.

## Action
```sql
-- Customer Insights & Behavior

-- Who are the most frequent travelers, and how often do they fly?
SELECT	
	first_name, 
	last_name,
	COUNT(c.customer_id) AS travel_count 
FROM	
	customers c JOIN passengers_on_flights pof
ON		
	c.customer_id = pof.customer_id
GROUP BY 
	first_name, last_name
ORDER BY 
	travel_count DESC;


-- What is the distribution of passengers by age group and gender?
ALTER TABLE customers  
ADD age INT;

UPDATE customers  
SET age = DATEDIFF(YEAR, date_of_birth, CAST(GETDATE() AS DATE)); 

WITH age_group AS(
SELECT *, 
		CASE WHEN age between 10 and 18 THEN '10-18'
			 WHEN age between 19 and 30 THEN '19-30'
			 WHEN age between 31 and 40 THEN '31-40'
			 ELSE '40+' END AS Age_group
FROM customers
)
SELECT 
	age_group , 
	COUNT(*) AS pessanger_count 
FROM age_group
GROUP BY Age_group;


SELECT 
	 gender, COUNT(*) AS pessanger_count 
FROM customers
GROUP BY gender;


-- What percentage of passengers are first-time flyers vs. repeat customers?

WITH passanger_count AS (
	SELECT customer_id, COUNT(customer_id) AS passacger_count  
	FROM passengers_on_flights
	GROUP BY customer_id
)
SELECT 
	 SUM(CASE WHEN passacger_count = 1 THEN 1 ELSE 0 end)*100/ COUNT(*) AS percent_1st_time,
	 SUM(CASE WHEN passacger_count > 1 THEN 1 ELSE 0 end)*100/ COUNT(*) AS percent_repeat_customer
FROM passanger_count;


-- Which airlines or brands do customers prefer the most?
SELECT 
	brand,
	COUNT(customer_id) AS Customer_count
FROM ticket_details
GROUP BY brand
ORDER BY Customer_count DESC;



-- Flight & Route Analysis

-- What are the busiest routes based on passenger volume?
SELECT 
	r.route_id,
	origin_airport, 
	destination_airport,
	COUNT(p.customer_id) AS Passenger_volume
FROM routes r JOIN passengers_on_flights p
ON r.route_id = p.route_id
GROUP BY r.route_id,origin_airport, destination_airport
ORDER BY Passenger_volume DESC;


-- Which airports have the highest departure and arrival traffic?
SELECT TOP 5
	arrival, 
	COUNT(aircraft_id) AS total_arrival 
FROM passengers_on_flights
GROUP BY arrival
ORDER BY total_arrival DESC;


SELECT TOP 5
	depart, 
	COUNT(aircraft_id) AS total_depart 
FROM passengers_on_flights
GROUP BY depart
ORDER BY total_depart DESC;

-- How does the travel class distribution vary across different routes?

SELECT 
    p.depart AS departure,
    p.arrival AS arrival,
    t.class_id,
    COUNT(t.no_of_tickets) AS total_passengers
FROM passengers_on_flights p JOIN ticket_details t 
ON p.aircraft_id = t.aircraft_id
GROUP BY p.depart, p.arrival, t.class_id
ORDER BY departure, arrival, total_passengers DESC;



-- Revenue & Ticket Sales Analysis

-- What is the total revenue generated per month, and how does it vary seasonally?
SELECT 
	FORMAT(p_date, 'MMM') AS month,
	SUM(Price_per_ticket) AS Revenue,
	CASE WHEN MONTH(p_date) >=4 THEN MONTH(p_date) -3 ELSE MONTH(p_date)+9 END AS Month_
FROM ticket_details	
GROUP BY FORMAT(p_date, 'MMM'), CASE WHEN MONTH(p_date) >=4 THEN MONTH(p_date) -3 ELSE MONTH(p_date)+9  END
ORDER BY CASE WHEN MONTH(p_date) >=4 THEN MONTH(p_date) -3 ELSE MONTH(p_date)+9  END


SELECT 
	 FORMAT(p_date, 'MMM') AS month,
	 SUM(Price_per_ticket) AS Revenue,
	 DATEPART(MONTH, DATEADD(MONTH, -3, p_date)) AS Fiscal_Month
FROM ticket_details	
GROUP BY FORMAT(p_date, 'MMM'), DATEPART(MONTH, DATEADD(MONTH, -3, p_date)) 
ORDER BY DATEPART(MONTH, DATEADD(MONTH, -3, p_date)) 


-- Which routes generate the highest revenue?
SELECT 
	 r.origin_airport,r.destination_airport, 
	 SUM(Price_per_ticket) AS revenue 
FROM routes r Left Join ticket_details t
ON	 r.aircraft_id =t.aircraft_id
GROUP BY r.origin_airport,r.destination_airport
ORDER BY revenue DESC;

-- What is the average ticket price per travel class?
SELECT 
	class_id,
	AVG(Price_per_ticket)AS Avg_Price
FROM ticket_details
GROUP BY class_id;

-- What percentage of total revenue comes from Business Class passengers?

SELECT 
    Concat(ROUND(CAST(SUM(CASE WHEN class_id = 'Bussiness' THEN Price_per_ticket ELSE 0 END) * 100.0/ 
	SUM(Price_per_ticket) AS FLOAT),2), '%') AS revenue_percentage
FROM ticket_details;


-- Operational Efficiency & Optimization

-- What are the peak travel times for different routes?
SELECT 
	FORMAT(p_date,'MMM') AS Month_name,
	r.origin_airport,
	r.destination_airport,
	COUNT(t.customer_id) AS Customer_count
FROM 
	ticket_details t JOIN routes r
ON	 
	t.aircraft_id = r.aircraft_id
GROUP BY 
	FORMAT(p_date,'MMM'),r.origin_airport, r.destination_airport
ORDER BY 
	customer_count DESC

-- Are there underperforming routes with consistently low passenger traffic?
SELECT  TOP 5
	r.origin_airport,
	r.destination_airport,
	COUNT(t.customer_id) AS Customer_count
FROM 
	ticket_details t JOIN routes r
ON	 
	t.aircraft_id = r.aircraft_id
GROUP BY 
	r.origin_airport, r.destination_airport
ORDER BY 
	customer_count ASC

-- How does aircraft utilization vary across different flights and routes?
SELECT brand, route_id, COUNT(customer_id) AS total_passanger FROM ticket_details t JOIN routes r
ON t.aircraft_id = r.aircraft_id
GROUP BY brand, route_id
ORDER BY brand, total_passanger DESC
```


## Conclusion of the Air Cargo Analysis Project
The Air Cargo Analysis Project provided key insights into customer behavior, revenue trends, route performance and operational efficiency. The findings revealed:

* **Customer Loyalty & Preferences** – Identifying frequent travelers and preferred airlines helps in targeted marketing and loyalty programs.

* **Route & Traffic Optimization** – Busiest routes and high-traffic airports indicate areas requiring better scheduling and resource allocation.

* **Revenue Growth Opportunities** – Analysis of revenue trends, business class contribution and ticket pricing helps optimize profitability.

* **Operational Enhancements** – Identifying peak travel times and underperforming routes aids in improving aircraft utilization and cost efficiency.




