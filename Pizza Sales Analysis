USE pizza_sales;

SELECT * FROM [dbo].[pizzas];
SELECT * FROM [dbo].[pizza_types];
SELECT * FROM [dbo].[orders];
SELECT * FROM [dbo].[order_details];

----------------------------------------------------
-- Sum of total sales all
----------------------------------------------------

SELECT SUM(order_details.quantity * pizzas.price) AS total_sales
FROM pizzas 
	JOIN order_details 
		ON pizzas.pizza_id = order_details.pizza_id;

----------------------------------------------------
/*Total sales by category (join table pizzas, 
pizza_types & order_details)*/
----------------------------------------------------

SELECT 
	pizza_types.category,
	SUM(order_details.quantity * pizzas.price) AS total_sales
FROM pizzas 
	JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
	JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY total_sales DESC;

----------------------------------------------------
-- Show total sales in percentage by category
-- total sales per category / total sales all * 100
----------------------------------------------------

SELECT 
	pizza_types.category,
	SUM(order_details.quantity * pizzas.price) AS total_sales,
	SUM(order_details.quantity * pizzas.price) / 
	(SELECT SUM(order_details.quantity * pizzas.price)
	FROM pizzas 
		JOIN order_details 
			ON pizzas.pizza_id = order_details.pizza_id) * 100 AS percentage_of_total
FROM pizzas 
	JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
	JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY total_sales DESC;

----------------------------------------------------
-- Show order quantity & total sales by pizza size 
-- Sort total sales result from largest to smallest
----------------------------------------------------
SELECT pizzas.size,
	SUM(order_details.quantity) AS qty,
	SUM(order_details.quantity * pizzas.price) AS total_sales
FROM pizzas 
	JOIN order_details 
		ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY total_sales DESC;

----------------------------------------------------
-- Show monthly trend of sales
-- Show month name
----------------------------------------------------

SELECT DISTINCT YEAR(date) FROM orders;
SELECT DISTINCT MONTH(date) FROM orders;

SELECT month_name, total_sales
FROM(
	SELECT DATENAME(month, date) AS month_name,
		SUM(order_details.quantity * pizzas.price) AS total_sales,
		DATEPART(month,date) AS month_number
	FROM orders 
		JOIN order_details ON orders.order_id = order_details.order_id
		JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id
	GROUP BY DATENAME(month, date), DATEPART(month,date)
	) AS subquery
ORDER BY month_number;

----------------------------------------------------
--	TOP 3 Pizza name with largest sales
----------------------------------------------------
SELECT 
	TOP 3
	pizza_types.name,
	SUM(order_details.quantity * pizzas.price) AS total_sales
FROM pizza_types 
	JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
	JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY name
ORDER BY total_sales DESC;


/*----------------------------------------------------
Show pizza sales per month by category

month |     category      |
      -------------------
      | C1 | C2 | C3 | C4 | 
----------------------------------------------------*/

SELECT DISTINCT(pizza_types.category) FROM pizza_types;

SELECT
	month_name,
	ROUND(SUM(CASE WHEN category = 'Chicken' THEN total_sales ELSE 0 END ),2) AS Chicken,
	ROUND(SUM(CASE WHEN category = 'Classic' THEN total_sales ELSE 0 END ),2) AS Classic,
	ROUND(SUM(CASE WHEN category = 'Supreme' THEN total_sales ELSE 0 END ),2) AS Supreme,
	ROUND(SUM(CASE WHEN category = 'Veggie' THEN total_sales ELSE 0 END ),2) AS Veggie
FROM(
    
	SELECT DATENAME(MONTH, orders.date) AS month_name,
		pizza_types.category,
		SUM(order_details.quantity * pizzas.price) AS total_sales,
		DATEPART(MONTH, orders.date) AS month_number
	FROM orders 
		JOIN order_details ON orders.order_id = order_details.order_id
		JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id
		JOIN pizza_types ON pizza_types.pizza_type_id = pizzas.pizza_type_id
	GROUP BY DATENAME(MONTH, orders.date), DATEPART(MONTH, orders.date), pizza_types.category

) AS subquery
GROUP BY month_name, month_number
ORDER BY month_number ASC;			

-----------------------------------------------------------------------
-- Monthly Sales Growth --
   -- Percent increase (or decrease) = (Month 2 - Month 1) / Month 1 * 100
   -- DATEDIFF : Return the difference between two date values, in years
   -- DATEADD : Add one year to a date, then return the date
   -- LAG : Provide access to a row at a given physical offset that comes before the current row.
-----------------------------------------------------------------------
WITH MonthlySales AS(
	SELECT 
	DATEADD(MONTH,DATEDIFF(MONTH,0,orders.date),0) AS month_start,
	SUM(order_details.quantity * pizzas.price) AS total_sales
	FROM orders 
		JOIN order_details ON orders.order_id = order_details.order_id
		JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id
		JOIN pizza_types ON pizza_types.pizza_type_id = pizzas.pizza_type_id
	GROUP BY DATEADD(MONTH,DATEDIFF(MONTH,0,orders.date),0)
)

SELECT 
	current_month_start,
	LAG(current_month_start) OVER (ORDER BY current_month_start) AS previous_month,
	current_month_sales,
	LAG(current_month_sales) OVER (ORDER BY current_month_start) AS previous_month_sales,
	(current_month_sales - LAG(current_month_sales) OVER (ORDER BY current_month_start)) AS MoM_growth,

	-- percentage_growth
	CONCAT(
	ROUND(
		((current_month_sales - LAG(current_month_sales) OVER (ORDER BY current_month_start)) / 
		LAG(current_month_sales) OVER (ORDER BY current_month_start) *100),2),'%')
	AS MoM_percentage_growth
	
FROM(
	SELECT 
		month_start AS current_month_start,
		total_sales AS current_month_sales
	FROM MonthlySales
) AS current_month_data;










